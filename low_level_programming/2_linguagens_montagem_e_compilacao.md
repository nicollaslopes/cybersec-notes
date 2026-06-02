# Linguagens, montagem e compilação

## Objetivos

- Distinguir as etapas de tradução de códigos-fonte para código de máquina.
- Conhecer algumas das formas como linguagens de programação são implementadas.
- Utilizar o NASM para montar arquivos objeto de programas em Assembly.
- Utilizar o ligador `ld` para gerar binários executáveis a partir de objetos.
- Utilizar o `gcc` para compilar programas em C.
- Conhecer algumas ferramentas do GNU/Linux para inspecionar arquivos binários.
- Utilizar o programa `objdump` para desmontar conteúdos de binários executáveis.

## Do código-fonte ao binário

Quando programamos um computador, geralmente escrevemos textos que correspondem a instruções em linguagens que nós, humanos, conseguimos ler com maior ou menor facilidade. Os arquivos com os textos escritos nessas linguagens são chamados de _códigos-fonte_, mas o processador da máquina (a sua CPU) não é capaz de interpretá-los diretamente. Para a CPU, instruções são os conjuntos de sinais elétricos digitais que ela identifica como _códigos de operação_ (_OpCodes_) ou, de forma mais genérica, como _código de máquina_.

### Arquivos texto e binários

O caminho entre o que nós escrevemos nos códigos-fonte dos nossos programas e o código de máquina, que a CPU consegue executar, envolve toda uma série de transformações que resulta na montagem daquilo que nós chamamos de _arquivo binário executável_. No fundo, todo arquivo é binário, ou seja, todos eles contêm informações que podem ser convertidas em estados elétricos alto ou baixo que, no fim das contas, equivalem a números escritos na base 2 (números _binários_).

> **Curiosidade:** consegue notar que programas são apenas números gigantescos?

O que diferencia os arquivos texto, em relação aos binários, é que os números que eles contêm são escritos de modo a codificar a representação de caracteres utilizados para formar textos (codificação ASCII, por exemplo). Nos arquivos binários, por sua vez, o que importa é a codificação de dados, que podem representar diversos tipos de informações, como os _pixels_ de uma imagem, a intensidade sonora ao longo do tempo de uma música ou, no nosso caso, as instruções de um programa em código de máquina.

### Formato de binário executável

Voltando aos arquivos binários executáveis, além das instruções e dados do programa, eles devem conter informações estruturadas que serão utilizadas pelo sistema operacional para, primeiro, determinar se o arquivo é um binário executável válido e, segundo, para definir como suas várias partes serão copiadas para a memória, se ele puder ser executado. A estrutura do binário, portanto, deve obedecer às convenções específicas de cada sistema operacional para arquivos executáveis. Esse conjunto de regras é chamado de _formato de executável_ e no sistema GNU/Linux (e sistemas _Unix-like_, em geral) é utilizado o formato ELF (de _Executable and Linkable Format_), que é aplicado a:

- Arquivos binários executáveis
- Arquivos objeto
- Bibliotecas compartilhadas
- Imagens de memória geradas em casos de falha (_core dumps_)

A parte _"linkable"_ do formato ELF refere-se ao fato de que os arquivos objeto gerados podem ser ligados (isto é, combinados com outros objetos binários) para formar programas executáveis completos, seja por meio de ligações estáticas ou dinâmicas:

- **Ligação estática:** todos os objetos e bibliotecas são combinados em um único binário executável.
- **Ligação dinâmica:** o binário executável contém referências a bibliotecas externas que serão carregadas na memória em tempo de execução.

Seja qual for o modo de ligação, os objetos e bibliotecas compartilhadas devem seguir as especificações do formato ELF.

## Programação em código de máquina

É possível, embora raro e extremamente trabalhoso, escrever diretamente em código de máquina: ou seja, especificando manualmente os bytes que representam as instruções da CPU. Abaixo, nós temos o arquivo texto com os 96 bytes de um programa escrito diretamente em código de máquina (**ok.txt**):

```
7f 45 4c 46 01 01 01 00 00 00 00 00
00 00 00 00 02 00 03 00 01 00 00 00
54 80 04 08 34 00 00 00 00 00 00 00
00 00 00 00 34 00 20 00 01 00 28 00
00 00 00 00 01 00 00 00 54 00 00 00
54 80 04 08 00 00 00 00 0c 00 00 00
0c 00 00 00 05 00 00 00 00 10 00 00
b8 01 00 00 00 bb 00 00 00 00 cd 80
```

