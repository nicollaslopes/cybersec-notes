# O formato Binário ELF

## Objetivos

- Conhecer a estrutura de arquivos executáveis no formato ELF.
- Compreender a definição das seções do programa.
- Relacionar o binário ELF com seu conteúdo em Assembly.
- Explorar as seções do binário com `readelf`, `xxd` e `gdb`.

## O que é o formato ELF

O formato binário ELF, de _Executable and Linking Format_ (_Formato Executável e de Ligação_), foi originalmente desenvolvido e publicado como parte da ABI (_Interface Binária de Aplicações_) do Unix System V Release 4 (SVR4). Desde então, tornou-se o padrão adotado pela maioria dos sistemas _Unix-like_ para representar arquivos objeto binários, como executáveis e bibliotecas.

### Principais tipos de arquivos objeto

Os arquivos objeto, criados por montadores e editores de ligações (_link-editores_), são representações binárias de programas destinados a serem executados diretamente por um processador. As especificações do ELF para o Linux definem três tipos principais de arquivos objeto:

**Arquivo relocável**
	Contém código e dados preparados para serem ligados a outros objetos para criar um executável ou um objeto compartilhado.

**Arquivo executável**
	O arquivo de um programa que pode ser executado.

**Arquivo objeto compartilhado**
	Contém código e dados que podem ser ligados de duas formas: com outros arquivos relocáveis para criar um outro objeto, ou com um executável e outros objetos compartilhados para formar a imagem de um processo.

## Formato do arquivo

Os arquivos objeto participam tanto da link-edição quanto da execução de programas. Portanto, seu formato deve acomodar as estruturas necessárias para ambas as atividades. Na link-edição, o foco está nas seções e na tabela de cabeçalhos de seções, enquanto que, na execução, o carregador (_loader_) utiliza os cabeçalhos do programa para mapear segmentos na memória.

 ```
   LINK-EDIÇÃO (ARQUIVO)           EXECUÇÃO (MEMÓRIA)
  ┌────────────────────────┐     ┌────────────────────────┐
  │      CABEÇALHO ELF     │     │      CABEÇALHO ELF     │
  ├────────────────────────┤     ├────────────────────────┤
  │  TABELA DE CABEÇALHOS  │     │  TABELA DE CABEÇALHOS  │
  │ DO PROGRAMA (OPCIONAL) │     │       DO PROGRAMA      │
  ├────────────────────────┤     ├────────────────────────┤
  │        SEÇÃO 1         │     │                        │
  ├────────────────────────┤     │       SEGMENTO 1       │
  │        SEÇÃO 2         │     │                        │
  ├────────────────────────┤     ├────────────────────────┤
  │          ...           │     │                        │
  ├────────────────────────┤     │       SEGMENTO 2       │
  │        SEÇÃO N         │     │                        │
  ├────────────────────────┤     ├────────────────────────┤
  │          ...           │     │          ...           │
  ├────────────────────────┤     ├────────────────────────┤
  │  TABELA DE CABEÇALHOS  │     │  TABELA DE CABEÇALHOS  │
  │       DE SEÇÕES        │     │  DE SEÇÕES (OPCIONAL)  │
  └────────────────────────┘     └────────────────────────┘
 ```

**Importante!**

- O posicionamento real das tabelas de seções e do programa pode ser diferente de como está representado nos diagramas.
- Do mesmo modo, as seções e os seguimentos não têm uma ordem específica.
- Só o cabeçalho ELF tem uma posição fixa no arquivo.
- Um segmento na memória pode conter várias seções do arquivo.

No diagrama…

**Cabeçalho ELF**
	Escrito nos primeiros 52 ou 64 bytes do arquivo, o cabeçalho ELF contém um resumo da sua organização e diversas informações, como o formato do arquivo (32 ou 64 bits), a ordem de escrita dos bytes de dados (_little_ ou _big endian_), o tipo do arquivo objeto (se é relocável, executável ou compartilhado), a arquitetura do conjunto de instruções, entre outras definições. Os primeiros 4 bytes do cabeçalho ELF contêm a assinatura do formato, o seu _número mágico_: o byte `0x7F` seguido dos bytes dos caracteres `E`, `L` e `F` na tabela ASCII (`45 4C 46`).

