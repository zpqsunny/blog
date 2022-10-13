---
title: Mac os client_loop: send disconnect: broken pipe
date: 2022-10-13 22:31:50 +0800
categories: [Linux]
tags: [Iterm2, SSH, macos]
---


```
/etc/ssh/ssh_config

```

```
Host *
    ServerAliveInterval 60
    TCPKeepAlive no
```

> https://unix.stackexchange.com/questions/602518/ssh-connection-client-loop-send-disconnect-broken-pipe-or-connection-reset

Motivation:
TCPKeepAlive no means "do not send keepalive messages to the server". When the opposite, TCPKeepAlive yes, is set, then the client sends keepalive messages to the server and requires a response in order to maintain its end of the connection. This will detect if the server goes down, reboots, etc. The trouble with this is that if the connection between the client and server is broken for a brief period of time (due to flaky a network connection), this will cause the keepalive messages to fail, and the client will end the connection with "broken pipe".
Setting TCPKeepAlive no tells the client to just assume the connection is still good until proven otherwise by a user request, meaning that temporary connection breakages while your ssh term is sitting idle in the background won't kill the connection.
