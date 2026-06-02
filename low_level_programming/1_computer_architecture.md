# Computer Architecture

## Objetivos

- Compreender os principais componentes de um computador sob o modelo de von Neumann.
- Primeiro contato com os registradores da arquitetura x86_64.
- Entender a relação entre o hardware e código Assembly.
- Criar um primeiro programa em Assembly com uma chamada de sistema.

## Modelo de von Neumann

Um computador possui:

- Unidade de processamento (ALU + Controladora)
- Memória (armazenamento de instruções e dados)
- Dispositivos de entrada/saída

```
            ┌───────────────────────────────────────┐
            │  UNIDADE DE PROCESSAMENTO (CPU)       │
            │                                       │
            │  • Unidade Lógica e Aritmética (ALU)  │
            │  • Unidade de Controle (CU)           │
            └───────────────────┬───────────────────┘
                                │
            ┌───────────────────┴───────────────────┐
            │               BARRAMENTO              │
            └───────┬──────────────────────┬────────┘
                    │                      │
            ┌───────┴───────┐     ┌────────┴────────┐
            │    MEMÓRIA    │     │ DISPOSITIVOS DE │
            │               │     │ ENTRADA E SAÍDA │
            └───────────────┘     └────────┬────────┘
                                           │
                                  ┌────────┴────────┐
                                  │  ARMAZENAMENTO  │
                                  └─────────────────┘
```


A arquitetura de Von Neumann é um modelo de hardware no qual a Unidade Central de Processamento (CPU) e a memória utilizam um espaço único de armazenamento para guardar tanto as instruções dos programas quanto os dados. Isso significa que a CPU acessa a mesma memória para buscar instruções e para ler ou gravar dados, utilizando um único barramento para ambas as operações.

Resumindo, no modelo de Von Neumann:

- Instruções e dados compartilham o mesmo espaço de memória.
- A CPU executa o ciclo: busca → decodificação → execução.

Em outros modelos da época (como a arquitetura de Harvard):

- Instruções e dados armazenados em memórias separadas.
- A CPU executa o ciclo: busca → decodificação → execução, podendo buscar dados e instruções em paralelo.

### Influência nas arquiteturas modernas

O modelo de Von Neumann influenciou profundamente as arquiteturas modernas de computadores, estabelecendo a base para a maioria dos designs de sistemas computacionais atuais. Algumas das principais influências incluem:

**Memória unificada**
	A ideia de compartilhar a mesma memória para dados e instruções se manteve como o padrão em muitas arquiteturas modernas, simplificando o design dos sistemas, embora existam variações: como a memória cache, que separa dados e instruções em níveis mais próximos ao processador.

**Ciclo de busca, decodificação e execução**
	O modelo de Von Neumann introduziu o ciclo básico de execução de instruções, que ainda é fundamental em CPUs modernas. As arquiteturas atuais, como x86 e ARM, seguem esse ciclo de maneira semelhante, embora com otimizações como a _pipelining_, onde múltiplas fases do ciclo podem ser realizadas em paralelo.

**Programação e flexibilidade**
	O modelo permite que os programas sejam tratados como dados, o que possibilita a criação de sistemas que podem ser facilmente modificados e adaptados. Isso contribui para o desenvolvimento de linguagens de programação de alto nível, sistemas operacionais e aplicativos dinâmicos.

**Simplicidade e custo**
	O modelo de Von Neumann simplificou o design de computadores ao integrar a memória de instruções e dados. Isso contribuiu para a redução de custos de hardware, tornando os sistemas mais acessíveis e facilitando o desenvolvimento de sistemas comerciais e pessoais.

> **Nota:** A _pipelining_ (do conceito de _"linha de montagem"_) é uma técnica de execução paralela em que múltiplas instruções são processadas simultaneamente em diferentes _estágios_ do ciclo de execução para aumentar o desempenho da CPU. Isso difere do conceito geral de _paralelismo_, onde _tarefas completas_, e não estágios do processamento, são executadas simultaneamente.