**Seções**
	Contêm a organização dos dados do programa para efeito da edição das ligações no arquivo, como as instruções do código, dados globais contantes e variáveis, símbolos, informações de relocação, etc.

**Tabela de cabeçalhos do programa**
	Se existir, diz ao programa como montar uma imagem do programa quando ele for carregado na memória para execução. Logo, se o arquivo objeto for executável, ele terá que conter uma tabela de cabeçalhos do programa, o que é desnecessário em arquivos objeto relocáveis.

**Tabela de cabeçalhos de seções**
	Contém a descrição das seções do arquivo. Cada seção tem uma entrada na tabela com informações como seu nome, seu tamanho, a partir de onde pode ser encontrada no arquivo (_offset_), etc. Arquivos utilizados durante a edição de ligações precisam ter a tabela de cabeçalhos de seções, mas ela é desnecessária no caso de arquivos objeto que só serão ligados dinamicamente em tempo de execução.

## Seções especiais

Várias seções de um arquivo ELF são predefinidas para conter informações de controle utilizadas pelo sistema operacional. Na pŕática, programas executáveis são formados vários arquivos objeto e bibliotecas vinculados através do processo de ligação, que pode ser:

- **Estática:** Vários arquivo objeto são combinados em um só arquivo executável.
- **Dinâmica:** O objeto executável é ligado na memoria, em tempo de execução, com objetos compartilhados e bibliotecas disponíveis no sistema.

No fim das contas, o resultado será sempre um programa completo na memória, a diferença está em quando o objeto executável é construído e em quem resolve e processa as ligações.

No GNU/Linux:

- **Ligações estáticas:** Processadas pelo editor de ligações (`ld`, da suíte GNU Binutils).
- **Ligações dinâmicas:** Processadas pelo carregador e ligador dinâmico do sistema (`ld-linux`, da `glibc`).

Seja a ligação realizada estaticamente, durante a construção do executável, ou dinamicamente, no momento da execução, o correto processamento das ligações depende da organização interna do arquivo ELF. Essa organização é descrita por suas seções especiais, entre as quais podemos destacar:

|Seção|Descrição|
|---|---|
|`.bss`|Contém dados globais não inicializados ou inicializados com zero; ocupa espaço na memória, mas não ocupa espaço no arquivo executável.|
|`.data`|Contém dados globais inicializados; ocupa espaço tanto no arquivo quanto na memória.|
|`.rodata`|Contém dados que não podem ser alterados (_read only_), geralmente usada para constantes e strings literais.|
|`.text`|Seção que contém o código executável do programa (instruções da CPU).|
|`.symtab`|Tabela de símbolos completa, usada pelo ligador e depuradores para localizar símbolos.|
|`.strtab`|Tabela de strings que armazena os nomes dos símbolos referenciados em `.symtab`.|

## Tipos de segmentos

Na execução de um programa ELF, o sistema operacional utiliza a tabela de cabeçalhos do programa para saber quais partes do arquivo devem ser carregadas na memória e como tratá-las. Cada entrada nessa tabela descreve um segmento que representa uma região de interesse para o carregador, como código do programa, seus dados, informações de ligação dinâmica e o caminho do carregador e ligador dinâmico. Esses segmentos são identificados por tipos padronizados, cada um com uma finalidade específica no processo de carregamento e execução do programa.

Aqui estão alguns tipos de segmentos:

|Tipo|Descrição|
|---|---|
|`LOAD`|Segmento que deve ser carregado na memória, contendo código ou dados.|
|`INTERP`|Indica o caminho do carregador dinâmico (`ld-linux`) para execução.|
|`DYNAMIC`|Contém informações para ligação dinâmica, como símbolos e dependências.|
|`GNU_STACK`|Define permissões da pilha (stack), como se pode ou não ser executável.|
|`NOTE`|Armazena metadados usados pelo sistema operacional.|
|`PHDR`|Contém a própria Tabela de Cabeçalhos do Programa, usada por depuradores.|
|`GNU_EH_FRAME`|Informações para suporte à pilha de exceções (C, C++, etc.).|
|`GNU_RELRO`|Segmento de dados que será tornado somente leitura após inicialização.|

