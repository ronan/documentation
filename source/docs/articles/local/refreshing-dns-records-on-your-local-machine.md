---
title: Refreshing DNS Records on Your Local Machine
description: Flush your DNS cache to clear and update the data.
category:
  - developing

---

## Overview
DNS is the main naming system for the internet, allowing computers to exchange data over TCP/IP. It takes any domain name, such as "getpantheon.com" and "ties" it to an IP address like 24.102.112.16. The numbers identify computers to each other, and the names are easier for humanoids to remember.  






1. Start -> Run -> type cmd to open the command prompt
2. Type ipconfig /flushdns

1. Type /etc/rc.d/init.d/nscd restart in your terminal

1. type lookupd -flushcache in your terminal to flush the DNS resolver cache.
2. ex: bash-2.05a$ lookupd -flushcache

1. Type dscacheutil -flushcache in your terminal to flush the DNS resolver cache.
2. ex: bash-2.05a$ dscacheutil -flushcache