### Gargalo de Von Neumann

O gargalo de Von Neumann é uma limitação de desempenho causada pelo fato de que a CPU e a memória compartilham o mesmo barramento para acessar instruções e dados.

Isso significa que:

- A CPU não pode buscar uma instrução e acessar dados ao mesmo tempo.
- O tempo de espera entre operações aumenta, especialmente em programas que exigem muitos acessos à memória.
- A largura de banda do barramento se torna um fator crítico para o desempenho.

Esse gargalo levou ao desenvolvimento de soluções como:

- Memória cache (para reduzir acessos à RAM)
- Execução paralela e _pipelines_
- Arquiteturas modificadas (como o modelo Harvard modificado)

> **Nota:** A _arquitetura Harvard modificada_ é uma variação do modelo de Von Neumann que usa memórias separadas internamente (como em caches) para dados e instruções, mantendo um espaço de memória unificado do ponto de vista do programador.

## Arquiteturas x86

A arquitetura x86 é uma família de conjuntos de instruções (ISA – _Instruction Set Architecture_) baseada no modelo de Von Neumann e desenvolvida originalmente pela Intel a partir do processador 8086. Todas as CPUs x86 implementam (ou emulam) o conjunto de instruções da Intel 8086 (16 bits), lançada em 1978.

Como uma evolução prática do modelo de Von Neumann, a arquitetura x86 implementa otimizações como:

- Uso extensivo de memória cache.
- Execução fora de ordem (_out-of-order execution_).
- _Pipelines_ e paralelismo interno.

### Características

- ISA complexa (CISC – _Complex Instruction Set Computing_), com centenas de instruções e modos de endereçamento.
- Suporte a múltiplos tamanhos de palavra (16, 32 e 64 bits nas versões modernas).
- Registradores de uso geral (AX, BX, CX, etc.) e segmentados (CS, DS, SS…, não utilizados no Linux), herdados de versões mais antigas.
- Ampla compatibilidade com versões anteriores (retrocompatibilidade).
- Utilizada em desktops, laptops e servidores, sendo a base da maioria dos PCs atuais.

> **Nota:** Embora a arquitetura x86 inclua registradores segmentados (como CS, DS, ES, SS), o Linux não faz uso da segmentação de memória no modo protegido. Em vez disso, ele utiliza o chamado _endereçamento plano_, tratando toda a memória como um único espaço contínuo. A segmentação é mantida apenas em um nível mínimo para atender exigências da arquitetura (como troca de contexto e proteção básica), mas a segmentação lógica, como era usada no MS-DOS, é totalmente evitada.

### Gerações da família x86:

| Processador    | Ano  | Registradores | Endereçamento                  | Modos de operação        | Destaques principais                                            |
| -------------- | ---- | ------------- | ------------------------------ | ------------------------ | --------------------------------------------------------------- |
| 8086           | 1978 | 16 bits       | 20 bits (1 MB)                 | Real mode                | Primeiro da linha x86, sem proteção ou multitarefa              |
| 80286          | 1982 | 16 bits       | 24 bits (16 MB)                | Real, Protected          | Introduziu o modo protegido e privilégio de acesso por segmento |
| 80386          | 1985 | 32 bits       | 32 bits (4 GB)                 | Real, Protected, Virtual | Registradores de 32 bits, suporte à paginação, multitarefa      |
| 80486          | 1989 | 32 bits       | 32 bits                        | Idem 80386               | Pipeline simples, cache L1, primeira versão com FPU integrada   |
| Pentium        | 1993 | 32 bits       | 32 bits                        | Idem 80486               | Pipeline duplo, superscalar, introdução de MMX                  |
| Pentium Pro    | 1995 | 32 bits       | 36 bits (PAE)                  | Idem                     | Execução fora de ordem, cache L2 on-die, suporte a PAE          |
| x86-64 (AMD64) | 2003 | 64 bits       | 48 bits virtuais (até 57 hoje) | Real, Protected, Long    | ISA de 64 bits, registradores expandidos, compatível com x86    |

