ARQUITETURA DE COMPUTADOORES. 

Assunto de hoje.
- Assembly
- Compilacao. 
- Compilacao de C pra aseembly
- COmo assembly funciona(labels)
- Traducao do assembly
- Objeto gerado da traducao.
- Como funciiona a compunicacao e referenciamento das memorias, como L1s. 
- ... -> Linker(prog especifico(nao é do SO) para fazer isso, PLINKER outros) ->.exe
- .exe é totalmente binario

- 



MIPS

- maquina de 32 bits
- a gente nao vai usar todas as instrucoes do mIPS. 
- uma vez ele usou e disse q tinha 102 intrucoes. Mas vamos usar somente 10. 
- No livro os caras tambem usam muito menos de 102. 
- A propria empresa que fez o MIPS fez o ARQ
- Quais sao as instrucoes. Instrucao de atribuicao, desvio, chamada de procedimento. 
- Chama de procedimento, system call, chamar um progrmaa, uma funcao, etc. Call Return. 
-Instrucoes de atribuicao, por exemplo, a = a +1. a = b + c. Colocar em a o valor que esta em uma posicao de memoria X. Ou oclocar na momeria o registrador a. Atribuicoes. Registradores... subtracao tbm, funcoes logias, ou e end, formas de atribuicao. 
- Desvio, mais importante talvez juntamente com atribuicao. Instrucao de desvio, por que é tao importante? O desvio muda a posicao do "pc" o pc da depois da instrucao que esta snedo executada.Mas por que é imporatne. Porque sem o desvio vira uma maquina de calcula o computador. Capacidade de if ou else por exemplo, sao essas posibilidades de decisoes, capacidade de desviar envitar ou fazer. While, if, else. Desvio condicional, incondicional sao os que nao tem desvio nenhum, com ooperacoes matematicas simpels, ou ate tratamento de execoes que existentes no sistema. Se tiver uma pane por exemplo, precisa se gerar o desvio da execao para tratar isso. COndicional por exemplo, podemos ver a condicao flag.zero ou o flag."vai um", ou um flag."overflow", se o bit mais significativo é 0 ou 1, processor status word, varios flags que a maquina possui, PSW. Tem outros tbm, como o trap.flag. o controle que se passa para o sistema operacional, ou usar um periferico teria uma trap para ir e voltar para um user. Outro tbm, modified.flag, exclusive.flag, shared.flag, excluded.flag. Antigamente tinha uns 8 ou 12, hoje tem uns 30 tem flag que eu nem sei pra quer serve. Se voce fez uma operacao e o resultado deu 0 ou 1, vai determinar o flag respectivamente. Flags, ou mudar o valor do pc, mudando nas posicoes, tudo isso é logica de programacao, nao é decisao de SO.

instrucoes aratmeticas e logicas, vamos falar sobre.

vamos usar somente 5 instrucoes. 

add, sub, and, or, slt.

slt é set less than, ou seja, se o valor de um registrador for menor que o outro, ele seta o valor do registrador destino para 1, caso contrario seta para 0. COMPARACAO DE VALORES, COMPARACAO DE REGISTRADORES. Masvamos ver isso soenten qunado formos estudar for loops, while loops, if else, switch case, etc.

qual é a caracteristicas, elass chamam de tipo-r. 
instrucoes de tipo-r sao instrucoes que operam com registradores, ou seja, os operandos estao nos registradores. A instrucao vai pegar o valor de dois registradores, fazer a operacao e colocar o resultado em um terceiro registrador. Por exemplo, add $t0, $t1, $t2 significa que vai somar o valor do registrador $t1 com o valor do registrador $t2 e colocar o resultado no registrador $t0.

outro exemplo. 
add A, B, C

Nesse caso C é o Rt, B é o Rs e A é o Rd.
Rd = registrador destino
Rs = registrador fonte
Rt = registrador fonte

s e t sao fonte? sim, s e t sao registradores fonte, o que significa que eles fornecem os valores para a operacao. O registrador destino (Rd) é onde o resultado da operacao sera armazenado.

Imagine uma porta logica que recebe dois sinais de entrada (Rs e Rt) e produz um sinal de saída (Rd) com base na operacao especificada (add, sub, and, or, slt).

esse formato sempre. Entao como seria a subtracao? seria sub A, B, C. A = B - C. O mesmo formato se aplica para as outras operacoes logicas (and, or) e para a instrucao slt.

aristetica imediata, bremente é uma instrucao que utiliza um valor constante (imediato) em vez de um registrador como um dos operandos. Por exemplo, addi A, B, 5 significa que o valor do registrador B será somado com o valor imediato 5 e o resultado será armazenado no registrador A.

Em termos de arquitetura é somente setar um bit diferente na instrucao para indicar que é uma operacao imediata. O formato da instrucao imediata é diferente do formato tipo-r, pois inclui o valor imediato em vez de um registrador fonte.



o tipo r, o r veio de onde?

