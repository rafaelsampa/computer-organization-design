# Arquitetura de Computadores - Aula 5

## Instruções Básicas: A Linguagem de Máquina (MIPS)

Guia de estudo organizado por tópico a partir dos slides da aula. Foco da prova: programação MIPS é o assunto mais pesado, mas os conceitos de organização (toolchain, formatos de instrução, mapa de memória) também caem.

---

## 1. Linguagem Assembly

- Representação simbólica da codificação binária de um processador.
- Permite usar **labels** para nomear posições de memória que guardam instruções ou dados.
- **Assembler (montador)**: ferramenta que traduz da linguagem de montagem para binário.

Características:
- Mais primitiva que linguagens de alto nível (sem fluxo de controle sofisticado).
- Útil quando memória ou velocidade são críticos, ou para explorar recursos de hardware não acessíveis em alto nível.
- Muito restritiva (ex.: as instruções aritméticas do MIPS).
- O repertório do MIPS é similar a arquiteturas dos anos 80, usadas por NEC, Nintendo, Silicon Graphics e Sony.

**Objetivo do projetista da máquina:** projetar um conjunto de instruções que facilite construir o hardware e o compilador, maximize a performance, reduza custos e simplifique o equipamento.

**Objetivo da disciplina:** estudar o conjunto de instruções de um processador simples; mostrar a relação entre repertório de instruções, hardware e linguagem de alto nível; entender o conceito de programa armazenado; e ver as implementações monociclo, multiciclo e pipeline.

---

## 2. Princípios de Projeto RISC

1. A simplicidade é favorecida pela regularidade.
2. Quanto menor, mais rápido.
3. Um bom projeto demanda compromisso.
4. Torne o caso comum mais rápido.

---

## 3. Operações Aritméticas

- Toda instrução aritmética tem **exatamente 3 operandos** (simplifica o hardware).
- Cada linha contém no máximo uma instrução.

```mips
add a, b, c    # a = b + c
sub a, b, c    # a = b - c
```

**Compilação de expressão complexa** `f = (g + h) - (i + j)`:

```mips
add $t0, $s1, $s2    # $t0 = g + h
add $t1, $s3, $s4    # $t1 = i + j
sub $s0, $t0, $t1    # f = $t0 - $t1
```

---

## 4. Registradores

- Memórias rápidas e internas ao processador (ligado ao princípio 2: quanto menor, mais rápido).
- Variáveis são associadas a registradores: `$t0`, `$t1`, ... `$s0`, `$s1`, ... (número limitado).
- **Quantidade: 32 registradores.** Mais registradores aumentaria o ciclo de clock e deixaria a máquina mais lenta.
- **Tamanho: 32 bits por registrador** (uma palavra).

---

## 5. Acesso à Memória

Estruturas de dados complexas (arrays) ficam em posições de memória. Para operar sobre elas é preciso trazê-las para registradores.

**Load (carregar da memória para registrador):**

```mips
lw $t0, desl($s2)    # $t0 <- memória[$s2 + desl]
#   reg destino   reg base
```

Compilação de `g = h + A[8]`:

```mips
lw  $t0, 8($s3)      # $t0 <- A[8]
add $s1, $s2, $t0    # g = h + A[8]
```

**Store (armazenar de registrador para memória):**

```mips
sw $t0, desl($s2)    # memória[$s2 + desl] <- $t0
#   reg fonte    reg base
```

Compilação de `A[12] = h + A[8]`:

```mips
lw  $t0, 32($s3)     # $t0 <- A[8]
add $t0, $s2, $t0    # $t0 = h + A[8]
sw  $t0, 48($s3)     # A[12] <- resultado
```

### Interface Hardware / Software

O compilador:
- Associa variáveis (software) a registradores (hardware).
- Determina o endereço de arrays na memória.

Regras de endereçamento de memória:
- **Alinhamento:** endereços de palavras são múltiplos de 4.
- **Ordem dos bytes na palavra:** *big endian* ou *little endian*.
- **Indexação de arrays:** `endereço = (4 x índice) + conteúdo do registrador base`.

### Array com índice variável

Compilação de `g = h + A[i]` (i em `$s4`, base em `$s3`):

```mips
add $t1, $s4, $s4    # $t1 = 2 x i
add $t1, $t1, $t1    # $t1 = 4 x i
add $t1, $t1, $s3    # $t1 = endereço de A[i]
lw  $t0, 0($t1)      # $t0 <- A[i]
add $s1, $s2, $t0    # g = h + A[i]
```

