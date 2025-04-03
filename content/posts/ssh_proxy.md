---
title: "proxy"
date: 2020-12-18T01:46:34-03:00
draft: True
tags: ["hacker","security"]
categories: ["Linux"]
autor: "Lauro de Paula Gomes"
---

# Túnel ssh proxy socks

## Túnel para acesso interno

> ssh -q -L 1000:192.168.0.10:3389 user@200.0.0.1

O acesso é comum para a um host interno na rede de destino, o analista deve acessar o servidor SSH que esta aberto e através dele conectar no servidor de destino.
topologia1
Após a conexão no servidor ssh, use o rdesktop para conectar no terminal service.
#rdesktop 127.0.0.1:1000

## Proxy Socks

> ssh -q -D 1000 user@200.0.0.1

Após acessar o ssh, deve configurar o navegador para receber as conexões de socks.
topologia2
No navegador…