## Definindo seções ELF em NASM para Linux 64 bits

Nós podemos definir qualquer seção ELF em Assembly NASM com a _diretiva_ `section`, ou com seu sinônimo `segment`, oriundo de uma terminologia mais antiga para arquiteturas x86 de 16 e 32 bits, onde o conceito de segmentação de memória era explícito. Em ambientes modernos, `section` é mais comum, pois reflete a nomenclatura utilizada no formato ELF e que está padronizada no editor de ligações `ld`, no _loader_ `ld-linux` e em ferramentas como `readelf` e `objdump`.

```asm
; Arquivo    : sections.asm
; Descrição  : Demonstra a definição das seções .rodata, .data, .bss e .text
; Montagem   : nasm -f elf64 sections.asm
; Link-edição: ld sections.o -o sections

section .rodata
	msg db "Eu sou imutável!", 0x0a
	len equ $ - msg

section .data
	contador dq 0		; preenche 8 bytes (qword) com 0x00 em 'contador'

section .bss
	buffer resb 32		; reserva 32 bytes no endereço 'buffer'

section .text
global _start			; ponto de entrada do programa

_start:
	mov rax, 1		; syscall: write
	mov rdi, 1		; fd 1 = stdout
	mov rsi, msg		; endereço da mensagem
	mov rdx, len		; tamanho da mensagem
	syscall

	mov rax, [contador]	; copia o dado no endereço 'contador' para rax
	inc rax			; incrementa em 1 o valor em rax
	mov [contador], rax	; copia o valor em rax para o endereço 'contador'

	mov rax, 60		; syscall: exit
	xor rdi, rdi		; estado de término = 0
	syscall
```

Antes de falarmos das seções do programa, é importante saber que, no caso de programas em Assembly link-editados para serem carregados pelo sistema como executáveis ELF, a ordem das seções no código-fonte só é mantida no arquivo objeto montado (`.o`). Mas, quando o objeto é link-editado para gerar o arquivo executável, o link-editor (`ld`) reorganiza as seções de modo a atender as especificações do formato ELF.

Montando e executando o programa:

```
:~$ nasm -f elf64 sections.asm
:~$ ld sections.o -o sections
:~$ ./sections
Eu sou imutável!
```

Com o utilitário `readelf`, nós podemos listar os cabeçalhos do programa:

$ readelf -l sections

Tipo de ficheiro Elf é EXEC (ficheiro executável)
Entry point 0x401000
There are 4 program headers, starting at offset 64

```
$ readelf -l sections

Tipo de ficheiro Elf é EXEC (ficheiro executável)
Entry point 0x401000
There are 4 program headers, starting at offset 64

Cabeçalhos do programa:
  Tipo           Desvio             EndVirtl           EndFís
                 TamFich            TamMem              Bndrs  Alinh
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x0000000000000120 0x0000000000000120  R      0x1000
  LOAD           0x0000000000001000 0x0000000000401000 0x0000000000401000
                 0x0000000000000038 0x0000000000000038  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000402000 0x0000000000402000
                 0x0000000000000012 0x0000000000000012  R      0x1000
  LOAD           0x0000000000002014 0x0000000000403014 0x0000000000403014
                 0x0000000000000008 0x000000000000002c  RW     0x1000

 Secção para mapa do segmento:
  Secções do segmento...
   00
   01     .text
   02     .rodata
   03     .data .bss
```

Onde:

- A seção `.text` inicia no byte `0x1000` do arquivo e ocupa 56 bytes (`0x38`).
- A seção `.rodata` inicia no byte `0x2000` do arquivo e ocupa 18 bytes (`0x12`).
- A seção `.data` inicia no byte `0x2014` do arquivo e ocupa 8 bytes.
- A seção `.bss` é indicada para mapeamento na execução, mas não ocupa espaço no arquivo binário.

> **Nota:** Repare que as seções `.text` e `.rodata` só têm permissão de leitura (`R`), enquanto `.data` e `.bss` têm permissão de leitura e escrita (`RW`).

Nós podemos localizar a definição da seção `.bss` listando a tabela de cabeçalhos de seção, utilizada para orientar o mapeamento do executável nos segmentos da memória:

```
:~$ readelf -S sections
There are 8 section headers, starting at offset 0x2188:

Cabeçalhos de secção:
  [Nr] Nome              Tipo             Endereço          Desvio
       Tam.              Tam.Ent          Bands  Lig.  Info  Alinh
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000401000  00001000
       0000000000000038  0000000000000000  AX       0     0     16
  [ 2] .rodata           PROGBITS         0000000000402000  00002000
       0000000000000012  0000000000000000   A       0     0     4
  [ 3] .data             PROGBITS         0000000000403014  00002014
       0000000000000008  0000000000000000  WA       0     0     4
  [ 4] .bss              NOBITS           000000000040301c  0000201c
       0000000000000024  0000000000000000  WA       0     0     4
  [ 5] .symtab           SYMTAB           0000000000000000  00002020
       00000000000000f0  0000000000000018           6     6     8
  [ 6] .strtab           STRTAB           0000000000000000  00002110
       000000000000003e  0000000000000000           0     0     1
  [ 7] .shstrtab         STRTAB           0000000000000000  0000214e
       0000000000000034  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```

Aqui, nós podemos ver que `.bss` deve ocupar um espaço de 36 bytes na memória (`0x24`), a partir do endereço de _offset_ 0x40301c. A diferença entre os 32 bytes reservados no programa e os 36 bytes que vemos na tabela, é o resultado do alinhamento de 4 bytes aplicado pelo `ld`, como podemos ver na última coluna da tabela.

### Uma nota sobre alinhamento de dados

Em sistemas Linux 64 bits, é comum que dados sejam alinhados a endereços múltiplos de 8 bytes, o que corresponde à largura dos registradores e garante acesso eficiente e seguro à memória. Se pegarmos o _offset_ da seção `.bss` (`0x201c`) e somarmos os 32 bytes reservados no programa, nós veremos que o próximo dado teria que ser escrito no _offset_ `0x203c` (8252, em base 10), que não é múltiplo de 8 (`8252/8=1031.5`). Somando 4 ao próximo endereço, nós chegamos ao _offset_ `0x2040` (8256, em base 10), que é um múltiplo de 8 (`8256/8=1032.0`).

No entanto, observe que as seções `.rodata` e `.data` também tiveram alinhamento, mas seus tamanhos não são diferentes daqueles definidos no código-fonte. Isso aconteceu porque, diferente de `.bss`, os dados em `.rodata` e em `.data` foram inicializados e, por definição, `.bss` é uma região reservada para dados não inicializados – logo, o tamanho da seção inclui os bytes de alinhamento.

> **Nota:** O nome `.bss` vem da linguagem Fortran e significa _Block Started by Symbol_ (_bloco iniciado por um símbolo_, em tradução livre).

### Inspecionando a seção .rodata

```asm
section .rodata
	msg db "Eu sou imutável!", 0x0a
	len equ $ - msg
```

Aqui, o endereço de rótulo `msg` receberá a cadeia de bytes definida com a diretiva de _definição de bytes_ (`db`). Na montagem, esses bytes serão escritos na seção `.rodata`, onde não poderão ser alterados.

Como visto na tabela de cabeçalhos do programa, a seção `.rodata` está no _offset_ `0x002000` do arquivo e tem permissão apenas para leitura (`R`):

```
  Tipo           Desvio             EndVirtl           EndFís
                 TamFich            TamMem              Bndrs  Alinh
                 ...
  LOAD           0x0000000000002000 0x0000000000402000 0x0000000000402000
                 0x0000000000000012 0x0000000000000012  R      0x1000
                 ...
```

Seu conteúdo também pode ser visualizado com a opção `-x` do `readelf`:

```
:~$ readelf -x .rodata sections

Despejo máximo da secção ".rodata":
  0x00402000 45752073 6f752069 6d7574c3 a176656c Eu sou imut..vel
  0x00402010 210a                                !.
```

Ou com o `xxd`, utilizando as opções `-s` (_skip_) e `-l` (_length_):

```
:~$ xxd -s 0x2000 -l 32 sections
00002000: 4575 2073 6f75 2069 6d75 74c3 a176 656c  Eu sou imut..vel
00002010: 210a 0000 0000 0000 0000 0000 0000 0000  !...............
```

