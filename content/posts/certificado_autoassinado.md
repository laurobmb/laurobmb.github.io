---
title: "Certificado"
date: 2020-12-18T01:46:34-03:00
draft: True
tags: ["openssl","security"]
categories: ["Linux"]
autor: "Lauro de Paula Gomes"
---

# Certificado auto assinado com openssl

## Gerar certificado
> openssl req -out sav.csr -new -newkey rsa:2048 -nodes -keyout sav.key -subj “/C=BR/ST=Pernambuco/L=Olinda/O=Suporte Avançado/OU=SAV/CN=*.sav.com.br”

## auto-assinar certificado
> openssl x509 -req -days 365 -in sav.csr -signkey sav.key -out sav.crt

## chegar requisição do certificado
> openssl req -text -noout -in sav.csr

## chegar o certificado
> openssl x509 -in sav.crt -text -noout

### Exemplo:
```
/C= Pais → BR
/ST= Estado → Pernambuco
/L= Localização → Olinda
/O= organização → Suporte Avançado
/OU= Unidade Organizacional → SAV
/CN= Domínio → sav.com.br
```