O "R" em "tipo-R" vem de "Register", que significa registrador em inglês. As instruções do tipo-R são aquelas que operam diretamente com os registradores da CPU, utilizando-os como operandos para realizar operações aritméticas e lógicas. Essas instruções são fundamentais na arquitetura MIPS, pois permitem manipular dados de forma eficiente dentro do processador, sem a necessidade de acessar a memória principal para cada operação.


agora.

opcode, que é o campo de operação da instrução, é um conjunto de bits que especifica qual operação a CPU deve realizar. No caso das instruções do tipo-R, o campo op geralmente é definido como 0, e a operação específica é determinada por outro campo chamado funct (função). O campo funct indica a operação exata a ser executada, como adição, subtração, AND, OR, etc.

opcode tem 6 bits. como ele consegue identificar 64 instrucoes diferentes? Porque ele tem 6 bits, e com 6 bits podemos representar 2^6 = 64 combinações diferentes. Cada combinação pode ser mapeada para uma instrução específica, permitindo que a CPU reconheça e execute uma variedade de operações.

ai eles criaram um campo ai de campo de funcao. 
nesse campo de funcao, que também tem 6 bits, eles podem especificar a operação exata a ser realizada dentro do conjunto de instruções do tipo-R. Isso significa que, mesmo que o opcode seja o mesmo (por exemplo, 0 para todas as instruções do tipo-R), o campo funct permite diferenciar entre operações como add, sub, and, or, etc.

No computador em a unidade de controle e o caminho de dados. A Unidade de controle é o que vai direcionar, detrerminar os componentes, liverando somente semaforors, sinais de controle mesmo. Se a opracao for uma de acesso a memoria por exemplo, a unidade de controle vai liberar o semaforo de acesso a memoria, liberando o barramento de dados para que a memoria possa ser acessada. Se for uma operacao aritmetica, a unidade de controle vai liberar os semaforos do ALU (Unidade Aritmetica e Logica) para que a operacao seja realizada. A unidade de controle é responsável por coordenar todas as operações dentro do processador, garantindo que cada componente funcione corretamente e no momento certo.


Nesse caso do MIPS, pra resovler isso ai, precisamos de coisas amais. 

o opcode vai para o uc. os 6 bits do opcode são enviados para a unidade de controle (UC), que interpreta esses bits e determina qual operação deve ser realizada. A UC então gera sinais de controle apropriados para os outros componentes do processador, como a ALU, registradores e memória, garantindo que a operação seja executada corretamente.


agora tem uma outra entrada de bits para para a UC, que é o campo funct. O campo funct, que também possui 6 bits, é usado para especificar a operação exata a ser realizada dentro do conjunto de instruções do tipo-R. A UC utiliza tanto o opcode quanto o funct para determinar a operação específica que deve ser executada pela ALU.

"olha é para voce executar tal operacao, mas qual exatamente? ai o funct vai dizer qual operacao especifica voce vai executar. Ai a UC vai liberar os semaforos da ALU para que ela execute a operacao especifica determinada pelo funct."


Utilizando o endereco base do registrador, a UC pode acessar os registradores corretos para obter os operandos necessários para a operação. O endereço base indica qual registrador contém o valor que será usado na operação, permitindo que a ALU realize cálculos precisos com os dados corretos. Entao tem certas operacoes que nao precisa do campo de funct, como por exemplo, a instrucao de desvio incondicional. Nesse caso, o opcode é suficiente para determinar a operação a ser realizada, e o campo funct não é necessário.



o opcode o que é exatamente, ele é um campo de bits dentro da instrução que indica à CPU qual operação deve ser executada. Ele serve como um identificador para a operação específica que a instrução representa. No caso das instruções do tipo-R, o opcode geralmente é definido como 0, e a operação exata é determinada pelo campo funct. Em outras palavras, o opcode informa à CPU que se trata de uma instrução do tipo-R, enquanto o campo funct especifica qual operação aritmética ou lógica deve ser realizada. Mas onde o opcode fica exatamente? O opcode é um campo de bits localizado na parte inicial da instrução, geralmente nos primeiros 6 bits. A posição exata do opcode pode variar dependendo do formato da instrução (tipo-R, tipo-I, tipo-J), mas em geral, ele ocupa os primeiros 6 bits da instrução de 32 bits no MIPS. Entao sobram 26 bits para outros campos da instrução, como registradores fonte e destino, valor imediato, endereço de salto, etc. A estrutura da instrução é projetada para fornecer todas as informações necessárias para a execução correta da operação especificada pelo opcode e, se aplicável, pelo campo funct.

Cada operacao vai ter seu opcodes e funcoes especificas. Por exemplo, a operação de adição (add) terá um opcode específico e um campo funct correspondente que indica que a operação a ser realizada é a adição. Da mesma forma, outras operações como subtração (sub), AND, OR e SLT terão seus próprios opcodes e campos funct distintos, permitindo que a CPU diferencie entre as várias instruções do tipo-R e execute a operação correta com base nos valores dos registradores fornecidos.


tabela da verdade.

faltou a instrucao de acesso a memoria, como funciona? Para falar disso realemnte precisamos falar de organizacao de computadores, que é um assunto mais complexo. A instrução de acesso à memória envolve a leitura e escrita de dados na memória principal, e isso requer uma compreensão mais profunda da arquitetura do computador, incluindo o gerenciamento de memória, hierarquia de memória (como caches L1, L2), e como os endereços de memória são calculados e acessados.