### Inspeção da seção .data

```asm
section .data
	contador dq 0		; preenche 8 bytes (qword) com 0x00 em 'contador'
```

Neste exemplo, o valor `0` será escrito na seção `.data`, no endereço indicado pelo rótulo `contador`, e ocupara 8 bytes (diretiva `dq`), resultando em 8 bytes no arquivo (e na memória) preenchidos com zeros.

No NASM, os dados podem ser definidos com tamanhos de:

- 1 byte: `db`, de _define bytes_
- 2 bytes: `dw`, de _define word_
- 4 bytes: `dd`, de _define double word_
- 8 bytes `dq`, de _define quad word_

Como visto na tabela de cabeçalhos do programa, a seção `.data` inicia no _offset_ `0x2014` do arquivo e tem permissões de leitura e escrita (`RW`):

```
  Tipo           Desvio             EndVirtl           EndFís
                 TamFich            TamMem              Bndrs  Alinh
                 ...
  LOAD           0x0000000000002014 0x0000000000403014 0x0000000000403014
                 0x0000000000000008 0x000000000000002c  RW     0x1000
                 ...
```

Seu conteúdo pode ser visualizado com a opção `-x` do `readelf`:

```
:~$ readelf -x .data sections

Despejo máximo da secção ".data":
  0x00403014 00000000 00000000                   ........
```

Ou com o `xxd`, sabendo seu tamanho:

```
:~$ xxd -s 0x2014 -l 8 sections
00002014: 0000 0000 0000 0000                      ........
```

Ainda no exempĺo, nós escrevemos uma rotina para incrementar o valor no endereço identificado pelo rótulo `contador`:

```asm
mov rax, [contador] ; copia o dado no endereço 'contador' para rax
inc rax             ; incrementa em 1 o valor em rax
mov [contador], rax ; copia o valor em rax para o endereço 'contador'
```

> Em Assembly, isso é o mais próximo que podemos chegar do conceito de _variáveis_, próprio de linguagens de alto nível.

O efeito dessa rotina só pode ser examinado com o programa em execução, por exemplo, com o depurador GDB (GNU Debugger). Mas, antes, o programa terá que ser montado com símbolos de depuração (opção `-g` do `nasm`):

```
:~$ nasm -g -f elf64 sections.asm
:~$ ld sections.o -o sections
```

Carregando o programa com o GDB:

```
:~$ gdb sections
Reading symbols from sections...
```

Listando o código-fonte (`list`) para descobrir as linhas antes e depois da alteração do dado em `contador`:

```
(gdb) list
1	; Arquivo    : sections.asm
2	; Descrição  : Demonstra a definição das seções .rodata, .data, .bss e .text
3	; Montagem   : nasm -f elf64 sections.asm
4	; Link-edição: ld sections.o -o sections
5	
6	section .rodata
7		msg db "Eu sou imutável!", 0x0a
8		len equ $ - msg
9	
10	section .data
(gdb)
11		contador dq 0		; preenche 8 bytes (qword) com 0x00 em 'contador'
12	
13	section .bss
14		buffer resb 32		; reserva 32 bytes no endereço 'buffer'
15	
16	section .text
17	global _start			; ponto de entrada do programa
18	
19	_start:
20		mov rax, 1		; syscall: write
(gdb)
21		mov rdi, 1		; fd 1 = stdout
22		mov rsi, msg		; endereço da mensagem
23		mov rdx, len		; tamanho da mensagem
24		syscall
25	
26		mov rax, [contador]	; copia o dado no endereço 'contador' para rax
27		inc rax			; incrementa em 1 o valor em rax
28		mov [contador], rax	; copia o valor em rax para o endereço 'contador'
29	
30		mov rax, 60		; syscall: exit
(gdb)
31		xor rdi, rdi		; estado de término = 0
32		syscall
```

Definindo os pontos de parada (`break`) nas linhas 26 e 30:

```
(gdb) break 26
Breakpoint 1 at 0x40101b: file sections.asm, line 26.
(gdb) break 29
Breakpoint 2 at 0x40102e: file sections.asm, line 30.
```

Executando o programa (`run`):