**Spilling (derramamento):** processo de colocar na memória as variáveis menos usadas, quando o número de variáveis do programa é maior que o número de registradores.

---

## 6. Tamanho x Velocidade

O acesso à memória é mais lento que o acesso a registradores. Logo, os registradores do MIPS gastam menos tempo para acessar operandos e têm throughput mais alto que a memória.

- **Tempo de resposta (tempo de execução):** tempo entre o início e o fim da execução do programa.
- **Throughput:** quantidade total de trabalho executado num intervalo de tempo (ex.: bytes por segundo ou acessos a disco por segundo).

---

## 7. Conceito de Programa Armazenado

Princípios básicos da construção de computadores:
1. As instruções são representadas como se fossem números.
2. Os programas devem ser armazenados na memória antes de serem executados.

Consequência: o computador deixa de ser uma simples calculadora e vira máquina de escrever, editor de som ou de imagem, dependendo do programa que estiver na memória naquele instante, sob gerência do sistema operacional.

---

## 8. Representação de Caracteres

| Código | Bits | Observação |
|---|---|---|
| BCD (Binary Coded Decimal) | 6 | Usado pela IBM; só maiúsculas e alguns símbolos |
| EBCDIC | 8 | 256 símbolos, equipamentos IBM (S/370) |
| ASCII | 8 | 7 bits para símbolos + 1 bit de paridade; ASCII estendido usa os 8 bits para 256 símbolos |
| UNICODE | 16 | 65.536 símbolos; padrão mundial atual |

Notas:
- Alguns caracteres ASCII são de **controle** (controlam impressão ou procedimentos de comunicação).
- `011XXXX` no ASCII representa os dígitos 0 a 9 em BCD.

### Tabela ASCII

Bits altos (b7 b6 b5) nas colunas, bits baixos (b4 b3 b2 b1) nas linhas.

| b4b3b2b1 | 000 | 001 | 010 | 011 | 100 | 101 | 110 | 111 |
|---|---|---|---|---|---|---|---|---|
| 0000 | NULL | DLE | SP | 0 | @ | P | ` | p |
| 0001 | SOH | DC1 | ! | 1 | A | Q | a | q |
| 0010 | STX | DC2 | " | 2 | B | R | b | r |
| 0011 | ETX | DC3 | # | 3 | C | S | c | s |
| 0100 | EOT | DC4 | $ | 4 | D | T | d | t |
| 0101 | ENQ | NAK | % | 5 | E | U | e | u |
| 0110 | ACK | SYN | & | 6 | F | V | f | v |
| 0111 | BEL | ETB | ' | 7 | G | W | g | w |
| 1000 | BS | CAN | ( | 8 | H | X | h | x |
| 1001 | HT | EM | ) | 9 | I | Y | i | y |
| 1010 | LF | SUB | * | : | J | Z | j | z |
| 1011 | VT | ESC | + | ; | K | [ | k | { |
| 1100 | FF | FS | , | < | L | \ | l | \| |
| 1101 | CR | GS | - | = | M | ] | m | } |
| 1110 | SO | RS | . | > | N | ^ | n | ~ |
| 1111 | SI | US | / | ? | O | _ | o | DEL |

---

## 9. Representação das Instruções (Formatos)

As instruções são divididas em **campos** que viram números. Os registradores são mapeados em números:

- `$t0` a `$t7` -> 8 a 15
- `$s0` a `$s7` -> 16 a 23

**Exemplo de tradução** `add $t0, $s1, $s2`:

| campo | op | rs | rt | rd | shamt | funct |
|---|---|---|---|---|---|---|
| decimal | 0 | 17 | 18 | 8 | 0 | 32 |
| binário | 000000 | 10001 | 10010 | 01000 | 00000 | 100000 |

Note a ordem: `rs` e `rt` são as fontes, `rd` é o destino. Em `add $t0, $s1, $s2`, o destino `$t0` (8) aparece no campo `rd`, e as fontes `$s1` (17) e `$s2` (18) em `rs` e `rt`.

### Formato R (tipo R): aritméticas e lógicas

| op | rs | rt | rd | shamt | funct |
|---|---|---|---|---|---|
| 6 bits | 5 bits | 5 bits | 5 bits | 5 bits | 6 bits |

- **op**: código da operação (opcode).
- **rs**: registrador do 1º operando fonte.
- **rt**: registrador do 2º operando fonte.
- **rd**: registrador destino (resultado).
- **shamt**: quantidade de bits a deslocar.
- **funct**: código da função (para instruções aritméticas).

### Formato I (tipo I): imediatas e transferência de dados

| op | rs | rt | endereço / constante |
|---|---|---|---|
| 6 bits | 5 bits | 5 bits | 16 bits |

- **rt**: registrador do 2º operando fonte ou destino.
- **endereço/constante**: valor de deslocamento ou operando imediato.
- Com 16 bits de endereço: faixa de -2^15 a +2^15 bytes, ou -2^13 a +2^13 palavras.

### Compromisso de projeto (princípio 3)

O compromisso é entre **formato** (mesmos campos) e **tamanho** (instruções de 32 bits). A escolha do MIPS: manter o mesmo tamanho (32 bits) mas com campos diferentes conforme o tipo.

- Tipo R -> instruções aritméticas.
- Tipo I -> instruções de transferência de dados (`lw`, `sw`, `addi`, `ori`, ...).

**Exemplo completo** `A[300] = h + A[300]` (base em `$t1`, h em `$s2`):

```mips
lw  $t0, 1200($t1)   # $t0 <- A[300]    (300 x 4 = 1200)
add $t0, $s2, $t0    # $t0 = h + A[300]
sw  $t0, 1200($t1)   # A[300] <- resultado
```

| instrução | op | rs | rt | rd | shamt | funct |
|---|---|---|---|---|---|---|
| lw | 35 | 9 | 8 | \- | \- | 1200 (endereço) |
| add | 0 | 18 | 8 | 8 | 0 | 32 |
| sw | 43 | 9 | 8 | \- | \- | 1200 (endereço) |

---

## 10. Instruções de Desvio (Controle de Fluxo)

**Desvios condicionais:**

```mips
beq registrador1, registrador2, L1    # desvia se igual
bne registrador1, registrador2, L1    # desvia se diferente
```

### If simples

`if (i == j) goto L1; f = g + h; L1: f = f - i;` (f,g,h,i,j em `$s0`..`$s4`):

```mips
     beq $s3, $s4, L1     # se i == j, desvia
     add $s0, $s1, $s2    # f = g + h  (executa se i != j)
