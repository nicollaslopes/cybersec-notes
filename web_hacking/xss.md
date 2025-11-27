# XSS - Cross Site Scripting

O XSS é uma vulnerabilidade que basicamente permite inserir um código javascript do lado do usuário. Existem três tipos, são eles: reflected XSS, stored XSS e DOM-based XSS. É utilizado junto com a engenharia social para causar impacto, já que a ideia dessa falha é executar um javascript malicioso no navegador da vítima.

## Reflected XSS

Esse é o tipo de XSS mais simples e de menor impacto, como o próprio nome diz, ele apenas reflete o script quando for executado. Como no exemplo anterior, vamos supor que a aplicação possui uma entrada que permite passar um valor por parâmetro e exibe na tela esse valor.

Como esse parâmetro não é tratado com algum filtro corretamente no backend, a aplicação permite executar um código javascript que vai refletir para o usuário, por isso é chamado de Reflected XSS.

## DOM Based XSS

O DOM Based XSS ocorre quando o conteúdo é renderizado pelo próprio DOM, e não tratado no backend, nesse caso quando é utilizado as funções `eval()` ou `innerHTML` do javascript.

The `innerHTML` sink doesn't accept `script` elements on any modern browser, nor will `svg onload` events fire. This means you will need to use alternative elements like `img` or `iframe`. Event handlers such as `onload` and `onerror` can be used in conjunction with these elements. For example: 

`element.innerHTML='... <img src=1 onerror=alert(document.domain)> ...'`
## Stored XSS

O XSS armazenado, é de longe o que mais causa impacto para o usuário. Vamos imaginar que uma aplicação tem alguma parte que salva informações e as exiba, como por exemplo em campos de comentários, alguma tabela de usuários, etc. Se essa parte for vulnerável a XSS, o script malicioso que for injetado irá ficar salvo no banco de dados, e consequentemente irá ficar salvo nessa página que existir a falha. Sendo assim, toda vez que o usuário for entrar nessa página, irá ser executado esse script malicioso e com isso ele se torna persistente. Um dos impactos que essa vulnerabilidade possibilita, é usar um javascript que irá enviar o cookie do usuário que estiver na sessão logada para algum servidor nosso que armazena esse cookie para podermos utilizar e consequentemente logar na conta do mesmo. Lembrando que é necessário que o usuário entre nessa página que contém esse javascript. Com isso se algum administrador logado entrar nessa página, conseguimos roubar a sessão dele.

Após o envio deste comentário, qualquer usuário que visitar a postagem do blog receberá o seguinte na resposta do aplicativo:

```
<p>This post was extremely helpful.</p>
```

Supondo que o aplicativo não execute nenhum outro processamento dos dados, um invasor pode enviar um comentário malicioso como este:

```
<script>/* Bad stuff here... */</script>
```

Com isso, o browser de cada pessoa que acessar o blog, irá executar esse script. 

## Session Hijacking via XSS