Instrucoes de acesso a memoria. Vamos usar umas 10 a 15 instrucoes de acesso a memoria.
Temos entao a CPU, que nela tem um bocado de registradores la dentro, o ultimo geralmente sendo o pc. Temos a ULA, Cache, RAM, tudo isso dentro da CPU. A CPU vai acessar a memoria, que pode ser a RAM, ou a memoria cache, que é mais rapida. A CPU vai acessar a memoria para ler ou escrever dados, dependendo da instrução de acesso à memória que está sendo executada. As instruções de acesso à memória geralmente envolvem o uso de registradores para armazenar endereços de memória e valores a serem lidos ou escritos.

Observe que as instruceos de acesso a memoria podem ser classificadas em dois tipos principais: load (carregar) e store (armazenar). As instruções de load são usadas para ler dados da memória e colocá-los em um registrador, enquanto as instruções de store são usadas para escrever dados de um registrador de volta para a memória.

O que precisamos fazer. Vamos supor que temos algumas palavras de memoria, como uma de 4 bytes, ou seja, 31 bits. Nesse exemplo, estamos trabalhando com registradores com diferentes tipos de memorias diferentes. Vamos conhecer alguns, t0, t1, t2, t3, t4, t5, t6, t7, s0, s1, s2, s3, s4, s5, s6, s7. Esses registradores são usados para armazenar temporariamente valores durante a execução de programas. Preciso pegar um dado, por exemplo, uma operacao de soma, e armazenar o resultado em um registrador. Para isso, posso usar uma instrução de load para carregar um valor da memória para um registrador, realizar a operação desejada (como adição) usando os registradores, e depois usar uma instrução de store para escrever o resultado de volta na memória.

Mas ai quero fazer uma outra operacao, adicionar de novo, depois de uma adicao ja feita, salvando o resultado em outro registrador. Para isso, posso usar outra instrução de load para carregar o valor da memória para um registrador, realizar a operação de adição com os valores nos registradores, e então usar uma instrução de store para salvar o resultado de volta na memória.

No estados unidos esta perguntanddo e chamando programadores de baixo nivel, cmo sistemas embarcados, baixo nivel mesmo, cobol. 

Entao temos um store, que ele pode armazenar 1 byte, 2 bytes e 4 bytes. A instrução de store é usada para escrever dados de um registrador de volta para a memória. Dependendo do tamanho do dado que você deseja armazenar, você pode usar diferentes variantes da instrução de store, como store byte (sb), store halfword (sh), e store word (sw). Cada uma dessas instruções permite armazenar diferentes tamanhos de dados na memória, garantindo que os valores sejam corretamente escritos no local apropriado.

Assim como essa mesma terminacao pode ser usado para o load, que é a instrução de carregar dados da memória para um registrador. As variantes da instrução de load incluem load byte (lb), load halfword (lh), e load word (lw), permitindo que você leia diferentes tamanhos de dados da memória e os armazene nos registradores correspondentes.

Quando ele le byte a byte ele chama isso de store byte, quando ele le 2 bytes ele chama de store halfword, e quando ele le 4 bytes ele chama de store word. O mesmo vale para o load, load byte, load halfword, load word. Essas instruções são essenciais para manipular dados na memória de forma eficiente, permitindo que os programas acessem e modifiquem informações conforme necessário durante a execução. 

Por exemplo, em um exemplo com hexadecimal, se você tiver um valor armazenado na memória em um endereço específico, você pode usar a instrução load word (lw) para carregar esse valor de 4 bytes para um registrador. Se você quiser apenas um byte desse valor, você pode usar a instrução load byte (lb) para carregar apenas o byte desejado para o registrador. Da mesma forma, ao armazenar dados de volta na memória, você pode escolher a instrução apropriada (sb, sh, sw) com base no tamanho do dado que deseja escrever. Mas nao tem como fazer algo fora dessas 1, 2 ou 4 bytes. A memoria é organizada em palavras de 4 bytes, e as instrucoes de load e store sao projetadas para trabalhar com esses tamanhos padrao. Se voce precisar manipular dados de tamanhos diferentes, voce precisara usar combinacoes dessas instrucoes ou realizar manipulacoes adicionais nos registradores para obter o resultado desejado.

E como é que funciona o endereçamento da memoria? A memoria é organizada em endereços, onde cada endereço corresponde a uma posição específica na memória. No MIPS, os endereços de memória são geralmente representados em bytes, e cada palavra de memória (4 bytes) ocupa 4 endereços consecutivos. Por exemplo, se um valor está armazenado no endereço 0x1000, os próximos 3 bytes estariam nos endereços 0x1001, 0x1002 e 0x1003. 

Precisamos ter cuidado se o coomputador é big endian ou little endian, pois isso afeta a forma como os bytes são armazenados e acessados na memória. Em sistemas big-endian, o byte mais significativo é armazenado no menor endereço, enquanto em sistemas little-endian, o byte menos significativo é armazenado no menor endereço. Isso é importante ao trabalhar com instruções de load e store, pois você precisa garantir que está acessando os bytes corretos na ordem correta.