L1:  sub $s0, $s0, $s3    # f = f - i
```

O label `L1` corresponde ao endereço da instrução `sub` (resolvido pelo montador).

### If-Then-Else

`if (i == j) f = g + h; else f = g - h;`:

```mips
      bne $s3, $s4, Else
      add $s0, $s1, $s2    # then
      j   Exit
Else: sub $s0, $s1, $s2    # else
Exit:
```

### Loop com array

`Loop: g = g + A[i]; i = i + j; if (i != h) goto Loop;` (g,h,i,j em `$s1`,`$s2`,`$s3`,`$s4`; base em `$s5`):

```mips
Loop: add $t1, $s3, $s3    # $t1 = 2 x i
      add $t1, $t1, $t1    # $t1 = 4 x i
      add $t1, $t1, $s5    # $t1 = endereço de A[i]
      lw  $t0, 0($t1)      # $t0 <- A[i]
      add $s1, $s1, $t0    # g = g + A[i]
      add $s3, $s3, $s4    # i = i + j
      bne $s3, $s2, Loop   # se i != h, repete
```

**Bloco básico:** sequência de instruções sem desvio (exceto talvez no final) e sem labels de desvio (exceto talvez no início).

### Set on Less Than (comparações)

```mips
slt $t0, $s3, $s4    # se $s3 < $s4 então $t0 = 1, senão $t0 = 0
```

As relações de ordem (igual, diferente, menor, menor ou igual, maior, maior ou igual) são geradas pelo compilador combinando `slt`, `beq`, `bne` e o registrador `$zero` (constante 0, só leitura).

`if (a < b) goto Less`:

```mips
slt $t0, $s0, $s1       # $t0 = 1 se a < b
bne $t0, $zero, Less    # se $t0 != 0, desvia
```

Isso segue a recomendação de von Neumann sobre simplicidade: não aumenta o ciclo de clock nem exige muitos ciclos, o que aconteceria se a comparação fosse uma instrução dedicada.

### Case / Switch

Usa uma **tabela de endereços de desvio** (*jump table*): array de palavras indexado contendo os endereços dos labels do código.

`switch (k) { case 0: f=i+j; case 1: f=g+h; case 2: f=g-h; case 3: f=i-j; }`:

```mips
      slt $t3, $s5, $zero   # testa k < 0
      bne $t3, $zero, Exit  # se k < 0, sai
      slt $t3, $s5, $t2     # testa k < 4
      beq $t3, $zero, Exit  # se k >= 4, sai
      add $t1, $s5, $s5     # $t1 = 2 x k
      add $t1, $t1, $t1     # $t1 = 4 x k
      add $t1, $t1, $t4     # $t1 = endereço de JumpTable[k]  ($t4 = base da tabela)
      lw  $t0, 0($t1)       # $t0 <- JumpTable[k]
      jr  $t0               # desvia para o endereço em $t0