Para criar um arquivo binário com esses bytes, eu posso utilizar o utilitário `xxd`, geralmente disponível quando se instala o editor Vim:

```
:~$ xxd -r -p ok.txt > ok.bin
:~$ ls
ok.bin  ok.txt
```

Onde:

- `-r`: Impressão reversa (de texto para binário).
- `-p`: Despejo bruto hexadecimal (apenas os dígitos válidos para hexadecimais).
- `>`: Operador de redirecionamento de escrita do shell.

O `xxd` também pode ser utilizado para visualizar o conteúdo do arquivo binário:

```
:~$ xxd ok.bin
00000000: 7f45 4c46 0101 0100 0000 0000 0000 0000  .ELF............
00000010: 0200 0300 0100 0000 5480 0408 3400 0000  ........T...4...
00000020: 0000 0000 0000 0000 3400 2000 0100 2800  ........4. ...(.
00000030: 0000 0000 0100 0000 5400 0000 5480 0408  ........T...T...
00000040: 0000 0000 0c00 0000 0c00 0000 0500 0000  ................
00000050: 0010 0000 b801 0000 00bb 0000 0000 cd80  ................
```

Para facilitar as próximas demonstrações, nós vamos formatar a impressão em 12 colunas com grupos de apenas 1 byte:

```
:~$ xxd -c 12 -g 1 ok.bin
00000000: 7f 45 4c 46 01 01 01 00 00 00 00 00  .ELF........
0000000c: 00 00 00 00 02 00 03 00 01 00 00 00  ............
00000018: 54 80 04 08 34 00 00 00 00 00 00 00  T...4.......
00000024: 00 00 00 00 34 00 20 00 01 00 28 00  ....4. ...(.
00000030: 00 00 00 00 01 00 00 00 54 00 00 00  ........T...
0000003c: 54 80 04 08 00 00 00 00 0c 00 00 00  T...........
00000048: 0c 00 00 00 05 00 00 00 00 10 00 00  ............
00000054: b8 01 00 00 00 bb 00 00 00 00 cd 80  ............
```

Os primeiros 4 bytes (`7f 45 4c 46`) são o _número mágico_ que identifica o formato ELF para o sistema operacional. Na coluna da representação em ASCII, na saída do `xxd`, nós podemos ver que os 3 últimos bytes do número mágico correspondem aos caracteres `ELF`. Isso pode ser encontrado em qualquer arquivo binário, por exemplo, em um arquivo de imagem PNG:

```
:~$ xxd -c 12 -g 1 foto.png | head -1
00000000: 89 50 4e 47 0d 0a 1a 0a 00 00 00 0d  .PNG........
```

A primeira coluna da saída do `xxd` mostra, em hexadecimal a posição que o primeiro byte de cada linha impressa ocupa no arquivo. Com base nisso, observe que tudo que vem antes do byte `0x54` (byte na posição 84) são dados calculados ou convencionados para atender às especificações do formato de arquivos binários executáveis ELF32 (32 bits) – o programa em si, são apenas os 12 últimos bytes:

```
00000054: b8 01 00 00 00 bb 00 00 00 00 cd 80  ............
```

Aqui, estão os códigos de operação (_OpCodes_) da arquitetura x86 e seus respectivos operandos. Em Assembly 32 bits, eles corresponderiam a:

```
b8 01 00 00 00  ->  mov eax, 0x00000001 ; Registrar o valor 1 em EAX
bb 00 00 00 00  ->  mov ebx, 0x00000000 ; Registrar o valor 0 em EBX
cd 80           ->  int 0x80            ; Executar a interrupção 0x80
```

Nós falaremos sobre registradores e chamadas de sistemas mais adiante no curso, mas esta tradução "reversa" (de código de máquina para Assembly) mostra que, seguindo as convenções de chamadas de sistema do Linux, nos permite ver que:

- O valor `1` foi copiado no registrador `EAX`, para informar que eu queria executar a chamada de sistema `exit`;
- O valor `0` foi copiado no registrador `EBX`, para definir o valor que seria retornado pelo programa ao terminar (estado de término);
- E a interrupção `0x80` deveria ser executada para invocar a chamada de sistema definida em `EAX` com o argumento em `EBX`.

Resumindo, meu programa não faz nada além de terminar com estado `0` (que significa _sucesso_, para o sistema). Para confirmar, vamos dar permissão de execução ao arquivo binário e observar seu estado de término:

:~$ chmod +x ok.bin
:~$ ./ok.bin
:~$ echo $?
0

