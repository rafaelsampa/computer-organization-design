# Arquitetura de Computadores - Aula 5

## MIPS: a lógica da aula (com as nuances do professor)

Este resumo segue o raciocínio das aulas, não a ordem dos slides. Complementa o outro guia (que é a referência slide a slide). Aqui o foco é o **porquê** de cada coisa e as regras práticas que caem na prova.

---

## 1. O coração de tudo é o desvio

Uma instrução cai em uma de três categorias:
- **Atribuição:** `a = b + c`, mover valor entre registrador e memória, operações lógicas (and, or).
- **Desvio:** muda o `pc` (que normalmente aponta para a instrução seguinte).
- **Chamada de procedimento:** call/return, system call, chamar função.

Por que o desvio é a categoria mais importante (junto com atribuição): **sem desvio o computador é só uma calculadora.** É o desvio que dá `if`, `else`, `while`, tratamento de exceção. É a capacidade de decidir.

**Desvio condicional x incondicional:** o condicional olha uma condição; o incondicional sempre desvia. As condições vêm dos **flags** da máquina, guardados no **PSW** (Processor Status Word): flag zero, flag "vai um" (carry), flag de overflow, trap flag (troca de controle para o SO ou periférico), entre outros. Antigamente eram uns 8 a 12 flags, hoje passam de 30. Ponto do professor: mexer em flag e mudar o `pc` é **lógica de programação**, não decisão do sistema operacional.

---

## 2. A grande restrição da disciplina

O MIPS tem cerca de 102 instruções, mas **a disciplina usa só um punhado (umas 10).** O livro também usa bem menos que 102. Em especial:

**Só 5 instruções aritméticas/lógicas (Tipo-R):** `add`, `sub`, `and`, `or`, `slt`.

Conjunto permitido nos exercícios:

| Tipo | Instruções |
|---|---|
| R | `add`, `sub`, `and`, `or`, `slt` (e `jr` no retorno de função) |
| I | `lw`, `sw`, `beq`, `bne`, `addi`, `andi`, `ori`, `slti`, `lui` |
| J | `j`, `jal` |

Consequência prática que aparece o tempo todo: **não tem `mul` nem `sll` na lista.** Para multiplicar um índice por 4, faz dois `add`: `i + i = 2i`, depois `2i + 2i = 4i`.

---

## 3. Tipo-R e o modelo da porta lógica

Formato de sempre, 3 operandos:

```mips
add A, B, C    # A = B + C
sub A, B, C    # A = B - C
```

O "R" vem de **Register**: os operandos estão em registradores. Modelo mental do professor: é como uma **porta lógica** com duas entradas (Rs e Rt) e uma saída (Rd), e a operação (add/sub/and/or/slt) define o que a porta faz.

- **Rd** = registrador destino (onde vai o resultado).
- **Rs** e **Rt** = registradores fonte (fornecem os valores).

Atenção à ordem: na escrita é `add rd, rs, rt`, mas **no binário os campos vêm na ordem rs, rt, rd**.

### opcode e funct (e por que os dois existem)

O **opcode** tem 6 bits, então dá 2^6 = **64 operações** possíveis. Mas todas as instruções Tipo-R usam **opcode = 0**. Então como diferenciar add de sub de and? Criaram um segundo campo, o **funct** (também 6 bits), que diz qual operação exatamente.

Fluxo na máquina: opcode vai para a **Unidade de Controle (UC)**. A UC libera os sinais de controle ("semáforos") para os componentes: se é acesso à memória, libera o barramento; se é aritmética, libera a ALU. Para o Tipo-R a UC ainda olha o funct: "é para executar uma operação, mas qual? o funct diz qual, e aí a UC libera a ALU para aquela operação específica". Algumas instruções nem precisam de funct (ex.: desvio incondicional, o opcode já basta).

**Por que 5 bits para cada registrador?** Porque o MIPS tem 32 registradores, e 2^5 = 32. Com 4 bits só daria 16.

Valores de funct (decorar para montar binário):

