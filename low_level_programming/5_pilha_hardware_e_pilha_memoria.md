# Pilha de hardware e pilha de memória

## Objetivos

- Entender o que é uma pilha.
- Diferenciar pilha de hardware da pilha de memória.
- Explorar a movimentação do ponteiro da pilha.
- Observar a dinâmica dos quadros de pilha.
- Observar como a pilha se altera com a execução de sub-rotinas.
- Utilizar a pilha para salvar e restaurar estados de registradores.
- Escrever funções simples que manipulam a pilha explicitamente.

## O conceito de pilha (stack)

Pilhas são estruturas lineares de dados que seguem a política LIFO (_Last In, First Out_), ou seja, o último elemento inserido é o primeiro a ser removido.

```
┌──────────────┐ TOPO DA PILHA
│  ELEMENTO N  │
├──────────────┤
│ ELEMENTO N-1 │
├──────────────┤
│     ····     │
├──────────────┤
│  ELEMENTO 3  │
├──────────────┤
│  ELEMENTO 2  │
├──────────────┤
│  ELEMENTO 1  │
└──────────────┘ BASE DA PILHA
```

Por analogia, podemos imaginar uma pilha de pratos: os pratos são colocados um por cima do outro e, quando precisamos de um prato no meio da pilha, todos que estiverem acima dele terão que ser retirados primeiro. Isso ilustra bem a principal característica da estrutura, que é o acesso imediato restrito ao elemento que estiver no topo da pilha.

### Implementação da pilha como estrutura de dados

Uma pilha pode ser implementada de duas formas principais: utilizando uma área contígua de memória (como um vetor) ou uma estrutura que cresce conforme a necessidade (como uma sequência de elementos ligados uns aos outros).

**Como um vetor**
	A pilha é representada por um arranjo fixo de elementos e um índice que identifica o elemento que estiver no topo. Essa abordagem é simples e eficiente, mas exige que o tamanho máximo da pilha seja definido previamente.

**Como uma lista encadeada**
	Os elementos da pilha são criados conforme a necessidade, ligando cada novo elemento ao que já estiver ocupando a posição do topo. Assim, a pilha pode crescer ou diminuir dinamicamente, utilizando apenas o espaço de memória realmente necessário.

### Aplicações típicas

As pilhas são amplamente utilizadas por serem simples e pela sua propriedade de oferecerem acesso aos dados na ordem inversa de sua inserção. Algumas aplicações comuns incluem:

- Avaliação de expressões em notação pós-fixada (notação polonesa reversa).
- Operações de desfazer e refazer em editores.
- Verificação do balanceamento de estruturas aninhadas (parêntesis, chaves e colchetes).
- Algoritmos de busca e retrocesso (_backtracking_), como para explorar labirintos ou solucionar problemas combinatórios por tentativa e erro.
- Controle de chamadas de funções durante a execução de um programa.
- Tratamento de interrupções de hardware e chamadas de funções e sub-rotinas.

Entre tantas outras. No entanto, as aplicações que nos interessam dizem respeito às duas últimas da lista, como veremos um pouco mais adiante.

### Operações associadas às pilhas

De modo geral, as operações associadas a uma pilha são:

- **Push:** para inserir um novo elemento no topo da pilha;
- **Pop:** para acessar o elemento do topo da pilha e removê-lo;
- **Peek:** para acessar o elemento no topo da pilha sem removê-lo.

**Operação PUSH:**

```
                          TOPO DA PILHA (novo)
                            ┌──────┐
TOPO DA PILHA               │  23  │
  ┌──────┐                  ├──────┤
  │  64  │                  │  64  │
  ├──────┤                  ├──────┤
  │  42  │     PUSH 23      │  42  │
  ├──────┤                  ├──────┤
  │  16  │                  │  16  │
  └──────┘                  └──────┘
BASE DA PILHA             BASE DA PILHA
```

**Operação POP:**

```
TOPO DA PILHA
  ┌──────┐
  │  23  │               TOPO DA PILHA (removido)
  ├──────┤                 ┌──────┐
  │  64  │     A = ???     │  64  │    A = 23
  ├──────┤                 ├──────┤
  │  42  │     POP A       │  42  │
  ├──────┤                 ├──────┤
  │  16  │                 │  16  │
  └──────┘                 └──────┘
BASE DA PILHA            BASE DA PILHA
```

**Operação PEEK:**

```
TOPO DA PILHA          TOPO DA PILHA (não alterado)
  ┌──────┐               ┌──────┐
  │  23  │               │  23  │
  ├──────┤               ├──────┤
  │  64  │     A = ???   │  64  │   A = 23
  ├──────┤               ├──────┤
  │  42  │     PEEK A    │  42  │
  ├──────┤               ├──────┤
  │  16  │               │  16  │
  └──────┘               └──────┘
BASE DA PILHA          BASE DA PILHA
```