L0:   add $s0, $s3, $s4     # k=0: f = i + j
      j   Exit
L1:   add $s0, $s1, $s2     # k=1: f = g + h
      j   Exit
L2:   sub $s0, $s1, $s2     # k=2: f = g - h
      j   Exit
L3:   sub $s0, $s3, $s4     # k=3: f = i - j
Exit:
```

`jr $t0` (*jump register*) desvia para o endereço contido no registrador.

---

## 11. Suporte a Procedimentos

Procedimentos são ferramenta de estruturação: facilitam entendimento e reúso; os parâmetros são a barreira entre procedimento e programa.

Passos da execução (programa chama procedimento):
1. Colocar parâmetros em local acessível ao procedimento.
2. Transferir o controle para o procedimento.
3. Garantir recursos de memória para a execução.
4. Realizar a tarefa.
5. Colocar o resultado em local acessível ao programa.
6. Retornar o controle ao ponto de origem.

### Registradores da convenção de chamada

- `$a0` a `$a3`: argumentos (parâmetros de entrada).
- `$v0` a `$v1`: valores de retorno (saída).
- `$ra`: endereço de retorno ao programa.

Instruções associadas:

```mips
jal Endereço-do-Procedimento    # chama procedimento; salva endereço de retorno em $ra
jr  $ra                         # retorna ao programa
```

### Pilha

- Aumenta a quantidade de registradores disponíveis ao procedimento.
- Salva registradores na entrada e os restitui na saída.
- O apontador de pilha `$sp` endereça o **topo** da pilha.
- A pilha **cresce dos endereços mais altos para os mais baixos** (direção inversa da execução).
- A movimentação de `$sp` deve ser balanceada para evitar estouro.

### Procedimento folha (não chama outro)

`int leaf_example(int g,int h,int i,int j){ int f; f=(g+h)-(i+j); return f; }`:

```mips
leaf_example:
    sub $sp, $sp, 12     # aloca 3 palavras na pilha
    sw  $t1, 8($sp)      # salva registradores usados
    sw  $t0, 4($sp)
    sw  $s0, 0($sp)
    add $t0, $a0, $a1    # $t0 = g + h
    add $t1, $a2, $a3    # $t1 = i + j
    sub $s0, $t0, $t1    # $s0 = (g+h) - (i+j)
    add $v0, $s0, $zero  # resultado em $v0
    lw  $s0, 0($sp)      # restaura registradores
    lw  $t0, 4($sp)
    lw  $t1, 8($sp)
    add $sp, $sp, 12     # libera a pilha (balanceamento)
    jr  $ra              # retorna
