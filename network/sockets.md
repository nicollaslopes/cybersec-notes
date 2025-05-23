# Sockets

Em resumo bem resumido, pense em sockets literalmente como tomada da sua casa. Diferentes portas tendo diferentes padrões de buracos, como o padrão brasileiro sendo uma porta, padrão europeu sendo outra, padrão americano outra e assim vai. E só plugues corretos encaixam em determinadas tomadas. Em resumo bem resumido, pense em sockets literalmente como tomada da sua casa. Diferentes portas tendo diferentes padrões de buracos, como o padrão brasileiro sendo uma porta, padrão europeu sendo outra, padrão americano outra e assim vai. E só plugues corretos encaixam em determinadas tomadas. 

Sockets, na verdade, é uma forma de comunicação entre processos. Todo sistema operacional existe pra gerenciar processos. Processo é todo programa que tem rodando no seu computador, seja seu spotify ou banco de dados. No nível mais básico, um processo é representado por um número, um process ID ou PID que o sistema operacional dá pro seu programa. Quando do terminal de um Linux se digita `ps -ax` dá pra ver os processos e seus PIDs. O sistema operacional aloca uma certa quantidade de memória e internamente, seu processo acha que tem acesso a toda memória teórica de 64-bits, mas na realidade o sistema operacional é quem intermedia o acesso à RAM ou Swap de disco.

### Memória Protegida/Isolada Impede Comunicação

Isso é importante. Um processo é como uma criança rica mimada gastando cartão de crédito dos pais. Ele não tem noção que tem limite, e só vai gastando, e os pais, que é o sistema operacional, vai sempre só pagando, então essa criança tem a ilusão que tem dinheiro infinito. É mais ou menos assim. Seu PC tem só 8 giga de RAM real, mas seu processo acha que tem do endereço 0 até 2 elevado a 64 de memória, que seria o máximo teórico de 16 exabytes. Nenhum computador de verdade tem exabytes de RAM ainda. Isso é importante de entender então vou insistir com outra metáfora.

O sistema operacional intermedia e garante que um processo não consiga acessar a memória real inteira de outro processo. Por isso falamos que rodam isolados, sem um sobrescrever a memória do outro.



### Comunicação via pipes

Se um processo quiser se comunicar com outro processo, não tem como, porque não tem como gravar algo na memória que outro processo possa ler diretamente. As únicas duas formas de dois processos se comunicarem ou é via pipes ou via arquivos. Pipes de Linux é simplesmente ligar a saída padrão de um programa na entrada padrão de outro programa, o stdout ligado ao stdin via um pipe, ou cano. Que nem quando fazemos `cat algum_arquivo` pipe `grep alguma coisa`.

### Fluxo de conexão

Primeiro seu programa pede um bind pro sistema operacional. Bind é literalmente ligação, a ligação de um endereço IP com um outro número de 16 bits que chamamos de porta. Bind é um pedido de registro. Funciona assim. Todo processo quando inicia, o sistema operacional oferece um número de identificação, o tal PID. Só que esse PID muda o tempo todo. Hoje seu Spotify inicia com PID 10. Mas amanhã pode já ter outro programa com PID 10, então o sistema atribui PID 11 ou qualquer outro número. PIDs identificam programas rodando naquele momento, mas não servem pra identificar um programa individualmente a qualquer momento. Então precisamos de outro número que seja fixo que o programa possa pedir pra se registrar.

Digamos que seu programa peça pro sistema registrar ele com o número 3000. O sistema operacional mantém uma tabela em memória e poderia associar o PID atual do seu programa, digamos que seja 11, com esse número 3000, que ele chama de porta, ligados ou bound num endereço IP. Porta é um número que identifica um processo que o sistema operacional mantém numa tabela de controle na memória dele. Se ninguém antes pediu pra registrar essa porta, o sistema te associa nela e nenhum outro programa pode pedir pra se ligar na mesma porta. Porta é só isso, um identificador do programa rodando ligado ao endereço IP do computador.

Quando o sistema consegue te associar nessa porta 3000, devolve um ok pro programa e devolve o equivalente a um stream de bits, o cano aberto, como se tivesse acabado de conseguir abrir um arquivo. E agora seu programa pode entrar em modo de escuta, ou listen. Nesse momento ele fica bloqueado esperando o sistema te mandar alguma coisa por esse cano e não precisa fazer mais nada enquanto isso. Normalmente isso fica numa thread bloqueada ou é usado I/O assíncrono, que não é escopo explicar hoje. Só entenda que depois do bind, seu programa ficaria parado esperando e escutando o que vier através dessa porta 3000. O processo de bind e listen de sockets é similar ao processo de dar open e abrir um arquivo.