## Pilha de memória

No contexto de sistemas GNU/Linux, a _pilha de memória_ é uma região especial da memória virtual associada a um processo que cresce dinamicamente no sentido dos endereços mais baixo. Essa estrutura é usada para armazenar dados temporários durante a execução de um programa, especialmente:

- Endereços de retorno de chamadas de função.
- Valores recebidos pelas funções como argumentos.
- Dados em variáveis locais de funções.
- Dados nos registradores da CPU (_registros_) que foram salvos durante chamadas e interrupções.

A pilha de memória é controlada pelo hardware e pelo sistema operacional, e seu comportamento segue a política LIFO, como em toda estrutura de pilha: o último valor inserido é o primeiro a ser removido. Além disso, em arquiteturas x86_64, a pilha é acessada principalmente pelos registradores `rsp` (_stack pointer_) e `rbp` (_base pointer_), e seu uso está profundamente ligado à organização de chamadas de função e à convenção de chamadas da ABI do sistema.

## Pilha de hardware

Além da pilha de memória, gerenciada pelo sistema e pelas convenções de chamadas de função, o processador possui um mecanismo chamado _pilha de hardware_. Trata-se de um recurso interno da CPU que suporta operações de empilhar (_push_) e desempilhar (_pop_) valores diretamente, facilitando o gerenciamento da pilha de memória no nível do hardware.

Em processadores Intel 64, as instruções `push` e `pop` operam sobre a pilha de memória associada ao registrador `rsp` (_stack pointer_), que contém o endereço do dado que se encontra no topo da pilha.

Quando a instrução `push` é executada, o processador decrementa o valor em `rsp` (o endereço do dado no topo da pilha) e, em seguida, armazena o novo dado no endereço resultante. Já a instrução `pop` realiza o oposto: ela lê o valor no endereço atualmente indicado em `rsp`, copia esse valor para um registrador ou para outro local na memória, e então incrementa o endereço em `rsp`.

Resumindo:

- **Instrução `push`:** _Decrementa_ do endereço em `rsp` e insere um novo valor no topo da pilha (a pilha de memória cresce no sentido dos endereços mais baixos).

Exemplo:

```asm
push rax    ; Copia o valor em rax para um novo
            ; elemento no topo da pilha.
```

- **Instrução `pop`:** Copia o dado no topo da pilha para um destino e incrementa o endereço em `rsp`, o que faz com que o dado no endereço anterior seja ignorado (e eventualmente sobrescrito) em operações subsequentes.

Exemplo:

```asm
pop rax    ; Copia para rax o valor que será
           ; removido do topo da pilha.
```

> A implementação em hardware torna operações de pilha muito eficientes, e é uma das razões pelas quais a pilha de memória é utilizada para controle de fluxo, armazenamento temporário de dados, e manipulação de chamadas de função. Mas, embora a CPU tenha suporte para operações com a pilha, sua manipulação depende também do sistema operacional, da ABI e do compilador, que definem regras para uso, alinhamento e preservação de seus dados.

## Registradores e a pilha de memória

Como vimos, registradores são pequenas unidades de armazenamento internas da CPU para guardar valores temporários. Eles são utilizados constantemente durante a execução de programas, pois permitem a manipulação de dados sem depender da memória principal (RAM), que é bem mais lenta.

Antes da execução de funções e sub-rotinas, é comum que o conteúdo de alguns desses registradores precise ser preservado para que, ao retomar o controle da sua execução, o programa encontre os mesmos valores que estavam presentes antes da chamada. Para isso, os valores dos registradores costumam ser salvos na pilha de memória e restaurados posteriormente.

> A especificação de quais registradores precisam ser salvos depende do contexto e, principalmente, das convenções envolvidas, como veremos a seguir.

## Convenções de chamadas de funções (System V AMD64 ABI)

Ao compilar programas em linguagens como C para sistemas Linux 64 bits, o código gerado segue uma convenção de chamadas definida pela ABI (_Application Binary Interface_) do sistema. Essa convenção especifica:

- Quais registradores são usados para passar argumentos.
- Qual registrador é usado para o valor de retorno.
- Quais registradores devem ser preservados entre chamadas de função.
- Como a pilha deve ser utilizada.

### Ordem dos argumentos

Os primeiros seis argumentos de uma função são passados em registradores, na seguinte ordem:

|Argumento|Registrador|
|---|---|
|`arg1`|`RDI`|
|`arg2`|`RSI`|
|`arg3`|`RDX`|
|`arg4`|`RCX`|
|`arg5`|`R8`|
|`arg6`|`R9`|

> Argumentos adicionais são passados na pilha.

### Valor de retorno

O valor de retorno de uma função é armazenado em:

- `RAX`: para valores inteiros ou ponteiros.
- `RAX + RDX`: para estruturas de até 128 bits.
- `XMM0` (e seguintes): para valores em ponto flutuante.

### Registradores preservados e não preservados

Durante uma chamada de função, há uma distinção entre os registradores que devem ser preservados e os que podem ser sobrescritos…

Preservados pela função chamada (_callee-saved_):

- `RBX`
- `RBP`
- `R12` a `R15`

> `RSP` também deve ser restaurado à posição original.

Não presenrvados (_caller-saved_):

- `RAX`
- `RCX`
- `RDX`
- `RSI`
- `RDI`
- `R8` a `R11`

Os registradores não preservados podem ser modificados pela função chamada. Assim, se uma função chamadora deseja manter o valor de algum deles após a chamada, ela será responsável por salvá-los antes.

### Exemplo em C

Arquivo: `soma.c`

```c
#include <stdio.h>

int soma(int a, int b) {
    return a + b;
}

int main() {
    int x = 10;
    int y = 20;
    int r = soma(x, y);
    printf("Soma: %d\n", r);
    return 0;
}
```

Durante a execução de `soma(x, y)`, os valores de `x` e `y` são passados via `RDI` e `RSI`, respectivamente. O resultado é retornado em `RAX`. Se a função `soma` quiser usar, por exemplo, `RBX`, ela deverá salvá-lo na pilha no início da função (com `push rbx`) e restaurá-lo (com `pop rbx`) antes de retornar.

### Quadro de pilha (_stack frame_)

Em geral, cada função chamada cria seu próprio _quadro de pilha_, que inclui:

- Valores salvos de registradores preservados.
- Variáveis locais.
- Espaço para armazenar argumentos extras ou retorno de funções chamadas.

A base do quadro costuma ser registrada em `RBP`, como podemos demonstrar compilando o programa do exemplo e analisando o assembly desmontado pelo GDB.

**Compilação:**

```
:~$ gcc -O0 -no-pie -g -o soma soma.c
```

Aqui, nós desabilitamos as otimizações (`-O0`) e a geração de um executável independente de posição (`-no-pie`) para facilitar a análise dos endereços absolutos das instruções.

**Início do GDB:**

```
:~$ gdb soma
Reading symbols from soma...
```

**Desmontagem da função `soma`:**

```
(gdb) disassemble soma
Dump of assembler code for function soma:
   0x0000000000401126 <+0>:     push   rbp
   0x0000000000401127 <+1>:     mov    rbp,rsp
   0x000000000040112a <+4>:     mov    DWORD PTR [rbp-0x4],edi
   0x000000000040112d <+7>:     mov    DWORD PTR [rbp-0x8],esi
   0x0000000000401130 <+10>:    mov    edx,DWORD PTR [rbp-0x4]
   0x0000000000401133 <+13>:    mov    eax,DWORD PTR [rbp-0x8]
   0x0000000000401136 <+16>:    add    eax,edx
   0x0000000000401138 <+18>:    pop    rbp
   0x0000000000401139 <+19>:    ret
End of assembler dump.
```

Logo nas duas primeiras instruções, nós podemos ver a inicialização do quadro de pilha:

```
   0x0000000000401126 <+0>:     push   rbp
   0x0000000000401127 <+1>:     mov    rbp,rsp
```

Onde o endereço da base da pilha (em `rbp`) é inserido como um novo elemento no topo da pilha. Em seguida, o novo endereço do topo da pilha (em `rsp`) é copiado (em `rbp`) como o endereço da base da pilha, utilizado nas instruções seguintes como referencial fixo para a inserção dos dados da função.

Antes do retorno (`ret`), o endereço do topo da pilha é decrementado e o resultado, salvo em `rbp`, será o endereço que ele armazenava originalmente – em outras palavras, o estado original de `rbp` é restaurado.

```
   0x0000000000401138 <+18>:    pop    rbp
   0x0000000000401139 <+19>:    ret
```

Repare também que, entre o início e a remoção do quadro de pilha, os argumentos passados na chamada da função (registrados em `edi` e `esi`) são copiados em endereços, respectivamente, 4 e 8 bytes mais baixos que o endereço em `rbp`:

```
   0x000000000040112a <+4>:     mov    DWORD PTR [rbp-0x4],edi
   0x000000000040112d <+7>:     mov    DWORD PTR [rbp-0x8],esi
```

