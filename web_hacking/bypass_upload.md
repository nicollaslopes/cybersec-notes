# Bypass Upload

Quando a aplicação possui alguma funcionalidade de fazer um upload de arquivo, podemos testar esse vetor para conseguirmos uma shell enviando um arquivo malicioso.

## Extension Bypass

A primeira coisa a se testar, é enviar um arquivo que execute comandos do sistema para a aplicação (como a função sistem() do php). Só que algumas vezes, pode ser que seja bloqueado a extensão **.php** para impedir que isso seja feito. Uma das formas para tentar bypassar essa proteção, é colocarmos a extensão com alguma das letras maiúsculas, como por exemplo **.phP, .Php ou .pHp**, assim pode ser que a aplicação aceita.