| instrução | funct (dec) | funct (hex) |
|---|---|---|
| add | 32 | 0x20 |
| sub | 34 | 0x22 |
| and | 36 | 0x24 |
| or | 37 | 0x25 |
| slt | 42 | 0x2A |
| jr | 8 | 0x08 |

**Exemplo de montagem** `add $t0, $s1, $s2` (rs=$s1=17, rt=$s2=18, rd=$t0=8):

```
opcode  rs      rt      rd      shamt   funct
0       17      18      8       0       32
000000  10001   10010   01000   00000   100000
```

Agrupando de 4 em 4: `0000 0010 0011 0010 0100 0000 0010 0000` = **0x02324020**.

---

## 4. Aritmética imediata e o Tipo-I

Quando um operando é uma **constante** em vez de um registrador, usa a versão imediata. Em arquitetura, é só setar um bit diferente na instrução para indicar modo imediato, e o formato muda para caber a constante.

```mips
addi A, B, 5    # A = B + 5
```

Formato Tipo-I:

```
| opcode | rs (5) | rt (5) | imediato (16) |   = 32 bits
```

- **rt** aqui geralmente é o **destino** (em `addi`, `lw`). Exceções: em `sw` o rt é o valor a gravar, e em `beq`/`bne` o rt é o segundo valor comparado.
- **imediato**: 16 bits. Com sinal, vai de -32768 a +32767.

### Nuances que derrubam gente na prova

- **Não existe `subi`.** Para subtrair constante, `addi` com negativo: `addi $t0, $t0, -1`.
- **Extensão de sinal x extensão com zero.** O imediato tem 16 bits mas o registrador tem 32, então precisa preencher os 16 de cima:
  - `addi`, `slti`, `lw`, `sw`, `beq`, `bne`: estendem **com sinal** (copiam o bit mais alto; -1 vira 0xFFFFFFFF).
  - `andi`, `ori`: estendem **com zeros** (0xFFFF vira 0x0000FFFF).
- **`lwi` não existe** (aparece num rascunho, mas é engano). Para colocar constante grande num registrador, usa `lui` + `ori`. O montador tem a pseudo-instrução `li` que faz isso por baixo dos panos.

Opcodes do Tipo-I:

| instrução | opcode (dec/hex) | significado |
|---|---|---|
| lw | 35 / 0x23 | `lw $t0, 8($s3)` -> $t0 = Mem[$s3+8] |
| sw | 43 / 0x2B | `sw $t0, 8($s3)` -> Mem[$s3+8] = $t0 |
| beq | 4 / 0x04 | se $s1 == $s2, desvia |
| bne | 5 / 0x05 | se $s1 != $s2, desvia |
| addi | 8 / 0x08 | $t0 = $t0 + 1 |
| andi | 12 / 0x0C | máscara (pega só certos bits); estende com zero |
| ori | 13 / 0x0D | liga bits; estende com zero |
| slti | 10 / 0x0A | $t0 = 1 se $s0 < 10 |
| lui | 15 / 0x0F | `lui $t0, 0x1234` -> $t0 = 0x12340000 |

**Exemplo de montagem** `lw $t0, 32($s3)` (rs=$s3=19, rt=$t0=8, imediato=32):

```
opcode  rs      rt      imediato
35      19      8       32
100011  10011   01000   0000000000100000
```

= `1000 1110 0110 1000 0000 0000 0010 0000` = **0x8E680020**.

---

## 5. Acesso à memória

Vetor mora na **memória**, não no registrador. Para operar, precisa trazer com `lw` e devolver com `sw`.

Tamanhos: a memória é organizada em palavras de 4 bytes, e só dá para mexer em **1, 2 ou 4 bytes**:
- `lb`/`sb`: byte (1)
- `lh`/`sh`: halfword (2)
- `lw`/`sw`: word (4)

Endereçamento: cada palavra ocupa 4 endereços consecutivos de byte. O **offset do `lw`/`sw` é em bytes**, então `A[i]` fica em `4*i` a partir do endereço base.

