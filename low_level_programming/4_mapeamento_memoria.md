# Mapeamento de memória

## Objetivos

- Compreender como programas são executados no GNU/Linux.
- Descobrir como a execução de programas é gerenciada.
- Conhecer a organização do espaço de endereçamento de processos.
- Visualizar o layout de memória de programas reais.

## Como programas são executados

Como vimos, na criação de arquivos objeto no formato ELF, os dados e instruções são organizados em várias seções, o que inclui:

- O código do programa (seção `.text`)
- Dados globais constantes (seção `.rodata`)
- Dados globais variáveis inicializados (seção `.data`)
- Dados globais variáveis não inicializados (seção `.bss`)

Também vimos que arquivos objeto podem ser ligados dinâmica ou estaticamente entre si para formar um programa completo:

- **Ligação estática:** os objetos são vinculados para formar um único binário executável.
- **Ligação dinâmica:** um arquivo executável e os objetos compartilhados de que depende são carregados em um mesmo espaço de memória no momento da execução do programa.

Assim, quando um programa é executado, as seções de seu binário – bem como as seções de objetos compartilhados, se forem requeridos – devem ser mapeadas e organizadas em uma região da memória principal designada e gerenciada pelo sistema operacional. Cada programa em execução, portanto, terá seu próprio espaço isolado na memória onde seus dados e instruções serão organizados.

Contudo, a região da memória atribuída à execução do programa não contém apenas código e dados carregados do executável – e, eventualmente, de outros objetos compartilhados. Nela, também haverá estruturas adicionais, como a _pilha_, a _heap_ e outros dados de controle para o gerenciamento da execução. O conjunto individual de estruturas na memória associado à execução de um programa é o que nós chamamos de _processo_.

### O que são processos

Um processo pode ser entendido como uma entidade ativa no sistema operacional que representa a execução de um programa, incluindo sua identificação, os recursos a que tem acesso, seu contexto de execução, permissões e informações de gerenciamento. É por meio dos processos que o sistema operacional organiza e administra a execução simultânea de programas, garantindo que cada um deles funcione corretamente e de forma isolada dos demais.

Portanto, ao executar um programa, o sistema operacional:

- Cria um novo espaço de memória de uso exclusivo;
- Carrega as seções do executável e de objetos compartilhados (se necessário);
- Inicializa estruturas auxiliares (como _pilha_, _heap_ e dados de ambiente);
- Inicia a execução no ponto de entrada do programa;
- Monitora e controla suas atividades como um novo processo.

### Layout da memória virtual de um processo

Quando um programa é executado, ele não acessa diretamente a memória física (a memória RAM). Em vez disso, é designada para ele uma faixa contínua de endereços simbólicos, conhecida como _espaço de endereços virtuais_ ou, simplesmente, _memória virtual_. Assim, quando o programa acessa um endereço em seu espeço de memória virtual, o sistema, com auxílio do processador, traduz este acesso para o endereço correspondente na memória física.

O espaço de endereços virtuais é dividido em regiões específicas, organizadas de forma previsível, e o diagrama abaixo mostra uma representação de seu layout típico em sistemas GNU/Linux x86_64, com as seções dispostas do endereço mais alto para o mais baixo.

```
Endereços mais altos (ex: 0x7fffffffffff)
┌─────────────────────────────┐
│      ESPAÇO DO KERNEL       │ ← informações de controle interno do kernel.
├─────────────────────────────┤
│      VETOR DE AMBIENTE      │ ← Variáveis exportadas.
├─────────────────────────────┤
│     VETOR DE ARGUMENTOS     │ ← Argumentos de linha de comando.
├─────────────────────────────┤
│                             │
│        PILHA (STACK)        │
│                             │
├───────────── ▼ ─────────────┤
│                             │
│                             │ ← Espaço livre.
│                             │
├─────────────────────────────┤
│            VDSO             │ ← Assistente de chamadas de sistema.
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤
│            VVAR             │ ← Acesso a tempo real do kernel.
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤
│                             │
│            MMAP             │ ← Objetos compartilhados, arquivos mapeados,
│                             │   alocação com 'mmap', etc.
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┤
│                             │
│            HEAP             │ ← Memória alocada com 'malloc'.
│                             │
├─────────────────────────────┤ ─┐
│           .BSS              │  │
├─────────────────────────────┤  │
│           .DATA             │  │
├─────────────────────────────┤  │
│          .RODATA            │  ├─ Conteúdo mapeado do arquivo executável.
├─────────────────────────────┤  │
│           .TEXT             │  │
├─────────────────────────────┤  │
│        TABELAS ELF          │  │
└─────────────────────────────┘ ─┘
Endereços mais baixos (ex: 0x400000)
```