Mas podemos alterar o programa de modo que ele terminasse retornando, por exemplo, o valor `42`. Para isso, nós precisamos de um editor de arquivos binários, como o `hexedit` ou o `bvi`. Como sou mais familiarizado com os editores Vi e Vim, eu vou utilizar o `bvi`. Para iniciar a edição:

:~$ bvi ok.bin

No editor, nós veremos isso:

00000000  7F 45 4C 46 01 01 01 00 00 00 00 00 00 00 00 00 02 00 03 00 .ELF................
00000014  01 00 00 00 54 80 04 08 34 00 00 00 00 00 00 00 00 00 00 00 ....T...4...........
00000028  34 00 20 00 01 00 28 00 00 00 00 00 01 00 00 00 54 00 00 00 4. ...(.........T...
0000003C  54 80 04 08 00 00 00 00 0C 00 00 00 0C 00 00 00 05 00 00 00 T...................
00000050  00 10 00 00 B8 01 00 00 00 BB 00 00 00 00 CD 80             ................

O valor que eu quero alterar está na linha iniciada pelo byte na posição `0x50`, logo depois do _OpCode_ `BB` (`MOV EBX`). O valor atual é formado por quatro bytes (32 bits) escritos na ordem _little endian_, ou seja, os bytes menos significativos vêm antes dos mais significativos. Portanto, se nós queremos escrever `0x0000002a` (valor hexadecimal de 42 em 4 bytes), o último byte (`2a`) deverá vir na frente dos demais.

Para iniciar a edição do arquivo, nós precisamos executar o comando:

:set memmove

Agora, posicionando o cursor no byte que queremos substituir, nós teclamos `s` e digitamos `2a`, o que deixará a linha editada assim:

00000050  00 10 00 00 B8 01 00 00 00 BB 2A 00 00 00 CD 80             ..........*.....
                                        ↑

Teclando `ESC`, para voltar ao modo normal, nós podemos salvar e sair com o comando:

:wq

Verificando o novo conteúdo do arquivo binário:

:~$ xxd -c 12 -g 1 ok.bin
00000000: 7f 45 4c 46 01 01 01 00 00 00 00 00  .ELF........
0000000c: 00 00 00 00 02 00 03 00 01 00 00 00  ............
00000018: 54 80 04 08 34 00 00 00 00 00 00 00  T...4.......
00000024: 00 00 00 00 34 00 20 00 01 00 28 00  ....4. ...(.
00000030: 00 00 00 00 01 00 00 00 54 00 00 00  ........T...
0000003c: 54 80 04 08 00 00 00 00 0c 00 00 00  T...........
00000048: 0c 00 00 00 05 00 00 00 00 10 00 00  ............
00000054: b8 01 00 00 00 bb 2a 00 00 00 cd 80  ......*.....

Como o arquivo já tem as permissões necessárias, nós podemos executá-lo e verificar seu estado de término:

:~$ ./ok.bin
:~$ echo $?
42

## [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-6)Linguagens de baixo e alto nível

Criar e editar binários em código de máquina já foi comum nos primórdios da computação e ainda aparece em contextos de engenharia reversa, fins didáticos ou em casos altamente especializados, mas é impraticável para a criação de programas de maior complexidade. Por isso, foram desenvolvidas linguagens que, embora ainda muito próximas da escrita de código de máquina, tornaram esse processo mais viável.

É o caso da linguagem Assembly, classificada como uma linguagem de _baixo nível_ porque, embora abstraia a escrita do código de máquina por meio de mnemônicos e pseudo instruções legíveis por humanos, ainda nos obriga a descrever, passo a passo, cada uma das instruções que a CPU deverá executar para alcançar a finalidade do programa. Já nas linguagens de alto nível, os passos da CPU são abstraídos, de modo que nós podemos descrever nossos programas em termos de procedimentos que devem ser realizados, em vez de como esses procedimentos são implementados para serem executados pela máquina.

> **Nota:** É por isso que se diz que, com linguagens de alto nível, nos escrevemos _"o que fazer"_, ao passo que, com linguagens de baixo nível nós escrevemos _"como fazer"_.

## [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-7)Sistemas de tradução de linguagens

Independentemente de estar escrito em uma linguagem de baixo ou alto nível, todo programa precisa, em algum momento, ser convertido em código de máquina para que possa ser executado pela CPU. Isso pode ocorrer por meio de tradução direta – como nos processos de compilação e montagem – ou de forma indireta, através da interpretação por outro programa que já esteja traduzido para código de máquina. Em outras palavras, os códigos-fonte só poderão ser executados se forem compilados, montados ou interpretados, dependendo das características da linguagem utilizada.

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-8)Linguagens interpretadas

No caso das linguagens interpretadas, como Lua, Python ou PHP, por exemplo, é o binário do interpretador que é executado para ler, analisar e executar o conteúdo do código-fonte, conforme as suas instruções são processadas. Isso significa que o código de máquina é produzido em tempo real: o que é bastante flexível e prático, mas pode apresentar um desempenho inferior, se comparado com programas executados diretamente a partir de seus binários.

De modo geral, programas interpretados passam pelas seguintes etapas até serem executados:

Leitura

O programa interpretador lê uma linha ou um bloco do código-fonte.

Análise léxica

O trecho lido é dividido em palavras-chave, identificadores, operadores e outros símbolos significativos (um processo chamado de _tokenização_).

Análise sintática

Os elementos léxicos são organizados conforme as regras gramaticais da linguagem, formando estruturas como expressões, comandos e blocos de controle.

Execução

O interpretador, com base na estrutura sintática encontrada, executa a operação correspondente por meio de rotinas internas que já estão em código de máquina.

O processo se repete até que todo o conteúdo do código-fonte seja interpretado e suas instruções executadas.

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-9)Linguagens compiladas