```

### Convenção de salvamento

| Preservados (salvos pelo procedimento) | Não preservados |
|---|---|
| `$s0` a `$s7` (salvamento) | `$t0` a `$t9` (temporários) |
| `$sp` (stack pointer) | `$a0` a `$a3` (argumentos) |
| `$ra` (endereço de retorno) | `$v0` a `$v1` (retorno) |
| pilha acima do stack pointer | pilha abaixo do stack pointer |

Regra prática: se um procedimento usa um `$sX`, ele **precisa** salvar e restaurar via pilha. Os `$tX` podem ser sobrescritos livremente.

### Procedimentos aninhados

Procedimentos que chamam outros procedimentos (ex.: Programa principal -> Proc. A -> Proc. B, e o retorno percorre o caminho inverso).

Problemas ao aninhar:
- Argumentos de entrada em `$a0` se sobrepõem.
- Endereços de retorno (`$ra`) se sobrepõem.

Solução (uso da pilha):
- Na entrada do Proc. A, salvar `$a0` e `$ra` na pilha.
- No retorno do Proc. B, restituir `$a0` e `$ra` da pilha.

### Interrupções (mesma ideia de aninhamento)

- **Sequenciais:** o programa é interrompido, o handler X executa e retorna; mais tarde, outra interrupção chama o handler Y, que executa e retorna. Um handler termina antes do próximo começar.
- **Aninhadas:** enquanto o handler X executa, chega outra interrupção e o handler Y interrompe X; Y retorna a X, e X retorna ao programa. Um handler é interrompido por outro (tipicamente de maior prioridade).

---

## 12. Operandos Imediatos (Constantes)

Constantes são muito usadas. Colocá-las dentro da instrução é mais rápido que buscá-las na memória.

`sp = sp + 4`:

```mips
addi $sp, $sp, 4     # soma imediata
```

Tradução para máquina (formato I): op=8, rs=29, rt=29, imediato=4
`001000 11101 11101 0000000000000100`

Outras instruções com imediato:

```mips
slti $t0, $s2, 10    # $t0 = 1 se $s2 < 10  (set on less than imediato)
lui  $t0, 255        # load upper immediate
```

`lui`: copia o valor (ex.: 255 = `0000 0000 1111 1111`) nos **16 bits de mais alta ordem** do registrador. Usado junto com `addi`/`ori` para montar constantes de 32 bits.

---

## 13. Endereçamento dos Desvios

**Desvio incondicional** `j 10000` (formato J):

| op | endereço |
|---|---|
| 6 bits | 26 bits |

**Desvio condicional** `bne $s0, $s1, Exit` (formato I):

| op | rs | rt | endereço |
|---|---|---|---|
| 6 bits | 5 bits | 5 bits | 16 bits |

- **Problema:** o campo de 16 bits limita o tamanho do salto.
- **Solução (endereçamento relativo ao PC):** o endereço de desvio é `[PC + 4] + campo de endereço`.

Se um desvio condicional precisar ir além do limite dos 16 bits, o montador inverte a condição e insere um desvio incondicional (`j`), fazendo o condicional decidir apenas se pula ou não o `j`. Exemplo do padrão gerado:

```mips
     beq $s0, $s1, L1     # forma original (salto longo)
# vira:
     bne $s0, $s1, L2     # condição invertida
     j   L1
L2:  ...
```

---

## 14. Modos de Endereçamento do MIPS

1. **A registrador:** o operando está num registrador.
2. **Base ou deslocamento:** endereço = registrador base + deslocamento (usado por `lw`/`sw`, com byte, meia-palavra ou palavra).
3. **Imediato:** o operando é uma constante dentro da instrução.
4. **Relativo ao PC:** endereço = PC + campo de endereço (usado por desvios condicionais).
5. **Pseudo-direto:** endereço formado por concatenação de bits (usado por `j`).

Uma mesma operação pode usar mais de um modo: a soma pode ser imediata (`addi`) ou a registrador (`add`).

---

## 15. Convenção MIPS de Registradores (referência completa)

| Nome | Nº | Utilização | Preservado |
|---|---|---|---|
| `$zero` | 0 | constante 0 | só leitura |
| `$v0`-`$v1` | 2-3 | resultados e avaliação de expressões | não |
| `$a0`-`$a3` | 4-7 | argumentos | não |
| `$t0`-`$t7` | 8-15 | temporários | não |
| `$s0`-`$s7` | 16-23 | salvos | sim |
| `$t8`-`$t9` | 24-25 | mais temporários | não |
| `$gp` | 28 | global pointer | sim |
| `$sp` | 29 | stack pointer | sim |
| `$fp` | 30 | frame pointer | sim |
| `$ra` | 31 | endereço de retorno | sim |

Não mostrados na tabela do slide: `$at` (1) reservado ao montador, `$k0`-`$k1` (26-27) reservados ao sistema operacional.

---

## 16. Execução de um Programa (Toolchain)

```
Programa em C (.c / .txt)
        |  Compilador
Programa Assembly (.asm / .s)
        |  Assembler (montador)
Linguagem de Máquina (.obj / .o)  +  Rotinas de biblioteca (.obj)
        |  Ligador (linker)
Linguagem de Máquina (.exe / .out)
        |  Carregador (loader)
