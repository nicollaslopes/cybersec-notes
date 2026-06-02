# Vetor de ambiente (envp)

## Objetivos

- Entender o papel do vetor de ambiente (`envp`).
- Acessar as definições de variáveis exportadas para o processo.
- Exemplificar o uso do conteúdo de `envp` em programas escritos em baixo nível.

## Revisão: o quadro inicial da pilha do processo

Como vimos no último tópico, a pilha do processo é inicializada com diversos conjuntos de dados relativos ao ambiente de execução do programa e outras informações auxiliares para uso da _runtime_ da linguagem C. Também vimos que o conteúdo inicial da pilha pode ser representado desta forma:

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
## Localizando o vetor de ambiente

O endereço inicial do vetor de ambiente (`envp`) é carregado na pilha 8 bytes depois do endereço do separador nulo do vetor de argumentos. Sendo assim, nós podemos determinar sua posição com:

```
rsp + (8 * argc) + 16
```

Para confirmar, vamos utilizar novamente o programa `exit42.asm` e o GDB:

Arquivo: `exit42.asm`

```asm
; Retorna 42 como estado de término

section .text
    global _start

_start:
    mov     rax, 60        ; syscall: exit
    mov     rdi, 42        ; código de saída
    syscall
```

Montagem, início do GDB e execução do programa:

```
:~$ nasm -g -f elf64 exit42.asm
:~$ ld -o exit42 exit42.o
:~$ gdb ./exit42
Reading symbols from ./exit42...
(gdb) run
Starting program: /home/blau/git/pbn/curso/exemplos/07/exit42

Breakpoint 1, _start () at exit42.asm:7
7	    mov     rax, 60        ; syscall: exit
```

O separador nulo, entre `argv` e `envp` deve estar em `rsp+16`, e nós podemos verificar se é verdade com:

```
(gdb) x /1gx $rsp+16
0x7fffffffdfe0:	0x0000000000000000
```

Como o programa foi executado sem argumentos, `argc` terá valor `1`. Portanto, o endereço de `envp[0]` estará 8 bytes depois em `rsp+24`:

```
(gdb) x /1gx $rsp+24
0x7fffffffdfe8:	0x00007fffffffe318
```

Nós poderíamos exibir a string neste endereço copiando e colando a saída do comando anterior, mas também é possível exibi-la com a sintaxe:

```
x /1s *((char **)($rsp + 24))
```

Onde:

- `x`: comando _execute_.
- `/1s`: exibir conteúdo formatado como uma string (cadeia de caracteres terminada com `0x00`).
- `*(...ENDEREÇO...)`: exibir os dados em `ENDEREÇO`.
- `(char **)ENDEREÇO`: definição do tipo do dado em `ENDEREÇO` (ponteiro de ponteiro para caracteres).
- `($rsp + 24)`: o dado que está 24 bytes após o endereço em `rsp`.

Executando o comando:

```
(gdb) x /1s *((char **)($rsp + 24))
0x7fffffffe318:	"SHELL=/bin/bash"
```

E aqui nós temos a string da definição da primeira variável exportada no processo do nosso programa – uma string no formato `NOME=VALOR`. Infelizmente, diferente do vetor de argumentos, o vetor de ambiente não tem uma informação de contagem associada (como `argc`), então não temos como saber onde as variáveis terminam, a não ser percorrendo todos os endereços em `envp` até encontrarmos outro separador nulo.

Na prática, porém, isso não é um problema, pois as exportação de variáveis para processos tem usos muito específicos e são mais utilizadas por programas escritos em linguagens que, de algum modo, abstraem a busca pelos nomes das variáveis e a extração de seus respectivos valores. Este é o caso, por exemplo, da função `getenv`, da `glibc`, declarada em `stdlib.h` como:

```c
char *getenv(const char *name);
```

Recebendo o ponteiro para a string de um nome como argumento (`name`), a função retorna o endereço do byte da string onde o valor atribuído à variável exportada inicia:

```
     endereço de 'name'
     ↓
     NOME=VALOR
          ↑
          endereço retornado
```

## Extraindo valores de variáveis exportadas

O programa abaixo acessa as variáveis exportadas para o ambiente do processo (`envp`) e simula o funcionamento de `getenv`, imprimindo o valor de uma variável buscada, se ela existir. Com este exemplo, nossos objetivos são demonstrar o uso de argumentos de linha de comando e o acesso ao vetor de ambiente em um programa escrito sem qualquer abstração, totalmente em baixo nível.

> Os detalhes sobre as chamadas de sistema envolvidas e criação de sub-rotinas ficarão para os próximos tópicos.

Arquivo: `getenv.asm`

```asm
; ----------------------------------------------------------
; Montagem: nasm -f elf64 -o getenv.o getenv_v2.asm
; Ligação : ld -o getenv getenv.o
; Uso     : ./getenv NOME_VARIAVEL
; ----------------------------------------------------------
section .rodata
; ----------------------------------------------------------
        newline    db 10, 0                        ; caractere de nova linha
        msg_nf     db "Not found", 10, 0           ; mensagem de erro
        msg_usage  db "Usage: ./getenv VAR", 10, 0 ; mensagem de uso incorreto
; ----------------------------------------------------------
section .bss
; ----------------------------------------------------------
        argc   resq 1    ; quantidade de argumentos (argc)
        argv   resq 1    ; endereço do vetor de argumentos (argv)
        envp   resq 1    ; endereço do vetor de ambiente (envp)
; ----------------------------------------------------------
section .text
; ----------------------------------------------------------
global _start
_start:
; ----------------------------------------------------------
; Carregar e salvar argc, argv e envp
; ----------------------------------------------------------
        mov     rax, [rsp]              ; argc está no topo da pilha
        mov     [argc], rax             ; salva argc em argc

        lea     rax, [rsp + 8]          ; argv começa logo após argc
        mov     [argv], rax             ; salva ponteiro de argv

        mov     rbx, [argc]
        lea     rax, [rsp + rbx*8 + 16] ; envp vem após argc + argv[] + ponteiro nulo
        mov     [envp], rax             ; salva ponteiro de envp
; ----------------------------------------------------------
; Verificar se argc == 2
; ----------------------------------------------------------
        cmp     qword [argc], 2
        jl      show_usage              ; se argc < 2, mostrar uso correto
; ----------------------------------------------------------
; Obter endereço do nome da variável: argv[1]
; ----------------------------------------------------------
        mov     rax, [argv]
        mov     rsi, [rax + 8]          ; rsi = argv[1]
        ; ------------------------------------------
        ; Iterar sobre envp para buscar a variável
        ; ------------------------------------------
        mov     rcx, [envp]             ; rcx recebe endereço de envp[0]
.next_env:
        mov     rdx, [rcx]              ; rdx recebe endereço da string "VAR=valor"
        test    rdx, rdx                ; verifica se rdx = 0
        jz      not_found               ; se rdx = 0, termina com erro (não encontrada)
        ; ------------------------------------------
        ; Comparar caractere a caractere: se argv[1] é prefixo de rdx
        ; e é seguido por '=' → variável encontrada
        ; ------------------------------------------
        mov     r8, rsi                 ; r8 = nome buscado (argv[1])
        mov     r9, rdx                 ; r9 = string atual do envp
.cmp_loop:
        mov     al, [r8]                ; próximo char do nome buscado
        mov     bl, [r9]                ; próximo char da string do envp
        cmp     al, 0
        je      .check_equals           ; se al = 0, chegou ao fim do nome buscado: checar '='
        cmp     al, bl
        jne     .next                   ; se al != bl, não bateu, próxima variável
        inc     r8
        inc     r9
        jmp     .cmp_loop
        ; ------------------------------------------
        ; Comparar se o caractere no endereço é '='
        ; ------------------------------------------
.check_equals:
        cmp     byte [r9], '='          ; verifica se o caractere em envp é '='
        jne     .next                   ; não era uma '=': próximo elemento em envp
        inc     r9                      ; avança para o início do valor
        mov     rsi, r9                 ; rsi = endereço do primeiro byte do valor
        mov     rdi, 1                  ; stdout
        call    print_string            ; imprime valor
        mov     rsi, newline            ; carrega endereço de '\n' em rsi
        call    print_string            ; imprime \n
        mov     rdi, 0                  ; estado de término 0 (sucesso)
        call    exit_program            ; termina o programa
.next:
        add     rcx, 8                  ; avança para próximo emdereço em envp
        jmp     .next_env
; ----------------------------------------------------------
; Caso variável não seja encontrada
; ----------------------------------------------------------
not_found:
        lea     rsi, [rel msg_nf]
        mov     rdi, 2                  ; stderr
        call    print_string
        mov     rdi, 1                  ; código de erro
        call    exit_program
; ----------------------------------------------------------
; Caso uso incorreto (menos de 2 argumentos)
; ----------------------------------------------------------
show_usage:
        lea     rsi, [rel msg_usage]    ; endereço da mensagem de uso
        mov     rdi, 2                  ; fd 2 = stderr
        call    print_string            ; imprime a mensagem
        mov     rdi, 1                  ; estado de término 1 (erro)
        call    exit_program            ; termina o programa
; ----------------------------------------------------------
; SUB-ROTINAS
; ----------------------------------------------------------
print_string:
; ----------------------------------------------------------
; Imprime string terminada em '\0' no descritor indicado
; Entrada:
;   rdi = file descriptor (1 = stdout, 2 = stderr)
;   rsi = ponteiro para string terminada em '\0'
; ----------------------------------------------------------
        push    rdi                     ; salva rdi (fd)
        mov     rax, rsi
        xor     rcx, rcx                ; contador = 0
.count:
        cmp     byte [rax + rcx], 0     ; fim da string?
        je      .write
        inc     rcx
        jmp     .count
.write:
        mov     rdx, rcx                ; número de bytes a escrever
        pop     rax                     ; restaura fd original
        mov     rdi, rax
        mov     rax, 1                  ; syscall: write
        syscall
        ret
; ----------------------------------------------------------
exit_program:
; ----------------------------------------------------------
; Encerra o programa com código em rdi
; Entrada:
;   rdi = código de saída (0 = sucesso, 1 = erro)
; ----------------------------------------------------------
        mov     rax, 60                 ; syscall: exit
        syscall
```