O termo _compilação_ é utilizado para designar, de forma geral, o processo de traduzir um código escrito numa linguagem de alto nível (como C, C++, Rust ou Pascal) para um arquivo binário no formato que o sistema computacional em questão seja capaz de carregar na memória e executar. Essa tradução é feita por programas chamados de _compiladores_, mas eles podem atuar de formas diferentes, a depender do conceito que pretendam implementar.

Tomando a linguagem C como exemplo, compiladores como o `cc` (comum em sistemas BSD e Unix) e o `gcc` (do Projeto GNU) realizam o processo de compilação em até quatro etapas principais:

Pré-processamento

As diretivas encontradas no código-fonte (como `#include` e `#define`), além de outros requisitos implícitos, são processadas para gerar uma versão expandida do programa, ainda escrita em C.

Podemos interromper a compilação após esta etapa e inspecionar o resultado com:

gcc -E arquivo.c

O resultado é impresso no terminal, sem gerar nenhum arquivo.

Compilação

O código C pré-processado é traduzido para linguagem Assembly. Esse passo pode ocorrer de forma interna, mas também pode ser observado explicitamente gerando um arquivo com extensão `.s`:

gcc -S arquivo.c

Note que esta etapa também é chamada de _compilação_, porque o termo se aplica tanto ao processo completo quanto a essa etapa específica.

Montagem

O código em Assembly (interno ou gerado) é convertido em um arquivo objeto binário, normalmente com extensão `.o`. Podemos interromper a compilação aqui com:

gcc -c arquivo.c

Link-edição (ou apenas _ligação_)

O arquivo objeto é combinado com outros arquivos objeto (caso haja) e com as bibliotecas necessárias. Isso pode resultar na junção de código proveniente de bibliotecas estáticas (ligações estáticas) e na inserção de referências a bibliotecas compartilhadas (ligações dinâmicas).

Ao final, nós teremos um binário executável formatado de acordo com o padrão do sistema (como ELF, no Linux). Essa etapa é realizada pelo programa `ld`, geralmente invocado automaticamente pelo compilador. Contudo, mesmo no caso da linguagem C, existem compiladores que adotam propostas diferentes, gerando código de máquina sem passar explicitamente por uma etapa intermediária de tradução para Assembly.

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-10)Linguagens de montagem

Quando se trata de linguagens de baixo nível, o que nós temos é, basicamente, um conjunto de mnemônicos para os _OpCodes_ da CPU e algumas pseudoinstruções para definir estruturas de dados e organizar as seções do que virá a ser o arquivo binário do programa. Em outras palavras, um código em Assembly pode ser visto como um "manual de construção" do código de máquina: daí o nome _"assembly"_, que significa _"montagem"_ em inglês. Consequentemente, o programa utilizado para montar o arquivo binário com o código de máquina a partir de um fonte em linguagem de montagem é chamado de "montador" – ou _"assembler"_, em inglês.

As pequenas (embora importantes) diferenças entre as várias linguagens de montagem existentes são determinadas, essencialmente, pelas particularidades dos montadores que as implementam e pelas especificações das arquiteturas de hardware e software para as quais esses montadores foram projetados. Além disso, montadores como o NASM (_Netwide Assembler_) e GAS (_GNU Assembler_), por exemplo, não produzem arquivos binários executáveis diretamente. Isso tem a vantagem de possibilitar o uso dos arquivos objeto gerados de várias formas, mas requer que uma etapa posterior de link-edição seja realizada para que se tenha um binário executável. Já o montador FASM (_Fast Assembler_), também bastante utilizado, dispensa a etapa final de link-edição, mas sua integração com código em C não é tão direta ou automatizada quanto em montadores como o NASM, exigindo mais controle manual por parte de quem programa.

