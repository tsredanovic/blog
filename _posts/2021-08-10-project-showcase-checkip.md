---
layout: post
title:  Project Showcase - CheckIp
categories: Python
excerpt: CheckIp is a python package used to resolve public (WAN) IP from one of many supported providers.
---

## Project Showcase - [CheckIp](https://github.com/tsredanovic/checkip)

CheckIp is a python package used to resolve public (WAN) IP from one of many supported providers.

Usage is as simple as:
```python
>>> from checkip import get_ip
>>> get_ip('cloudflare')
'141.136.171.75'
```

It can even use multiple providers to resolve the public IP if one (or more) of them are down:
```python
>>> from checkip import resolve_ip
>>> resolve_ip(['cloudflare', 'dyndns', 'googledomains'])
'141.136.171.75'
```

Docs, installation, usage and more can be found [here](https://github.com/tsredanovic/checkip).