Onde, de cima para baixo, nós temos…

**Espaço do kernel**
	Informações internas do kernel para configuração da pilha, do ambiente de execução (argumentos e ambiente) e, em alguns casos, "trampolins de sinais". Esta região não pode ser acessada diretamente pelo programa.

**Vetor de ambiente**
	Vetor de strings no formato de atribuições de variáveis (`NOME=VALOR`) correspondendo às definições das variáveis exportadas para o processo.

**Vetor de argumentos**
	Também chamado de _vetor de parâmetros_, é um vetor de strings que contém, em ordem, as palavras da linha de comando usada para executar o programa. O primeiro elemento do vetor sempre existe e corresponde à primeira palavra da linha do comando – geralmente, o nome ou caminho do programa.

**Pilha (_stack_)**
	É uma estrutura dinâmica que _empilha_ blocos de dados conforme as funções do programa são chamadas, armazenando temporariamente informações como endereços de retorno e dados de variáveis locais. A cada nova chamada de função, um novo bloco (chamado de _quadro de pilha_) é empilhado. Quando a função termina, esse quadro é removido e a pilha volta ao seu estado anterior. Ao ser inicializada, a pilha recebe, do endereço mais alto para o mais baixo: os endereços dos elementos no vetor de ambiente, os endereços dos elementos do vetor de argumentos e, no topo da pilha, a quantidade de argumentos…

```
Endereço mais alto na pilha (sua base)
┌─────────────┐
│    NULL     │ ← Fim do vetor de ambiente
├──── ... ────┤
│   envp[1]   │
├─────────────┤
│   envp[0]   │ ← Definição da primeira variável exportada
├─────────────┤
│    NULL     │ ← Fim do vetor de argumentos
├──── ... ────┤
│   argv[1]   │
├─────────────┤
│   argv[0]   │ ← Nome/caminho do programa
├──── ... ────┤
│    argc     │ ← Quantidade de argumentos.
└─────────────┘
Endereço mais baixo da pilha (na direção de seu topo)
```

**Espaço livre entre pilha e endereços mapeados**
	Em versões mais modernas do GNU/Linux, a folga de segurança para o crescimento da pilha (_guard gap_) não é compartilhada com a região de dados alocados dinamicamente pelo programa (_heap_, de "amontoado"), como costumava ser. Atualmente, o sistema insere várias regiões mapeadas automaticamente com bibliotecas compartilhadas, inclusive do próprio sistema. Deste modo, a tradicional região do _heap_ deixa de ficar imediatamente abaixo da pilha no espaço de endereços virtuais.

**Região de mapeamento automático de bibliotecas**
	Atualmente, logo abaixo da folga de segurança, nós temos várias áreas mapeadas com objetos compartilhados e do sistema, como a biblioteca especial `linux-vdso.so.1` (que possibilita realizar chamadas de sistema sem utilizar interrupções), a biblioteca C padrão (`glibc`) e o carregador dinâmico (`ld-linux`). Essas bibliotecas são mapeadas automaticamente pelo sistema e podem variar de acordo com as dependências do programa e as configurações do ambiente de execução.

**Regiões mapeadas com 'mmap'**
	Áreas criadas com a chamada de sistema `mmap`, que possibilita mapear arquivos, carregar bibliotecas explicitamente e alocar de grandes blocos de memória.