o mais significativo fica na direita para little endian, e o mais significativo fica na esquerda para big endian. 

Deslocamento de bits, ou shift, é uma operação que move os bits de um valor para a esquerda ou para a direita. No MIPS, existem instruções específicas para realizar deslocamentos, como sll (shift left logical) e srl (shift right logical). Essas instruções permitem manipular os bits de um registrador, deslocando-os para a esquerda ou para a direita, o que pode ser útil para operações aritméticas, multiplicações e divisões por potências de dois, entre outras aplicações.

shift nao é para multiplicacao e divisao, mas sim para deslocamento de bits. Por exemplo, um deslocamento para a esquerda (sll) de um valor binário efetivamente multiplica o valor por 2 para cada posição deslocada, enquanto um deslocamento para a direita (srl) efetivamente divide o valor por 2 para cada posição deslocada. No entanto, essas operações são realizadas no nível de bits e não são equivalentes a multiplicação ou divisão aritmética direta.


assim como shift nao é um read, shift nao é um write, shift nao é um load, shift nao é um store. Shift é uma operacao logica de deslocamento de bits. 


Agora se eu quiser ler uma palavra em um "final"eu vou precisar ler todos os bytes que compoem essa palavra, ou seja, 4 bytes. Se eu quiser ler uma palavra de 4 bytes, preciso garantir que estou acessando os endereços corretos na memória e lendo todos os bytes que compõem essa palavra. Isso é importante para garantir que os dados sejam lidos corretamente e armazenados no registrador de destino.

o que seria o halfword? Halfword é uma unidade de dados que consiste em 2 bytes (16 bits). No contexto do MIPS, um halfword ocupa dois endereços consecutivos na memória. As instruções de load e store para halfwords permitem que você leia ou escreva 2 bytes de dados na memória, em vez de 1 byte (byte) ou 4 bytes (word).

------------------------------------------
PROXIMA AULA
------------------------------------------

Hoje, repertorio de instrucoes de linguagem assembly. Instrucoes basicas. 

Arquitetura de uma maquina RISC. MIPS é uma arquitetura RISC (Reduced Instruction Set Computer), que significa "Computador com Conjunto Reduzido de Instruções". Isso implica que a arquitetura MIPS utiliza um conjunto limitado de instruções, cada uma projetada para ser executada rapidamente e de forma eficiente. A filosofia RISC enfatiza a simplicidade das instruções, permitindo que cada instrução seja executada em um único ciclo de clock, o que resulta em melhor desempenho geral do processador.


Tende a ser mais simples possivel, pense no hardwarel. Aquilo nao vai ser um software, vai ser literalmente uma trilha de fios. Entao quanto mais simples for a instrução, mais simples vai ser o hardware. E quanto mais simples for o hardware, mais rápido ele vai ser. E quanto mais rápido ele for, mais barato ele vai ser. E quanto mais barato ele for, mais pessoas vão comprar. E quanto mais pessoas comprarem, mais dinheiro a empresa vai ganhar. E quanto mais dinheiro a empresa ganhar, mais ela vai investir em pesquisa e desenvolvimento. E quanto mais ela investir em pesquisa e desenvolvimento, mais ela vai melhorar a arquitetura. E quanto mais ela melhorar a arquitetura, mais pessoas vão comprar. E quanto mais pessoas comprarem, mais dinheiro a empresa vai ganhar. E quanto mais dinheiro a empresa ganhar, mais ela vai investir em pesquisa e desenvolvimento. E quanto mais ela investir em pesquisa e desenvolvimento, mais ela vai melhorar a arquitetura. E quanto mais ela melhorar a arquitetura, mais pessoas vão comprar. E quanto mais pessoas comprarem, mais dinheiro a empresa vai ganhar. E quanto mais dinheiro a empresa ganhar, mais ela vai investir em pesquisa e desenvolvimento. E quanto mais ela investir em pesquisa e desenvolvimento, mais ela vai melhorar a arquitetura. E quanto mais ela melhorar a arquitetura, mais pessoas vão comprar. E quanto mais pessoas comprarem, mais dinheiro a empresa vai ganhar. E quanto mais dinheiro a empresa ganhar, mais ela vai investir em pesquisa e desenvolvimento. E quanto mais ela investir em pesquisa e desenvolvimento, mais ela vai melhorar a arquitetura. E quanto mais ela melhorar a arquitetura, mais pessoas vão comprar. E quanto mais pessoas comprarem, mais dinheiro a empresa vai ganhar. E quanto mais dinheiro a empresa ganhar, mais ela vai investir em pesquisa e desenvolvimento. E quanto mais ela investir em pesquisa e desenvolvimento, mais ela vai melhorar a arquitetura. E quanto mais ela melhorar a arquitetura, menos instruções vão existir.

Diferentes formatos, mas todas as intrucoes tem 32 bits. 

