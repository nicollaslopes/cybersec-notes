# Introdução à linguagem Assembly (NASM)

## Objetivos

- Apresentar as características e os elementos básicos da linguagem Assembly.
- Diferenciar conceitos da programação em alto e baixo nível.
- Conhecer as características da implementação NASM para Linux 64 bits.
- Aprender as instruções e diretivas essenciais para começar a programar.
- Criar executar um primeiro programa em Assembly.
## Máquina de Turing

No começo do século 20, surge uma questão no campo da matemática: até onde seria possível solucionar problemas seguindo estritamente um conjunto de regras fixas estabelecidas em um _algoritmo_?

> **Nota:** em matemática, _algoritmos_ são sequências finitas de ações executáveis que visam obter soluções para determinados tipos de problema.

Nos anos 1930, o matemático Alan Turing estava trabalhando em um problema de lógica formal (campo que estuda a estrutura de enunciados e suas regras). Como parte do que estava tentando demonstrar, ele precisava encontrar uma forma geral para descrever como algoritmos poderiam ser implementados e executados mecanicamente.

A partir disso, ele formulou o conceito matemático de uma máquina abstrata que manipularia automaticamente os símbolos em uma fita de acordo com uma tabela de regras: a sua _a-machine_ (de _"automatic machine"_), mais tarde chamada de _Máquina de Turing_ por seu orientador de doutorado – termo que utilizamos até hoje.

Então, a Máquina de Turing é um modelo matemático que descreve um dispositivo muito simples que seria capaz de expressar computações arbitrárias e, com isso, explorar as propriedades e as limitações da computação mecânica.

> **Nota:** _modelos matemáticos computacionais_ são modelos que descrevem como a saída de uma função matemática é computada a partir de uma dada entrada.

Turing descreveu a imagem física de sua máquina como consistindo de…

- Uma fita dividida em células contendo símbolos ou espaços vazios;
- Um cabeçote para ler e escrever símbolos na fita;
- Um registrador de estados, para armazenar o estado corrente de um conjunto finito de estados possíveis;
- Uma tabela de instruções que, dado o estado atual e o que estiver sendo lido na fita, diz à máquina o que fazer na sequência.

Por exemplo:

```
Estado atual: q1

Fita:
   ┌───┬───┬───┬───┬───┬───┬───┐
   │ ␣ │ 1 │ 1 │ 0 │ ␣ │ ␣ │ ␣ │
...└───┴───┴───┴───┴───┴───┴───┘...
                 ↑
              Cabeçote

Regra:
Se (q1, 0) -> escreva 1, mova para a direita, mude estado para q2
```

Regra:
Se (q1, 0) -> escreva 1, mova para a direita, mude estado para q2

