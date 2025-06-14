# Open Redirect

O open redirect, é uma vulnerabilidade que ocorre quando a aplicação web possui algum ponto de redirecionamento que ao clicar, será redirecionado para um outro link, como por exemplo, logar no Facebook. Pode ser observado na url se possui algum parâmetro que faz esse redirecionamento. O impacto dessa falha é que o atacante pode trocar o link que for passado por parâmetro por algum outro e se caso a aplicação permitir, pode-se de criar uma nova url que redireciona o para qualquer lugar que deseja, como por exemplo, uma página falsa ou baixar um arquivo malicioso de algum servidor.

Example of vulnerable code in php:

```html
<a id="redirectLink" href="UrlController.php?url=http://example.com/">Click here to go to the website</a>
```

```php
$url = $_GET['url'];
header("Location: $url");
```

Nós podemos então trocar o parâmetro `url` para a aplicação nos redirecionar para outro site

```
http://localhost:8000/UrlController.php?url=http://malicious-site.com
```