A transição da arquitetura x86 de 32 para 64 bits foi liderada pela AMD com a criação da AMD64 em 2003. Essa extensão manteve compatibilidade com o conjunto de instruções x86 original (IA-32), mas adicionou registradores de 64 bits e suporte a endereçamento ampliado. Posteriormente, a Intel adotou essa mesma arquitetura sob o nome Intel 64 (anteriormente chamada EM64T), e o termo x86_64 passou a ser usado de forma genérica para se referir à arquitetura compatível com AMD64. Assim, o AMD64 é a origem técnica da arquitetura x86 de 64 bits utilizada na maioria dos sistemas modernos.

### Comparativo com outras arquiteturas

Todas as arquiteturas modernas seguem, em maior ou menor grau, os princípios do modelo de Von Neumann. No entanto, diferem na forma como organizam e executam instruções. Veja o comparativo:

**x86 (CISC – _Complex Instruction Set Computing_):**

- Conjunto de instruções extenso e complexo.
- Instruções de vários tamanhos e com múltiplos modos de endereçamento.
- Maior consumo de energia, mas com alta compatibilidade e maior desempenho bruto.
- Retrocompatibilidade com código legado.
- Comum em PCs, laptops e servidores.

**ARM (RISC – _Reduced Instruction Set Computing_)**

- Conjunto de instruções reduzido e regular.
- Foco em simplicidade, baixa energia e eficiência.
- Desempenho por watt muito superior ao x86.
- Predominante em dispositivos móveis (smartphones, tablets) e embarcados.
- ARM64 (AArch64) é a versão de 64 bits moderna.

**RISC-V (RISC e _open source_):**

- ISA aberta, modular e extensível.
- Sem royalties: qualquer um pode implementar.
- Design limpo e simples, adequado para pesquisa, educação e sistemas customizados.
- Crescimento em sistemas embarcados e processadores personalizados.

**Resumo comparativo:**

|Arquitetura|Tipo|Complexidade|Consumo|Uso comum|
|---|---|---|---|---|
|x86|CISC|Alta|Alto|PCs, laptops, servidores|
|ARM|RISC|Média|Baixo|Celulares, IoT, Apple M1+|
|RISC-V|RISC|Baixa|Baixo|Pesquisa, sistemas embarcados|

## Componentes de uma CPU x86_64

**Unidade de Controle (_Control Unit_ - CU)**
	Gerencia o fluxo de dados e as instruções dentro da CPU, coordenando as operações de execução.

**Unidade Lógica e Aritmética (ALU – _Arithmetic and Logic Unit_)**
	Responsável pela execução de operações aritméticas (soma, subtração, etc.) e lógicas (AND, OR, NOT, etc.).

**Registradores de uso geral**
	Utilizados para armazenar dados temporários durante a execução de instruções (ex.: RAX, RBX, RCX, etc.).

**Registradores de propósito específico**
	Como o ponteiro de pilha (RSP), ponteiro de instrução (RIP) e flags (como EFLAGS).

**Registradores de segmentos**
	Armazenam endereços de segmentos de memória para código (CS), dados (DS) e pilha (SS), além dos registradores de segmentos adicionais para dados (ES, FS e GS).

**Cache**
	Memória de acesso ultrarrápido usada para armazenar dados frequentemente acessados, visando reduzir o tempo de acesso à memória principal. Normalmente dividida em L1, L2 e, em algumas CPUs, L3.

**Barramentos**
	Conjunto de trilhas de comunicação que transportam dados entre a CPU e outros componentes, como a memória, dispositivos de entrada/saída e outros núcleos.

**Unidade de Execução (_Execution Unit_ – EU)**
	Responsável por executar as instruções. Em CPUs modernas, pode haver múltiplas unidades de execução para executar diferentes tipos de operações em paralelo.

**Decodificador de Instruções**
	Interpreta as instruções da linguagem de máquina (bytecode) e as converte para operações que podem ser executadas pela ALU ou outras unidades de execução.

