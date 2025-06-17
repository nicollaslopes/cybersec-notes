# XSS - Cross Site Scripting

O XSS é uma vulnerabilidade que basicamente permite inserir um código javascript do lado do usuário. Existem três tipos, são eles: reflected XSS, stored XSS e DOM-based XSS. É utilizado junto com a engenharia social para causar impacto, já que a ideia dessa falha é executar um javascript malicioso no navegador da vítima.

## Reflected XSS

Esse é o tipo de XSS mais simples e de menor impacto, como o próprio nome diz, ele apenas reflete o script quando for executado. Como no exemplo anterior, vamos supor que a aplicação possui uma entrada que permite passar um valor por parâmetro e exibe na tela esse valor.

Como esse parâmetro não é tratado com algum filtro corretamente no backend, a aplicação permite executar um código javascript que vai refletir para o usuário.

## Self-XSS

## Stored XSS