**Endianness:** big endian guarda o byte mais significativo no menor endereço; little endian guarda o menos significativo no menor endereço. **A disciplina usa little endian.**

**Não dá para ter instrução com 4 registradores.** Se a conta precisa de mais operandos, quebra em várias instruções. Exemplo `g = h + A[i]` (i=$s4, base=$s3, h=$s2):

```mips
add $t1, $s4, $s4    # 2i
add $t1, $t1, $t1    # 4i
add $t1, $t1, $s3    # endereço de A[i]
lw  $t0, 0($t1)      # A[i]
add $s1, $s2, $t0    # g = h + A[i]
```

Sobre **shift** (`sll`, `srl`): deslocar 1 posição para a esquerda equivale a multiplicar por 2, para a direita a dividir por 2. Mas shift é operação **lógica de bits**, não é load, não é store, não é read/write. (E não está no conjunto permitido, por isso multiplica-se por 4 com dois `add`.)

---

## 6. O banco de registradores

Os registradores são da **CPU** (hardware), muito mais rápidos que a memória. O banco é **sempre o mesmo**, não importa o programa.

- **`pc`** (program counter, "apontador de programa"): endereço da próxima instrução. Desvio muda o `pc`.
- **`$sp`** (stack pointer): aponta o topo da pilha.
- **`$at`** (assembler temporary): uso interno do montador, não usar direto.
- **`$t0`-`$t9`** (temporários): **não são preservados** entre chamadas.
- **`$s0`-`$s7`** (salvos): **precisam ser preservados**. Se um programa usa um `$s`, tem que salvar antes de chamar função e restaurar depois.
- **`$a0`-`$a3`** (argumentos): passam parâmetros para funções. **Máximo 4**; extras vão para a pilha.
- **`$v0`-`$v1`** (retorno): valores devolvidos pela função.

Números dos registradores (necessário para montar binário):

| reg | nº | reg | nº |
|---|---|---|---|
| $zero | 0 | $t8-$t9 | 24-25 |
| $at | 1 | $k0-$k1 | 26-27 |
| $v0-$v1 | 2-3 | $gp | 28 |
| $a0-$a3 | 4-7 | $sp | 29 |
| $t0-$t7 | 8-15 | $fp | 30 |
| $s0-$s7 | 16-23 | $ra | 31 |

**Por que 4 registradores de argumento?** Convenção que equilibra eficiência (acesso rápido, sem ir à memória) e simplicidade. Nuance do professor: na prática **os parâmetros vão nos registradores, não na pilha.** O padrão da disciplina é `$a0` = endereço-base do vetor e `$a1` = tamanho do vetor. A pilha serve mais para variáveis locais e para salvar o estado dos registradores antes de uma chamada. (Ex.: uma função `check_sum` recebe base em `$a0`, tamanho em `$a1`, e devolve a soma em `$v0`.)

Como usar a pilha na prática (empilha decrementando, cresce para baixo):

```mips
addi $sp, $sp, -4    # abre espaço (push)
sw   $t0, 0($sp)     # guarda
# ...
lw   $t0, 0($sp)     # recupera (pop)
addi $sp, $sp, 4     # libera
```

---

## 7. Por que RISC (a lógica econômica)

MIPS é RISC (Reduced Instruction Set Computer). A ideia: instrução tende a ser o mais simples possível, porque **no fim é uma trilha de fios, não software.**

A cadeia que o professor desenha: instrução simples -> hardware simples -> mais rápido -> mais barato -> mais gente compra -> mais dinheiro -> mais pesquisa -> arquitetura melhor -> menos instruções (o ciclo se retroalimenta).

Consequências de projeto:
- Todas as instruções têm **32 bits** (só muda o formato).
- Instrução simples tende a rodar em **um ciclo de clock**.
- Fica mais fácil de **paralelizar** (pipeline).
- "Variável é registrador; o que não é variável é constante (imediato)."

---

## 8. Desempenho (o bloco de fórmulas)

### Throughput x tempo de resposta

