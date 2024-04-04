---
layout: post
title:  Encrypted volumes part 3 / Automatic mounting
categories: [Automatic,Mount,Cloudflare,Worker,Disk]
excerpt:  Part 3 of creating an encrypted logical volume using LVM and LUKS with automatic mounting.
---

![]({{site.baseurl}}/images/2024-02-06-encrypted-volumes-part-3-automatic-mounting.png)

## Encrypted volumes part 3 / Automatic mounting

Automatically mounting encrypted volumes is pretty simple if the passphrase is stored locally. But then what is even the point of encrypting volumes if the passphrase for unlocking is sitting right next to them. That is why we will set up a remote service from which the passphrase can be fetched.

### Passphrases

We will need to generate 2 more passphrases, let's call them `pp1` and `pp2`:
- `pp1` will be used to unlock the encrypted volume and will be stored in the remote service
- `pp2` will be used to fetch `pp1` from the online service and will be stored locally

This way we can prevent our encrypted volumes from being accesseable by simply deleting the passphrase from the remote service.

> It is important to note that this approach can not be used to encrypted the volume which has system's OS installed on it, that one must remain unencrypted to do the auto mounting.

We can create the 2 passphrases (`pp1` and `pp2`) using the `dd` command 2 times:
```bash
dd bs=32 count=1 if=/dev/random | base64
```

Now add `pp1` to the encrypted volume we created in part 2:
```bash
cryptsetup luksAddKey /dev/vg0/lv0
```

The command will prompt us for any existing passphrase (input the one we set up in part 2) and for the new passphrase (`pp1`).

### Remote service

We will use [Cloudflare Workers](https://workers.cloudflare.com/) to write a simple service for fetching the passphrase. Both `pp1` and `pp2` will be stored as service's environment variables and encrypted.

From Cloudflare dashboard go to `Workers & Pages` -> `Create application` -> `Workers` -> `Create Worker`. Give it a name `passphrases-service` and click `Deploy`.

Now click `Edit code` and paste in this Node JS function:
```javascript
export default {
  async fetch(request, env, ctx) {
    const { searchParams } = new URL(request.url)

    let name = searchParams.get('name')
    let in_key = searchParams.get('key')
    if (typeof name != 'string' || typeof in_key != 'string') {
      return new Response(null, { status: 400 })
    }
    let in_key_var = name.concat('-key')
    let out_key_var = name.concat('-value')

    if (env[in_key_var] != in_key) {
      return new Response(null, { status: 400 })
    }

    return new Response(env[out_key_var]);
  },
};
```

It supports fetching multiple passphrases in case we decide to encypt more volumes later on. Here is a breakdown of how it works:

- takes `name` and `key` as parameters from the URL
  - `name` is identifier for this encrypted volume
  - `key` is the second passphrase (`pp2`) used to fetch the one doing volume unlock (`pp1`)
- checks if `key` (`pp2`) is valid for `name` and returns `pp1` if it is

Click `Save and deploy`.

Now all that's left is to configure `pp1` and `pp2` in our `passphrases-service`. We can do so by going to `Settings` -> `Variables` and adding 2 varibles:
- `vg0-lv0-key`: value of `pp2`
- `vg0-lv0-value`: value of `pp1`

Make sure to `Encrypt` both of them and click `Save and deploy`.

To make sure everything is working correctly we can test it using `curl` (replace `_cloudflare_username_` and `_value_of_pp2_`):
```bash
curl --location 'https://passphrases-service._cloudflare_username_.workers.dev/?name=vg0-lv0&key=_value_of_pp2_'
```

There should be the returned value of `pp1` in output.

> Note that if there is a `+` sign in `pp2` passphrase it needs to be encoded as `%2b` because `+` sign holds special meaning (it means whitespace: ` `) in URLs.

### Fetching the passphrase

Back in our system with encypted volume we need to write a script which fetches `pp1`.
Make a new directory for it:
```bash
mkdir /etc/luks
```
Open the editor:
```bash
nano /etc/luks/vg0-lv0-fetch-passphrase.sh
```
Paste in the following (use the same `curl` command we used to test things out with replaced `_cloudflare_username_` and `_value_of_pp2_`):
```bash
#!/bin/bash
set -e
curl --location 'https://passphrases-service._cloudflare_username_.workers.dev/?name=vg0-lv0&key=_value_of_pp2_'
```

Give ownership of this script to the `root` user with `read` and `execute` permissions:
```bash
chown root:root /etc/luks/vg0-lv0-fetch-passphrase.sh
chmod 0500 /etc/luks/vg0-lv0-fetch-passphrase.sh
```

Run the script to make sure it works:
```bash
sudo /etc/luks/vg0-lv0-fetch-passphrase.sh
```

If all is well there should be the value of `pp1` in output.

Check if the encrypted volume is open with:
```bash
ls /dev/mapper
```
If it is open there will be `open-lv0` in the output and we should close it for further testing:
```bash
cryptsetup close open-lv0
```

Now let's see if we can open the encrypted volume using the script we just wrote:
```bash
/etc/luks/vg0-lv0-fetch-passphrase.sh | /sbin/cryptsetup open /dev/mapper/vg0-lv0 open-lv0 -
```

Use the `ls /dev/mapper` again to check if `open-lv0` is in the output which means we successfully opened the encrypted volume.

### Open and mount

We will create one systemd unit for opening the encrypted volume:
```bash
nano /etc/systemd/system/open-lv0.service
```
Paste in the following configuration:
```
[Unit]
Description=Open encrypted lv0
After=network-online.target
Wants=network-online.target
StopWhenUnneeded=true

[Service]
Type=oneshot
ExecStart=/bin/sh -c '/etc/luks/vg0-lv0-fetch-passphrase.sh | /sbin/cryptsetup open /dev/mapper/vg0-lv0 open-lv0 -'
RemainAfterExit=true
ExecStop=/sbin/cryptsetup close open-lv0
```

And one for mounting the open volume to `/mnt/lv0`:
```bash
nano /etc/systemd/system/mnt-lv0.mount
```
Paste in the following configuration:
```
[Unit]
Requires=open-lv0.service
After=open-lv0.service

[Mount]
What=/dev/mapper/open-lv0
Where=/mnt/lv0
Type=ext4
Options=defaults,noatime,_netdev

[Install]
WantedBy=multi-user.target
```

To check if everything is working we can run the mount unit:
```bash
systemctl start mnt-lv0.mount
```

And then check if our open encrypted volume got mounted:
```bash
ls /mnt/
```

All that is left is to enable the mount unit so that it runs on every system boot:
```bash
systemctl enable mnt-lv0.mount
```

Now try rebooting the system and witness the magic.
```bash
reboot
```