Neste curso, a escolha do montador NASM deve-se a alguns motivos:

- Utiliza a sintaxe Intel por padrão.
- Tem um suporte a macros simples e poderoso.
- Tem um ótimo suporte à integração com código em C.
- É uma linguagem fartamente documentada.
- Pode ser aprendida com muita facilidade por iniciantes.

## [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-11)Um programa em Assembly

Aqui, nós temos um programa escrito em Assembly 64 bits (NASM) para imprimir a mensagem `Salve, simpatia!` no terminal e sair com estado de término `0`…

Arquivo: [salve.asm](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/exemplos/02/salve.asm)

```nasm
section .rodata
    msg db "Salve, simpatia!", 10
    len equ $ - msg

section .text
    global _start

_start:
    mov rax, 1          ; syscall write
    mov rdi, 1          ; stdout
    mov rsi, msg
    mov rdx, len
    syscall

    mov rax, 60         ; syscall exit
    mov rdi, 0          ; código de saída 0
    syscall
```

Por enquanto, não importa entender o que está no código, mas observar seu aspecto geral que, claramente, define passo a passo o que a CPU deve fazer e organiza os dados e as instruções nas seções que deverão ocupar no arquivo binário resultante. A mensagem que queremos imprimir, por exemplo, está na seção chamada de `.rodata`, que é onde dados constantes serão escritos no arquivo binário.

Montando e link-editando o programa:

:~$ nasm -f elf64 -o salve.o salve.asm
:~$ ld -o salve salve.o

Como não foram exibidas mensagens de erro ou de aviso, nós sabemos que tudo correu bem e já podemos executar o programa:

:~$ ./salve
Salve, simpatia!

A partir daqui, vamos utilizar diversas ferramentas disponíveis para o GNU/Linux que nos ajudarão o obter informações sobre arquivos binários de programas.

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-12)Tamanho do binário em bytes

Com o utilitário `ls`:

:~$ ls -l salve
-rwxrwxr-x 1 blau blau 8880 mai 16 10:15 salve

Com o utilitário `du`:

:~$ du -b salve
8880

Com o utilitário `stat`:

:~$ stat -c '%s' salve
8880

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-13)Informações gerais do arquivo

Com o utilitário `file`:

:~$ file salve
salve: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically
linked, not stripped

Com o utilitário `stat`:

:~$ stat salve
    Arquivo: salve
    Tamanho: 8880      	Blocos: 24         bloco de E/S: 4096   regular file
Dispositivo: 8,18	Inode: 4856471     Ligações: 1
     Acesso: (0775/-rwxrwxr-x)  Uid: ( 1000/    blau)   Gid: ( 1000/    blau)
     Acesso: 2025-05-16 10:15:50.611903536 -0300
Modificação: 2025-05-16 10:15:46.279952507 -0300
  Alteração: 2025-05-16 10:15:46.279952507 -0300
    Criação: 2025-05-16 10:15:46.275952553 -0300

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-14)Cabeçalho do formato ELF

:~$ readelf -h salve
Cabeçalho ELF:
  Magia:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Classe:                            ELF64
  Dados:                             complemento 2, little endian
  Versão:                            1 (actual)
  OS/ABI:                            UNIX - System V
  Versão ABI:                        0
  Tipo:                              EXEC (ficheiro executável)
  Máquina:                           Advanced Micro Devices X86-64
  Versão:                            0x1
  Endereço do ponto de entrada:               0x401000
  Início dos cabeçalhos do programa:          64 (bytes no ficheiro)
  Start of section headers:          8496 (bytes no ficheiro)
  Bandeiras:                         0x0
  Tamanho deste cabeçalho:           64 (bytes)
  Tamanho dos cabeçalhos do programa:56 (bytes)
  Nº de cabeçalhos do programa:      3
  Tamanho dos cabeçalhos de secção:  64 (bytes)
  Nº dos cabeçalhos de secção:       6
  Índice de tabela de cadeias da secção: 5

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-15)Lista de seções do programa

:~$ readelf -l salve

Tipo de ficheiro Elf é EXEC (ficheiro executável)
Entry point 0x401000
There are 3 program headers, starting at offset 64