Memória (execução)
```

### Montador (Assembler)

- Instruções: implementadas em hardware.
- **Pseudo-instruções:** facilitam a tradução e disciplinam a programação. Ex.: `move $t0, $t1` equivale a `add $t0, $zero, $t1`.
- Diretivas de layout de dados: `.asciiz "texto"`, `.byte 84, 35, 100, 122`.
- Desvio longo = instrução de branch + jump.
- Permite constantes de 32 bits (mesmo com imediatos de 16 bits) via `lui` + `addi`.
- Repertório do assembly do MIPS é mais rico que o hardware (instruções + pseudo-instruções).
- Tipos de dados: binário, decimal, hexadecimal.
- Precisa do registrador especial `$at`.
- **Tabela de símbolos:** define os endereços dos labels.

Limitações do assembly: ligado à arquitetura; programas mais longos que em alto nível; falta de estrutura, difíceis de ler e mais sujeitos a erro.

### Formato do arquivo-objeto (Unix)

- **Cabeçalho:** descreve os segmentos de texto e dados e seus tamanhos.
- **Segmento de texto (código):** código em linguagem de máquina.
- **Segmento de dados:** representação binária dos dados fonte (estáticos alocados pelo programa; dinâmicos alocados na execução).
- **Informação de relocação:** instruções e dados que dependem de endereço absoluto durante a carga.
- **Tabela de símbolos:** labels não definidos, como referências externas.

### Ligador (Linker)

Objetivos: unir os módulos em linguagem de máquina em um só; otimizar o executável a partir de módulos independentes; permitir compilar/montar procedimentos separadamente.

Passos:
1. Colocar módulos de dados e código simbolicamente na memória.
2. Determinar os endereços dos labels via informação de relocação e tabela de símbolos.
3. Resolver referências internas e externas.

O **arquivo executável** tem o mesmo formato do arquivo-objeto, mas sem referências não resolvidas e sem tabela de símbolos.

### Carregador (Loader)

Transfere o `.exe` para a memória. Passos:
1. Ler o cabeçalho para determinar o tamanho dos segmentos de código e dados.
2. Criar espaço de endereçamento para código e dados.
3. Copiar instruções e dados para a memória.
4. Copiar parâmetros para a pilha do programa principal (se houver).
5. Inicializar registradores; `$sp` aponta para o 1º endereço livre.
6. Desviar para a rotina de inicialização, que copia parâmetros nos registradores de argumento e chama a rotina principal.

### Mapa de memória do MIPS

Do endereço mais alto para o mais baixo:

| Segmento | Endereço | Cresce |
|---|---|---|
| Pilha (stack) | `$sp` = 0x7fff fffc | para baixo |
| Dados dinâmicos (heap) | (entre a pilha e os dados estáticos) | para cima |
| Dados estáticos | `$gp` = 0x1000 8000 / base 0x1000 0000 | \- |
| Texto (código) | `pc` = 0x0040 0000 | \- |
| Posições reservadas | 0 | \- |

Pilha e heap crescem em direções opostas para aproveitar melhor o espaço livre entre eles.

---

## 17. Exercícios

### Exercício 1 - Trace do código

Fragmento que processa um array de 400 palavras (índices 0 a 399), com endereço-base em `$a0` e tamanho 400 em `$a1`. Pede-se descrever o que o código faz, o que retorna em `$v0`, `$v1` e `$s7`, e os valores dos labels.

```mips
(80000000) Proc1:
      Add  $a1, $a1, $a1
      Add  $a1, $a1, $a1
      Addi $t0, $zero, 0
      Addi $t2, $zero, 1
      Addi $v0, $zero, 0
      Addi $v1, $zero, 0
      Add  $s4, $a0, $t0
      Lw   $s4, 0($s4)
      And  $t3, $s4, $t2
      Bne  $t3, $zero, Outro
      Addi $v0, $v0, 1
      J    Interno
Outro:    Addi $v1, $v1, 1
Interno:  Addi $s7, $s4, 0
Seguinte: Addi $t0, $t0, 4
      Slt  $t1, $t0, $a1
      Beq  $t1, $zero, Exit
      Add  $s4, $a0, $t0
      Lw   $s4, 0($s4)
      And  $t3, $s4, $t2
      Bne  $t3, $zero, Proximo
      Addi $v0, $v0, 1
      J    Teste
Proximo:  Addi $v1, $v1, 1
Teste:    Slt  $t1, $s4, $s7
      Bne  $t1, $zero, Next1
      Add  $s7, $s4, $zero
