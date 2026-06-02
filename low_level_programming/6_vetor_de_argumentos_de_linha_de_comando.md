# Vetor de argumentos de linha de comando

## Objetivos

- Compreender a passagem de argumentos de linha de comando para processos.
- Descobrir como o kernel Linux estrutura o vetor de argumentos.
- Acessar a quantidade e os elementos do vetor de argumentos.
- Criar programas em Assembly que trabalhem com argumentos de linha de comando.
- Implementar parsing manual de argumentos no Assembly.

## O quadro inicial da pilha do processo

Ao iniciar a pilha de um processo, o kernel inclui diversos conjuntos de dados relativos ao ambiente de execução do programa e outras informações auxiliares para uso do _runtime_ da linguagem C (`glibc`) e do carregados dinâmico do sistema (`ld-linux`). Na arquitetura Linux x86_64, dos endereços mais altos para os mais baixos, nós temos:

- Strings dos argumentos de linha de comando;
- Strings das definições de variáveis exportadas para o ambiente do processo;
- Semente aleatória (`AT_RANDOM`);
- Nome do executável (`AT_EXECFN`);
- Vetor auxiliar (`auxv`);
- Vetor dos endereços dos dados de ambiente (`envp`);
- Vetor dos endereços dos argumentos (`argv`);
- Contador de argumentos (`argc`).

O diagrama abaixo é uma representação simplificada do estado inicial da pilha, segundo especificado pela _System V Application Binary Interface_ (ABI) para a arquitetura AMD64:

```
                     ┌────────────────────────────────┐
                     │        NÃO ESPECIFICADO        │  ENDEREÇOS MAIS ALTOS
                     ├────────────────────────────────┤  ─┐
                     │     STRINGS DE ARGUMENTOS      │   │
                     ├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┤   │
                     │      STRINGS DE AMBIENTE       │   ├─ BLOCO DE INFORMAÇÕES
                     ├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┤   │(SEM ORDEM ESPECÍFICA)
                     │   DADOS BINÁRIOS AUXILIARES    │   │
                     ├────────────────────────────────┤  ─┘
                     │        NÃO ESPECIFICADO        │
                     ├────────────────────────────────┤
                     │ ENTRADA NULA DO VETOR AUXILIAR │  QWORD (0x00)
                     ├────────────────────────────────┤
                     │   ENTRADAS DO VETOR AUXILIAR   │  2 QWORDS POR ENTRADA
                     ├────────────────────────────────┤
                     │        SEPARADOR NULO          │  QWORD (0X00)
                     ├────────────────────────────────┤
                     │     VETOR AMBIENTE (ENVP)      │  1 QWORD POR ENDEREÇO
                     ├────────────────────────────────┤
RSP + (8 * ARGC) + 8 │        SEPARADOR NULO          │  QWORD (0X00)
                     ├────────────────────────────────┤
   RSP + 8 (ARGV[0]) │   VETOR DE ARGUMENTOS (ARGV)   │  1 QWORD POR ENDEREÇO
                     ├────────────────────────────────┤
                 RSP │   ENDEREÇO DO VALOR DE ARGC    │  1 QWORD
                     ├────────────────────────────────┤
                     │         NÃO DEFINIDO           │  ENDEREÇO MAIS BAIXO
                     └────────────────────────────────┘

QWORD = 8 BYTES (64 BITS)
```

QWORD = 8 BYTES (64 BITS)

