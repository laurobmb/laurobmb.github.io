---
title: "DNSSEC"
date: 2020-12-18T01:46:34-03:00
draft: True
tags: ["dnssec","bind"]
categories: ["Linux"]
moods: ["happy"]
autor: "Lauro de Paula Gomes"
---

# DNSSEC 

DNSSEC é um padrão internacional que estende a tecnologia DNS. DNSSEC adiciona um sistema de resolução de nomes mais seguro, reduzindo o risco de manipulação de dados e informações, pois garante autenticidade e integridade ao sistema DNS. O mecanismo utilizado pelo DNSSEC é baseado na tecnologia de criptografia de chaves públicas.

 Em fim … vamos começar
Um KSK significa chave de assinatura chave. Um KSK é um par de chaves público / privado. A chave privada KSK é usada para gerar uma assinatura digital para o ZSK. A chave pública KSK é armazenada no DNS a ser usado para autenticar o ZSK.
Um ZSK é uma chave de assinatura de zona. Um ZSK é um par de chaves público / privado. A chave privada ZSK é usada para gerar uma assinatura digital, conhecida como RRSIG (Resource Record Signature), para cada um dos conjuntos de registros de recursos (RRSET) em uma zona. A chave pública ZSK é armazenada no DNS para autenticar um RRSIG.
Cada nome dentro de uma zona assinada DNSSEC será coberto por um RRSIG.
Sobre os algoritmos que podem ser usados nas chaves, a escolha do tamanho da chave depende do algoritmo utilizado.

 * RSAMD5,
 * RSASHA1,
 * DSA,
 * NSEC3RSASHA1,
 * NSEC3DSA,
 * RSASHA256,
 * RSASHA512,
 * ECCGOST,
 * ECDSAP256SHA256
 * ECDSAP384SHA384.

Para TSIG/TKEY, o valor deve ser DH (Diffie Hellman),

 * HMAC-MD5,
 * HMAC-SHA1,
 * HMAC-SHA224,
 * HMAC-SHA256,
 * HMAC-SHA384,
 * HMAC-SHA512.

As chaves RSA devem ter entre 512 e 2048 bits. As chaves Diffie Hellman devem ter entre 128 e 4096 bits. As chaves DSA devem ter entre 512 e 1024 bits e um múltiplo exato de 64. As chaves HMAC devem ter entre 1 e 512 bits. Algoritmos de curva elíptica não precisam desse parâmetro.
O tamanho da chave não precisa ser especificado se estiver usando um algoritmo padrão. O tamanho da chave padrão é 1024 bits para chaves de assinatura de zona (ZSK) e 2048 bits para chaves de assinatura de chave (KSK, gerado com -f KSK). No entanto, se um algoritmo é explicitamente especificado com o -a, então não há tamanho de chave padrão, e o -b deve ser usado.

O arquivo de configuração named.conf, antes de iniciar a configuração do DNSSEC, deve conter as seguintes opções habilitadas:

> dnssec-enable yes;

> dnssec-validation yes;

> dnssec-lookaside auto;

Gerar Chave KSK 
> dnssec-keygen -r /dev/urandom -a RSASHA256 -b 4096 -f KSK -n ZONE olinda.com.br

Gerar chave ZSK 
> dnssec-keygen -r /dev/urandom -a RSASHA256 -b 1024 -n ZONE olinda.com.br

Os arquivos que contém a chave devem ser incluídos dentro do arquivo e zona do domínio, para colocar basta adicionar um “$include” dentro do arquivo

> vim olinda.com.br.zone

```
$include Kolinda.com.br.+008+14200.key
$include Kolinda.com.br.+008+31950.key
```

Assinando o domínio junto com a chave privada
dnssec-signzone -S -z -o olinda.com.br olinda.com.br.zone Kolinda.com.br.*.private

Script para reassinar a zona após novas alterações alteração

```
#!/bin/bash
PDIR=`pwd`
ZONEDIR="/var/named"
ZONE=$1
ZONEFILE=$2
DNSSERVICE="named"
cd $ZONEDIR
SERIAL=`named-checkzone $ZONE $ZONEFILE | awk '{print $5}'| head -n1`
sed -i s/$SERIAL/"$(($SERIAL+1))"/g $ZONEFILE
dnssec-signzone -S -z -o $1 $2 K$1.*.private
```

Para realizar verificação do DNSSEC do domínio, pode usar o seguinte comando:
dig DNSKEY olinda.com.br @192.168.15.7 +multiline

> /etc/named.conf