Next1:    J    Seguinte
Exit:     Jr   $ra
```

Análise (para conferir seu raciocínio):
- As duas primeiras somas fazem `$a1 = 400 x 4 = 1600` (tamanho do array em bytes), usado como limite do índice de byte `$t0`.
- `$t2 = 1` é uma máscara. `And $t3, $s4, $t2` isola o bit menos significativo do elemento, testando **par ou ímpar**.
  - Se o bit for 1 (ímpar), desvia para `Outro`, que incrementa `$v1`.
  - Se for 0 (par), incrementa `$v0`.
- `Slt $t1, $s4, $s7` com `Add $s7, $s4, $zero` quando o atual é maior mantém em `$s7` o **maior valor** do array.
- Retornos: `$v0` = quantidade de elementos **pares**; `$v1` = quantidade de elementos **ímpares**; `$s7` = **maior valor** do array.

Valores dos labels (cada instrução ocupa 4 bytes a partir de 0x80000000):

| Label | Endereço |
|---|---|
| Outro | 0x80000030 |
| Interno | 0x80000034 |
| Seguinte | 0x80000038 |
| Proximo | 0x8000005C |
| Teste | 0x80000060 |
| Next1 | 0x8000006C |
| Exit | 0x80000070 |

### Exercício 2 - Controle de qualidade de jeans (com procedimentos e pilha)

Enunciado: a fábrica produz 1000 calças por dia. Um array na memória guarda, para cada peça, o código 1 (perfeita), 2 (semi-perfeita) ou 3 (defeituosa). Base do array em `$s3`, quantidade total em `$s2`. O programa conta cada tipo e chama **três procedimentos** distintos: lucro das perfeitas (R$ 10,00 cada), lucro das semi-perfeitas (R$ 5,00 cada) e prejuízo das defeituosas (R$ 7,00 cada). O principal calcula o resultado diário. Todos devem seguir as regras de salvamento em pilha.

Programa principal (transcrito dos slides, com correções de digitação):

```mips
# $s2 = qtd de peças (1000); $s3 = base do array
# $s4 = código perfeita (1); $s5 = semi-perfeita (2); $s6 = defeituosa (3)
Start:
      add  $s0, $zero, $zero    # índice
      addi $s2, $zero, 1000     # quantidade total
      addi $s4, $zero, 1        # perfeita
      addi $s5, $zero, 2        # semi-perfeita
      addi $s6, $zero, 3        # defeituosa
      add  $t1, $zero, $zero    # contador perfeitas
      add  $t2, $zero, $zero    # contador semi-perfeitas
      add  $t3, $zero, $zero    # contador defeituosas
Loop: add  $t0, $s0, $s0        # 2 x índice
      add  $t0, $t0, $t0        # 4 x índice
      add  $t0, $t0, $s3        # base + deslocamento
      lw   $t0, 0($t0)          # lê a peça
      beq  $t0, $s5, L1         # semi-perfeita?
      beq  $t0, $s6, L2         # defeituosa?
      addi $t1, $t1, 1          # perfeita: incrementa
      j    Fim
L1:   addi $t2, $t2, 1          # semi-perfeita: incrementa
      j    Fim
L2:   addi $t3, $t3, 1          # defeituosa: incrementa
Fim:  addi $s0, $s0, 1          # incrementa índice
      slt  $t0, $s0, $s2        # fim do array?
      bne  $t0, $zero, Loop

      addi $sp, $sp, -8         # salva contadores na pilha
      sw   $t2, 0($sp)
      sw   $t3, 4($sp)
      add  $a0, $t1, $zero      # argumento: qtd perfeitas
      jal  Proc1
      add  $s1, $v0, $zero      # lucro perfeitas
      lw   $t2, 0($sp)
      add  $a0, $t2, $zero      # argumento: qtd semi-perfeitas
      jal  Proc2
      add  $s1, $s1, $v0        # lucro acumulado
      lw   $t3, 4($sp)
      addi $sp, $sp, 8          # balanceamento da pilha
      add  $a0, $t3, $zero      # argumento: qtd defeituosas
      jal  Proc3
      sub  $t4, $s1, $v0        # lucro - prejuízo
      add  $v0, $t4, $zero      # resultado financeiro
      slt  $t3, $t4, $zero
      beq  $t3, $zero, Lucro
      addi $v1, $zero, 1        # prejuízo ($v1 = 1)
      j    End
Lucro: add $v1, $zero, $zero    # lucro ($v1 = 0)
End:
```

Procedimentos (cada um recebe a quantidade em `$a0` e retorna o total em `$v0`):

```mips
Proc1:                          # lucro peças perfeitas
      addi $v0, $zero, 0        # lucro acumulado
      add  $t5, $zero, $zero    # contador
Inicio1:
      slt  $t6, $t5, $a0
      beq  $t6, $zero, ExitProc1
      addi $v0, $v0, 10         # R$ 10 por peça
      addi $t5, $t5, 1
      j    Inicio1
ExitProc1: jr $ra

Proc2:                          # lucro peças semi-perfeitas
      addi $v0, $zero, 0
      add  $t5, $zero, $zero