- **Tempo de resposta** (latência, tempo de execução): quanto demora **uma** tarefa, do início ao fim. Visão do usuário.
- **Throughput** (vazão): quantas tarefas terminam **por unidade de tempo**. Visão do servidor/data center. Ex.: 200 requisições/s.

Relação: trocar por um processador mais rápido melhora **os dois**. Colocar mais processadores (mais núcleos, mais máquinas) melhora **o throughput**, mas cada tarefa individual não fica necessariamente mais rápida.

```
Desempenho = 1 / Tempo de execução
```

### Speedup

Quanto uma coisa ficou mais rápida que outra. É razão, não tem unidade.

```
Speedup = Desempenho_novo / Desempenho_antigo = Tempo_antigo / Tempo_novo
```

Ex.: de 10 s para 4 s -> speedup 2,5. "A é n vezes mais rápido que B" quer dizer `Tempo_B / Tempo_A = n`.

### Lei de Amdahl (quase sempre vem junto)

Melhorar uma parte só acelera a fração de tempo em que essa parte é usada. O resto continua igual.

```
Tempo_novo = (Tempo afetado / fator de melhoria) + Tempo não afetado
```

Exemplo do livro: programa de 100 s, sendo 80 s de multiplicação.
- Ficar 4x mais rápido (25 s): `80/n + 20 = 25` -> `n = 16`.
- Ficar 5x mais rápido (20 s): `80/n + 20 = 20` -> `80/n = 0`, **impossível**. Mesmo com multiplicação instantânea, os outros 20 s ficam lá.

Moral: **"torne o caso comum mais rápido"**. Não adianta otimizar o que ocupa pouco do tempo total.

### Tempo de CPU

```
Tempo de CPU = Nº de instruções x CPI x Tempo de ciclo do clock
             = Nº de instruções x CPI / Frequência do clock
```

- **Nº de instruções:** depende do programa e do compilador.
- **CPI** (ciclos por instrução): depende da arquitetura e da implementação.
- **Tempo de ciclo:** depende do hardware (frequência, ex.: 1 / 3 GHz).

É por isso que o RISC aposta em instruções simples: CPI baixo e ciclo curto, mesmo que precise de mais instruções.

---

## 9. Os 3 formatos lado a lado

```
Tipo-R: | op(6) | rs(5) | rt(5) | rd(5) | shamt(5) | funct(6) |   add, sub, and, or, slt, jr
Tipo-I: | op(6) | rs(5) | rt(5) |        imediato(16)         |   lw, sw, beq, bne, addi, andi, ori, slti, lui
Tipo-J: | op(6) |             endereço(26)                    |   j, jal
```

**Por que 3 formatos e não 1?** Princípio "um bom projeto exige bons compromissos". Queriam manter 32 bits fixos (hardware simples, rápido de buscar e decodificar), mas instrução só com registradores não tem espaço para constante ou endereço grande. Solução: tamanho fixo, formato variável.

Detalhe esperto: **`rs` e `rt` ficam na mesma posição** nos tipos R e I. Assim o hardware já começa a ler esses registradores do banco antes de terminar de decodificar ("a simplicidade é favorecida pela regularidade"). O `shamt` tem **5 bits** e o nome é **shamt** (shift amount), usado só em `sll`/`srl`.

---

## 10. Desvios condicionais e endereçamento relativo ao PC

```mips
beq $s1, $s2, L    # branch if equal: se $s1 == $s2, desvia
bne $s1, $s2, L    # branch if not equal: se $s1 != $s2, desvia
```

São Tipo-I. O campo imediato **não é o endereço** de L: é a **distância em instruções** contada a partir da instrução seguinte (PC + 4). Isso é o **endereçamento relativo ao PC**.

```
se a condição for verdadeira:  PC = (PC + 4) + (imediato x 4)
```

Faz sentido porque desvio condicional quase sempre pula para perto (if, while, for), e 16 bits com sinal alcançam ~32 mil instruções para frente ou para trás. Para ir longe, usa `j`.