**Região de alocação dinâmica de memória (_heap_)**
	É um grande espaço de endereços reservado para que o programa possa carregar e manipular dados dinamicamente – ou seja, dados que não foram associados previamente a variáveis ou vetores, o que geralmente é feito com funções da família `malloc`. Em termos gerais, diz-se que o _heap_ cresce no sentido dos endereços mais altos, a partir de um ponto inicial acima das regiões estáticas do programa. Contudo, novas alocações não seguem necessariamente um padrão linear, podendo ocupar regiões já liberadas ou até mesmo fora do espaço tradicional do _heap_, via `mmap`. De todo modo, o limite de crescimento do _heap_ está nos espaços superiores já ocupados com mapeamentos automáticos feitos pelo sistema.

**Conteúdo estático do arquivo executável**
	No início da faixa de endereços da memória virtual, ficam os dados carregados diretamente do arquivo executável, o que inclui as seções do binário e as tabelas de cabeçalhos e segmentos do formato ELF.

## Explorando os espaços de endereços de processos

Conhecendo o aspecto geral do layout de organização da memória virtual de processos em execução em sistemas GNU/Linux, é importante enfatizar que o conteúdo nesse espaço virtual de endereços pode variar em função de diversos fatores como, entre outros:

- O tipo do programa e seu grau de abstração;
- A necessidade, ou não, de carregar objetos compartilhados;
- A forma como a memória é alocada em tempo de execução;
- A versão do sistema operacional.

Sendo assim, a melhor forma de consolidar a compreensão sobre os espaços de memória é observando diretamente como eles são organizados durante a execução de programas reais, escritos em diferentes linguagens de programação e com diferentes níveis de abstração.

## Exemplo 1: mapeamento de memória de um programa em C

Arquivo: mappings.c

```c
#include <stdio.h>
#include <stdlib.h>

int global_var = 42;         // .data
static int static_var = 7;   // .data
int uninit_var;              // .bss
const int const_var = 99;    // .rodata

int main() {
    int local_var = 10;                  // stack
    static int static_main = 5;          // .data
    int *heap_var = malloc(sizeof(int)); // heap

    *heap_var = 123;

    printf("local_var  : %p : %d\n", &local_var, local_var);
    printf("*heap_var  : %p : %d\n", heap_var, *heap_var);
    printf("uninit_var : %p : %d\n", &uninit_var, uninit_var);
    printf("static_main: %p : %d\n", &static_main, static_main);
    printf("static_var : %p : %d\n", &static_var, static_var);
    printf("global_var : %p : %d\n", &global_var, global_var);
    printf("const_var  : %p : %d\n", &const_var, const_var);

    // Código da função main (.text)...
    printf("main()     : %p\n", &main);

    // Pausa para análise na linha de comandos...
    puts("Tecle algo para terminar...");
    getchar();
    
    free(heap_var);
    
    return 0;
}
```

Compilando e executando, nós termos algo como…

```
:~$ gcc -Wall mappings.c
:~$ ./a.out
local_var  : 0x7fff8803eb94 : 10
*heap_var  : 0x563f84c752a0 : 123
uninit_var : 0x563f63a47048 : 0
static_main: 0x563f63a47040 : 5
static_var : 0x563f63a4703c : 7
global_var : 0x563f63a47038 : 42
const_var  : 0x563f63a45004 : 99
main()     : 0x563f63a44179
Tecle algo para terminar...
```

Como podemos ver, a impressão foi planejada de modo a apresentar os endereços na ordem em que seriam mapeados na memória virtual, do mais alto para o mais baixo. Isso facilitará a continuação do experimento, que é a localização desses endereços nas faixas de mapeamento obtidas com o utilitário `pmap` e no arquivo `/proc/<pid>/maps`.

### Análise com 'pmap'

O utilitário `pmap` exibe o mapa de memória de um ou mais processos:

```
pmap [opções] PID [PID ...]
```

Por padrão, o utilitário exibe o mapeamento de forma simplificada (ideal para os nossos propósitos), mas outros detalhes podem ser exibidos com as opções listadas em:

```
pmap --help
```

Para utilizá-lo nos nossos experimentos, com o programa de exemplo ainda em execução, nós teremos que abrir outro terminal e executar:

```
$ pmap $(pidof a.out)
```

O resultado impresso deve ser algo assim:

```
2782565:   ./a.out
0000563f63a43000      4K r---- a.out
0000563f63a44000      4K r-x-- a.out
0000563f63a45000      4K r---- a.out
0000563f63a46000      4K r---- a.out
0000563f63a47000      4K rw--- a.out
0000563f84c75000    132K rw---   [ anôn ]
00007f6207f5f000     12K rw---   [ anôn ]
00007f6207f62000    160K r---- libc.so.6
00007f6207f8a000   1428K r-x-- libc.so.6
00007f62080ef000    344K r---- libc.so.6
00007f6208145000     16K r---- libc.so.6
00007f6208149000      8K rw--- libc.so.6
00007f620814b000     52K rw---   [ anôn ]
00007f620817b000      8K rw---   [ anôn ]
00007f620817d000     16K r----   [ anôn ]
00007f6208181000      8K r-x--   [ anôn ]
00007f6208183000      4K r---- ld-linux-x86-64.so.2
00007f6208184000    160K r-x-- ld-linux-x86-64.so.2
00007f62081ac000     44K r---- ld-linux-x86-64.so.2
00007f62081b7000      8K r---- ld-linux-x86-64.so.2
00007f62081b9000      4K rw--- ld-linux-x86-64.so.2
00007f62081ba000      4K rw---   [ anôn ]
00007fff88020000    132K rw---   [ pilha ]
 total             2560K
```

Na primeira linha…

```
2782565:   ./a.out
```

Nós temos o número do processo do nosso programa e o comando que invocou sua execução.

Nas linhas seguintes, as informações seguem este padrão:

```
ENDEREÇO_INICIAL  TAMANHO(BYTES)  PERMISSÕES  MAPEAMENTO
```

Finalmente, na última linha, nós temos o tamanho total da memória mapeada para o processo:

```
total             2560K
```

Nosso principal objetivo é comparar os endereços na saída do nosso programa com os endereços iniciais listados por `pmap` para localizar onde os dados e instruções foram posicionados na memória:

```
2782565:   ./a.out
0000563f63a43000      4K r---- a.out
0000563f63a44000      4K r-x-- a.out  <-------- main()
0000563f63a45000      4K r---- a.out  <-------- const_var
0000563f63a46000      4K r---- a.out  
0000563f63a47000      4K rw--- a.out  <-------- global_var até uninit_var
0000563f84c75000    132K rw---   [ anôn ]  <--- *heap_var 
00007f6207f5f000     12K rw---   [ anôn ]
00007f6207f62000    160K r---- libc.so.6
00007f6207f8a000   1428K r-x-- libc.so.6
00007f62080ef000    344K r---- libc.so.6
00007f6208145000     16K r---- libc.so.6
00007f6208149000      8K rw--- libc.so.6
00007f620814b000     52K rw---   [ anôn ]
00007f620817b000      8K rw---   [ anôn ]
00007f620817d000     16K r----   [ anôn ]
00007f6208181000      8K r-x--   [ anôn ]
00007f6208183000      4K r---- ld-linux-x86-64.so.2
00007f6208184000    160K r-x-- ld-linux-x86-64.so.2
00007f62081ac000     44K r---- ld-linux-x86-64.so.2
00007f62081b7000      8K r---- ld-linux-x86-64.so.2
00007f62081b9000      4K rw--- ld-linux-x86-64.so.2
00007f62081ba000      4K rw---   [ anôn ]
00007fff88020000    132K rw---   [ pilha ]  <-- local_var
 total             2560K
```

Note que a região iniciada em `0x563f63a47000` recebeu os dados de todas as variáveis globais e estáticas inicializadas e não inicializadas:

```
uninit_var : 0x563f63a47048 : 0
static_main: 0x563f63a47040 : 5
static_var : 0x563f63a4703c : 7
global_var : 0x563f63a47038 : 42
```

Isso mostra que, pelo menos neste caso, a seção `.bss` iniciou imediatamente após o fim da seção `.data` na mesma faixa de endereços – aqui, ocupando um total de 4kbytes:

```
0000563f63a47000      4K rw--- a.out  <-------- global_var até uninit_var
```

Nosso segundo objetivo é comparar o mapeamento com o layout típico da memória virtual de um processo. Portanto, nós temos…

