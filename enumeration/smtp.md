# SMTP

O SMTP é um protocolo de transferência de e-mail, ele roda na porta 25.

Podemos conectar usando o comando `HELO` ou `EHLO` e verificar se um usuário existe com o comando `VRFY`.

```
# nc -v 172.30.0.128 25                                                                                        1 ⨯
172.30.0.128: inverse host lookup failed: Unknown host
(UNKNOWN) [172.30.0.128] 25 (smtp) open
HELO teste
220 ubuntu.bloi.com.br ESMTP Postfix (Ubuntu)
250 ubuntu.bloi.com.br
EHLO teste
250-ubuntu.bloi.com.br
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250 SMTPUTF8
VRFY testeuser
550 5.1.1 <testeuser>: Recipient address rejected: User unknown in local recipient table
VRFY root
252 2.0.0 root
VRFY www-data
252 2.0.0 www-data
```

Para enviar um e-mail, passamos nosso usuário com o comando `mail from: <usuário>`

Passamos para quem queremos enviar com o comando `rcpt to: <receptor>`

E para enviar os dados `DATA`

Após escrever, damos um ponto final e um enter para enviar.

```
$ nc -v 172.30.0.128 25 
172.30.0.128: inverse host lookup failed: Unknown host
(UNKNOWN) [172.30.0.128] 25 (smtp) open
mail from: teste
220 ubuntu.bloi.com.br ESMTP Postfix (Ubuntu)
250 2.1.0 Ok
rcpt to: root
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
Teste email
.
250 2.0.0 Ok: queued as 03917C172B
```