Cabeçalhos do programa:
  Tipo           Desvio             EndVirtl           EndFís
                 TamFich            TamMem              Bndrs  Alinh
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x00000000000000e8 0x00000000000000e8  R      0x1000
  LOAD           0x0000000000001000 0x0000000000401000 0x0000000000401000
                 0x0000000000000025 0x0000000000000025  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000402000 0x0000000000402000
                 0x0000000000000011 0x0000000000000011  R      0x1000

 Secção para mapa do segmento:
  Secções do segmento...
   00
   01     .text
   02     .rodata

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-16)Despejo do conteúdo (em hexa) da seção .text

:~$ readelf -x .text salve

Despejo máximo da secção ".text":
  0x00401000 b8010000 00bf0100 000048be 00204000 ..........H.. @.
  0x00401010 00000000 ba110000 000f05b8 3c000000 ............<...
  0x00401020 4831ff0f 05                         H1...

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-17)Despejo do conteúdo (em hexa) da seção .rodata

:~$ readelf -x .rodata salve

Despejo máximo da secção ".rodata":
  0x00402000 53616c76 652c2073 696d7061 74696121 Salve, simpatia!
  0x00402010 0a                                  .

## [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-18)Uma versão equivalente em C

Tentando manter a maior proximidade possível com o que foi escrito em Assembly, esta seria uma possível versão equivalente em C…

Arquivo: [salve.c](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/exemplos/02/salve.c)

```c
#include <unistd.h>

// msg db "Salve, simpatia!", 10 
const char msg[] = "Salve, simpatia!\n";

// len equ $ - msg 
#define LEN sizeof(msg) - 1

int main(void) {
    /*
    mov rax, 1          ; syscall write
    mov rdi, 1          ; stdout
    mov rsi, msg
    mov rdx, len
    syscall
    ,*/
    write(1, msg, LEN);

    /*
    mov rax, 60         ; syscall exit
    mov rdi, 0          ; código de saída 0
    syscall
    ,*/
    _exit(0);
}
```

Compilando e executando:

:~$ gcc -o salvec salve.c
:~$ ./salvec
Salve, simpatia!

Apesar de fazer a mesma coisa praticamente do mesmo modo, o arquivo binário gerado a partir de um código em C tem diferenças importantes, como veremos a seguir.

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-19)Tamanho do binário em bytes:

:~$ du -b salvec
16032

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-20)Informações gerais do arquivo

:~$ file salvec
salvec: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically
linked, interpreter /lib64/ld-linux-x86-64.so.2,
BuildID[sha1]=351e5395c71e0b758b9cc05a68f3813f8f130817, for GNU/Linux 3.2.0,
not stripped

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-21)Cabeçalho do formato ELF

:~$ readelf -h salvec
Cabeçalho ELF:
  Magia:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Classe:                            ELF64
  Dados:                             complemento 2, little endian
  Versão:                            1 (actual)
  OS/ABI:                            UNIX - System V
  Versão ABI:                        0
  Tipo:                              DYN (Position-Independent Executable file)
  Máquina:                           Advanced Micro Devices X86-64
  Versão:                            0x1
  Endereço do ponto de entrada:               0x1060
  Início dos cabeçalhos do programa:          64 (bytes no ficheiro)
  Start of section headers:          14048 (bytes no ficheiro)
  Bandeiras:                         0x0
  Tamanho deste cabeçalho:           64 (bytes)
  Tamanho dos cabeçalhos do programa:56 (bytes)
  Nº de cabeçalhos do programa:      14
  Tamanho dos cabeçalhos de secção:  64 (bytes)
  Nº dos cabeçalhos de secção:       31
  Índice de tabela de cadeias da secção: 30

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-22)Lista de seções do programa

:~$ readelf -l salvec

Tipo de ficheiro Elf é DYN (Position-Independent Executable file)
Entry point 0x1060
There are 14 program headers, starting at offset 64