#### Tabelas de cabeçalhos e seções ELF

A primeira parte do arquivo mapeada na memória é o conteúdo das tabelas de cabeçalhos e seções:

```
0000563f63a43000      4K r---- a.out
```

#### Seção `.text`

Em seguida, é mapeado o conteúdo da seção `.text` do arquivo:

```
0000563f63a44000      4K r-x-- a.out  <-- main()
```

Observe que esta região foi mapeada com permissões apenas para leitura e execução (`r-x--`) e é nela que estão as instruções da função `main` do nosso programa.

#### Seção `.rodata`

A seção seguinte a ser mapeada é `.rodata`, onde temos os dados que não poderão ser alterados:

```
0000563f63a45000      4K r---- a.out  <-- const_var
```

A região mapeada na memória, portanto, só tem permissão para leitura (`r`).

#### Seções `.data` e `.bss`

```
0000563f63a47000      4K rw--- a.out  <-- global_var até uninit_var
```

Com permissões de leitura e escrita, esta região de endereços recebe todos os dados globais e estáticos que poderão ser alterados ao longo da execução do programa.

#### Fim do mapeamento do arquivo executável

Até aqui, todos os dados mapeados correspondem ao conteúdo do arquivo executável do programa (`a.out`). Agora, porém, nós entraremos nas regiões dos endereços mapeados pelo sistema para viabilizar a sua execução como um processo.

#### Região de alocação dinâmica (heap)

```
0000563f84c75000    132K rw---   [ anôn ]  <--- *heap_var
```

#### Região de mapeamento automático e com 'mmap'

```
00007f6207f5f000     12K rw---   [ anôn ]
00007f6207f62000    160K r---- libc.so.6
00007f6207f8a000   1428K r-x-- libc.so.6
00007f62080ef000    344K r---- libc.so.6
00007f6208145000     16K r---- libc.so.6
00007f6208149000      8K rw--- libc.so.6
00007f620814b000     52K rw---   [ anôn ]
00007f620817b000      8K rw---   [ anôn ]
00007f620817d000     16K r----   [ anôn ]
00007f6208181000      8K r-x--   [ anôn ]
00007f6208183000      4K r---- ld-linux-x86-64.so.2
00007f6208184000    160K r-x-- ld-linux-x86-64.so.2
00007f62081ac000     44K r---- ld-linux-x86-64.so.2
00007f62081b7000      8K r---- ld-linux-x86-64.so.2
00007f62081b9000      4K rw--- ld-linux-x86-64.so.2
00007f62081ba000      4K rw---   [ anôn ]
```

Observe que é nesta região que o conteúdo da `glibc` (`libc.so.6`) e do carregador do sistema (`ld-linux-x86-64.so.2`) são mapeados.

#### Pilha (stack)

A partir dos endereços mais altos da memória virtual, nós temos a pilha do processo, onde são carregados os dados locais das funções à medida em que elas são chamadas, como é o caso da variável `local_var`.

```
00007fff88020000    132K rw---   [ pilha ]  <-- local_var
```
### Análise com o arquivo /proc/`<pid>`/maps

Nós também podemos analisar o mapeamento de memória do processo associado à execução do nosso programa pela mesma fonte de dados utilizada pelo `pmap`: o arquivo `/proc/<pid>/maps`. Para isso, com o nosso programa rodando, nós podemos executar:

```
:~$ cat /proc/$(pidof a.out)/maps
563f63a43000-563f63a44000 r--p 00000000 08:12 4856488    /home/blau/git/pbn/curso/exemplos/04/a.out
563f63a44000-563f63a45000 r-xp 00001000 08:12 4856488    /home/blau/git/pbn/curso/exemplos/04/a.out
563f63a45000-563f63a46000 r--p 00002000 08:12 4856488    /home/blau/git/pbn/curso/exemplos/04/a.out
563f63a46000-563f63a47000 r--p 00002000 08:12 4856488    /home/blau/git/pbn/curso/exemplos/04/a.out
563f63a47000-563f63a48000 rw-p 00003000 08:12 4856488    /home/blau/git/pbn/curso/exemplos/04/a.out
563f84c75000-563f84c96000 rw-p 00000000 00:00 0          [heap]
7f6207f5f000-7f6207f62000 rw-p 00000000 00:00 0 
7f6207f62000-7f6207f8a000 r--p 00000000 08:12 6584950    /usr/lib/x86_64-linux-gnu/libc.so.6
7f6207f8a000-7f62080ef000 r-xp 00028000 08:12 6584950    /usr/lib/x86_64-linux-gnu/libc.so.6
7f62080ef000-7f6208145000 r--p 0018d000 08:12 6584950    /usr/lib/x86_64-linux-gnu/libc.so.6
7f6208145000-7f6208149000 r--p 001e2000 08:12 6584950    /usr/lib/x86_64-linux-gnu/libc.so.6
7f6208149000-7f620814b000 rw-p 001e6000 08:12 6584950    /usr/lib/x86_64-linux-gnu/libc.so.6
7f620814b000-7f6208158000 rw-p 00000000 00:00 0 
7f620817b000-7f620817d000 rw-p 00000000 00:00 0 
7f620817d000-7f6208181000 r--p 00000000 00:00 0          [vvar]
7f6208181000-7f6208183000 r-xp 00000000 00:00 0          [vdso]
7f6208183000-7f6208184000 r--p 00000000 08:12 6584935    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7f6208184000-7f62081ac000 r-xp 00001000 08:12 6584935    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7f62081ac000-7f62081b7000 r--p 00029000 08:12 6584935    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7f62081b7000-7f62081b9000 r--p 00034000 08:12 6584935    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7f62081b9000-7f62081ba000 rw-p 00036000 08:12 6584935    /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
7f62081ba000-7f62081bb000 rw-p 00000000 00:00 0 
7fff88020000-7fff88041000 rw-p 00000000 00:00 0          [stack]
```

Neste arquivo, os dados são brutos e menos fáceis de analisar, mas nós podemos observar algumas informações que ficaram de fora da listagem simplificada do utilitário `pmap`. Por exemplo, aqui é possível visualizar onde as funções e variáveis auxiliares do kernel foram mapeadas:

```
7f620817d000-7f6208181000 r--p 00000000 00:00 0          [vvar]
7f6208181000-7f6208183000 r-xp 00000000 00:00 0          [vdso]
```

Na listagem do `pmap`, essas mesmas regiões aparecem como "anônimas", mas é possível ver seus tamanhos:

```
00007f620817d000     16K r----   [ anôn ]
00007f6208181000      8K r-x--   [ anôn ]
```

Outra região mais facilmente identificável no arquivo é o _heap_:

```
563f84c75000-563f84c96000 rw-p 00000000 00:00 0          [heap]
```

Listada pelo `pmap` apenas como:

```
0000563f84c75000    132K rw---   [ anôn ]
```

## Exemplo 2: mapeamento de memória de um programa em Assembly

Alocar dados no _heap_ (via chamadas de sistema `brk` ou `sbrk`) seria complexo demais para este momento dos nossos estudos. Imprimir os valores e endereços também exigiria um código muito maior e a antecipação de técnicas e instruções que não compensariam ser vistas agora, Por isso, vamos nos limitar à definição de dados.

Arquivo: `mappings.asm`

```asm
section .rodata
    msg db `Salve, simpatia!\n`
    len equ $ - msg
    
section .data
    global_data dq 42

section .bss
    uninit_data resq 1

section .text
    global _start

_start:
   ; Fazer algo com os dados em .data e .bss
    mov rax, [rel global_data]  ; acessando .data
    mov rcx, [rel uninit_data]  ; acessando .bss

    ; Imprimir a mensagem em .rodata
    mov rax, 1                  ; syscall: write
    mov rdi, 1                  ; fd 1 = stdout
    mov rsi, msg                ; endereço da mensagem
    mov rdx, len                ; tamanho da mensagem
    syscall
    
    ; Encerrar processo
    mov rax, 60         ; syscall: exit
    xor rdi, rdi        ; status 0
    syscall
```

Montando e executando o exemplo:

```
:~$ nasm -f elf64 -g mappings.asm
:~$ ld -o mappings mappings.o
:~$ ./mappings
Salve, simpatia!
```

### Análise com o GNU Debugger (GDB)

Note que a opção `-g` foi utilizada para incluir os símbolos de depuração do programa. Isso permite que a análise do mapeamento seja feita com o `gdb`, já que precisamos observar o processo em execução.

**Inicializando o GDB:**

```
:~$ gdb mappings
Reading symbols from mappings...
```

**Definindo um ponto de parada:**

```
(gdb) break _start
Breakpoint 1 at 0x401000: file mappings.asm, line 16.
```

**Executando o programa:**

```
(gdb) run
Starting program: /home/blau/git/pbn/curso/exemplos/04/mappings

Breakpoint 1, _start () at mappings.asm:16
16	    mov rax, [rel global_data]  ; acessando .data
```

**Listando o mapeamento de memória do processo:**

```
(gdb) info proc mappings
process 2793082
Mapped address spaces:

Start Addr         End Addr           Size         Offset     Perms File
0x0000000000400000 0x0000000000401000 0x1000       0x0        r--p  /home/blau/git/pbn/curso/exemplos/04/mappings
0x0000000000401000 0x0000000000402000 0x1000       0x1000     r-xp  /home/blau/git/pbn/curso/exemplos/04/mappings
0x0000000000402000 0x0000000000403000 0x1000       0x2000     r--p  /home/blau/git/pbn/curso/exemplos/04/mappings
0x0000000000403000 0x0000000000404000 0x1000       0x2000     rw-p  /home/blau/git/pbn/curso/exemplos/04/mappings
0x00007ffff7ff9000 0x00007ffff7ffd000 0x4000       0x0        r--p  [vvar]
0x00007ffff7ffd000 0x00007ffff7fff000 0x2000       0x0        r-xp  [vdso]
0x00007ffffffde000 0x00007ffffffff000 0x21000      0x0        rw-p  [stack]
```

Observe que a listagem segue o estilo do arquivo `/proc/<pid>/maps`, que, como o programa está em execução, nós também podemos exibir em outro terminal com:

```
:~$ cat /proc/$(pidof mappings)/maps
00400000-00401000 r--p 00000000 08:12 4856707        /home/blau/git/pbn/curso/exemplos/04/mappings
00401000-00402000 r-xp 00001000 08:12 4856707        /home/blau/git/pbn/curso/exemplos/04/mappings
00402000-00403000 r--p 00002000 08:12 4856707        /home/blau/git/pbn/curso/exemplos/04/mappings
00403000-00404000 rw-p 00002000 08:12 4856707        /home/blau/git/pbn/curso/exemplos/04/mappings
7ffff7ff9000-7ffff7ffd000 r--p 00000000 00:00 0      [vvar]
7ffff7ffd000-7ffff7fff000 r-xp 00000000 00:00 0      [vdso]
7ffffffde000-7ffffffff000 rw-p 00000000 00:00 0      [stack]
```

Ou com o `pmap`:

```
:~$ pmap $(pidof mappings)
2793082:   /home/blau/git/pbn/curso/exemplos/04/mappings
0000000000400000      4K r---- mappings
0000000000401000      4K r-x-- mappings
0000000000402000      4K r---- mappings
0000000000403000      4K rw--- mappings
00007ffff7ff9000     16K r----   [ anôn ]
00007ffff7ffd000      8K r-x--   [ anôn ]
00007ffffffde000    132K rw---   [ pilha ]
 total              172K
```

De qualquer forma, as regiões do layout de memória do processo são facilmente localizadas:

```
0000000000400000      4K r---- mappings      <-- Tabelas ELF
0000000000401000      4K r-x-- mappings      <-- .text
0000000000402000      4K r---- mappings      <-- .rodata
0000000000403000      4K rw--- mappings      <-- .data .bss
00007ffff7ff9000     16K r----   [ anôn ]    <-- vvar
00007ffff7ffd000      8K r-x--   [ anôn ]    <-- vdso
00007ffffffde000    132K rw---   [ pilha ]   <-- pilha
```

#### Localizando o dado em `.rodata`

O dado iniciado no rótulo `msg` tem 17 bytes, portanto:

```
(gdb) x /17bx &msg
0x402000 <msg>:	0x53	0x61	0x6c	0x76	0x65	0x2c	0x20	0x73
0x402008:	0x69	0x6d	0x70	0x61	0x74	0x69	0x61	0x21
0x402010:	0x0a
```

Como podemos ver, o endereço inicial dos dados no rótulo `msg` é `0x402000`, que corresponde ao mapeamento da seção `.rodata`:

```
0000000000402000      4K r---- mappings      <-- .rodata
```

#### Localizando os dados em `.data` e `.bss`

O dado no rótulo `global_data` foi inicializado com o valor 42 (`0x2c`, em hexa) e ocupa o espaço de uma _quad-word_ (8 bytes), que podemos exibir assim:

```
(gdb) x /8bx &global_data
0x403014 <global_data>:	0x2a	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

Localizando seu endereço (`0x403014`) no mapeamento, nós vemos que os 8 bytes foram escritos na região da seção compartilhada por `.data` e `.bss`:

```
0000000000403000      4K rw--- mappings      <-- .data .bss
```

O mesmo ocorre com o espaço de 8 bytes preenchidos com zeros reservados no endereço identificado pelo rótulo `uninit_data`:

```
(gdb) x /8bx &uninit_data
0x40301c <uninit_data>:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

Repare, também, que o dado em `uninit_data` inicia exatamente 8 bytes após o dado em `global_data`:

```
:~$ echo $((0x40301c - 0x403014))
8
```

Novamente demonstrando que as seções `.data` e `.bss` são vizinhas.

### Executáveis independentes de posição (PIE)

Uma coisa que você deve ter notado, é a diferença de distância entre o mapeamento do executável gerado com `nasm+ld` em relação aos mapeamentos do sistema, se comparado com o mapeamento do exemplo em C:

```
0x563f63a43000 --> 0x7fff88020000 : Programa compilado com gcc
0x000000400000 --> 0x7ffffffde000 : Programa montado com o nasm+ld
```

Isso acontece porque, por padrão, o `gcc` produz arquivos executáveis do tipo PIE (_Position-Independent Executable_) que são programas que podem ser carregados em qualquer lugar da memória, pois seu código não depende de endereços fixos.

Isso permite que o sistema operacional utilize técnicas para aumentar a segurança da memória (como ASLR, para randomizar o layout de memória), dificultando explorações de falhas como, por exemplo, de _buffer overflow_.

Programas em Assembly também podem gerar executáveis PIE, mas isso depende de que o código seja escrito com vistas à sua montagem e link-edição para este tipo de executável. Por enquanto, é mais fácil demonstrar as diferenças de arquivos PIE e não-PIE compilando o exemplo em C com a opção `-no-pie`:

```
:~$ gcc -Wall mappings.c
:~$ ./a.out
local_var  : 0x7fff26577c94 : 10
*heap_var  : 0x13aa92a0 : 123
uninit_var : 0x404048 : 0
static_main: 0x404040 : 5
static_var : 0x40403c : 7
global_var : 0x404038 : 42
const_var  : 0x402004 : 99
main()     : 0x401166
Tecle algo para terminar...
```

Isso mostra como os endereços mapeados se parecem mais com os que vimos com o exemplo em Assembly.

## Exercícios propostos

1. Adicione um vetor global de 10 inteiros no exemplo em C e verifique onde seus elementos são mapeados.
2. Ainda no exemplo em C, investigue o efeito do qualificador `volatile` no mapeamento de variáveis.
3. Modifique o código em C para usar `mmap` em vez de `malloc` e veja se há mudanças quanto a região mapeada.
4. No exemplo em Assembly, mova as definições de `msg` e `len` para a seção `.data` e investigue o que se altera.

## Referências

- `man pmap`
- `man proc_pid_maps`
- `man 3 maloc`
- `man 2 mmap`
- `man 2 brk`
- [Mel Gorman: Understanding The Linux Virtual Memory Manager](https://www.kernel.org/doc/gorman/pdf/understand.pdf)
- [Linux: Memory Management Documentation](https://docs.kernel.org/mm/)