Mais facil para paralelizar. "torne o caso comum mais rapido". Por exemplo, bottom neck seria nesse contexto, a instrução mais comum que é executada com mais frequência. Ao otimizar o desempenho dessa instrução específica, o processador pode melhorar significativamente o desempenho geral do sistema. Isso é alcançado através de técnicas como pipelining, onde várias instruções são sobrepostas em execução, permitindo que o processador execute múltiplas instruções simultaneamente e reduza o tempo de execução total.

Variaveis sao registradores, seria o que vimos aula passada. Variaveis sao registradores, e o que nao for variavel, vai ser uma constante. A constante vai ser um valor imediato, que é um valor fixo codificado diretamente na instrução. Por exemplo, em uma instrução de adição, você pode ter um registrador que contém uma variável e um valor imediato que representa uma constante a ser adicionada a essa variável. O uso de valores imediatos permite que certas operações sejam realizadas sem a necessidade de acessar a memória para obter os operandos, tornando a execução mais rápida e eficiente.

Lembrando, so vamos usar 5 instrucoes, add, sub, and, or, slt. TIPOR

Instrucoes de acesso a memoria, load e store. Load e store sao instrucoes de acesso a memoria. Load carrega da memoria para o registrador, store armazena do registrador para a memoria. Load e store sao instrucoes de acesso a memoria. Load carrega da memoria para o registrador, store armazena do registrador para a memoria. Load e store sao instrucoes de acesso a memoria. Load carrega da memoria para o registrador, store armazena do registrador para a memoria. Load e store sao instrucoes de acesso a memoria. Load carrega da memoria para o registrador, store armazena do registrador para a memoria. Load e store sao instrucoes de acesso a memoria. Load carrega da memoria para o registrador, store armazena do registrador para a memoria. Load e store sao instrucoes de acesso a memoria. Load carrega da memoria para o registrador, store armazena do registrador para a memoria. Load e store sao instrucoes de acesso a memoria. Load carrega da memoria para o registrador, store armazena do registrador para a memoria.

Agora lembrando, tempo a sh, sb e sw. Cada uma dessas instrucoes de store tem um tamanho especifico de dados que ela manipula. A instrucao sb (store byte) armazena 1 byte de dados na memoria, a instrucao sh (store halfword) armazena 2 bytes de dados, e a instrucao sw (store word) armazena 4 bytes de dados. Da mesma forma, as instrucoes de load correspondentes (lb, lh, lw) carregam os mesmos tamanhos de dados da memoria para os registradores.


Para que serve o registrador pc, em portugues, "apontador de programa". O registrador pc (program counter) é responsável por armazenar o endereço da próxima instrução a ser executada pelo processador. Ele é atualizado automaticamente após cada instrução, garantindo que o fluxo de execução do programa siga a sequência correta. Em caso de desvios ou chamadas de função, o valor do pc pode ser alterado para apontar para um endereço diferente, permitindo que o processador execute instruções fora da sequência linear.


O registrador pc é crucial para o controle do fluxo de execução do programa, pois ele determina qual instrução será buscada e executada a seguir. Quando uma instrução é executada, o pc é incrementado para apontar para a próxima instrução na memória. No caso de instruções de desvio ou chamadas de função, o pc pode ser modificado para apontar para um endereço específico, permitindo que o processador altere o fluxo de execução conforme necessário.

Agora temos o sp ele aponta para o topo da pilha, que é uma área de memória usada para armazenar informações temporárias, como variáveis locais e endereços de retorno de funções. O registrador sp (stack pointer) é atualizado conforme os dados são empilhados ou desempilhados, garantindo que o acesso à pilha seja eficiente e organizado.

Tem um que nao usamos muito, o At (assembler temporary), que é um registrador temporário usado pelo montador (assembler) para armazenar valores intermediários durante a tradução de código assembly para código de máquina. Ele é geralmente utilizado para operações internas do montador e não é destinado ao uso direto por programas de usuário.

Tem tambem registradores de uso geral, como os registradores t0 a t9 (temporários) e s0 a s7 (especificos, salvos), que são usados para armazenar valores temporários e variáveis durante a execução de programas. Os registradores temporários (t0 a t9) são usados para armazenar valores que não precisam ser preservados entre chamadas de função, enquanto os registradores salvos (s0 a s7) são usados para armazenar valores que devem ser preservados entre chamadas de função. Se umprograma for usar qualquer s, ele precisa salvar o valor antes de chamar uma função e restaurar o valor depois que a função retornar, garantindo que os dados importantes não sejam perdidos durante a execução do programa. O banco de registradores é sempre o mesmo, independente do programa que esta sendo executado. O banco de registradores é uma parte fundamental da arquitetura do processador, fornecendo armazenamento rápido e eficiente para dados temporários e variáveis durante a execução de programas. Ele é projetado para ser acessado rapidamente pela CPU, permitindo que as operações sejam realizadas de forma eficiente sem a necessidade de acessar a memória principal com frequência.


Registradores tambem $a 0 a $a3, que sao usados para passar argumentos para funções. Quando uma função é chamada, os argumentos são colocados nesses registradores, permitindo que a função acesse os valores necessários para sua execução. Se houver mais de quatro argumentos, os adicionais são passados na pilha.Tbm tem as do output elas sao $v0 e $v1, que são usadas para armazenar valores de retorno de funções. Quando uma função retorna um valor, ele é colocado nesses registradores, permitindo que o chamador acesse o resultado da função. Se a função retornar mais de um valor, os valores adicionais podem ser passados em outros registradores ou na pilha, dependendo da convenção de chamada utilizada.