Montando e executando, nós teremos algo como:

```
:~$ nasm -f elf64 getenv.asm
:~$ ld -o getenv getenv.o
:~$ ./getenv USER
blau
:~$ ./getenv
Usage: ./getenv VAR
:~$ ./getenv TESTE
Not found
:~$ TESTE=123 ./getenv TESTE
123
```

## Implementação com funções da linguagem C

As funções da `glibc` abstraem quase todo o trabalho que tivemos com o Assembly, como podemos ver no exemplo a seguir.

Arquivo: `cgetenv.asm`

```asm
; ----------------------------------------------------------
; Montagem: nasm -f elf64 cgetenv.asm
; Ligação : gcc -no-pie -o cgetenv cgetenv.o
; Uso     : ./cgetenv PATH
; ----------------------------------------------------------
; Funções importadas da glibc...
; ----------------------------------------------------------
extern  getenv
extern  puts
extern  fprintf
extern  exit
extern  stderr
; ----------------------------------------------------------
section .rodata
; ----------------------------------------------------------
        msg_nf     db "Not found", 10, 0
        msg_usage  db "Usage: ./getenv VAR", 10, 0
; ----------------------------------------------------------
section .bss
; ----------------------------------------------------------
        argc   resq 1             ; recebe quantidade de argumentos
        argv   resq 1             ; recebe endereço de argv[0]
        envp   resq 1             ; recebe endereço de envp[0]
; ----------------------------------------------------------
section .text
; ----------------------------------------------------------
global main
main:
; ----------------------------------------------------------
; Iniciar quadro de pilha para main...
; ----------------------------------------------------------
        push rbp
        mov rbp, rsp
; ----------------------------------------------------------
; Salvar dados de argumentos e ambiente...
; ----------------------------------------------------------
        mov     [argc], rdi       ; salvar valor de argc
        mov     [argv], rsi       ; salvar endereço de argv[0]
        mov     [envp], rdx       ; salvar endereço de envp[0]
; ----------------------------------------------------------
; Verificar se argc < 2
; ----------------------------------------------------------
        cmp     rdi, 2
        jl      show_usage
; ----------------------------------------------------------
; getenv(argv[1]);
; ----------------------------------------------------------
        mov     rax, [argv]       ; endereço de argv[0]
        mov     rdi, [rax + 8]    ; argumento 1: endereço de argv[1]
        call    getenv            ; chamar getenv
        test    rax, rax          ; testar retorno (0 = não encontrada)
        jz      not_found
; ----------------------------------------------------------
; Imprimir valor encontrado: puts(char *valor)
; ----------------------------------------------------------
        mov     rdi, rax          ; argumento 1: endereço do valor em rax
        call    puts              ; chamar puts
; ----------------------------------------------------------
; Término com sucesso: exit(int status);
; ----------------------------------------------------------
        mov     edi, 0            ; argumento 1: (int)0 - sucesso
        call    exit              ; chamar exit
; ----------------------------------------------------------
; Se o nome da variável não for encontrado...
; ----------------------------------------------------------
not_found:
; ----------------------------------------------------------
; Mensagem de erro: fprintf(stderr, "Not found\n");
; ----------------------------------------------------------
        mov     rdi, [stderr]     ; argumento 1: stream de saída
        lea     rsi, [msg_nf]     ; argumento 2: string de formato
        call    fprintf           ; chamar fprintf
        jmp exit_error            ; termina com erro
; ----------------------------------------------------------
; Quantidade de argumentos incorrerta...
; ----------------------------------------------------------
show_usage:
; ----------------------------------------------------------
; Mensagem de erro: fprintf(stderr, "Usage: ./getenv VAR\n");
; ----------------------------------------------------------
        mov     rdi, [stderr]     ; argumento 1: stream de saída
        lea     rsi, [msg_usage]  ; argumento 2: string de formato
        call    fprintf           ; chamar fprintf
; ----------------------------------------------------------
; Término com erro
; ----------------------------------------------------------
exit_error:
        mov     edi, 1            ; argumento 1: (int)1 - erro
        call    exit              ; chamar exit
```

Montagem, compilação e testes:

```
:~$ nasm -f elf64 cgetenv.asm
:~$ gcc -no-pie -o cgetenv cgetenv.o
:~$ ./cgetenv USER
blau
:~$ ./cgetenv
Usage: ./getenv VAR
:~$ ./cgetenv TESTE
Not found
:~$ TESTE=123 ./cgetenv TESTE
123
```

## Exercícios propostos

1. Utilizando o GDB, execute os programas de exemplo buscando pela variável exportada `LANG` e examine as strings obtidas até que a ela seja encontrada.
2. No exemplo `cgetenv.asm`, comente a inicialização da pilha, observe o resultado e explique por que ele acontece.
3. Crie o programa `salve.asm` para imprimir `Salve, <nome>!`, onde `nome` pode ser recebido tanto como um argumento quanto pela exportação da variável `NOME`.

## Referências

- `man 7 environ`
- `man 3 getenv`
- [GNU libc: Environment Variables](https://www.gnu.org/software/libc/manual/html_node/Environment-Variables.html)
- [OSDev: Calling Conventions](https://wiki.osdev.org/Calling_Conventions)