Deste modo, os dados são inseridos na pilha sem que novos elementos sejam empilhados, ou seja, para todos os efeitos, o endereço do topo da pilha ainda será o registrado em `rbp`.

## Convenções de chamadas de sistema

Diferente das chamadas de função ,que utilizam a pilha de memória para armazenar argumentos e preservar o estado dos registradores, chamadas de sistema interagem diretamente com o kernel e seguem uma convenção distinta, mais enxuta e orientada ao uso de registradores.

Em sistemas Linux x86-64, a transição entre espaço de usuário e espaço do kernel é feita por meio da instrução `syscall`. Os argumentos são passados exclusivamente por registradores:

|Registrador|Conteúdo|
|---|---|
|`rax`|Número da chamada de sistema|
|`rdi`|1º argumento|
|`rsi`|2º argumento|
|`rdx`|3º argumento|
|`r10`|4º argumento|
|`r8`|5º argumento|
|`r9`|6º argumento|
|`rax`|Valor de retorno|

A pilha permanece intacta durante a chamada de sistema — ou seja, não há necessidade de alocar um novo quadro de pilha nem de salvar/restaurar registradores preservados, como ocorre nas convenções de chamadas entre funções. Em vez disso, o kernel assume que os registradores não preservados (`rax`, `rcx` e `r11`) podem ser sobrescritos e o processo que realiza a chamada é que deve se encarregar de salvar qualquer valor que deseje manter.

Essa característica destaca o papel dos registradores temporários: eles são utilizados livremente pela chamada de sistema, e qualquer valor importante deve ser salvo na pilha pelo chamador se necessário, o que geralmente é feito com `push <registrador>`, antes da chamada de sistema, e `pop <registrador>` após a chamada.

### Exemplo em Assembly

Arquivo: `salve.asm`

```asm
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

Com este exemplo, nós queremos demonstrar que, nas chamadas de sistema:

- Nenhum dado é empilhado.
- Nenhum registrador é preservado.
- O controle é transferido diretamente ao kernel por meio de `syscall`.
- Após o retorno do kernel, o processo continua sua execução ou é finalizado.

Para isso, vamos montá-lo e abrir o executável com o GDB:

```
:~$ nasm -f elf64 -g salve.asm
:~$ ld -o salve salve.o
:~$ gdb salve
Reading symbols from salve...
(gdb)
```

Com o comando `list`, nós descobrimos que a instrução que processará a chamada de sistema `write` (`sycall`) está na linha 13:

```
(gdb) list
1	section .rodata
2	    msg db "Salve, simpatia!", 10
3	    len equ $ - msg
4	
5	section .text
6	    global _start
7	
8	_start:
9	    mov rax, 1          ; syscall write
10	    mov rdi, 1          ; stdout
(gdb)
11	    mov rsi, msg
12	    mov rdx, len
13	    syscall
14	
15	    mov rax, 60         ; syscall exit
16	    mov rdi, 0          ; código de saída 0
17	    syscall
```

Sendo assim, ela será o nosso ponto de parada:

```
(gdb) break 13
Breakpoint 1 at 0x401019: file salve.asm, line 13.
```

Nosso objetivo é comparar o estado dos registradores antes e depois da execução do ponto de parada para determinar o que mudou. Então, vamos executar o programa e inspecionar o estado corrente dos registradores:

```
(gdb) run
Starting program: /home/blau/git/pbn/curso/exemplos/05/salve