Mas por que o 4 parametros? Bem, isso é uma convenção de chamada de função na arquitetura MIPS. A escolha de quatro registradores para passar argumentos é baseada em um equilíbrio entre eficiência e simplicidade. Usar registradores para os primeiros quatro argumentos permite acesso rápido e eficiente aos valores necessários para a execução da função, enquanto argumentos adicionais podem ser passados na pilha, garantindo que a função tenha acesso a todos os dados necessários sem sobrecarregar o conjunto de registradores disponíveis. Essa abordagem ajuda a otimizar o desempenho do programa, minimizando o tempo gasto em operações de acesso à memória e mantendo a simplicidade na implementação das funções. Qual a diferenca de usar a pilha para usar o $a0 para o primeiro elemento e o $a1 para o tamanho do vetor? A diferença está na convenção de passagem de parâmetros e na eficiência do acesso aos dados. Usar registradores como $a0 e $a1 para passar o primeiro elemento e o tamanho do vetor permite acesso rápido e direto aos valores necessários para a execução da função, sem a necessidade de acessar a memória. Isso resulta em melhor desempenho, especialmente em funções que são chamadas com frequência. Entao ninguem usa a pilha? Na verdade, a pilha ainda é usada, mas principalmente para armazenar argumentos adicionais quando há mais de quatro parâmetros, bem como para salvar o estado do registrador antes de chamar uma função e restaurá-lo após a função retornar. A pilha também é utilizada para armazenar variáveis locais e endereços de retorno, garantindo que o fluxo de execução do programa seja mantido corretamente. Portanto, enquanto os registradores são preferidos para os primeiros quatro argumentos devido à sua eficiência, a pilha continua sendo uma parte essencial da arquitetura MIPS para gerenciar dados adicionais e manter a integridade do programa durante chamadas de função. Agora para usar uma pilha na pratica seria como? Para usar uma pilha na prática em MIPS, você precisa manipular o registrador de ponteiro de pilha (sp) para empilhar e desempilhar dados. Aqui está um exemplo básico de como você pode fazer isso:
1. Inicialize o registrador sp para apontar para o topo da pilha. Normalmente, a pilha cresce para baixo na memória, então você começaria com o sp apontando para um endereço alto.
2. Para empilhar (push) um valor, você decrementa o sp para criar espaço na pilha e, em seguida, armazena o valor no endereço apontado pelo sp. Por exemplo:
   ```
   addi $sp, $sp, -4  # Decrementa o sp para criar espaço
   sw $t0, 0($sp)     # Armazena o valor de $t0 na pilha
   ```