Cabeçalhos do programa:
  Tipo           Desvio             EndVirtl           EndFís
                 TamFich            TamMem              Bndrs  Alinh
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x0000000000000310 0x0000000000000310  R      0x8
  INTERP         0x0000000000000394 0x0000000000000394 0x0000000000000394
                 0x000000000000001c 0x000000000000001c  R      0x1
      [A pedir interpretador do programa: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000660 0x0000000000000660  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x0000000000000179 0x0000000000000179  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x0000000000000118 0x0000000000000118  R      0x1000
  LOAD           0x0000000000002dd0 0x0000000000003dd0 0x0000000000003dd0
                 0x0000000000000250 0x0000000000000258  RW     0x1000
  DYNAMIC        0x0000000000002de0 0x0000000000003de0 0x0000000000003de0
                 0x00000000000001e0 0x00000000000001e0  RW     0x8
  NOTE           0x0000000000000350 0x0000000000000350 0x0000000000000350
                 0x0000000000000020 0x0000000000000020  R      0x8
  NOTE           0x0000000000000370 0x0000000000000370 0x0000000000000370
                 0x0000000000000024 0x0000000000000024  R      0x4
  NOTE           0x00000000000020f8 0x00000000000020f8 0x00000000000020f8
                 0x0000000000000020 0x0000000000000020  R      0x4
  GNU_PROPERTY   0x0000000000000350 0x0000000000000350 0x0000000000000350
                 0x0000000000000020 0x0000000000000020  R      0x8
  GNU_EH_FRAME   0x0000000000002024 0x0000000000002024 0x0000000000002024
                 0x000000000000002c 0x000000000000002c  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000002dd0 0x0000000000003dd0 0x0000000000003dd0
                 0x0000000000000230 0x0000000000000230  R      0x1

 Secção para mapa do segmento:
  Secções do segmento...
   00
   01     .interp
   02     .note.gnu.property .note.gnu.build-id .interp .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt
   03     .init .plt .plt.got .text .fini
   04     .rodata .eh_frame_hdr .eh_frame .note.ABI-tag
   05     .init_array .fini_array .dynamic .got .got.plt .data .bss
   06     .dynamic
   07     .note.gnu.property
   08     .note.gnu.build-id
   09     .note.ABI-tag
   10     .note.gnu.property
   11     .eh_frame_hdr
   12
   13     .init_array .fini_array .dynamic .got

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-23)Despejo do conteúdo (em hexa) da seção .text

:~$ readelf -x .text salvec

Despejo máximo da secção ".text":
  0x00001060 31ed4989 d15e4889 e24883e4 f0505445 1.I..^H..H...PTE
  0x00001070 31c031c9 488d3dce 000000ff 153f2f00 1.1.H.=......?/.
  0x00001080 00f4662e 0f1f8400 00000000 0f1f4000 ..f...........@.
  0x00001090 488d3d89 2f000048 8d05822f 00004839 H.=./..H.../..H9
  0x000010a0 f8741548 8b051e2f 00004885 c07409ff .t.H.../..H..t..
  0x000010b0 e00f1f80 00000000 c30f1f80 00000000 ................
  0x000010c0 488d3d59 2f000048 8d35522f 00004829 H.=Y/..H.5R/..H)
  0x000010d0 fe4889f0 48c1ee3f 48c1f803 4801c648 .H..H..?H...H..H
  0x000010e0 d1fe7414 488b05ed 2e000048 85c07408 ..t.H......H..t.
  0x000010f0 ffe0660f 1f440000 c30f1f80 00000000 ..f..D..........
  0x00001100 f30f1efa 803d152f 00000075 2b554883 .....=./...u+UH.
  0x00001110 3dca2e00 00004889 e5740c48 8b3df62e =.....H..t.H.=..
  0x00001120 0000e829 ffffffe8 64ffffff c605ed2e ...)....d.......
  0x00001130 0000015d c30f1f00 c30f1f80 00000000 ...]............
  0x00001140 f30f1efa e977ffff ff554889 e5ba1100 .....w...UH.....
  0x00001150 0000488d 05b70e00 004889c6 bf010000 ..H......H......
  0x00001160 00e8dafe ffffbf00 000000e8 c0feffff ................

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-24)Despejo do conteúdo (em hexa) da seção .rodata

:~$ readelf -x .rodata salvec

Despejo máximo da secção ".rodata":
  0x00002000 01000200 00000000 00000000 00000000 ................
  0x00002010 53616c76 652c2073 696d7061 74696121 Salve, simpatia!
  0x00002020 0a00                                ..

## [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-25)Desmontagem comparativa

Com a opção `-d`, o utilitário `objdump` pode desmontar arquivos binários objeto ou executáveis, ou seja, ele exibe um código em Assembly que corresponde ao código de máquina encontrado no binário. Por exemplo, com o binário do nosso programa `salve.asm`, nós podemos executar:

:~$ objdump -d salve

salve:     formato de ficheiro elf64-x86-64


Desmontagem da secção .text:

0000000000401000 <_start>:
  401000:	b8 01 00 00 00       	mov    $0x1,%eax
  401005:	bf 01 00 00 00       	mov    $0x1,%edi
  40100a:	48 be 00 20 40 00 00 	movabs $0x402000,%rsi
  401011:	00 00 00
  401014:	ba 11 00 00 00       	mov    $0x11,%edx
  401019:	0f 05                	syscall
  40101b:	b8 3c 00 00 00       	mov    $0x3c,%eax
  401020:	48 31 ff             	xor    %rdi,%rdi
  401023:	0f 05                	syscall