Breakpoint 1, _start () at salve.asm:13
13	    syscall
(gdb) info registers
rax            0x1                 1
rbx            0x0                 0
rcx            0x0                 0
rdx            0x11                17
rsi            0x402000            4202496
rdi            0x1                 1
rbp            0x0                 0x0
rsp            0x7fffffffdfd0      0x7fffffffdfd0
r8             0x0                 0
r9             0x0                 0
r10            0x0                 0
r11            0x0                 0
r12            0x0                 0
r13            0x0                 0
r14            0x0                 0
r15            0x0                 0
rip            0x401019            0x401019 <_start+25>
eflags         0x202               [ IF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
fs_base        0x0                 0
gs_base        0x0                 0
```

Note que, até aqui, nós utilizamos apenas os registradores…

- `RAX`: Identificação da chamada de sistema `write` (`0x1`);
- `RDX`: Quantidade de bytes a serem impressos (`0x11 = 17`);
- `RSI`: O endereço do primeiro byte da mensagem (`0x402000`);
- `RDI`: O número do descritor de arquivos (`0x1 = stdout`).

Todos os demais estão em seus estados originais, menos `RIP`, o ponteiro de instruções, que sempre é atualizado para indicar o endereço da próxima instrução a ser executada. No caso, `RIP` contém o endereço `0x401019`, que é exatamente o endereço da instrução `syscall`, no nosso ponto de parada.

Mas vejamos o que acontece quando a chamada de sistema é processada:

```
(gdb) next
Salve, simpatia!
15	    mov rax, 60         ; syscall exit
(gdb) info registers
rax            0x11                17
rbx            0x0                 0
rcx            0x401004            4198404
rdx            0x11                17
rsi            0x402000            4202496
rdi            0x1                 1
rbp            0x0                 0x0
rsp            0x7fffffffdfd0      0x7fffffffdfd0
r8             0x0                 0
r9             0x0                 0
r10            0x0                 0
r11            0x302               770
r12            0x0                 0
r13            0x0                 0
r14            0x0                 0
r15            0x0                 0
rip            0x40101b            0x40101b <_start+27>
eflags         0x202               [ IF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
fs_base        0x0                 0
gs_base        0x0                 0
```

Observe que apenas três registradores foram alterados:

|Registrador|Antes|Depois|
|---|---|---|
|`RAX`|`0x1`|`0x11 (17)`|
|`RCX`|`0x0`|`0x401004`|
|`R11`|`0x0`|`0x302`|

**Causa da mudança em RAX:**

O registrador `RAX` é utilizado como retorno da chamada de sistema `write` e recebe a quantidade de bytes impressos (17).

**Causa da mudança em RCX:**

Quando uma chamada de sistema é processada, o endereço da instrução seguinte (originalmente em `RIP`) é salvo em `RCX` e restaurado para `RIP` antes do retorno da chamada, mas isso não significa que, no espaço de usuário, após a linha de `syscall`, nós encontraremos em `RCX` o mesmo endereço registrado em `RIP`. Not e que os valores em `RCX` e `RIP` são diferentes após a chamada de sistema:

```
(gdb) info registers rcx rip
rcx            0x401004            4198404
rip            0x40101b            0x40101b <_start+27>
```

Acontece que, depois de restaurado o endereço em `RIP`, o kernel está livre para utilizar `RCX` antes de retornar ao espaço de usuário. Por isso, em chamadas de sistema, `RCX` é considerado um registrador _volátil_, ou seja, seu conteúdo pode ser alterado a qualquer momento pelos próprios mecanismos internos da transição entre os modos de execução, sem qualquer garantia de preservação.

**Causa da mudança em R11:**

Internamente, em uma chamada de sistema, `R11` é utilizado para salvar e restaurar o estado do registrador `RFLAGS` (`eflags` no GDB). Contudo, assim como acontece com `RCX`, após a restauração de `RFLAGS` o kernel está livre para utilizar `R11` no processo de transição de volta ao espaço de usuário, o que justifica a diferença encontrada nesses dois registradores:

```
(gdb) info registers r11 eflags
r11            0x302               770
eflags         0x202               [ IF ]
```

### Preservação de registradores com a pilha

Se for necessário, é possível salvar e restaurar os registradores `RAX`, `RCX` e `R11` para a execução de uma chamada de sistema. A técnica mais comum em arquiteturas x86_64 envolve a utilização da pilha de memória:

```asm
push rax    ; salva rax como elemento N da pilha
push rcx    ; salva rcx como elemento N+1 da pilha
push r11    ; salva r11 como elemento N+2 da pilha (atual topo)

; código da chamada de sistema...

pop r11     ; restaura e remove elemento N+2 da pilha (atual topo)
pop rcx     ; restaura e remove elemento N+1 da pilha
pop rax     ; restaura e remove elemento N da pilha
```

É importante observar a ordem de salvamento e restauração, porque estamos lidando com uma pilha, e o elemento em seu topo sempre será o último que tiver sido empilhado.

### Resumo comparativo com chamadas de funções

Finalmente, aqui está uma tabela comparando as principais diferenças entre as convenções de chamadas de funções e a ABI do kernel para chamadas de sistema:

|Aspecto|Chamadas de função|Chamadas de sistema|
|---|---|---|
|Interface|Conveção System V ABI|ABI do kernel (syscall ABI)|
|Usa a pilha de memória|Sim (quadro de pilha)|Não (usa registradores)|
|Preserva registradores|Sim|Não (registradores temporários)|
|Instrução de transição|`call`, `ret`|`syscall`|
|Participação do kernel|Não|Sim|

## Referências

- man 2 syscall
- [OS Dev: Calling Conventions](https://wiki.osdev.org/Calling_Conventions)
- [OS Dev: Stack](https://wiki.osdev.org/Stack)