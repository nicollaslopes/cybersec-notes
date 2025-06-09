# Brute force in passwords

A técnica de brute force, é basicamente tentar descobrir uma senha através de tentativa e erro. É feito através de uma wordlist que pode ser personalizada ou feita especialmente para determinado alvo. Existem sistemas que bloqueiam o acesso do usuário caso ele erre a senha determinadas vezes, nesse caso um ataque de brute force é pouco efetivo.

## Wordlists

As wordlists do Kali Linux ficam localizadas em `/usr/share/wordlists`

Podemos instalar o seclists ou visualizar no github, que já vem com diversas wordlists prontas para vários serviços.

```
$ sudo apt install seclists
```

{% embed url="https://github.com/danielmiessler/SecLists" %}

### Gerando mutação em wordlist

Podemos usar o john para criar uma wordlist onde passamos algumas palavras e será feito uma mutação, ou seja, irá ser gerado com essas palavras várias outras porém com algum número adicionado. com letra maiúscula, caracteres especiais, entre outras coisas.

```
$ cat wordlist1
steve
stev
steven

$ john --wordlist=wordlist1 --rules --stdout > final_wordlist.txt
```

## Gerando wordlists personalizadas

Para personalizar uma wordlist, podemos utilizar uma ferramenta chamada `crunch`.

```
$ crunch <min-char> <max-char> -t <char> -o <output-file>
```

Example:

```
$ crunch 7 7 -t steve%^ -o wordlist
```

% dígitos

^ caracteres especiais

@ caracteres minúsculos

, caracteres maiúsculos

## Hydra

O hydra é uma ferramenta bem antiga e conhecida para realizar ataques de brute force.

```
$ hydra -v -l <usuário> -p <senha> <host> <protocolo>
```

Se utilizarmos os parâmetros maiúsculo, devemos passar uma lista para ser carregada.

```
$ hydra -v -L <users.txt> -P <passwords.txt> <host> <protocol>
```

*Se o host utilizar uma outra porta para determinado serviço e não a padrão, pode-se usar o parâmetro -s para indicar a porta. É interessante também utilizar o parâmetro -W para informar um tempo para cada requisição, se caso houver algum bloqueio de tentativas no alvo.*

## Low Hanging Fruit

A ideia dessa técnica é buscar os pontos mais fracos como por exemplo credenciais padrões. Podemos utilizar credenciais padrões de serviços ou até mesmo credenciais válidas de usuários assim que conseguimos algum acesso no sistema.

Para começar, vamos descobrir todos os hosts que possui determinado serviço disponível e jogar a saída para um arquivo `hosts`.

```
$ sudo nmap -v --open -sS -p 22 -oG ssh.txt 172.16.1.0/24
$ cat ssh.txt | grep "Up" | cut -d " " -f 2 > hosts
```

Agora podemos jogar no hydra para só testar nos hosts que descobrimos.

```
$ hydra -v -l root -p root -M hosts ssh
```