```
(gdb) run
Starting program: /home/blau/git/pbn/curso/exemplos/03/sections
Eu sou imutável!

Breakpoint 1, _start () at sections.asm:26
26		mov rax, [contador]	; copia o dado no endereço 'contador' para rax
```

Imprimindo o valor no endereço `contador` (8 bytes equivale ao tipo `long`):

```
(gdb) print (long *)contador
$1 = (long *) 0x0
```

Continuando a execução (`continue`):

```
(gdb) continue
Continuing.

Breakpoint 2, _start () at sections.asm:30
30		mov rax, 60		; syscall: exit
```

Verificando a alteração do valor no endereço `contador`:

```
(gdb) print (long *)contador
$2 = (long *) 0x1
```

### Inspecionando a seção .bss

```asm
section .bss
	buffer resb 32		; reserva 32 bytes no endereço 'buffer
```

Neste trecho, nós utilizamos a diretiva `resb` para _reservar_ 32 bytes no endereço representado pelo rótulo `buffer`. Mas, como vimos, a seção `.bss` é referenciada nas tabelas de seções e do programa, mas não ocupa espaço no arquivo. Portanto, não é possível visualizar seu conteúdo se o programa não estiver sendo executado ou com um depurador. Por isso, nós examinaremos os dados na seção com o GDB:

```
:~$ gdb sections
Reading symbols from sections...
(gdb)
```

Desta vez, vamos utilizar o ponto de entrada do programa como ponto de parada e vamos executá-lo:

```
(gdb) break _start
Breakpoint 1 at 0x401000: file sections.asm, line 20.
(gdb) run
Starting program: /home/blau/git/pbn/curso/exemplos/03/sections

Breakpoint 1, _start () at sections.asm:20
20		mov rax, 1		; syscall: write
```

Neste ponto, todas as seções de dados estão carregadas e nós podemos examiná-las (comando `x`):

```
(gdb) x /1s &msg
0x402000 <msg>:	"Eu sou imutável!\n"
(gdb) x /1gx &contador
0x403014 <contador>:	0x0000000000000000
(gdb) x /32bx &buffer
0x40301c <buffer>:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0x403024:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0x40302c:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0x403034:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

Como podemos ver, a diretiva `resb` preencheu com zeros os 32 bytes reservados na seção `.bss`.

**Notas sobre o comando `x`, do GDB:**

- Examina os dados em um endereço de memória.
- O endereço pode ser passado na forma `&RÓTULO` (endereço do rótulo).
- A formatação dos dados é feita com `/<QTD><TIPO><BASE>`.
- O `TIPO` pode ser bytes (`b`), 2 bytes (`h`, de _half-word_), 4 bytes (`w`, _word_), 8 bytes (`g`, de _giant word_), string (`s`) ou caracteres (`c`).
- A `BASE` `x` refere-se à exibição de números em hexadecimal.

## Exercícios propostos

1. Crie um exemplo em C onde seja possível observar as seções `.text`, `.rodata`, `.data` e `.bss`.
2. Com o arquivo objeto do exemplo `sections.asm`, tente interpretar as informações presentes no seu cabeçalho ELF (a descrição dos campos pode ser encontrada na página "ELF" da wiki OS Dev, nas referências).
3. Em seguida, interprete o cabeçalho ELF do arquivo executável e anote as diferenças, se houver.

### Desafio

Sabendo que o NASM pode montar binários puros, tente criar um programa que resulte em um arquivo executável 64 bits utilizando apenas as especificações ELF para definir os bytes de seu conteúdo, que deverá ser funcional sem a edição de ligações. O programa não precisa fazer nada além de terminar com estado de saída 42.

Para montar e dar permissões de execução:

```
:~$ nasm -f bin -o elf_exit42 elf_exit42.asm
:~$ chmod +x elf_exit42
```

Para testá-lo:

```
$ ./elf_exit42
$ echo $?
42
```

## Referências

- `man 5 elf`
- [OS Dev: ELF](https://wiki.osdev.org/ELF)
- [https://refspecs.linuxfoundation.org/elf/elf.pdf](https://refspecs.linuxfoundation.org/elf/elf.pdf)
- [Anotações do curso de introdução ao GDB](https://bolha.dev/blau_araujo/gdb-pratico)