**Unidade de carga e armazenamento (_Load-Store Unit_ - LSU)**
	Controla o carregamento e armazenamento de dados na memória, realizando operações de leitura e escrita.

**Unidade de Endereçamento (_Address Generation Unit_ - AGU)**
	Calcula endereços de memória, especialmente no contexto de operações de acesso à memória e cálculo de ponteiros.

**Controlador de Interrupções (_Interrupt Controller_)**
	Responsável por lidar com interrupções externas e internas, gerenciando a prioridade e o tratamento adequado das interrupções no sistema.

## Principais registradores e seus propósitos (64 bits)

- `RAX`: acumulador/propósito geral
- `RBX`: base/propósito geral
- `RCX`: contador/propósito geral
- `RDX`: dados/propósito geral
- `RSI`: índice de origem
- `RDI`: índice de destino
- `RSP`: ponteiro da pilha
- `RBP`: base da pilha
- `RIP`: ponteiro de instrução
- `RFLAGS`: sinalizações diversas

A arquitetura x86_64 ainda inclui oito registradores de propósito geral, de `R8` a `R15`.

### Relação com outras arquiteturas x86

Cada um dos registradores de 64 bits pode ser dividido em partes que, de certo modo, correspondem às capacidades de 32 e 16 bits de outras arquiteturas da família x86. Tomando o registrador `RAX` como exemplo:

```
┌────────────────────────────────────────────────────┐
│                       RAX                          │ 64 bits
└────────────────────────┬───────────────────────────┤
                         │           EAX             │ 32 bits
                         └────────────┬──────────────┤
                                      │      AX      │ 16 bits
                                      ├───────┬──────┤
                                      │  AH   │  AL  │
                                      └───────┴──────┘
```

> **Importante:** O diagrama não mostra registradores diferentes, mas os nomes pelos quais as subdivisões do registrador podem ser acessadas!

Aplicando a mesma ideia aos principais registradores, os nomes de suas subdivisões seriam:

| 64 bits | 32 bits                   | 16 bits | 8 bits "altos" | 8 bits "baixos" |
| ------- | ------------------------- | ------- | -------------- | --------------- |
| RAX     | EAX (Accumulator)         | AX      | AH             | AL              |
| RBX     | EBX (Base)                | BX      | BH             | BL              |
| RCX     | ECX (Counter)             | CX      | CH             | CL              |
| RDX     | EDX (Data)                | DX      | DH             | DL              |
| RSI     | ESI (Source Index)        | SI      | (sem acesso)   | SIL             |
| RDI     | EDI (Destination Index)   | DI      | (sem acesso)   | DIL             |
| RSP     | ESP (Stack Pointer)       | SP      | (sem acesso)   | SPL             |
| RBP     | EBP (Bse Pointer)         | BP      | (sem acesso)   | BPL             |
| RIP     | EIP (Instruction Pointer) | IP      | (sem acesso)   | (sem acesso)    |
| R8      | R8D                       | R8W     | (sem acesso)   | R8B             |
| …       | …                         | …       | …              | …               |
| R15     | R15D                      | R15W    | (sem acesso)   | R15B            |

> **Nota:** Assim como os registradores de R8 a R15, nem todas as subdivisões existiam nas arquiteturas de 32 e 16 bits da família x86, ou seja, elas só foram introduzidas na arquitetura x86_64.

## Primeiro exemplo em Assembly x86_64

Apenas para apresentar a aparência de um código em Assembly, aqui está o código de um programa que não faz nada além de terminar com o estado de término `42`…

Arquivo `exit42.asm`

```asm
; Retorna 42 como estado de término

section .text
    global _start

_start:
    mov     rax, 60        ; syscall: exit
    mov     rdi, 42        ; código de saída
    syscall
```

O programa é pequeno, mas nos dá a oportunidade de conhecer muitos dos elementos de um programa escrito com a linguagem Assembly.

### Seção do código executável