```
options {
listen-on port 53 { 127.0.0.1; 192.168.15.7; 192.168.1.34; };
listen-on-v6 port 53 { ::1; };
directory "/var/named";
dump-file "/var/named/data/cache_dump.db";
statistics-file "/var/named/data/named_stats.txt";
memstatistics-file "/var/named/data/named_mem_stats.txt";
recursion yes;
dnssec-enable yes;
dnssec-validation yes;
dnssec-lookaside auto;
bindkeys-file "/etc/named.iscdlv.key";
managed-keys-directory "/var/named/dynamic";
pid-file "/run/named/named.pid";
session-keyfile "/run/named/session.key";
forwarders {
8.8.8.8;
8.8.4.4;
};

blackhole { black-hats; };
allow-query { red-hats; };
allow-recursion { red-hats; };
};

acl black-hats {
192.168.15.6/32;
};

acl red-hats {
192.168.15.0/24;
192.168.1.0/24;
};

logging {
channel default_debug {
file “data/named.run”;
severity dynamic;
};
};

zone “.” IN {
type hint;
file “named.ca”;
};

include “/etc/named.rfc1912.zones”;
include “/etc/named.root.key”;

zone “olinda.com.br” IN {
type master;
file “olinda.com.br.zone.signed”;
allow-update { none; };
};

zone “paulista.com.br” IN {
type master;
file “paulista.com.br.zone.signed”;
allow-update { none; };
};

zone “15.168.192.in-addr.arpa” IN {
type master;
file “15.168.192.in-addr.arpa.zone”;
allow-update { none; };
};

zone “1.168.192.in-addr.arpa” IN {
type master;
file “1.168.192.in-addr.arpa.zone”;
allow-update { none; };
};

“paulista.com.br.zone”

$TTL 0
paulista.com.br. IN SOA ns1.paulista.com.br. hostmaster.paulista.com.br. (
201704151 ; serial
21600 ; refresh after 6 hours
3600 ; retry after 1 hour
604800 ; expire after 1 week
86400 ) ; minimum TTL of 1 day

paulista.com.br. IN NS ns1.paulista.com.br.
paulista.com.br. IN NS ns2.paulista.com.br.

paulista.com.br. IN MX 10 mail
paulista.com.br. IN MX 20 mail2

server1 IN A 192.168.1.100
server2 IN A 192.168.1.101
ns1 IN A 192.168.1.102
ns2 IN A 192.168.1.103

ftp IN CNAME server1
www IN CNAME server2

mail IN A 192.168.1.110
mail2 IN A 192.168.1.111

paulista.com.br. 3600 TXT “v=spf1 a mx ptr -all”

“olinda.com.br.zone”

$TTL 0
olinda.com.br. IN SOA ns1.olinda.com.br. hostmaster.olinda.com.br. (
201704153 ; serial
21600 ; refresh after 6 hours
3600 ; retry after 1 hour
604800 ; expire after 1 week
86400 ) ; minimum TTL of 1 day

$include Kolinda.com.br.+008+14200.key
$include Kolinda.com.br.+008+31950.key

olinda.com.br. IN NS ns1.olinda.com.br.
olinda.com.br. IN NS ns2.olinda.com.br.

olinda.com.br. IN MX 10 mail
olinda.com.br. IN MX 20 mail2

server1 IN A 192.168.15.100
server2 IN A 192.168.15.101
ns1 IN A 192.168.15.102
ns2 IN A 192.168.15.103

ftp IN CNAME server1
www IN CNAME server2

mail IN A 192.168.15.110
mail2 IN A 192.168.15.111

olinda.com.br. 3600 TXT “v=spf1 a mx ptr -all”

“15.168.192.in-addr.arpa.zone”

$TTL 0
@ SOA dns1.example.com. hostmaster.example.com. (
201704151 ; serial
21600 ; refresh after 6 hours
3600 ; retry after 1 hour
604800 ; expire after 1 week
86400 ) ; minimum TTL of 1 day

IN NS ns1.example.com.
IN NS ns2.example.com.

100 IN PTR server1.example.com.
101 IN PTR server2.example.com.
110 IN PTR mail.example.com.
111 IN PTR mail2.example.com.
```


Fontes:

[registro.br](ftp://ftp.registro.br/pub/doc/configuracao_dnssec_dominio.pdf)

[zendesk](https://auda.zendesk.com/hc/en-us/articles/201442880-What-is-a-KSK-ZSK-RRSIG-)

[registro-dnssec](https://registro.br/tecnologia/dnssec.html?secao=dnssec)