Embora a ABI diga que o conteúdo do bloco de informações não precisa seguir uma ordem específica, o Linux padronizou uma ordem em sua implementação da carga de programas (em [kernel.org](https://github.com/torvalds/linux/blob/94305e83eccb3120c921cd3a015cd74731140bac/fs/binfmt_elf.c#L824)), dos endereços mais altos para os mais baixos:

- Outros eventuais dados auxiliares
- Strings das definições das variáveis exportadas
- Strings de argumentos
- Semente aleatória (`AT_RANDOM`, ponteiro para 16 bytes aleatórios)
- Endereço da string da arquitetura (`AT_PLATAFORM`)
- Endereço da string do caminho do executável (`AT_EXECFN`)

No contexto deste e do próximo tópico dos nossos estudos, interessam apenas os dados do quadro inicial da pilha que, de algum modo, propiciam alguma forma de interação do usuário com os nossos programas: os argumentos passados pela linha de comandos e as variáveis exportadas para o processo do programa.

## Inspecionando argumentos e variáveis exportadas

Para investigar o estado do quadro inicial da pilha, nós vamos utilizar o programa `exit42.asm`.

```asm
; Retorna 42 como estado de término

section .text
    global _start

_start:
    mov     rax, 60        ; syscall: exit
    mov     rdi, 42        ; código de saída
    syscall
```

**Montagem e link-edição:**

```
:~$ nasm -f elf64 -g exit42.asm
:~$ ld -o exit42 exit42.o
```

Agora, vamos abrir o executável no GDB, definir `_start` como ponto de parada e iniciar sua execução:

```
:~$ gdb exit42
Reading symbols from exit42...
(gdb) break _start
Breakpoint 1 at 0x401000: file exit42.asm, line 7.
(gdb) run
Starting program: /home/blau/git/pbn/curso/exemplos/06/exit42

Breakpoint 1, _start () at exit42.asm:7
7	    mov     rax, 60        ; syscall: exit
(gdb)
```

### Contador de argumentos

Primeiro, vamos examinar o conteúdo do registrador `rsp` (_stack pointer_):

```
(gdb) info registers rsp
rsp            0x7fffffffdfd0      0x7fffffffdfd0
```

Como resultado, nós temos um endereço que, segundo o diagrama anterior, deve nos levar à quantidade de argumentos de linha de comando escritos para invocar o programa (`argc`). Como o programa não foi iniciado com argumentos, o valor de `argc` deve ser `1` (escrito em 8 bytes), correspondendo ao caminho do nosso executável. Sendo assim, vamos utilizar o comando `x` para exibir uma _giant-word_ em hexa (`/1gx`) a partir do endereço em `rsp` (`$rsp`):

```
(gdb) x /1gx $rsp
0x7fffffffdfd0:	0x0000000000000001
```

E lá está o valor `1` escrito em 8 bytes.

> **Nota:** É por conta dessa ambiguidade do termo _argumentos_, utilizado tanto no sentido de _"argumentos passados para um programa"_ quanto de _"todas as palavras utilizadas para invocar o programa"_, que o vetor de argumentos também é chamado de _vetor de parâmetros_, bem mais adequado para expressar o que nós encontramos no quadro inicial da pilha (todas as palavras utilizadas para invocar o programa). Mas, feita a observação, vamos nos manter na terminologia da documentação da ABI e do kernel.

### Primeiro elemento do vetor de argumentos

Lembrando que o registrador `rsp` sempre contém o dado que está no topo da pilha (ou seja, em seu endereço mais baixo) e sabendo que o dado anterior está 8 bytes acima, nós podemos obter o endereço do primeiro elemento do vetor de argumentos (`argv`) desta forma:

```
(gdb) x /1gx $rsp+8
0x7fffffffdfd8:	0x00007fffffffe2ec
```

Se o endereço estiver correto, ele nos levará à string do caminho completo do nosso executável. Para confirmar, vamos examinar com o formato `/1s`, de _"uma string"_:

```
(gdb) x /1s 0x00007fffffffe2ec
0x7fffffffe2ec:	"/home/blau/git/pbn/curso/exemplos/06/exit42"
```

Confirmado!

### Separador nulo do vetor de argumentos

Ainda segundo o diagrama do quadro inicial da pilha, tendo apenas um argumento, a posição seguinte na pilha (`rsp+16`) deve conter um endereço que nos leva a uma cadeia de 8 bytes zerados (_separador nulo_). Então, vejamos o que mostra o GDB:

```
(gdb) x /1gx $rsp+16
0x7fffffffdfe0:	0x0000000000000000
```

Não é o que estamos vendo… Na verdade, o que esperávamos resulta de uma interpretação equivocada bastante comum das especificações da ABI. Os dados no bloco de informações são compactados, ou seja, escritos sem separação de forma contígua. Sendo assim, a delimitação dos vetores (os _separadores nulos_) é feita pelos valores dos elementos mais acima na pilha (nos endereços mais baixos), como vimos no GDB. A posição do separador nulo de `argv` pode ser calculado com:

```
rsp + (argc * 8) + 8
```

Como o valor em `argc` é `1`:

```
rsp + (1 * 8) + 8 = rsp + 8 + 8 = rsp + 16
```

Ou seja, nós teríamos que encontrar o valor `0` escrito com 8 bytes na posição da pilha resultante de `rsp + 16` – e encontramos.

### Primeiro elemento do vetor ambiente

A posição imediatamente abaixo da pilha (`rsp + 24`) deve conter o endereço da primeira string com a definição de uma variável exportada para o processo do programa – ou seja, a string em `envp[0]`. Então, vamos examinar novamente:

```
(gdb) x /1gx $rsp+24
0x7fffffffdfe8:	0x00007fffffffe318
(gdb) x /1s 0x00007fffffffe318
0x7fffffffe318:	"SHELL=/bin/bash"
```

O quadro inicial da pilha não tem uma informação sobre a quantidade de variáveis exportadas para o ambiente do processo, então é difícil saber onde ela termina sem continuar a percorrer a pilha até encontrar o separador nulo de `envp`. No meu sistema, eu testei manualmente e descobri que o endereço do último elemento de `envp` está em `rsp + 368`:

```
(gdb) x /1gx $rsp+368
0x7fffffffe140:	0x00007fffffffefa3
(gdb) x /1s 0x00007fffffffefa3
0x7fffffffefa3:	"OLDPWD=/home/blau/git/pbn/curso/exemplos"
```

Portanto, em `rsp + 376` eu devo encontrar o separador nulo:

```
(gdb) x /1gx $rsp+376
0x7fffffffe148:	0x0000000000000000
```

E lá está ele!

### Resultados com mais de um argumento

Também é interessante investigar o que acontece com o quadro inicial da pilha quando nós executamos o programa com outros argumentos de linha de comando além, é claro, de seu próprio caminho. Para isso, vamos iniciar novamente o GDB, mas vamos rodar o programa com argumentos:

```
$ gdb exit42
Reading symbols from exit42...
(gdb) b _start
Breakpoint 1 at 0x401000: file exit42.asm, line 7.
(gdb) run banana laranja
Starting program: /home/blau/git/pbn/curso/exemplos/06/exit42 banana laranja

Breakpoint 1, _start () at exit42.asm:7
7	    mov     rax, 60        ; syscall: exit
```

**Conferindo `argc`:**

```
(gdb) info registers rsp
rsp            0x7fffffffdfb0      0x7fffffffdfb0
(gdb) x /1gx 0x7fffffffdfb0
0x7fffffffdfb0:	0x0000000000000003
```

**Valor em `argv[0]`:**

```
(gdb) x /1gx $rsp + 8
0x7fffffffdfb8:	0x00007fffffffe2dd
(gdb) x /1s 0x00007fffffffe2dd
0x7fffffffe2dd:	"/home/blau/git/pbn/curso/exemplos/06/exit42"
```

**Valor em `argv[1]`:**

```
(gdb) x /1gx $rsp + 16
0x7fffffffdfc0:	0x00007fffffffe309
(gdb) x /1s 0x00007fffffffe309
0x7fffffffe309:	"banana"
```

**Valor em `argv[2]`:**

```
(gdb) x /1gx $rsp + 24
0x7fffffffdfc8:	0x00007fffffffe310
(gdb) x /1s 0x00007fffffffe310
0x7fffffffe310:	"laranja"
```

**Separador nulo de `argv`:**

```
(gdb) x /1gx $rsp + 32
0x7fffffffdfd0:	0x0000000000000000
```

## Acesso aos argumentos em baixo nível

Em Assembly, as duas técnicas mais comuns para acessar os argumentos de linha de comando são a leitura da pilha após o ponto de entrada (`_start`) e, no caso de programas híbridos que seguem as convenções da linguagem C, exportando uma função `main`, deixando que a `glibc` cuide da pilha e das convenções de chamada.

### Acesso direto à pilha

Basta salvar o endereço em `rsp` e utilizá-lo para obter `argc` e os demais elementos em `argv`:

```asm
section .bss
    argc: resq 1    ; reservar 8 bytes para receber o valor de argc.
    argv: resq 1    ; reservar 8 bytes para receber o endereço de argv[0].

section .text
global _start

_start:
    ; ----------------------------------------------------
    ; Inicialização de argc e argv...
    ; ----------------------------------------------------
    mov rbx, rsp        ; endereço de argc
    mov rax, [rbx]      ; valor de argc
    lea rcx, [rbx + 8]  ; endereço de argv[0]
    mov [argc], rax
    mov [argv], rcx

    ; Agora podemos usar [argc] e [argv] em qualquer parte do código.
    ; ...
```

> **Nota:** a instrução `lea` (_Load Effective Address_) calcula um endereço e carrega o resultado em um registrador de destino.

### Acesso em Assembly exportando main

Exportando a rotina principal do programa com o nome `main`, nós podemos gerar o executável com o `gcc` que, utilizando o objeto de inicialização da linguagem C (`crt0.o`), executará todas as tarefas de preparação para o acesso aos dados de argumentos e ambiente, o que inclui:

- Carregar `argc` em `rdi`;
- Carregar o endereço de `argv[0]` em `rsi`;
- Carregar o endereço de `envp[0]` em `rdx`;
- Configurar variáveis globais para a `glibc` (se usada);
- Chamar `main`, esperando que retorne um `int` em `eax`;
- Terminar o programa chamando a função `exit` (`glibc`).

O exemplo abaixo imprime o nome do programa (string `argv[0]`) utilizando a função `printf`, da glibc:

```asm
section .data
    fmt db "Programa: %s", 10, 0

global main        ; símbolo chamado por _start de crt0.o
extern printf      ; (opcional) usar funções da libc

section .text
main:
    ; argc → rdi
    ; argv → rsi
    ; envp → rdx (não obrigatório)

    ; Exemplo: imprimir argv[0]
    mov rdi, fmt
    mov rsi, [rsi]         ; argv[0]
    xor rax, rax           ; terminação de argumentos (printf é variádica)
    call printf

    mov eax, 0             ; retorno de 'main' = (int)0
    ret
```

Para montar e compilar:

```
:~$ nasm -g -f elf64 prog.asm
:~$ gcc -no-pie -o prog prog.o
```

Atenção para a opção `-no-pie` no `gcc`: Isso é necessário porque o exemplo não foi escrito nem montado prevendo suporte a endereçamento relativo (PIE).

Executando:

```
:~$ ./prog
Programa: ./prog
```

Mas note que, ao preparar a chamada de `printf`, nós tivemos que seguir as convenções da ABI e passar o primeiro argumento em `rdi` e o segundo em `rsi`: que são justamente os registradores onde as informações de argumentos foram carregadas. Portanto, se quisermos reutilizar esses dados, nós devemos salvá-los em outro lugar, como fizemos antes:

```asm
section .bss
    argc: resq 1    ; reservar 8 bytes para receber o valor de argc.
    argv: resq 1    ; reservar 8 bytes para receber o endereço de argv[0].

section .data
    fmt db "Programa: %s", 10, 0

global main        ; símbolo chamado por _start de crt0.o
extern printf      ; (opcional) usar funções da libc

section .text
main:
    ; argc → rdi
    ; argv → rsi
    ; envp → rdx (não obrigatório)
    mov [argc], rdi        ; salvar argc
    mov [argv], rsi        ; salvar endereço de argv[0]

    ; Exemplo: imprimir argv[0]
    mov rdi, fmt
    mov rsi, [argv]        ; argv[0]
    xor rax, rax           ; terminação de argumentos (printf é variádica)
    call printf

    mov eax, 0             ; retorno de 'main' = (int)0
    ret
```

## Listando todos os dados de argumentos

Para encerrar este tópico, vamos criar dois programas que acessam dos dados de argumentos na pilha e lista seus valores no terminal: um utilizando o _runtime_ da linguagem C (`crt0.o`) e outro totalmente em Assembly, mas com um pequeno módulo de sub-rotinas para conversão e impressão que só será estudada nos próximos tópicos.

### Exemplo em Assembly com main e gcc

Arquivo: `cargs.asm`

```asm
section .bss
    argc: resq 1      ; 8 bytes para armazenar argc
    argv: resq 1      ; 8 bytes para armazenar ponteiro para argv[0]

section .data
    fmt_argc db "argc   : %ld", 10, 0
    fmt_argv db "argv[%d]: %s", 10, 0

global main
extern printf

section .text
main:
    ; rdi = argc
    ; rsi = argv
    mov [argc], rdi         ; salvar argc
    mov [argv], rsi         ; salvar argv[0]

    ; Imprimir com: printf("argc   : %ld", argc);
    mov rdi, fmt_argc       ; primeiro argumento: formato
    mov rsi, [argc]         ; segundo argumento: argc (qword = long)
    xor rax, rax            ; terminação de argumentos
    call printf

    ; preparar o laço: i = 0
    xor rcx, rcx            ; i = 0
    mov rbx, [argv]         ; rbx = argv base
    mov r8, [argc]          ; r8 = argc

.loop:
    cmp rcx, r8             ; compara i com argc
    jge .done               ; se i >= argc, fim

    ; Imprimir com: printf("argv[%d]: %s\n", i, argv[i]);
    mov rdi, fmt_argv       ; primeiro argumento: formato
    mov rsi, rcx            ; segundo argumento: i
    mov rdx, [rbx + rcx*8]  ; terceiro argumento: argv[i]
    xor rax, rax            ; terminação de argumentos
    push rcx                ; salvar rcx (caller saved) na pilha
    push r8                 ; salvar r8 (caller saved) na pilha
    call printf
    pop r8                  ; restaurar rcx
    pop rcx                 ; restaurar rcx

    inc rcx
    jmp .loop

.done:
    xor eax, eax            ; retorno de main: (int)0
    ret
```

Compilando e executando:

```
:~$ nasm -g -felf64 cargs.asm
:~$ gcc -no-pie -o cargs cargs.o
:~$ ./cargs a b c
argc   : 4
argv[0]: ./cargs
argv[1]: a
argv[2]: b
argv[3]: c
```

#### Nota sobre registradores _"caller saved"_

```asm
push rcx                ; salvar rcx (caller saved) na pilha
push r8                 ; salvar r8 (caller saved) na pilha
call printf
pop r8                  ; restaurar rcx
pop rcx                 ; restaurar rcx
```

Nesse trecho, nós salvamos na pilha os registradores `rcx` e `r8`, usados como referências do nosso _loop_, antes de chamar a função `printf`. Isso foi necessário porque, segundo a ABI, eles estão entre os registradores _"caller saved"_, ou seja: registradores que podem ser alterados pela chamada de uma função com a instrução `call` e, portanto, terão que ser salvos pela função chamadora, caso estejam em uso para outros propósitos.

São registradores _caller-saved_ segundo a ABI (System V AMD64):

- `rax`: usado para retorno de funções;
- `rcx`: usado como argumento 4 em chamadas de funções;
- `rdx`: usado como argumento 3 em chamadas de funções;
- `rsi`: usado como argumento 2 em chamadas de funções;
- `rdi`: usado como argumento 1 em chamadas de funções;
- `r8` e `r9`: usados como argumentos de 5 e 6 em chamadas de funções;
- `r10` e `r11`: uso interno em funções.

A volatilidade desses registradores não está relacionada com seu uso antes da chamada da função (`call`), mas com o fato da ABI não obrigar a função chamada a salvar e restaurar seus conteúdos.

### Exemplo com acesso direto à pilha

* Arquivo: `args.asm `
* Módulo: `print_utils.asm`

Neste exemplo, não há necessidade de salvar `argc` e `argv` na memória.

```asm
; ----------------------------------------------------------
; Montar módulo com   : nasm -f elf64 print_utils.asm
; Montar programa com : nasm -f elf64 args.asm
; Gerar executável com: ls -o args args.o print_utils.o
; ----------------------------------------------------------
section .rodata
    pref_argc db "argc   : ", 0  ; Prefixo da impressão de argc
    pref_argv db "argv[", 0   ; Prefixo da impressão de argv
    posf_argv db "]: ", 0     ; Sufixo da impressão de argv

; sub-rotinas do módulo print_utils.o
extern print_int
extern print_str
extern print_char

section .text
global _start

_start:
; ----------------------------------------------------------
; Impressão de argc
; ----------------------------------------------------------
    mov rdi, pref_argc    ; endereço de pref_argc
    call print_str        ; imprime pref_argc
    
    mov rax, [rsp]        ; rax = argc
    call print_int        ; imprime argc

    mov rdi, 10           ; rdi = \n
    call print_char       ; imprime \n
; ----------------------------------------------------------
; Impressão de argv
; ----------------------------------------------------------
    mov rcx, 0        ; índice i = 0
    lea rbx, [rsp + 8]    ; rbx = endereço de argv[0]

.print_argv_loop:
    cmp rcx, [rsp]        ; enquanto i < argc
    jge .done_argv

    mov rdi, pref_argv    ; imprime prefixo "argv["
    call print_str

    mov rax, rcx        ; rax = i
    call print_int        ; imprime índice

    mov rdi, posf_argv    ; imprime sufixo "]: "
    call print_str

    mov rdi, [rbx + rcx*8]    ; rdi = argv[i]
    call print_str

    mov rdi, 10        ; imprime \n
    call print_char

    inc rcx
    jmp .print_argv_loop

.done_argv:
; ----------------------------------------------------------
; Termina o programa com exit(0)
; ----------------------------------------------------------
    mov rax, 60        ; syscall: exit
    xor rdi, rdi        ; status = 0
    syscall
```

Montagem, link-edição e execução:

```
:~$ nasm -f elf64 print_utils.asm
:~$ nasm -f elf64 args.asm
:~$ ld -o args args.o print_utils.o
:~$ ./args a b c d
argc   : 5
argv[0]: ./args
argv[1]: a
argv[2]: b
argv[3]: c
argv[4]: d
```

## Exercícios propostos

1. Crie um programa em Assembly que receba seu nome como argumento e imprima `Salve, NOME!`.
2. Crie uma versão desse mesmo programa em Assembly chamando funções da linguagem C.
3. Utilizando o GDB e o exemplo `cargs.asm`, analise o comportamento dos registradores voláteis nas chamadas de funções da `glibc`.
4. Com base nos exemplos do tópico, crie um programa que liste as variáveis no vetor de ambiente.

## Referências

- [System V Application Binary Interface – AMD64 Architecture Processor Supplement](https://refspecs.linuxfoundation.org/)
- [Linux kernel: fs/binfmt_elf.c](https://github.com/torvalds/linux/blob/master/fs/binfmt_elf.c)
- `man 2 execve`
- `man 3 environ`
- `man 7 exec`