```asm
section .text
```

A diretiva `section` não é uma instrução da CPU, mas uma orientação sobre como o binário do programa deverá ser montado. No caso do exemplo, estamos dizendo ao montador que tudo que vier depois de `section`, até que se encontre outra definição de seção, deve ser montado na seção `.text` do binário. O nome `.text` é uma convenção que se refere à seção das instruções do programa em si (o seu código, essencialmente).

### O ponto de entrada

```asm
global _start
```

A diretiva `global` diz ao montador que um determinado _rótulo_ (como `_start`) deve ser visível fora do arquivo em que é escrito. Em outras palavras, o símbolo marcado com `global` pode ser referenciado por outros arquivos durante o processo de link-edição. No exemplo, `_start` é o rótulo que representa, por padrão, o endereço da primeira instrução a ser executada em um programa ou, como se costuma dizer, o seu _ponto de entrada_.

### Chamada de sistema

Nem todo programa em Assembly fará chamadas de sistema, que são funções internas do kernel para diversas finalidades que envolvam o acesso aos recursos do hardware ou controlados pelo sistema. No exemplo, porém, nós queremos utilizar a chamada de sistema `exit` para terminar a execução do programa retornando o valor `42` para o sistema operacional.

Para isso, nós temos que seguir as convenções de chamada, definidas pelo kernel para a arquitetura x86_64. Essas convenções estabelecem, por exemplo, quais registradores devem ser utilizados para receber a identificação da chamada de sistema e os argumentos que serão passados para ela.

No caso da chamada de sistema `exit`:

- O registrador `rax` deve receber o valor `60` (identificação da chamada `exit`).
- O registrador `rdi` deve receber o valor que será retornado como estado de término (`42`).

Sendo assim, nós utilizamos as instruções `mov` para carregar (mover) os valores esperados (argumentos) nos registradores apropriados:

```asm
mov     rax, 60        ; syscall: exit
mov     rdi, 42        ; código de saída
```

Em seguida, nós utilizamos a instrução `syscall` para informar à CPU que ela deveria invocar a chamada de sistema definida:

```asm
syscall
```

### Montagem e execução (no terminal)

```
:~$ nasm -f elf64 -o exit42.o exit42.asm
```

O resultado desse comando é a geração de um arquivo binário que nós chamamos de _objeto_ (com terminação `.o`). O arquivo objeto contém a tradução de todo o código em Assembly para código de máquina e já poderia, por exemplo, ser compilado com outros objetos para compor um programa completo, mas ainda não é capaz de ser executado.

Para isso, é necessário submeter o objeto a diversos procedimentos finais chamados de _link-edição_ – no caso, com o editor de ligações `ld`:

```
:~$ ld -o exit42 exit42.o
```

Assim, o editor de ligações `ld`:

- Resolve endereços e símbolos (como `_start`);
- Organiza o layout final do programa;
- Adiciona seções obrigatórias (como os cabeçalhos ELF);
- Produz um binário executável.

Com o binário final gerado, nós podemos executá-lo e testar seu estado de término com:

```
:~$ ./exit42
:~$ echo $?
42
```

Onde `$?` é a expansão do parâmetro especial do shell `?`, que registra o estado de término do último comando executado. Então, com o valor em `?` expandido, nós podemos imprimi-lo com o comando interno `echo`.

## Referências

- [Organização e Projeto de Computadores – David A. Patterson e John L. Hennesy](https://a.co/d/8Dm1Z93)
- [Organização Estruturada de Computadores – Andrew S. Tanenbaum](https://a.co/d/9QtcLPI)
- [Intel® 64 and IA-32 Instruction Set Reference](https://namazso.github.io/x86/)
- [OS Dev: CPU Registers x86-64](https://wiki.osdev.org/CPU_Registers_x86-64)
- [Linux System Call Table for x86 64](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)
- [Manual do Bash: Estado de Saída](https://www.gnu.org/software/bash/manual/bash.html#Exit-Status)
- `man 2 exit`