Contando o offset:

```mips
      bne $s3, $s4, Else    # PC+4 aponta para o add abaixo
      add $s0, $s1, $s2     # +0
      j   Fim               # +1
Else: sub $s0, $s1, $s2     # +2  -> imediato do bne = 2
Fim:
```

---

## 11. Comparações sem `blt`/`bgt`: `slt` + `beq`/`bne`

Não tem `blt`, `bgt`, `ble`, `bge`. Motivo: comparar **igualdade** é muito mais barato em hardware do que comparar menor/maior, e o desvio precisa ser rápido. A solução é combinar `slt` (ou `slti`) com `beq`/`bne`, comparando o resultado com `$zero`:

| em C | em MIPS (a=$s0, b=$s1) |
|---|---|
| `if (a < b) goto L` | `slt $t0, $s0, $s1` / `bne $t0, $zero, L` |
| `if (a >= b) goto L` | `slt $t0, $s0, $s1` / `beq $t0, $zero, L` |
| `if (a > b) goto L` | `slt $t0, $s1, $s0` / `bne $t0, $zero, L` |
| `if (a <= b) goto L` | `slt $t0, $s1, $s0` / `beq $t0, $zero, L` |

Lógica: `slt` deixa 1 se for menor, 0 se não. Depois é só perguntar "é diferente de zero?" (`bne`) ou "é zero?" (`beq`). Para `>`, troca a ordem dos operandos, porque `a > b` é o mesmo que `b < a`.

**Truque do `$zero`:** ele é sempre 0 e não muda. Então `beq $zero, $zero, L` é **sempre verdadeiro**, virando um desvio incondicional. Útil quando o exercício não deixa usar `j`.

---

## 12. Tipo-J: saltos

```mips
j   Loop      # PC = endereço de Loop
jal funcao    # $ra = PC + 4 (endereço de volta), depois PC = funcao
```

O retorno é `jr $ra`, que é **Tipo-R** (opcode 0, funct 8), porque o destino está num registrador.

**Como 26 bits viram endereço de 32 bits:**
1. Toda instrução tem 4 bytes, então todo endereço é múltiplo de 4: os 2 últimos bits são sempre `00`. Não precisa guardar, basta multiplicar por 4 (shift de 2). 26 + 2 = 28 bits.
2. Os 4 bits que faltam vêm dos 4 bits mais altos do PC atual.

```
endereço final = [4 bits altos do PC+4][26 bits do campo][00]
```

Ex.: `Loop` em 0x00400000 -> o campo guarda 0x00400000 / 4 = 0x00100000.

O endereço 0 costuma ser reservado (restart/reset do sistema).

---

## 13. Resolvendo exercícios: "só com essas instruções"

Formato típico da prova: dado um código em C, traduzir para MIPS usando **somente** o conjunto permitido (Seção 2).

### Roteiro

1. **Mapear variáveis em registradores** (o enunciado costuma dar).
2. **Vetor está na memória** -> `lw`/`sw`, com `A[i]` em `base + 4*i`.
3. **if / while:** inverter a condição e desviar para "fora" (para o else ou para o fim). Se em C é `if (i == j)`, em MIPS o desvio é `bne` para o else.
4. **< > <= >=:** `slt`/`slti` + `beq`/`bne` com `$zero` (tabela da Seção 11).
5. **Subtrair constante:** `addi` com número negativo.
6. **Constante maior que 16 bits:** `lui` + `ori`.
7. **Zerar registrador:** `add $t0, $zero, $zero`.

### if / else

```c
if (i == j) f = g + h; else f = g - h;   // f=$s0, g=$s1, h=$s2, i=$s3, j=$s4
```

```mips
      bne $s3, $s4, Else    # condição invertida: se i != j, vai pro else
      add $s0, $s1, $s2     # f = g + h
      j   Fim               # sem isso, executaria os dois blocos!
Else: sub $s0, $s1, $s2     # f = g - h
Fim:
```

### while com vetor