**Ilustração moderna:** [vídeo de Mike Davey](https://www.youtube.com/watch?v=E3keLeMwfHY)

Esse modelo simples, apesar de abstrato, revelou-se poderoso o suficiente para representar qualquer cálculo que possa ser feito por um algoritmo. Contudo, a computação sequencial das células de uma fita é lenta demais para aplicações práticas modernas. Mesmo assim, a máquina que Turing descreveu tornou-se uma das principais inspirações para a construção das _unidades de processamento_, utilizadas nos computadores até os nossos dias.

### Uma longa fita de bits

Na computação moderna, os nossos programas ainda podem ser vistos como uma fita contendo uma longa sequência de bits (valores binários `0` e `1`). Mas, diferente da máquina de Turing:

- Os bits são armazenados na memória principal do computador, onde podem ser acessados de forma aleatória e quase instantânea;
- Em vez de símbolos individuais por célula e uma tabela de estados, os bits são interpretados em blocos de tamanho fixo, expressando as instruções que a CPU deve executar e os dados envolvidos nessas operações;
- A tabela de estados agora está incorporada nos processadores na forma de um conjunto fixo de instruções específico de cada CPU, conhecido como ISA (_Instruction Set Architecture_).

Portanto, _escrever um programa_ significa, em essência, representar, nessa longa fita de bits, os códigos relativos aos dados e instruções que uma máquina em particular será capaz de interpretar e executar. Isso é o que chamamos de _código de máquina_ (ou, popularmente, _linguagem de máquina_) e, em vários momentos da história da computação, os programas foram escritos desta forma.

Obviamente, escrever diretamente o código de máquina é um processo demorado, tedioso e propenso a erros. Por isso, já nos anos 1950, começaram a ser criados programas dedicados à tradução de representações simbólicas de instruções para as representações binárias do código de máquina. Na prática, o que esses programas fazem é utilizar as representações simbólicas como um "manual de instruções" para montar o código de máquina correspondente – daí serem chamados de _montadores_ (ou _assemblers_, em inglês).

Vale observar que, inicialmente, essas representações simbólicas não eram escritas em arquivos de texto como fazemos hoje. Em geral, os códigos das instruções e dados eram registrados fisicamente, em fitas ou cartões perfurados, que depois eram lidos por máquinas. Assim, uma _linguagem de montagem_ (_assembly language_) já existia, mas ainda se passava por uma longa fase onde o processo de escrita de programas era manual e bastante sujeito a erros mecânicos e operacionais.

## O que é Assembly

Assembly é uma linguagem de programação que oferece formas de representar, em texto legível, os códigos numéricos das operações que o hardware da máquina é capaz de processar. A rigor, qualquer linguagem faz isso, mas um programa em Assembly se diferencia por expressar instruções que correspondem, cada uma, a apenas um passo daquilo que o processador (a CPU) deve fazer – sem abstrações e numa relação praticamente direta com o código de máquina.

### Linguagem dependente da arquitetura

Como representa diretamente as instruções executadas por um processador, a linguagem Assembly depende necessariamente das características da arquitetura desse processador. Em especial, cada CPU é projetada com seu próprio conjunto de instruções (ISA, do inglês _Instruction Set Architecture_). A ISA é, por definição, a própria interface entre o hardware e o software de um sistema computacional. É ela que define, entre outras coisas:

- Quais operações a CPU pode executar;
- Quais registradores estão disponíveis;
- Como as instruções são codificadas em binário;
- Como os operandos são manipulados;
- Como a memória pode ser utilizada.

Sendo assim, não há apenas uma linguagem Assembly, a não ser como um termo genérico que engloba diversas linguagens, todas baseadas na representação simbólica das operações de arquiteturas específicas.

No contexto dos nossos estudos, a linguagem Assembly que utilizaremos é aquela voltada à arquitetura Intel x86_64, tal como implementada no _montador_ NASM (_Netwide Assembler_), e é a ela que estaremos nos referindo todas as vezes que dissermos _"Assembly"_.

### Linguagem de montagem

A palavra _"assembly"_ vem do inglês e significa _"montagem"_. Portanto, o nome deriva da primeira etapa do processo que transforma os programas escritos na linguagem (os _códigos-fonte_) nos códigos binários executáveis pela máquina (_código de máquina_). Esse processo se chama _"montagem"_ e é feita por um programa chamado _"montador"_ (ou _"assembler"_, em inglês).

> **Importante!** A linguagem se chama _"Assembly"_, não _"assembler"_, que é como chamamos o programa que implementa a linguagem e faz a montagem do código de máquina.

### Classificações da linguagem

O Assembly pode ser classificado como uma linguagem…

#### De baixo nível de abstração (segunda geração)

Trabalha diretamente com os recursos da arquitetura da máquina, como registradores, endereços de memória e instruções específicas da CPU. Não há camadas intermediárias entre o código e o hardware.

#### Imperativa

As instruções são dadas em uma sequência explícita, descrevendo passo a passo o que deve ser feito. O controle do fluxo é manual, baseado em saltos e comparações, sem abstrações de alto nível.

#### De tipagem inexistente

Não há distinção formal entre tipos de dados. Tudo é interpretado como sequências de bytes, cabendo a quem programa decidir como os dados devem ser lidos ou manipulados.

#### Com gerenciamento manual de memória

Não há alocação automática ou coleta de lixo. O programador deve lidar com endereços de memória diretamente, reservando, acessando e liberando espaços manualmente, de acordo com a necessidade.

#### Compilada via montagem

O código Assembly não é interpretado nem compilado. Em vez disso, ele é traduzido diretamente para código de máquina pelo montador (_assembler_), que gera um binário executável diretamente ou um arquivo binário do tipo _objeto_. Para que tenhamos um executável de fato, ainda pode ser necessário realizar uma etapa de _ligação_, em que utilizamos um _editor de ligações_ para adequar o binário montado às especificações do formato de executáveis do sistema operacional.

#### Paradigmas

Também pode ser considerada uma linguagem procedural, mas com ressalvas, já que é uma linguagem que só oferece instruções e, portanto, não impõe nenhum paradigma. Isso quer dizer que nós podemos escrever programas de forma procedural, mas também de forma totalmente linear e até orientada a macros.

#### De uso específico

Apesar de ser uma linguagem completa, o Assembly é geralmente usado para finalidades bem específicas, como a implementação de trechos críticos de desempenho, rotinas de acesso direto ao hardware, criação de módulos de sistemas operacionais ou interação com o sistema em níveis muito mais baixos. Raramente é usado para o desenvolvimento de aplicações inteiras, principalmente devido à sua complexidade, verbosidade e baixa portabilidade.

### Comparação com outras linguagens

Por ser uma linguagem de segunda geração, o Assembly opera em um nível muito mais próximo do hardware do que linguagens de alto nível. Isso traz uma série de diferenças fundamentais em relação à forma como os programas são escritos, estruturados e compreendidos.

Aqui estão algumas dessas diferenças…

#### Não há estruturas de controle

Em Assembly nós não temos estruturas de controle abstratas, como `if`, `case`, `for` ou `while`. Em vez disso, todo o controle de fluxo do programa tem que ser implementado com instruções de comparação e saltos para outras partes do programa.

#### Não há o conceito de variáveis

Os dados são associados diretamente a registradores da CPU ou a endereços de memória identificados por _rótulos_. Toda a manipulação de dados é feita por meio de instruções de cópia, como resultado de operações lógicas e aritméticas ou em definições que serão processadas durante a montagem do arquivo binário.

#### Ponteiros, só por analogia

Como não há variáveis, também não existe exatamente o conceito de _ponteiro_, a não ser por analogia. Os rótulos, por exemplo, representam endereços de memória, e nós até podemos pensar neles como os ponteiros e vetores da linguagem C, mas isso pode causar algumas confusões conceituais e deve ser evitado.

#### Em vez de tipos, quantidades de bytes

Novamente: todos os dados são escritos diretamente em registradores ou na memória. Se usarmos registradores, as suas capacidades determinarão quantos bytes poderão (ou deverão) ser escritos. A escrita direta na memória, por outro lado, não tem um limite fixo, mas depende da quantidade de memória endereçável e da maneira como o programa a utiliza.

> **Nota:** uma das principais finalidades do conceito de _tipos_ é a delimitação do acesso aos dados. Por exemplo, declarar uma variável como do tipo `int` limita o acesso a apenas 4 bytes (em arquiteturas x86_64), mas não existe essa forma de restrição no Assembly. Nós somos livres para ler ou escrever quantos bytes quisermos, em qualquer endereço, desde que tenhamos permissão do sistema.

## Por que aprender Assembly?

### Entendimento do funcionamento de computadores

Aprender Assembly proporciona uma visão detalhada do que realmente acontece durante a execução de um programa, como o ciclo de execução de instruções, o uso de registradores, o papel da memória RAM, o funcionamento da pilha e tantos outros conhecimentos fundamentais para a compreensão da arquitetura e da organização de sistemas computacionais modernos.

### Depuração e análise

Ferramentas de depuração, análise, segurança e engenharia reversa são muito mais eficazes nas mãos de quem entende Assembly, especialmente quando tudo que temos são o código de máquina ou a _desmontagem_ do binário de um programa.

### Integração com C

É comum utilizar Assembly para otimizações pontuais em programas escritos em C, como em trechos críticos de desempenho ou quando se deseja acesso direto a instruções específicas da arquitetura. Também é necessário para a escrita de rotinas de sistema, drivers ou na manipulação direta de hardware, especialmente em ambientes com recursos limitados ou sem suporte a bibliotecas padrão.

### Aprendizado

Assembly é uma tremenda ferramenta didática em disciplinas como arquitetura e organização de computadores, sistemas operacionais, design de linguagens e construção de compiladores. Com ele, nós podemos demonstrar e observar diretamente conceitos que, de outra forma, seriam ocultados por várias camadas de abstração.

### Principais sintaxes

Embora as instruções de máquina sejam definidas pela arquitetura da CPU, a maneira como essas instruções são escritas em Assembly pode variar bastante, dependendo da sintaxe adotada pelo montador. Para as arquiteturas x86 e x86_64, destacam-se duas sintaxes principais: a sintaxe _Intel_ e a sintaxe _AT&T_.

#### Sintaxe AT&T

A sintaxe AT&T foi popularizada no contexto dos sistemas Unix e do projeto GNU. Ela é usada por padrão pelo montador GAS (_GNU Assembler_) e é a sintaxe adotada implicitamente pelo GCC ao gerar código Assembly. Os operandos seguem a ordem _origem → destino_, com prefixos `%` obrigatórios para registradores e `$` para valores (ex.: `movl $1, %rax`).

**"Salve, simpatia!" em sintaxe AT&T (GAS):**

```assembly
# salve-att.s
# Montar com: as salve-att.s -o salve-att.o
# Linkar com: ld salve-att.o -o salve-att

    .section .data
msg:
    .ascii "Salve, simpatia!\n"
len = . - msg

    .section .text
    .global _start

_start:
    # write(1, msg, len)
    mov    $1, %rax        # syscall: write
    mov    $1, %rdi        # stdout
    lea    msg(%rip), %rsi # endereço da mensagem
    mov    $len, %rdx      # tamanho da mensagem
    syscall

    # exit(0)
    mov    $60, %rax       # syscall: exit
    xor    %rdi, %rdi      # status 0
    syscall
```

#### Sintaxe Intel

Mais comum em livros didáticos, manuais da Intel e tutoriais voltados ao desenvolvimento em baixo nível, especialmente em ambientes _bare-metal_. Ela apresenta os operandos na ordem _destino ← origem_ (ex.: `mov rax, 1`), não usa prefixos nos registradores e é adotada por montadores como o NASM, que utilizaremos nos nossos estudos.

**"Salve, simpatia!" em sintaxe Intel (NASM):**

```asm
; salve-intel.asm
; Montar com: nasm -f elf64 salve-intel.asm
; Linkar com: ld salve-intel.o -o salve-intel

section .data
    msg db "Salve, simpatia!", 10  ; 10 = '\n'
    len equ $ - msg

section .text
    global _start

_start:
    ; write(1, msg, len)
    mov rax, 1          ; syscall número 1: write
    mov rdi, 1          ; stdout
    mov rsi, msg        ; endereço da mensagem
    mov rdx, len        ; tamanho da mensagem
    syscall

    ; exit(0)
    mov rax, 60         ; syscall número 60: exit
    xor rdi, rdi        ; status 0
    syscall
```

#### Compatibilidade

Embora o GAS (executável `as`) use a sintaxe AT&T por padrão, ele também aceita código em sintaxe Intel com a diretiva `.intel_syntax` no início do código-fonte, geralmente acompanhada de `noprefix`. Alguns compiladores e depuradores, como o GCC e o GDB, também oferecem suporte a ambas as sintaxes – por exemplo, com a opção `-masm=intel`, no GCC, e o comando `set disassembly-flavor`, no GDB.

## O Netwide Assembler (NASM)

O _Netwide Assembler_ (NASM) é um montador livre muito utilizado na programação em Assembly, especialmente para arquiteturas x86 e x86_64. Ele é conhecido por sua sintaxe clara e direta e por oferecer suporte a formatos de saída para várias plataforma.

|Característica|Descrição|
|---|---|
|**Sintaxe**|Intel (não suporta AT&T)|
|**Arquiteturas suportadas**|x86 (16/32 bits), x86_64 (64 bits)|
|**Formatos de saída**|ELF, COFF, Mach-O, bin, RDOFF|
|**Processo de montagem**|Gera binários brutos (_raw_) e arquivos objeto (exige link-edição separada)|
|**Controle de código**|Total controle sobre instruções, endereços e seções|
|**Pré-processamento**|Diretivas simples; sem pré-processador complexo como no C|
|**Suporte a macros**|Sim, com argumentos e controle condicional|
|**Integração com o sistema**|Compatível com chamadas de sistema Linux, Windows e Unix em geral|
|**Otimizações**|Não realiza otimizações nem análises semânticas|
|**Documentação**|Completa e com muitos exemplos|

### Sintaxe de instruções

A sintaxe geral de uma instrução em NASM segue uma forma bastante regular e compreensível:

```
[rótulo:]   instrução   operando1, operando2   ; comentário
```

Onde…

|Elemento|Obrigatório|Exemplos|Descrição|
|---|---|---|---|
|**Rótulo**|Opcional|`loop_start:`|Nome associado a um endereço. Pode ser usado como destino de saltos.|
|**Instrução**|Sim|`mov`, `add`, `jmp`|A operação a ser executada.|
|**Operando 1**|Depende|`rax`, `[msg]`|Primeiro operando: geralmente o destino (registrador ou memória).|
|**Operando 2**|Depende|`rbx`, `1`, `[valor]`|Segundo operando: geralmente a origem dos dados (registrador, memória ou um valor imediato).|
|**Comentário**|Opcional|`; isso é um salto`|Começa com `;` e vai até o fim da linha. Ignorado pelo montador.|

#### Diretivas para definições de dados

Além das instruções, mas ainda seguindo a estrutura geral da sintaxe, o NASM oferece várias diretivas para definir dados ou reservar espaços na memória, como:

|Diretiva|Uso|Ação|Equivalente aproximado em C|
|---|---|---|---|
|`db`|`db 0x48`|Define um byte (define byte)|`char x = 0x48;`|
|`dw`|`dw 1234h`|Define uma **word** (2 bytes)|`short x = 0x1234;`|
|`dd`|`dd 1.0`|Define uma **double word** (4 bytes)|`int x = 1;` / `float x = 1.0f;`|
|`dq`|`dq 3.14159`|Define uma **quad word** (8 bytes)|`double x = 3.14159;`|
|`dt`|`dt 1.23e400`|Define uma **ten-byte** (80 bits, float ext)|`long double x = …` (FPU específico)|
|`resb`|`resb 64`|Reserva 64 bytes (sem inicializar)|`char buffer[64];` (sem valor)|
|`resw`|`resw 8`|Reserva 8 words (16 bytes)|`short arr[8];`|
|`resd`|`resd 4`|Reserva 4 dwords (16 bytes)|`int arr[4];`|
|`resq`|`resq 2`|Reserva 2 qwords (16 bytes)|`long arr[2];`|

Essas diretivas podem ou não ser antecedidas de rótulos para indicar onde esses dados serão inseridos na memória, por exemplo:

```asm
msg:    db "Salve, simpatia!", 0   ; 'msg' é um rótulo para o endereço do primeiro byte
buffer: resb 64                    ; 'buffer' é um rótulo para a área reservada
```

É permitido definir e reservar espaços sem rótulos, mas os dados só poderão ser acessados com base em cálculos de posições relativas.

### Sintaxe de metaprogramação da montagem

A sintaxe de metaprogramação da montagem refere-se ao conjunto de diretivas, macros e comandos especiais fornecidos pelo montador para escrever código que gera ou manipula outro código durante a montagem. Essa sintaxe vai além das instruções padrão da CPU e possibilita:

- Definir constantes simbólicas e macros reutilizáveis.
- Realizar substituições textuais antes da montagem real.
- Implementar estruturas condicionais e laços para controle do fluxo de montagem.
- Gerar código de forma dinâmica e programável, facilitando a automação e a abstração.

Na tabela, nós temos alguns exemplos de diretivas e comandos usados na sintaxe de metaprogramação de montagem do NASM:

|Diretiva / Comando|Categoria|Descrição breve|Exemplo simples|
|---|---|---|---|
|`equ`|Definição simbólica|Define uma constante simbólica imutável|`MAXLEN equ 32`|
|`=`|Definição simbólica|Define símbolo numérico, pode ser redefinido|`counter = 0`|
|`%define`|Macros simbólicas|Define uma macro simples (substituição textual)|`%define MSG "Olá!"`|
|`%assign`|Macros simbólicas|Define símbolo numérico que pode ser alterado|`%assign i 10`|
|`%macro`|Macros com parâmetros|Define macros com parâmetros e corpo expansível|`%macro PRINT_MSG 0`|
|`%ifdef` / `%ifndef`|Controle condicional|Compila bloco se símbolo estiver (ou não) definido|`%ifdef DEBUG`|
|`%include`|Inclusão de arquivos|Insere outro arquivo fonte no ponto da montagem|`%include "utils.inc"`|
|`%rep` / `%endrep`|Laços e repetição|Repete um bloco de código um número fixo de vezes|`%rep 4`|

### Operadores e símbolos

No NASM, a montagem de código também envolve o uso de operadores e símbolos especiais que permitem calcular endereços, manipular valores e controlar a geração do programa, tudo em tempo de montagem.

Aqui estão alguns exemplos:

| Símbolo                   | Descrição                                         | Uso / Exemplo                                  |
| ------------------------- | ------------------------------------------------- | ---------------------------------------------- |
| `$`                       | Endereço da instrução atual                       | `jmp $+5` — salta 5 bytes à frente             |
| `$$`                      | Endereço do início do segmento ou bloco atual     | `jmp $$-$` — salto relativo ao início do bloco |
| `$+n` / `$-n`             | Endereço relativo deslocado `n` bytes             | `jmp $-10` — volta 10 bytes                    |
| `%`                       | Indica que um símbolo é um valor numérico (macro) | `%define X 10`                                 |
| `:`                       | Define ou qualifica rótulos                       | `loop_start:`                                  |
| `*`                       | Multiplicação em expressões                       | `len equ 4 * 10`                               |
| `+`, `-`, `/`, `<<`, `>>` | Operadores aritméticos e bitwise                  | `size equ 1 << 4`                              |
| `[...]`                   | Acesso indireto à memória (endereços)             | `mov eax, [ebx]`                               |
| `?`                       | Operador ternário em macros                       | `%if ?(cond) ... %endif`                       |

### Seções no NASM

Quando escrevemos um programa em Assembly, é preciso organizar o código e os dados de forma que o sistema operacional possa carregá-lo corretamente na memória. Essa organização é feita por meio das _seções_ (ou _segmentos_), que dividem o programa em partes distintas com propósitos diferentes. Mas isso não é apenas uma convenção de organização: a separação das seções é exigida pelo formato do executável (como ELF no Linux ou PE no Windows) e é fundamental para que o _carregador_ do sistema (_loader_) saiba onde colocar cada parte do programa na memória.

No NASM, nós usamos a diretiva `section` para declarar as divisões do código, que podem ser:

- `section .text`: armazena o código executável do programa (as instruções);
- `section .data`: contém dados inicializados (variáveis com valores definidos);
- `section .rodata`: usada para dados constantes somente leitura (como strings fixas);
- `section .bss`: reserva espaço para dados não inicializados (variáveis com valor padrão, geralmente zero).

### Importação e exportação de símbolos

Símbolos são nomes associados a endereços e valores em um programa. Eles podem representar rótulos de rotinas, posições de dados na memória ou constantes definidas em tempo de montagem. Em Assembly, esses símbolos servem como pontos de referência tanto para o código quanto para o montador e o editor de ligações (_link-editor_).

Quando escrevemos programas em NASM que serão ligados com outros módulos, como bibliotecas externas ou arquivos objeto, nós precisamos informar ao editor de ligações quais símbolos devem ser _exportados_ para esses módulos e quais devem ser _importados_ de outros lugares. No NASM, isso é feito com as diretivas `global` (exportação) e `extern` (importação).

No GNU/Linux, especialmente quando não usamos uma linguagem de mais alto nível (como C), nós precisamos especificar manualmente o ponto de entrada do programa — ou seja, a função onde o sistema operacional deve começar a execução. Por padrão, o Linux espera que o ponto de entrada seja chamado de `_start`, que deve ser exportado com a diretiva:

```asm
global _start
```

Essa instrução diz ao NASM que o símbolo `_start` deve ser incluído na tabela de símbolos do arquivo objeto montado. Assim, o editor de ligações (`ld`) poderá encontrá-lo. Nó código, `_start` geralmente é escrito no início da seção `.text`:

```
global _start

section .text
_start:

    ; O código de início do programa vem aqui...
```

### Tipos de operandos em instruções

Toda instrução Assembly opera sobre um ou mais valores que nós chamamos de operandos. Eles indicam a origem e o destino dos dados e podem representar:

- Registradores;
- Endereços de memória;
- Valores imediatos (constantes escritas diretamente no código).

A combinação entre esses tipos de operandos determina a validade e o comportamento das instruções. Por exemplo, uma instrução `mov` pode copiar…

- Dados de um registrador para outro registrador;
- Um valor imediato para um registrador;
- Dados em um endereço de memória para um registrador;
- Dados de um registrador para um endereço de memória.

Mas não pode, por exemplo, copiar dados de um registrador, ou de um endereço de memória, para um valor imediato, e o NASM exige que essas combinações estejam de acordo com a sintaxe e a semântica definida pela arquitetura.

## Instruções essenciais

A arquitetura x86_64 tem centenas de instruções, mas nós só precisamos conhecer algumas delas para começar a acompanhar os exemplos dos próximos tópicos – com o tempo, nós acrescentaremos organicamente algumas outras ao nosso arsenal da linguagem Assembly.

### Instrução `mov`

A instrução `mov` é usada para copiar valores de um operando para outro. É uma das instruções mais utilizadas em Assembly, pois quase todo código precisa manipular dados em registradores, na memória ou com valores imediatos.

**Forma geral:**

mov destino, origem

O valor de `origem` é copiado para `destino`.

> **Importante!** Isso não é uma troca de dados entre operandos, é uma cópia de um operando para outro.

**Combinações válidas:**

|Origem|Destino|Exemplo|Observações|
|---|---|---|---|
|Imediato|Registrador|`mov rax, 42`|Valor literal (imediato) para registrador|
|Registrador|Registrador|`mov rbx, rax`|Cópia entre registradores|
|Registrador|Memória|`mov [var], rax`|Salva conteúdo em memória|
|Memória|Registrador|`mov rax, [var]`|Carrega conteúdo da memória|
|Imediato|Memória|`mov [var], 10`|Atribuição direta a endereço|

O NASM não permite movimentos diretos de memória para memória.

**Exemplo de uso:**

```asm
section .data
    msg db 42         ; variável na memória

section .text
    global _start
_start:
    mov rax, [msg]    ; carrega o valor de msg em rax
    mov rbx, rax      ; copia rax para rbx
    mov [msg], 99     ; sobrescreve msg com novo valor
```

### Instrução `lea`

Carrega o endereço efetivo de um operando de memória (não o valor na memória).

> O endereço efetivo é o resultado da avaliação de uma expressão que resulta no endereço que será usado para acessar a memória.

**Forma geral:**

```
lea destino, [base + índice*escala + deslocamento]
```

**Combinações válidas:**

|Destino|Origem (memória)|Exemplo|
|---|---|---|
|Registrador|Memória simbólica|`lea rax, [rbx+4]`|
|Registrador|Endereços|`lea rsi, [rip+msg]`|

**Exemplo de uso:**

```asm
section .rodata
    msg: db "Olá, mundo!", 10 ; string com quebra de linha

section .text
global _start

_start:
    lea rsi, [rel msg] ; carrega o endereço de msg em rsi
    mov rax, 1         ; syscall: write
    mov rdi, 1         ; descritor: stdout
    mov rdx, 13        ; tamanho da mensagem
    syscall
                       ; ...
```

### Instruções aritméticas básicas (`inc`, `dec`, `add` e `sub`)

Estas instruções realizam operações aritméticas simples diretamente em registradores ou na memória.

#### Instrução `inc`

Incrementa (soma 1) o valor de um operando.

**Forma geral:**

```
inc destino
```

**Operandos válidos:** registrador ou endereço de memória.

**Exemplo:**

```asm
mov rcx, 5
inc rcx      ; rcx passa a valer 6
```

#### Instrução `dec`

Decrementa (subtrai 1) o valor de um operando.

**Forma geral:**

```
dec destino
```

**Operandos válidos:** registrador ou endereço de memória.

**Exemplo:**

```asm
mov rcx, 5
dec rcx      ; rcx passa a valer 4
```

#### Instrução `add`

Soma um operando ao destino e armazena o resultado no destino.

**Forma geral:**

```
add destino, origem
```

**Combinações válidas:**

| Destino     | Origem         | Exemplo        |
| ----------- | -------------- | -------------- |
| Registrador | Registrador    | add rax, rbx   |
| Registrador | Valor imediato | add rax, 10    |
| Registrador | Memória        | add rax, [var] |
| Memória     | Registrador    | add [var], rax |
| Memória     | Valor imediato | add [var], 1   |

**Exemplo:**

```asm
mov rax, 3
add rax, 7    ; rax passa a valer 10
```

#### Instrução `sub`

Subtrai um operando do destino e armazena o resultado no destino.

**Forma geral:**

```
sub destino, origem
```

**Combinações válidas:**

|Destino|Origem|Exemplo|
|---|---|---|
|Registrador|Registrador|sub rax, rbx|
|Registrador|Valor imediato|sub rax, 10|
|Registrador|Memória|sub rax, [var]|
|Memória|Registrador|sub [var], rax|
|Memória|Valor imediato|sub [var], 1|

**Exemplo:**

```asm
mov rax, 10
sub rax, 3    ; rax passa a valer 7
```

### Instruções de multiplicação e divisão (`mul`, `imul`, `div` e `idiv`)

Estas instruções realizam operações aritméticas de multiplicação e divisão.

#### Instrução `mul` (multiplicação sem sinal)

Realiza multiplicação sem sinal entre o valor no registrador acumulador e um operando, armazenando o resultado em registradores específicos.

**Funcionamento:**

|Tamanho dos operandos|Multiplicação|Resultado|
|---|---|---|
|8 bits|`AL` por operando|`AX`|
|16 bits|`AX` por operando|`DX:AX`|
|32 bits|`EAX` por operando|`EDX:EAX`|
|64 bits|`RAX` por operando|`RDX:RAX`|

**Forma geral:**

```
mul operando
```

**Operando:** registrador ou memória, sem sinal.

**Exemplo:**

```asm
mov rax, 5
mul qword [valor]  ; rax * [valor], resultado em RDX:RAX
```

#### Instrução `imul` (multiplicação com sinal)

Mais flexível que `mul`, pode usar 1, 2 ou 3 operandos.

**Forma geral unária (1 operando):**

```
imul operando
```

|Tamanho dos operandos|Multiplicação|Resultado|
|---|---|---|
|8 bits|`AL` por operando|`AX`|
|16 bits|`AX` por operando|`DX:AX`|
|32 bits|`EAX` por operando|`EDX:EAX`|
|64 bits|`RAX` por operando|`RDX:RAX`|

**Forma geral binária (2 operandos):**

```
imul destino, origem (resultado em destino)
```

- Multiplica `destino` por `origem` e armazena o resultado em `destino`.
- Se o resultado exceder a capacidade de `destino`, apenas os bytes mais baixos serão armazenados (não há extensão de registradores).
- Afeta as flags `CF` (_carry_) e `OF` (_overflow_) no caso de estouro da capacidade de `destino`.

**Forma geral ternária (3 operandos):**

```
imul destino, origem, imediato   (resultado em destino)
```

- Multiplica `origem` pelo valor de `imediato` e armazena o resultado em `destino`.
- Se o resultado exceder a capacidade de `destino`, apenas os bytes mais baixos serão armazenados (não há extensão de registradores).
- Afeta as flags `CF` e `OF` no caso de estouro da capacidade de `destino`.

**Operandos válidos para `imul`:**

|Forma|Operando 1 (destino)|Operando 2 (origem)|Operando 3 (imediato)|
|---|---|---|---|
|1 operando|`RAX`, `EAX`, `AX` ou `AL` (implícito)|Qualquer registrador ou memória|(Não há)|
|2 operandos|Registrador|Registrador ou memória|(Não há)|
|3 operandos|Registrador|Registrador ou memória|Valor imediato|

**Exemplo unário:**

```asm
mov rax, 10
mov rbx, -4
imul rbx    ; rdx:rax = rax * rbx (com sinal)
```

**Exemplo binário:**

```asm
mov rax, 5
mov rbx, -3
imul rax, rbx    ; rax = rax * rbx (com sinal)
```

**Exemplo ternário:**

```asm
mov rbx, 7
imul rax, rbx, -2    ; rax = rbx * -2 (com sinal)
```

#### Instrução `div` (divisão sem sinal)

Realiza divisão sem sinal entre o valor no registrador acumulador estendido e um operando.

**Funcionamento:**

|Tamanho dos operandos|Divisão|Quociente|Resto|
|---|---|---|---|
|8 bits|`AX` por operando|`AL`|`AH`|
|16 bits|`DX:AX` por operando|`AX`|`DX`|
|32 bits|`EDX:EAX` por operando|`EAX`|`EDX`|
|64 bits|`RDX:RAX` por operando|`RAX`|`RDX`|

**Forma geral:**

```
div operando
```

**Operando:** registrador ou memória sem sinal.

**Exemplo:**

```asm
mov rax, 20
xor rdx, rdx         ; limpa (zera) RDX antes da divisão
div qword [divisor]  ; divide RDX:RAX por [divisor]
```

#### Instrução `idiv` (divisão com sinal)

Realiza divisão com sinal entre o valor no registrador acumulador estendido e um operando, considerando números negativos.

**Funcionamento:**

|Tamanho dos operandos|Divisão|Quociente|Resto|
|---|---|---|---|
|8 bits|`AX` por operando|`AL`|`AH`|
|16 bits|`DX:AX` por operando|`AX`|`DX`|
|32 bits|`EDX:EAX` por operando|`EAX`|`EDX`|
|64 bits|`RDX:RAX` por operando|`RAX`|`RDX`|

**Forma geral:**

```
idiv operando
```

**Operando:** registrador ou memória com sinal.

**Exemplo:**

```asm
mov rax, -50
cqto                  ; extende o sinal de RAX para RDX:RAX (antes de idiv)
idiv qword [divisor]  ; divide RDX:RAX por [divisor]
```

> **Nota:** Antes de usar `idiv`, é necessário preparar o registrador de extensão (RDX) com o valor correto usando `cqto` (_Convert Quadword to Octaword_), que estende o sinal de RAX para RDX:RAX. Para operandos menores, são utilizados `cwd` (16 bits) ou `cdq` (32 bits).

### Instrução `jmp` (salto incondicional)

Realiza um salto incondicional para o endereço ou rótulo especificado. A execução do programa continua a partir da posição de destino, sem avaliar qualquer condição.

**Forma geral:**

```
jmp destino
```

**Combinações válidas:**

|Destino|Tipo de operando|Exemplo|
|---|---|---|
|Rótulo|Endereço relativo|`jmp fim`|
|Registrador|Endereço absoluto|`jmp rax`|
|Memória|Ponteiro para endereço|`jmp [rax]`|
|Imediato|Offset relativo|`jmp $+5`|

**Exemplo de uso:**

```asm
section .text
global _start

_start:
    mov rax, 1         ; syscall: write
    mov rdi, 1         ; descritor: stdout
    mov rsi, msg       ; ponteiro para a string
    mov rdx, 16        ; tamanho da string
    syscall

    jmp fim            ; salta incondicionalmente para o final

meio:
    ; código que nunca será executado
    mov rax, 60
    mov rdi, 1
    syscall

fim:
    mov rax, 60        ; syscall: exit
    xor rdi, rdi       ; Estado de término 0
    syscall

section .rodata
    msg: db "Salve, simpatia!", 10
```

### Instruções de salto condicional

As instruções de salto condicional (ou desvio condicional) alteram o fluxo de execução com base no estado das flags da CPU, definidas por instruções de comparação como `cmp` ou `test`.

**Forma geral:**

```
jXX destino
```

Onde `jXX` representa uma das variantes condicionais e `destino` é um rótulo no código.

**Flags envolvidas:**

- `ZF` (_Zero Flag_): Ativa (`1`) se o resultado de uma operação for zero.
- `SF` (_Sign Flag_): Reflete o bit de sinal do resultado (`0` para positivo, `1` para negativo).
- `OF` (_Overflow Flag_): Ativa (`1`) se ocorreu _overflow_ em operação aritmética com sinal.
- `CF` (_Carry Flag_): Ativa se houve transporte (_carry_) ou empréstimo (_borrow_) em operações aritméticas com números sem sinal.

**Tabela de saltos comuns (em inteiros com sinal):**

|Instrução|Sinônimo|Condição|Descrição|
|---|---|---|---|
|`je`|`jz`|`ZF = 1`|Salta se igual (zero)|
|`jne`|`jnz`|`ZF = 0`|Salta se diferente (não zero)|
|`jg`|`jnle`|`ZF = 0` e `SF = OF`|Salta se maior|
|`jge`|`jnl`|`SF = OF`|Salta se maior ou igual|
|`jl`|`jnge`|`SF != OF`|Salta se menor|
|`jle`|`jng`|`ZF = 1` ou `SF != OF`|Salta se menor ou igual|

**Outras variantes comuns (sem sinal):**

|Instrução|Condição|Descrição|
|---|---|---|
|`ja`|`CF = 0` e `ZF = 0`|Salta se acima|
|`jae`|`CF = 0`|Salta se acima ou igual|
|`jb`|`CF = 1`|Salta se abaixo|
|`jbe`|`CF = 1` ou `ZF = 1`|Salta se abaixo ou igual|

**Exemplo de uso:**

```asm
section .text
global _start

_start:
    mov rax, 5
    mov rbx, 3
    cmp rax, rbx      ; compara rax com rbx

    jg maior          ; salta se rax > rbx
    jl menor          ; salta se rax < rbx
    je igual          ; salta se rax == rbx

maior:
    ; código para o caso de rax > rbx
    jmp fim

menor:
    ; código para o caso de rax < rbx
    jmp fim

igual:
    ; código para o caso de rax == rbx

fim:
    mov rax, 60       ; syscall: exit
    xor rdi, rdi      ; Resula em 0
    syscall
```

### Instrução `cmp`

Compara dois operandos realizando uma subtração entre eles, mas sem armazenar o resultado. O objetivo da instrução é atualizar as flags da CPU, que podem ser usadas posteriormente em instruções de salto condicional.

**Forma geral:**

```
cmp operando1, operando2
```

Internamente, equivale a:

```
sub operando1 operando2
```

Mas sem alterar o conteúdo de `operando1`.

**Combinações válidas:**

|Operando 1|Operando 2|Exemplo|
|---|---|---|
|Registrador|Registrador|`cmp rax, rbx`|
|Registrador|Imediato|`cmp rsi, 10`|
|Registrador|Memória|`cmp rcx, [array]`|
|Memória|Registrador|`cmp [rdi], rax`|
|Memória|Imediato|`cmp [rsi], 1`|

**Exemplo de uso:**

```asm
section .data
    valor: dq 42

section .text
global _start

_start:
    mov rax, [valor]     ; carrega o valor da memória
    cmp rax, 42          ; compara com 42
    je igual             ; salta se for igual

    mov rdi, 1           ; Termina com erro: não é igual
    jmp fim

igual:
    mov rdi, 0           ; Termina com sucesso: é igual

fim:
    mov rax, 60          ; syscall: exit
    syscall
```

### Instrução `test`

Realiza uma operação lógica `AND` entre dois operandos e atualiza as flags de acordo com o resultado. A instrução é usada principalmente para testar, de forma não destrutiva, se bits estão ligados ou se valores são zero.

> A instrução `cmp` testa diferença (via subtração), enquanto `test` testa presença de bits em comum (`AND` bit a bit) entre dois valores.

**Forma geral:**

```
test operando1, operando2
```

O resultado da operação `operando1 AND operando2` não é armazenado, apenas as flags da CPU (zero, sinal, paridade, etc.) são atualizadas.

**Combinações válidas:**

|Operando 1|Operando 2|Exemplo|
|---|---|---|
|Registrador|Registrador|`test rax, rax`|
|Registrador|Imediato|`test rbx, 1`|
|Memória|Registrador|`test [rdi], rax`|
|Memória|Imediato|`test [rsi], 0xFF`|

**Exemplo de uso:**

```asm
section .text
global _start

_start:
    mov rax, 0         ; valor a testar
    test rax, rax      ; testa se é zero
    jz deu_zero        ; salta se resultado for zero

    ; não deu zero
    mov rdi, 1
    jmp sair

deu_zero:
    ; era zero
    mov rdi, 0

sair:
    ; Termina com o valor recebido em rdi...
    mov rax, 60        ; syscall: exit
    syscall
```

### Instruções `call` e `ret`

As instruções `call` e `ret` são utilizadas para implementar chamadas e retornos de sub-rotinas em Assembly, o que possibilita a estruturação do código em blocos reutilizáveis. A instrução `call` salva o endereço da próxima instrução (retorno) na pilha e salta para o endereço especificado. A instrução `ret`, por sua vez, recupera o endereço de retorno na pilha e desvia a execução de volta para o ponto após a chamada.

**Forma geral:**

```
call destino
...
destino:
...
ret
```

**Combinações válidas:**

|Instrução|Operando|Exemplo|
|---|---|---|
|`call`|Rótulo|`call uma_sub`|
|`call`|Registrador|`call rax`|
|`call`|Ponteiro de memória|`call [rax]`|
|`ret`|(sem operando)|`ret`|

**Observações:**

- As instruções `call` e `ret` afetam o registrador de instrução (`RIP`), controlando o fluxo de execução.
- A instrução `call` também é utilizada para chamar funções importadas com `extern`.
- Em x86_64, os argumentos de função geralmente são passados por registradores, conforme a convenção de chamada de funções do sistema.

**Exemplo de uso:**

```asm
section .text
global _start

_start:
    call salve          ; chama a sub-rotina

    mov rax, 60         ; syscall: exit
    xor rdi, rdi        ; estado de término 0
    syscall

salve:
    mov rax, 1          ; syscall: write
    mov rdi, 1          ; stdout
    mov rsi, msg
    mov rdx, msglen
    syscall
    ret

section .rodata
    msg: db "Salve, simpatia!", 10
    msglen equ $ - msg
```

### Instrução `syscall`

A instrução `syscall` realiza uma chamada ao kernel do sistema operacional em 64 bits (em 32 bits, seria a instrução `int 80`), transferindo para ele o controle temporário da execução do programa. É o principal meio de acesso a serviços como leitura e escrita em arquivos, alocação de memória, término do programa, etc.

> Ao contrário das funções da biblioteca padrão (como `printf` ou `exit`), que são abstrações em C, a `syscall` faz chamadas diretas ao sistema.

**Forma geral:**

```
mov rax, numero_da_syscall
mov rdi, argumento1
mov rsi, argumento2
mov rdx, argumento3
mov r10, argumento4
mov r8,  argumento5
mov r9,  argumento6
syscall
```

**Registradores de argumento:**

|Posição|Registrador|Descrição|
|---|---|---|
|1º|`rdi`|Primeiro argumento|
|2º|`rsi`|Segundo argumento|
|3º|`rdx`|Terceiro argumento|
|4º|`r10`|Quarto argumento|
|5º|`r8`|Quinto argumento|
|6º|`r9`|Sexto argumento|

**Resultado:**

- O valor de retorno da _syscall_ é carregado em `rax`.
- Erros são indicados com valores negativos em `rax`.

**Exemplo de uso (escrevendo na saída padrão):**

```asm
section .rodata
    msg: db "Salve, simpatia!", 10
    len: equ $ - msg

section .text
global _start

_start:
    mov rax, 1      ; syscall: write
    mov rdi, 1      ; descritor de saída: stdout
    mov rsi, msg    ; ponteiro para os dados
    mov rdx, len    ; tamanho da mensagem
    syscall

    mov rax, 60     ; syscall: exit
    xor rdi, rdi    ; código de saída: 0
    syscall
```

> **Nota:** A lista de números das chamadas varia conforme o sistema operacional.

### Padrões frequentemente utilizados

Aqui estão alguns padrões de código muito utilizados em Assembly, seja por performance, clareza para o processador, ou por simples abreviação da escrita do programa.

Por exemplo, em vez de usar:

```asm
mov reg, 0
```

É comum escrever:

```asm
xor reg, reg   ; operação XOR bit a bit
```

Motivação:

- É mais rápido em alguns processadores.
- Utiliza menos bytes de código.
- Não depende de carregar um imediato `0`.

Nessa mesma linha, existem outros padrões bastante encontrados, como:

|Padrão|Descrição|
|---|---|
|`inc reg`|Incrementa o registrador em 1 em vez de usar `add`.|
|`dec reg`|Decrementa o registrador em 1 em vez de usar `sub`.|
|`neg reg`|Inverte o sinal do registrador em vez de multiplicar por `-1`.|

## Montagem e execução de programas em Assembly NASM

### Requisitos

Para garantir que você tenha todas as ferramentas necessárias para acompanhar nossos exemplos e demonstrações, verifique se os programas abaixo estão instalados no seu sistema:

- Um editor de código da sua preferência (Vim, Emacs, Geany, etc)
- `git`
- `nasm` e `ndisasm`
- `gcc`
- `as`
- `ld`
- `gdb`
- `readelf`
- `objdump`
- `objcopy`
- `nm`
- `ldd`
- `hexedit`
- `xxd` (geralmente instalado com o editor Vim)

No Debian e derivados, a maioria deles é instalada com os comandos:

```
sudo apt update
sudo apt install build-essential git nasm gdb binutils hexedit
```

### Exemplo: "Salve, simpatia!"

Tomando como exemplo o programa salve-intel.asm, escrito com a sintaxe Intel implementada pelo NASM:

```asm
; salve-intel.asm
; Montar com: nasm -f elf64 salve-intel.asm
; Linkar com: ld salve-intel.o -o salve-intel

section .data
    msg db "Salve, simpatia!", 10  ; 10 = '\n'
    len equ $ - msg

section .text
    global _start

_start:
    ; write(1, msg, len)
    mov rax, 1          ; syscall número 1: write
    mov rdi, 1          ; stdout
    mov rsi, msg        ; endereço da mensagem
    mov rdx, len        ; tamanho da mensagem
    syscall

    ; exit(0)
    mov rax, 60         ; syscall número 60: exit
    xor rdi, rdi        ; status 0
    syscall
```

O processo de montagem envolve a execução do montador `nasm` informando o formato de saída esperado (no caso, formato ELF de 64 bits):

```
:~$ nasm -f elf64 salve-intel.asm
```

O `nasm` gera o binário montado em um arquivo com mesmo nome do código-fonte, mas com a extensão `.o`, de _objeto_:

```
ls
salve-intel.asm  salve-intel.o
```

Esse arquivo objeto até poderia ser vinculado a outros binários para compor um programa (_ligação estática_), mas ele ainda não pode ser executado como está, pois ainda não contém todas as informações que o sistema operacional requer para carregá-lo na memória e executá-lo. Para isso, o arquivo objeto precisa ser processado por um editor de ligações (_link-editor_ ou simplesmente _ligador_):

```
:~$ ld salve-intel.o -o salve-intel
```

Como resultado, nós teremos um arquivo binário executável (inclusive com as devidas permissões) de nome `salve-intel`:

```
:~$ ls
salve-intel  salve-intel.asm  salve-intel.o
```

> Se não tivéssemos definido um nome de saída (opção `-o`), o executável seria chamado, por padrão, de `a.out`.

Para testar, basta executar:

```
:~$ ./salve-intel
Salve, simpatia!
```

## Links e referências úteis

- [Manual do NASM (2.16.02)](https://www.nasm.us/xdoc/2.16.02/html/)
- [Guia de referência das instruções Intel 64](https://namazso.github.io/x86/)
- [Tabela de chamadas de sistema Linux x86_64](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)