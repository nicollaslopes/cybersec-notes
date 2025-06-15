# SQL Injection

O SQL injection, é uma das técnicas de vulnerabilidades web mais conhecida. Ela consiste em basicamente passar uma query SQL em um input que se o banco de dados não tiver um tratamento correto dessa entrada, pode acabar resultado em um vetor de ataque caso essa query seja maliciosa.

## Basic Bypass Authentication

Vamos supor que o backend trata a entrada de um login da seguinte forma abaixo.

```php
$user = $_POST['user'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE user = '$user' AND pass '$password'"; 
```

O problema é que a query que irá para o banco de dados não está sendo filtrada corretamente, passando os parâmetros exatamente como o usuário digita e isso pode ocasionar a vulnerabilidade. 

Vamos então utilizar o payload mais básico para explorar essa falha. A ideia a seguir, é utilizar o campo do usuário para bypassar a consulta. Primeiro temos que fechar as aspas simples e utilizar uma condição de **ou** para que assim a query retorne **true** e por fim comentar o resto para que seja ignorado, com isso irá trazer os resultados no banco de dados. O payload final fica da seguinte forma: `' or 1=1 # ` ou `' or 1=1 --`. (*o comentário # e -- varia de acordo com o banco de dados*.)

Com isso, no banco de dados vai retornar o primeiro usuário que a consulta buscar, autenticando assim na aplicação com este usuário.

```sql
SELECT * FROM users WHERE user = '' or 1=1 -- ' AND pass 'password123" 
```