```c
while (save[i] == k) i += 1;   // i=$s3, k=$s5, base de save=$s6
```

```mips
Loop: add  $t1, $s3, $s3    # 2i
      add  $t1, $t1, $t1    # 4i
      add  $t1, $t1, $s6    # endereço de save[i]
      lw   $t0, 0($t1)      # save[i]
      bne  $t0, $s5, Fim    # se save[i] != k, sai
      addi $s3, $s3, 1      # i++
      j    Loop
Fim:
```

### for somando vetor (com slt)

```c
soma = 0; for (i = 0; i < n; i++) soma += A[i];   // soma=$s0, base=$s1, n=$s2, i=$t0
```

```mips
      add  $s0, $zero, $zero   # soma = 0
      add  $t0, $zero, $zero   # i = 0
Loop: slt  $t1, $t0, $s2       # $t1 = 1 se i < n
      beq  $t1, $zero, Fim     # se NÃO (i < n), sai
      add  $t2, $t0, $t0       # 2i
      add  $t2, $t2, $t2       # 4i
      add  $t2, $t2, $s1       # endereço de A[i]
      lw   $t3, 0($t2)         # A[i]
      add  $s0, $s0, $t3       # soma += A[i]
      addi $t0, $t0, 1         # i++
      j    Loop
Fim:
```

### constante de 32 bits

```c
x = 4000000;   // 4.000.000 = 0x003D0900, não cabe em 16 bits.  x = $s0
```

```mips
lui $s0, 0x003D          # $s0 = 0x003D0000
ori $s0, $s0, 0x0900     # $s0 = 0x003D0900
```

Por que `ori` e não `addi`? Porque `addi` estende com sinal: se a parte de baixo fosse 0x8000 ou maior, viraria negativo e bagunçaria a parte de cima. `ori` estende com zero, só encaixa os 16 bits de baixo.

### par ou ímpar (com máscara)

```c
if ((x & 1) == 0) par = 1; else par = 0;   // x = $s0, par = $s1
```

```mips
      andi $t0, $s0, 1         # último bit de x (0 = par, 1 = ímpar)
      addi $s1, $zero, 0       # par = 0
      bne  $t0, $zero, Fim     # se bit = 1, é ímpar, mantém 0
      addi $s1, $zero, 1       # par = 1
Fim:
```

### condição composta (exemplo do professor)

```c
if (A == B && (C != D || A == C)) A = 20; else A = 30;
// A=$t0, B=$s0, C=$t1, D=$s1
```

```mips
      bne $t0, $s0, Else    # A != B  -> falso, vai pro else
      beq $t0, $t1, Then    # A == C  -> verdadeiro
      bne $t1, $s1, Then    # C != D  -> verdadeiro
Else: addi $t0, $zero, 30   # cai aqui se A==B, A!=C e C==D (falso)
      j    Fim
Then: addi $t0, $zero, 20
Fim:
```

A lógica é avaliação em curto-circuito: se `A != B` já falha tudo; passando disso, `A == C` ou `C != D` bastam para ser verdadeiro (pulam para Then); se nenhum dos dois, cai no Else.

---

## Erros comuns (não errar isso)

- **Ordem dos campos:** escreve `add rd, rs, rt`, mas no binário é `rs, rt, rd`.
- **`shamt` tem 5 bits** e o nome é shamt (não "shunt", não 4 bits).
- **`subi` não existe:** use `addi` negativo. **`lwi` não existe:** use `lui` + `ori` (ou o pseudo `li`).
- **Extensão de sinal x zero:** `addi/slti/lw/sw/beq/bne` com sinal; `andi/ori` com zero.
- **Esquecer o `j` depois do bloco "then"** num if/else: sem ele, os dois blocos executam.
- **Multiplicar índice por 4** com dois `add` (não tem `sll` nem `mul` no conjunto).
- **Offset de `lw`/`sw` é em bytes:** `A[i]` está em `4*i`, não em `i`.
- **Imediato do `beq`/`bne` é distância em instruções** a partir de PC+4, não o endereço do label.
