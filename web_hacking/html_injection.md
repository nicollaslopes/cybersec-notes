# HTML Injection

O HTML Injection é uma falha que permite que o usuário seja capaz de injetar códigos HTML em algum ponto de entrada na aplicação. É uma vulnerabilidade que pode ser combinada com engenharia social, pois como é possível injetar um código HTML, é possível criar um formulário de login falso e enviar para alguém e assim conseguir algo.

Vejamos a seguir como funciona, da maneira mais simples possível.

```php
<?php

$message = $_GET['msg'];

echo $message;
```

Uma possibilidade é colocar um <a h=ref> para ser redirecionado para uma página falsa.
