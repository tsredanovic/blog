---
layout: post
title:  Project Showcase / Emby Webhooks
categories: [Emby,Discord]
excerpt: Emby Webhooks is a service which catches Emby webhooks, generates messages and sends Discord notifications.
---

![]({{site.baseurl}}/images/2021-10-28-project-showcase-emby-webhooks.png)

## Project Showcase / [EmbyWebhooks](https://github.com/tsredanovic/emby-webhooks)

Emby Webhooks is a service which catches [Emby](https://emby.media/) webhooks, generates messages and sends Discord notifications.

The following events are supported:
- `playback.start`
- `playback.pause`
- `playback.unpause`
- `playback.stop`
- `item.rate`
- `item.markplayed`
- `item.markunplayed`

Messages are generated using the [Jinja](https://jinja.palletsprojects.com/) templating engine and are fully customizable.

It can be set up to run either as a [Systemd](https://systemd.io/) service or with [Docker](https://www.docker.com/).


Docs, installation, and usage can be found [here](https://github.com/tsredanovic/emby-webhooks).
