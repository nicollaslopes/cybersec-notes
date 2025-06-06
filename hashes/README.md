# Hashes

Uma função de hash criptográfico, muitas vezes é conhecida simplesmente como hash – é um algoritmo matemático que transforma qualquer bloco de dados em uma série de caracteres de comprimento

## Modos de uso

### Armazenamento de senhas no banco de dados

As aplicações atualmente, armazenam as senhas dos usuários em formato de hash, pois uma vez que o hash é gerado, não é possível voltar o valor original, ou seja, ele é irreversível (one way). Para que a autenticação então aconteça, o que a aplicação faz é comparar o valor inserido pelo usuário, gerar o hash desse valor e comparar com o hash que já está salvo no banco de dados, se caso os hashes forem idênticos irá autenticar o usuário, caso contrário não irá.

### Integridade

Alguns programas ou arquivos que baixamos em determinados sites, possui um checksum que gerado assim que ele foi criado. Isso garante que o conteúdo dentro desse programa esteja íntegro com base nesse hash gerado. Suponhamos que há outro site que disponibiliza esse mesmo arquivo ou programa, pode-se então tirar o checksum dele e comparar com o arquivo original para ver se os hashes são os mesmos. Se caso não for, esse programa foi modificado de alguma forma e com isso sabemos que ele não é íntegro, ou seja, não é confiável.

## Gerando um hash

Se gerarmos o hash da string "xisde", temos alguns exemplos das saídas a seguir (lembrando que usamos o parâmetro `-n` para não pular a linha).

Exemplo em MD5

```
$ echo -n "xisde" | md5sum
8933f5410778ad2e66e72cc6248bbab0
```

Exemplo em SHA 256

```
echo -n "xisde" | sha256sum 
5bb140d27e2cb1708f82b34305b78434c6c3b96cbd5c2e3a89724e17e0fffd43
```

Independente se está sendo gerado uma hash de uma única string, o hash sempre terá esses tamanhos fixos de acordo com o tipo.

Se quisermos gerar um hash em MD5 de determinado arquivo, basta utilizar da seguinte forma.

```
md5sum <file>
```

## One way x Two way

Uma vez que o hash é gerado, não é possível voltar o valor original, ou seja, ele é irreversível (**one way**), não existe um "decode" para o hash. Para "quebrar um hash" então, é necessário que haja uma wordlist com diversas possíveis senhas e sejam gerados os hashes dessas senhas, fazendo assim um bruteforce até descobrir se alguma das possibilidades bate de acordo com o hash verdadeiro.

No entanto, existem métodos de codificação, por exemplo base64, que é próprio para codificar e decodificar (diferente do hash).

```
$ echo -n "xisde" | base64 
eGlzZGU=

$ echo -n "eGlzZGU=" | base64 -d
xisde
```

Com isso, conseguimos passar um valor para codificar e se passarmos o resultado da codificação, é possível fazer o processo reverso, sendo assim `two way`.

Podemos também codificar arquivos com bas64 e depois reverter, fazendo virar um arquivo novamente.