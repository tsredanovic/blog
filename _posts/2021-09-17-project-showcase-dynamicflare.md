---
layout: post
title:  Project Showcase - DynamicFlare
categories: Python
excerpt: DynamicFlare is a python script which can be set up as a cron job to update dynamic DNS entries (dynamic IP) for accounts on CloudFlare's DNS service.
---

## Project Showcase - [DynamicFlare](https://github.com/tsredanovic/dynamicflare)

DynamicFlare is a python script which can be set up as a cron job to update dynamic DNS entries (dynamic IP) for accounts on [CloudFlare](https://www.cloudflare.com/) DNS service.

It only requires a `config.json` file with your CloudFlare API token and a list of A record objects to update:

```json
{
    "cloudflare_token": "_your_cloudflare_api_token_",
    "records": [
        {
            "domain": "domainone.com",
            "record": "domainone.com"
        },
        {
            "domain": "domainone.com",
            "record": "subone.domainone.com"
        },
        {
            "domain": "domaintwo.com",
            "record": "domaintwo.com"
        }
    ]
}
```

Optionally you can also set up logging to a `syslog` file and discord notification to be sent out on every IP update.

Docs, installation, usage and more can be found [here](https://github.com/tsredanovic/dynamicflare).