3. Para desempilhar (pop) um valor, você lê o valor do endereço apontado pelo sp e, em seguida, incrementa o sp para remover o valor da pilha. Por exemplo:
   ```
    lw $t0, 0($sp)     # Lê o valor da pilha para $t0
    addi $sp, $sp, 4   # Incrementa o sp para remover o valor da pilha

agora tem uma coisa é certa, nao precisamos usar a pilha para passar parametr, segundo o professor realmente nao se ustiliza, utilizamos esse modo com o a0 como primeiro elemento e o a1 como tamanho do vetor. A pilha é mais comumente usada para armazenar variáveis locais e salvar o estado dos registradores antes de chamar uma função, garantindo que os dados importantes não sejam perdidos durante a execução do programa. No entanto, para passar parâmetros, especialmente quando há apenas alguns argumentos, é mais eficiente usar os registradores $a0 a $a3, pois isso permite acesso rápido e direto aos valores necessários para a execução da função.


Agora um exemplo mais espexcifico o check_sum, ele vai receber um vetor de inteiros e o tamanho do vetor, e vai calcular a soma de todos os elementos do vetor. O primeiro elemento do vetor será passado no registrador $a0, e o tamanho do vetor será passado no registrador $a1. A função check_sum irá iterar sobre os elementos do vetor, somando-os e retornando o resultado final. Usa uma tabela para armazenar os elementos do vetor na memória, e a função irá acessar esses elementos usando o registrador $a0 como base para o endereço do primeiro elemento e o registrador $a1 para determinar quantos elementos devem ser somados. O resultado da soma será retornado no registrador $v0, que é usado para armazenar valores de retorno de funções em MIPS.


Registradores sao da CPU. Ou seja, eles são parte do hardware do processador e são usados para armazenar dados temporários durante a execução de programas. Eles são extremamente rápidos em comparação com a memória principal, permitindo que o processador acesse e manipule dados de forma eficiente. Os registradores são essenciais para o desempenho do processador, pois permitem operações rápidas sem a necessidade de acessar a memória principal com frequência.

Lembrar que o hexadecimal nesse caso é apenas uma representação dos valores binários armazenados nos registradores. Cada registrador armazena dados em formato binário, mas para facilitar a leitura e compreensão, os valores podem ser representados em hexadecimal. Por exemplo, um valor binário de 32 bits pode ser representado como um número hexadecimal de 8 dígitos, tornando mais fácil visualizar e trabalhar com os dados armazenados nos registradores durante a programação em assembly.

endereco 0 tem um restart, curiosidade, o que é isso? O endereço 0 na memória é frequentemente reservado para fins especiais, como reinicialização do sistema ou tratamento de exceções. Em muitos sistemas, o endereço 0 é usado como um ponto de entrada para o código de inicialização do sistema, permitindo que o processador execute instruções específicas ao ligar ou reiniciar. Além disso, o endereço 0 pode ser utilizado para armazenar informações críticas do sistema ou para lidar com interrupções e exceções, garantindo que o processador possa responder adequadamente a eventos inesperados durante a execução do programa.


Vamos revisar hexadecimal:
1. Hexadecimal é um sistema de numeração base 16, que utiliza os dígitos de 0 a 9 e as letras A a F para representar valores. Cada dígito hexadecimal representa 4 bits (ou meio byte), tornando-o uma forma compacta e eficiente de representar valores binários.
2. Conversão entre binário e hexadecimal: Para converter um número binário para hexadecimal, você pode agrupar os bits em conjuntos de 4, começando da direita para a esquerda, e então substituir cada grupo pelo dígito hexadecimal correspondente. Por exemplo, o número binário 11010110 pode ser agrupado como 1101 0110, que corresponde aos dígitos hexadecimais D6.
3. Conversão entre hexadecimal e decimal: Para converter um número hexadecimal para decimal, você pode multiplicar cada dígito pelo valor da base (16) elevado à posição do dígito, somando os resultados. Por exemplo, o número hexadecimal 1A3 pode ser convertido para decimal como (1 * 16^2) + (10 * 16^1) + (3 * 16^0) = 256 + 160 + 3 = 419.
4. Conversão entre decimal e hexadecimal: Para converter um número decimal para hexadecimal, você pode dividir o número por 16 repetidamente, registrando os restos em cada divisão. Os restos correspondem aos dígitos hexadecimais, que podem ser lidos de baixo para cima para formar o número hexadecimal final. Por exemplo, para converter o número decimal 255 para hexadecimal, você divide 255 por 16, obtendo um quociente de 15 e um resto de 15 (F em hexadecimal). O próximo quociente é 0, então o número hexadecimal é FF.
5. Utilização do hexadecimal em programação: O hexadecimal é amplamente utilizado em programação e desenvolvimento de software, especialmente em linguagens de baixo nível, como assembly e C. Ele é frequentemente usado para representar endereços de memória, valores binários compactos e cores em gráficos digitais. A representação hexadecimal facilita a leitura e compreensão dos dados, tornando-a uma ferramenta valiosa para programadores e engenheiros de software.

Logo, o hexadecimal é uma forma eficiente e prática de representar valores binários, sendo amplamente utilizado em programação e desenvolvimento de software, especialmente em contextos de baixo nível, como assembly e manipulação direta de memória. Ele permite uma visualização mais clara e compacta dos dados, facilitando a compreensão e o trabalho com valores binários complexos.

O que é o multipllexador? Um multiplexador (MUX) é um dispositivo eletrônico que seleciona uma das várias entradas de dados e a direciona para uma única saída. Ele funciona como um "comutador" que permite escolher qual entrada será transmitida para a saída com base em sinais de controle. Em termos de arquitetura de computadores, os multiplexadores são usados para gerenciar o fluxo de dados entre diferentes componentes, como registradores, memória e unidades de processamento, permitindo que o processador selecione e utilize os dados corretos conforme necessário durante a execução das instruções.


Exemplo de um lw com o registradores, na verdade o rt é o registrador de destino, que é o registrador onde o valor carregado da memória será armazenado. O registrador rt é especificado na instrução lw (load word) e indica qual registrador deve receber o valor lido da memória. Por exemplo, em uma instrução lw $t0, 0($t1), o registrador $t0 é o rt, e ele receberá o valor armazenado no endereço de memória calculado a partir do conteúdo do registrador $t1 mais o deslocamento de 0.


O multiplex é umcomponente extremamente util na arquitetura de computadores, pois permite que a CPU selecione entre diferentes fontes de dados e direcione o fluxo de informações de maneira eficiente. Ele é frequentemente usado em conjunto com a unidade de controle para garantir que os dados corretos sejam acessados e processados durante a execução das instruções, contribuindo para o desempenho geral do sistema.

Tanto que se pensamos nos 4 fios de controle, eles podem ser usados para controlar o multiplexador, determinando qual entrada será selecionada para a saída. Por exemplo, se temos duas entradas de dados e queremos escolher entre elas com base em um sinal de controle, podemos usar os fios de controle para indicar qual entrada deve ser direcionada para a saída do multiplexador. Isso permite que a CPU gerencie eficientemente o fluxo de dados entre diferentes componentes, garantindo que as operações sejam realizadas corretamente e no momento certo.


No caso de pensar em um rs, rt, e rd , o multiplexador pode ser usado para selecionar qual registrador será usado como fonte de dados para a operação. Por exemplo, se temos duas entradas de registradores (rs e rt) e queremos escolher qual delas será usada como operando para a ALU, podemos usar um multiplexador controlado por sinais da unidade de controle para direcionar o valor correto para a ALU. Isso permite que a CPU execute operações aritméticas e lógicas com os dados corretos, garantindo que as instruções sejam processadas de maneira eficiente e precisa.

Tem tambem um multiplex de 8 entradas, que pode ser usado para selecionar entre diferentes fontes de dados, como registradores, memória e valores imediatos. O multiplexador de 8 entradas permite que a CPU escolha entre várias opções de entrada com base em sinais de controle, garantindo que os dados corretos sejam direcionados para a saída desejada. Isso é especialmente útil em arquiteturas complexas, onde múltiplas fontes de dados podem estar disponíveis para uma operação específica.

Se pensamos no sentido de outros tipos de operacao como a de store , o multiplexador pode ser usado para selecionar qual registrador será usado como fonte de dados para a operação de armazenamento. Por exemplo, se temos várias entradas de registradores e queremos escolher qual delas será usada para armazenar um valor na memória, podemos usar um multiplexador controlado por sinais da unidade de controle para direcionar o valor correto para a operação de store. Isso permite que a CPU gerencie eficientemente o fluxo de dados entre os registradores e a memória, garantindo que as operações de armazenamento sejam realizadas corretamente e no momento certo. Ou seja, o multiplexador desempenha um papel crucial na arquitetura de computadores, permitindo que a CPU selecione entre diferentes fontes de dados e direcione o fluxo de informações de maneira eficiente. Ele é usado em conjunto com a unidade de controle para garantir que os dados corretos sejam acessados e processados durante a execução das instruções, contribuindo para o desempenho geral do sistema. De maneira pratica o rt é o registrador de destino, que recebe o valor carregado da memória na instrução lw. O multiplexador pode ser usado para selecionar entre diferentes registradores de destino, garantindo que o valor correto seja armazenado no registrador apropriado após a execução da instrução. Isso permite que a CPU gerencie eficientemente o fluxo de dados entre os registradores e a memória, garantindo que as operações sejam realizadas corretamente e no momento certo.


Dispositivo FPGA sigla FPGA significa "Field-Programmable Gate Array", que em português pode ser traduzido como "Matriz de Portas Programáveis em Campo". Um FPGA é um dispositivo eletrônico que pode ser programado após a fabricação para realizar funções lógicas específicas. Ele consiste em uma matriz de blocos lógicos configuráveis, interconexões programáveis e recursos de entrada/saída, permitindo que os engenheiros projetem circuitos digitais personalizados para atender a requisitos específicos. Como que programamos esse hardware? Mudando os valores do circuitos, aqueles fios que ligam os componentes, mudando a forma como eles se conectam. A programação de um FPGA é feita através de linguagens de descrição de hardware (HDL), como VHDL ou Verilog, que permitem aos engenheiros definir o comportamento e a estrutura do circuito digital. Ao escrever o código HDL, os engenheiros especificam como os blocos lógicos devem ser configurados e interconectados para realizar as funções desejadas. Em seguida, o código é sintetizado e carregado no FPGA, configurando os blocos lógicos e as interconexões de acordo com o projeto definido pelo engenheiro. Isso permite que o FPGA execute tarefas específicas de forma eficiente, tornando-o uma ferramenta poderosa para prototipagem e desenvolvimento de sistemas digitais personalizados.

Existem mais de um rd,rs,rt? Sim, em uma arquitetura de processador como a MIPS, existem múltiplos registradores que podem ser usados como rd (registrador de destino), rs (registrador fonte) e rt (registrador fonte). Cada instrução pode especificar diferentes registradores para essas funções, permitindo que o processador execute operações com diferentes conjuntos de dados. Por exemplo, em uma instrução de adição, você pode ter:

```
add $t0, $t1, $t2
```


FF em decimal é 255. O valor hexadecimal FF representa o número decimal 255, pois cada dígito hexadecimal corresponde a 4 bits, e FF em binário é 11111111, que é igual a 255 em decimal.

lwi significa "load word immediate", que é uma instrução usada para carregar um valor imediato (constante) de 32 bits diretamente em um registrador. Essa instrução permite que você inicialize um registrador com um valor específico sem precisar acessar a memória, tornando a operação mais rápida e eficiente. Por exemplo, a instrução lwi $t0, 0xFF00 carregaria o valor hexadecimal FF00 diretamente no registrador $t0.









vetor é na memoria, nao no registrador. Nos vamos usar o little endian. 

nao podemos ter uma instrucao com 4 variaveis, com 4 registradores. Precismos dividir em mais intrucoes. 

(C) g = h + [i]
(MIPS)
add $t1, $s4, $s4 // $t1 <- 2 * i
add $t1, $t1, $t1 // $t1 <- 4 * i
add $t1, $t1, $s3 // $t1 <- A[i] = (4*i + $s3)
lw $t0, 0($t1) // $t0 <- A[i]
add $s1, $s2, $t0 // g <- h + A[i]