O resultado foi a impressão de um código em Assembly escrito com a sintaxe AT&T. Se quisermos que a desmontagem seja feita com a sintaxe Intel, nós precisaremos incluir a opção `-M intel`:

:~$ objdump -d -M intel salve

salve:     formato de ficheiro elf64-x86-64


Desmontagem da secção .text:

0000000000401000 <_start>:
  401000:	b8 01 00 00 00       	mov    eax,0x1
  401005:	bf 01 00 00 00       	mov    edi,0x1
  40100a:	48 be 00 20 40 00 00 	movabs rsi,0x402000
  401011:	00 00 00
  401014:	ba 11 00 00 00       	mov    edx,0x11
  401019:	0f 05                	syscall
  40101b:	b8 3c 00 00 00       	mov    eax,0x3c
  401020:	48 31 ff             	xor    rdi,rdi
  401023:	0f 05                	syscall

Se colocarmos o código desmontado ao lado do código original, nós veremos que eles são praticamente os mesmos:

|Código original|Código desmontado|Diferença|
|---|---|---|
|`mov rax, 1`|`mov eax, 0x1`|Utilizados apenas 4 bytes de RAX (EAX)|
|`mov rdi, 1`|`mov edi, 0x1`|Utilizados apenas 4 bytes de RDI (EDI)|
|`mov rsi, msg`|`movabs rsi, 0x402000`|Rótulo `msg` substituído pelo seu endereço absoluto|
|`mox rdx, len`|`mov edx, 0x11`|Macro `len` substituída pelo seu valor (`0x11` = `17`)|
|`syscall`|`syscall`||
|`mov rax, 60`|`move eax, 0x3c`|Utilizados apenas 4 bytes de RAX (EAX)|
|`mov rdi, 0`|`xor rdi, rdi`|O valor `0` foi obtido por uma operação XOR bit a bit com o operador RDI|
|`syscall`|`syscall`||

Essas diferenças não foram "inventadas" pelo `objdump`: elas estão no binário gerado pelo montador NASM e resultam do preprocessamento do fonte e de várias estratégias de otimização, como:

- Antes da montagem começar, a expressão `equ`, no rótulo `len`, é avaliada e o valor resultante é substituído em todas as ocorrências de `len` no fonte.
- Rótulos de dados, como `msg`, são substituídos pelo deslocamento de endereços (_offset_) que representam (`0x402000`, no nosso arquivo binário).
- Por que ocupar 8 bytes (RAX) com um valor numérico que pode ser escrito com apenas 4 bytes (EAX)?
- Por que ocupar 5 bytes (`mov rdi, 0`) se o mesmo resultado pode ser obtido com apenas 3 bytes (`xor rdi, rdi`)?
- O mnemônico `movabs` é uma convenção de alguns montadores 64 bits (como o GAS) para explicitar que o valor copiado para o registrador deve ser escrito com 8 bytes e não pode ser otimizado para 4 bytes.

> **Nota:** O NASM trata automaticamente a necessidade, ou não, de utilizar os 64bits do registrador e não precisamos utilizar `movabs`. Na desmontagem, porém, o `objdump` tenta deixar explícito que é isso que está acontecendo no binário e, portanto, escreve `mov rdi, <valor>` como `movabs rdi, <valor>`. Contudo, os dados binários ainda correspondem à operação `mov` (`48 BE <valor com 8 bytes>`).

## [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-26)Exercícios propostos

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-27)1. Desmonte e analise o programa em C

Desmonte o binário executável `salvc`, pesquise e analise as causas das muitas diferenças encontradas em relação à desmontagem do programa `salve`.

### [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-28)2. Desmonte e analise o programa em código de máquina

Com o comando abaixo, desmonte o binário `ok.bin`, pesquise e analise o código em assembly exibido:

objdump -b binary -m i386 -M intel -D ok.bin

No mínimo, tente localizar e analisar o código relativo ao que o programa efetivamente faz.

## [](https://bolha.dev/blau_araujo/pbn/src/branch/main/curso/aula-02.org#headline-29)Referências

- `man xxd`
- `man 2 write`
- `man 2 exit`
- `man bvi`
- `man readelf`
- `man du`
- `man stat`
- `man file`
- `man objdump`
- [https://nasm.us/doc/](https://nasm.us/doc/)