Inicio2:
      slt  $t6, $t5, $a0
      beq  $t6, $zero, ExitProc2
      addi $v0, $v0, 5          # R$ 5 por peça
      addi $t5, $t5, 1
      j    Inicio2
ExitProc2: jr $ra

Proc3:                          # prejuízo peças defeituosas
      addi $v0, $zero, 0
      add  $t5, $zero, $zero
Inicio3:
      slt  $t6, $t5, $a0
      beq  $t6, $zero, ExitProc3
      addi $v0, $v0, 7          # R$ 7 por peça
      addi $t5, $t5, 1
      j    Inicio3
ExitProc3: jr $ra
```

Observações sobre os slides deste exercício:
- Nos slides o terceiro procedimento aparece rotulado como `Proc1`; o correto é `Proc3`.
- Aparece `$zer0` em um `addi`; leia como `$zero`.
- A soma dos três contadores (`$t1 + $t2 + $t3`) deve fechar em 1000.

### Exercício 3 - strcpy (cópia de string)

`void strcpy(char x[], char y[]){ int i=0; while ((x[i]=y[i]) != 0) i=i+1; }` (base de x em `$a0`, de y em `$a1`, i em `$s0`):

```mips
strcpy:
      addi $sp, $sp, -4
      sw   $s0, 0($sp)          # salva $s0
      add  $s0, $zero, $zero    # i = 0
L1:   add  $t1, $a1, $s0        # endereço de y[i]
      lb   $t2, 0($t1)          # $t2 <- y[i]   (load byte)
      add  $t3, $a0, $s0        # endereço de x[i]
      sb   $t2, 0($t3)          # x[i] <- $t2   (store byte)
      add  $s0, $s0, 1          # i = i + 1
      bne  $t2, $zero, L1       # repete enquanto o byte != 0
      lw   $s0, 0($sp)          # restaura $s0
      add  $sp, $sp, 4
      jr   $ra
```

Ponto-chave: strings usam `lb`/`sb` (byte), não `lw`/`sw`. O laço copia o byte e para quando copia o terminador nulo (0).

### Exercício 4 - Fatorial recursivo

`int fact(int n){ if (n<1) return 1; else return n * fact(n-1); }`:

```mips
fact:
      addi $sp, $sp, -8         # aloca 2 palavras (ver nota)
      sw   $ra, 4($sp)          # salva endereço de retorno
      sw   $a0, 0($sp)          # salva argumento n
      slti $t0, $a0, 1          # $t0 = 1 se n < 1
      beq  $t0, $zero, L1       # se n >= 1, ramo else
      add  $v0, $zero, 1        # caso base: retorna 1
      addi $sp, $sp, 8          # libera pilha
      jr   $ra
L1:   addi $a0, $a0, -1         # n - 1
      jal  fact                 # chamada recursiva
      lw   $a0, 0($sp)          # restaura n
      lw   $ra, 4($sp)          # restaura retorno
      addi $sp, $sp, 8          # libera pilha
      mult $v0, $a0, $v0        # $v0 = n * fact(n-1)
      jr   $ra
```

Nota: o slide mostra `addi $sp, $sp, 8` no início, mas como a pilha cresce para baixo, alocar espaço exige **subtrair**: o correto é `addi $sp, $sp, -8`. Este é o padrão clássico de procedimento aninhado/recursivo: salvar `$ra` e `$a0` na entrada e restaurar antes do retorno.

---

## Checklist de revisão

- [ ] 4 princípios RISC e por que 32 registradores de 32 bits
- [ ] `lw`/`sw` e a fórmula de indexação de array `(4 x i) + base`
- [ ] Alinhamento, endianness e spilling
- [ ] Formatos R, I e J: nome e tamanho de cada campo
- [ ] Traduzir uma instrução aritmética para binário (mapeamento de registradores)
- [ ] `beq`, `bne`, `j`, `slt` e a montagem de if / if-else / loop / switch
- [ ] Convenção `$a`, `$v`, `$ra`, `jal`, `jr` e uso da pilha (`$sp`)
- [ ] Registradores preservados x não preservados
- [ ] Procedimentos aninhados e recursivos (o que salvar na pilha)
- [ ] `addi`, `slti`, `lui` e a construção de constantes de 32 bits
- [ ] Endereçamento relativo ao PC e o truque do desvio longo
- [ ] Os 5 modos de endereçamento
- [ ] Papéis de montador, ligador e carregador; formato objeto x executável
- [ ] Mapa de memória (pilha, heap, dados estáticos, texto)
