
___________________________________________________________________

PROXIMA AULA



## Desempenho: throughput e speedup

throughput: é co... (completando) é a **vazão**, ou seja, a quantidade de trabalho que o computador termina por unidade de tempo. Exemplo: um servidor que processa 200 requisições por segundo tem throughput de 200 req/s.

Não confundir com **tempo de resposta** (ou tempo de execução, latência), que é quanto tempo leva para terminar UMA tarefa, do início ao fim.

- Tempo de resposta: "quanto tempo demora a minha tarefa?" (visão do usuário)
- Throughput: "quantas tarefas terminam por segundo?" (visão do servidor / data center)

Eles se relacionam, mas não são a mesma coisa:
- Trocar o processador por um mais rápido: melhora o tempo de resposta **e** o throughput.
- Colocar mais processadores (mais núcleos, mais máquinas): melhora o throughput, mas cada tarefa individual não fica necessariamente mais rápida.

Desempenho é o inverso do tempo de execução:

```
Desempenho = 1 / Tempo de execução
```

Quanto menor o tempo, maior o desempenho.

agora é speedup

**Speedup** (aceleração) é o quanto uma coisa ficou mais rápida que outra. É uma razão, não tem unidade.

```
Speedup = Desempenho_novo / Desempenho_antigo = Tempo_antigo / Tempo_novo
```

Exemplo: um programa rodava em 10 s, depois de uma melhoria roda em 4 s.
Speedup = 10 / 4 = 2,5. O sistema novo é 2,5 vezes mais rápido.

Outro jeito de falar a mesma coisa: "A é n vezes mais rápido que B" significa `Tempo_B / Tempo_A = n`.
Se A leva 10 s e B leva 15 s, A é 15/10 = 1,5 vezes mais rápido que B.

### Complemento: Lei de Amdahl (quase sempre vem junto com speedup)

A melhoria de uma parte do sistema só acelera a parte do tempo em que essa parte é usada. O resto continua igual.

```
Tempo_novo = (Tempo afetado pela melhoria / fator de melhoria) + Tempo não afetado
```

Exemplo clássico do livro: um programa leva 100 s, e 80 s disso são multiplicações.
- Quero que o programa fique 4x mais rápido (25 s): `80/n + 20 = 25`, então `80/n = 5`, logo `n = 16`. A multiplicação tem que ficar 16x mais rápida.
- Quero que fique 5x mais rápido (20 s): `80/n + 20 = 20`, então `80/n = 0`. **Impossível.** Nem com multiplicação instantânea, porque os outros 20 s continuam lá.

Moral: "torne o caso comum mais rápido" (a mesma ideia do rascunho-01). Não adianta otimizar muito algo que ocupa pouco do tempo total.

### Complemento: tempo de CPU

```
Tempo de CPU = Nº de instruções × CPI × Tempo de ciclo do clock
             = Nº de instruções × CPI / Frequência do clock
```

- **Nº de instruções**: depende do programa e do compilador.
- **CPI** (ciclos de clock por instrução): depende da arquitetura e da implementação.
- **Tempo de ciclo**: depende do hardware (frequência do clock, ex. 1 / 3 GHz).

Por isso o RISC tenta ter instruções simples: CPI baixo e ciclo de clock curto, mesmo que precise de mais instruções.




## Formatos de instrução do MIPS

temos 3 formatos, tipo-r, tipo-i... e tipo-j.

Todas têm **32 bits**, sempre. O que muda é como esses 32 bits são divididos. Os primeiros 6 bits são sempre o **opcode**, e é olhando para ele que a Unidade de Controle sabe qual formato é e como ler o resto.

Por que 3 formatos e não 1 só? Princípio de projeto do livro: **"bom projeto exige bons compromissos"**. Queriam manter todas as instruções com 32 bits (hardware simples, mais rápido pra buscar e decodificar), mas uma instrução só com registradores não tem espaço pra uma constante ou endereço grande. Solução: manter o tamanho fixo e variar o formato.

Outro detalhe esperto: `rs` e `rt` ficam **na mesma posição** nos formatos R e I. Assim o hardware já pode ir lendo esses registradores do banco antes mesmo de terminar de decodificar a instrução ("simplicidade favorece regularidade").

### Tipo-R (Register)

```
| opcode | rs     | rt     | rd     | shamt  | funct  |
| 6 bits | 5 bits | 5 bits | 5 bits | 5 bits | 6 bits |   = 32 bits
```

(corrigido: na anotação eu tinha escrito 4 bits e "shunt". São **5 bits** e o nome é **shamt**)

- **opcode**: sempre 0 no tipo-R.
- **rs**: primeiro registrador fonte.
- **rt**: segundo registrador fonte.
- **rd**: registrador destino (onde vai o resultado).
- **shamt** (shift amount): quantos bits deslocar. Só é usado em `sll`/`srl`, nas outras fica 0.
- **funct**: diz qual operação exatamente (add, sub, and...).

Por que 5 bits para registrador? Porque o MIPS tem 32 registradores, e 2^5 = 32. Com 4 bits só daria para 16.

Valores do funct:

| instrução | funct (decimal) | funct (hex) |
|-----------|-----------------|-------------|
| add       | 32              | 0x20        |
| sub       | 34              | 0x22        |
| and       | 36              | 0x24        |
| or        | 37              | 0x25        |
| slt       | 42              | 0x2A        |
| jr        | 8               | 0x08        |

Números dos registradores (precisa disso para montar o binário):

| registrador | número | registrador | número |
|-------------|--------|-------------|--------|
| $zero       | 0      | $t8 - $t9   | 24-25  |
| $at         | 1      | $k0 - $k1   | 26-27  |
| $v0 - $v1   | 2-3    | $gp         | 28     |
| $a0 - $a3   | 4-7    | $sp         | 29     |
| $t0 - $t7   | 8-15   | $fp         | 30     |
| $s0 - $s7   | 16-23  | $ra         | 31     |

**Exemplo montando o binário:** `add $t0, $s1, $s2`

Atenção: na escrita é `add rd, rs, rt`, mas no binário a ordem dos campos é rs, rt, rd.

```
opcode  rs($s1)  rt($s2)  rd($t0)  shamt  funct(add)
0       17       18       8        0      32
000000  10001    10010    01000    00000  100000
```

Juntando e agrupando de 4 em 4: `0000 0010 0011 0010 0100 0000 0010 0000` = **0x02324020**

### Tipo-I (Immediate)

tipo-i. o que é?

É o formato para instruções que precisam de uma **constante** (valor imediato) ou de um **deslocamento**. O "I" vem de *Immediate*. Como não tem rd, shamt e funct, esses 16 bits viram espaço para a constante.

```
| opcode | rs     | rt     | imediato (constante ou offset) |
| 6 bits | 5 bits | 5 bits | 16 bits                        |   = 32 bits
```

- **rs**: registrador base / fonte.
- **rt**: aqui o rt geralmente é o **destino** (addi, lw...), com exceção do `sw` (rt é o valor a ser gravado) e do `beq`/`bne` (rt é o segundo valor comparado).
- **imediato**: número de 16 bits. Com sinal vai de -32768 a +32767.

lw, sw, beq, bne, addi, andi, ori, slti, lui

| instrução | opcode (dec / hex) | exemplo              | significado                                 |
|-----------|--------------------|----------------------|---------------------------------------------|
| lw        | 35 / 0x23          | `lw $t0, 8($s3)`     | $t0 = Memória[$s3 + 8]                       |
| sw        | 43 / 0x2B          | `sw $t0, 8($s3)`     | Memória[$s3 + 8] = $t0                       |
| beq       | 4 / 0x04           | `beq $s1, $s2, L`    | se $s1 == $s2, pula para L                   |
| bne       | 5 / 0x05           | `bne $s1, $s2, L`    | se $s1 != $s2, pula para L                   |
| addi      | 8 / 0x08           | `addi $t0, $t0, 1`   | $t0 = $t0 + 1                                |
| andi      | 12 / 0x0C          | `andi $t0, $s0, 0xFF`| $t0 = $s0 AND 0xFF (máscara, pega só o byte baixo) |
| ori       | 13 / 0x0D          | `ori $t0, $s0, 0x1`  | $t0 = $s0 OR 1 (liga bits)                   |
| slti      | 10 / 0x0A          | `slti $t0, $s0, 10`  | $t0 = 1 se $s0 < 10, senão 0                 |
| lui       | 15 / 0x0F          | `lui $t0, 0x1234`    | $t0 = 0x12340000 (constante nos 16 bits altos) |

Detalhes importantes de cada uma:

- **Não existe `subi`**. Para subtrair uma constante, usa `addi` com número negativo: `addi $t0, $t0, -1`.
- **Extensão de sinal**: o imediato tem 16 bits mas o registrador tem 32. Em `addi`, `slti`, `lw`, `sw`, `beq`, `bne` o número é estendido **com sinal** (copia o bit mais significativo pra esquerda, então -1 vira 0xFFFFFFFF). Em `andi` e `ori` é estendido **com zeros** (0xFFFF vira 0x0000FFFF).
- **`lui`** (load upper immediate): coloca a constante nos 16 bits de cima e zera os 16 de baixo. Serve para montar constantes de 32 bits junto com `ori` (ver exercício mais abaixo). Obs: o "lwi" do rascunho-01 não é instrução real do MIPS. O jeito de colocar uma constante grande num registrador é `lui` + `ori` (o montador tem uma pseudo-instrução `li` que faz isso por você).
- **`lw`/`sw`**: o offset é em **bytes**. Como cada word tem 4 bytes, o elemento `A[i]` fica em `4*i` a partir do endereço base.

**Exemplo montando o binário:** `lw $t0, 32($s3)`

```
opcode(lw)  rs($s3)  rt($t0)  imediato
35          19       8        32
100011      10011    01000    0000000000100000
```

= `1000 1110 0110 1000 0000 0000 0010 0000` = **0x8E680020**

### Tipo-J (Jump)

tipo-j. o que é?

É o formato do **desvio incondicional** (salto). Só tem opcode e endereço, sem registradores, para sobrar o máximo de bits para o endereço de destino.

```
| opcode | endereço |
| 6 bits | 26 bits  |   = 32 bits
```

| instrução | opcode | exemplo      | significado                                         |
|-----------|--------|--------------|-----------------------------------------------------|
| j         | 2      | `j Loop`     | PC = endereço de Loop                               |
| jal       | 3      | `jal funcao` | $ra = PC + 4 (endereço de volta), depois PC = funcao |

E o retorno da função? É o `jr $ra` (jump register), que é **tipo-R** (opcode 0, funct 8), porque o destino está num registrador.

Como 26 bits viram um endereço de 32 bits?
1. Toda instrução tem 4 bytes, então todo endereço de instrução é múltiplo de 4, ou seja, os 2 últimos bits são **sempre 00**. Não precisa guardar, basta multiplicar por 4 (shift de 2 para a esquerda). 26 + 2 = 28 bits.
2. Os 4 bits que faltam são copiados dos 4 bits mais altos do PC atual.

```
endereço final = [4 bits altos do PC+4] [26 bits do campo] [00]
```

Exemplo: se `Loop` está em 0x00400000, o campo endereço guarda 0x00400000 / 4 = 0x00100000.

### Resumo dos 3 formatos

```
Tipo-R: | op (6) | rs (5) | rt (5) | rd (5) | shamt (5) | funct (6) |   add, sub, and, or, slt, jr
Tipo-I: | op (6) | rs (5) | rt (5) |       imediato (16)            |   lw, sw, beq, bne, addi, andi, ori, slti, lui
Tipo-J: | op (6) |              endereço (26)                       |   j, jal
```




## Desvios condicionais: beq e bne

Desvie se igual
beq  →  **b**ranch if **eq**ual

Desvie se NAO igual
bne  →  **b**ranch if **n**ot **e**qual

```
beq $s1, $s2, L    # se $s1 == $s2, vai para L; senão segue para a próxima instrução
bne $s1, $s2, L    # se $s1 != $s2, vai para L; senão segue para a próxima instrução
```

São tipo-I. O campo imediato **não é o endereço** de L, é a **distância em instruções** contada a partir da instrução seguinte (PC + 4). Isso se chama **endereçamento relativo ao PC**.

```
se a condição for verdadeira:  PC = (PC + 4) + (imediato × 4)
```

Faz sentido porque desvios condicionais quase sempre pulam para perto (if, while, for), e com 16 bits com sinal dá para ir até ~32 mil instruções pra frente ou pra trás. Para ir longe usa-se o `j`.

Exemplo de como contar o offset:

```
      bne $s3, $s4, Else    # PC+4 aponta para o add abaixo
      add $s0, $s1, $s2     # +0
      j   Fim               # +1
Else: sub $s0, $s1, $s2     # +2  → imediato do bne = 2
Fim:
```

### E o "menor que", "maior que"?

Não tem `blt`, `bgt`, `ble`, `bge` na lista. Por quê? Comparar igualdade é muito mais barato em hardware do que comparar menor/maior, e o desvio precisa ser rápido. A solução é **combinar `slt` (ou `slti`) com `beq`/`bne`**, comparando o resultado com `$zero`:

| em C               | em MIPS                                          |
|--------------------|--------------------------------------------------|
| `if (a <  b) goto L` | `slt $t0, $s0, $s1` / `bne $t0, $zero, L`      |
| `if (a >= b) goto L` | `slt $t0, $s0, $s1` / `beq $t0, $zero, L`      |
| `if (a >  b) goto L` | `slt $t0, $s1, $s0` / `bne $t0, $zero, L`      |
| `if (a <= b) goto L` | `slt $t0, $s1, $s0` / `beq $t0, $zero, L`      |

(a = $s0, b = $s1)

A lógica: `slt` deixa 1 em $t0 se for menor, 0 se não for. Depois é só perguntar "$t0 é diferente de zero?" (bne) ou "$t0 é zero?" (beq). Para `>` é só trocar a ordem dos operandos, porque `a > b` é o mesmo que `b < a`.

**Truque do `$zero`**: o registrador `$zero` é sempre 0 e não pode ser alterado. Então `beq $zero, $zero, L` é **sempre verdadeiro**, vira um desvio incondicional. Útil se o exercício não deixar usar o `j`.




## Exercícios: "só pode com essas instruções"

pega as instrucoes, so pode com essas ai, nenhum a mais, resolva esse problema.

Ou seja: dado um código em C, traduzir para MIPS usando **somente** o conjunto permitido:
- Tipo-R: `add, sub, and, or, slt`
- Tipo-I: `lw, sw, beq, bne, addi, andi, ori, slti, lui`
- Tipo-J: `j` (se o professor liberar; senão usa `beq $zero, $zero, L`)

Repare que nem `sll` nem `mul` estão na lista. Então, para multiplicar o índice por 4, faz `add` duas vezes (i + i = 2i, 2i + 2i = 4i), igual ao exemplo do final do rascunho-01.

### Roteiro para resolver

1. **Mapear variáveis em registradores** (o enunciado geralmente dá: f = $s0, g = $s1...).
2. **Vetor está na memória**, então precisa de `lw`/`sw`. Endereço de `A[i]` = base + 4*i.
3. **if / while**: inverter a condição e desviar para "fora" (para o else ou para o fim do laço). Se em C é `if (i == j)`, em MIPS é `bne` para o else.
4. **< > <= >=**: `slt`/`slti` + `beq`/`bne` com `$zero` (tabela acima).
5. **Subtrair constante**: `addi` com número negativo.
6. **Constante maior que 16 bits**: `lui` + `ori`.
7. **Zerar registrador**: `add $t0, $zero, $zero` ou `addi $t0, $zero, 0`.

### Exercício 1: acesso a vetor

```c
A[12] = h + A[8];      // h = $s2, endereço base de A = $s3
```

```
lw  $t0, 32($s3)       # $t0 = A[8]     (8 × 4 = 32)
add $t0, $s2, $t0      # $t0 = h + A[8]
sw  $t0, 48($s3)       # A[12] = $t0    (12 × 4 = 48)
```

### Exercício 2: if / else

```c
if (i == j) f = g + h;
else        f = g - h;
// f = $s0, g = $s1, h = $s2, i = $s3, j = $s4
```

```
      bne $s3, $s4, Else    # condição invertida: se i != j, vai para o else
      add $s0, $s1, $s2     # f = g + h
      j   Fim               # pula o else (sem isso executaria os dois!)
Else: sub $s0, $s1, $s2     # f = g - h
Fim:
```

### Exercício 3: while com vetor

```c
while (save[i] == k)
    i += 1;
// i = $s3, k = $s5, endereço base de save = $s6
```

```
Loop: add  $t1, $s3, $s3    # $t1 = 2i
      add  $t1, $t1, $t1    # $t1 = 4i
      add  $t1, $t1, $s6    # $t1 = endereço de save[i]
      lw   $t0, 0($t1)      # $t0 = save[i]
      bne  $t0, $s5, Fim    # se save[i] != k, sai do laço
      addi $s3, $s3, 1      # i = i + 1
      j    Loop             # volta para testar de novo
Fim:
```

### Exercício 4: for somando um vetor (usa slt)

```c
soma = 0;
for (i = 0; i < n; i++)
    soma += A[i];
// soma = $s0, base de A = $s1, n = $s2, i = $t0
```

```
      add  $s0, $zero, $zero   # soma = 0
      add  $t0, $zero, $zero   # i = 0
Loop: slt  $t1, $t0, $s2       # $t1 = 1 se i < n
      beq  $t1, $zero, Fim     # se NÃO (i < n), sai
      add  $t2, $t0, $t0       # 2i
      add  $t2, $t2, $t2       # 4i
      add  $t2, $t2, $s1       # endereço de A[i]
      lw   $t3, 0($t2)         # $t3 = A[i]
      add  $s0, $s0, $t3       # soma += A[i]
      addi $t0, $t0, 1         # i++
      j    Loop
Fim:
```

### Exercício 5: maior de dois números

```c
if (a > b) max = a;
else       max = b;
// a = $s0, b = $s1, max = $s2
```

```
      slt $t0, $s1, $s0        # $t0 = 1 se b < a  (ou seja, a > b)
      beq $t0, $zero, Else     # se não for a > b, vai para o else
      add $s2, $s0, $zero      # max = a
      beq $zero, $zero, Fim    # desvio incondicional sem usar j
Else: add $s2, $s1, $zero      # max = b
Fim:
```

### Exercício 6: constante de 32 bits

```c
x = 4000000;    // x = $s0.  4.000.000 = 0x003D0900, não cabe em 16 bits
```

```
lui $s0, 0x003D          # $s0 = 0x003D0000
ori $s0, $s0, 0x0900     # $s0 = 0x003D0900
```

Por que `ori` e não `addi`? Porque o `addi` estende com sinal. Se a parte de baixo fosse, por exemplo, 0x8000 ou maior, o `addi` entenderia como número negativo e bagunçaria a parte de cima. O `ori` estende com zeros, então só "encaixa" os 16 bits de baixo.

### Exercício 7: testar se um número é par

```c
if ((x & 1) == 0) par = 1;
else              par = 0;
// x = $s0, par = $s1
```

```
      andi $t0, $s0, 1         # $t0 = último bit de x (0 = par, 1 = ímpar)
      addi $s1, $zero, 0       # par = 0
      bne  $t0, $zero, Fim     # se o bit for 1, é ímpar, mantém 0
      addi $s1, $zero, 1       # par = 1
Fim:
```
















Exemplo do professor:


Se (A = B) && (C != D | A = C)
Then
    A = 20
else
    A = 30


Aparentmente solucao que ele deu

BNE $t0, $S0, else
BEQ $t0, $t1, then
BNE $t1, $s1, then
else
addi $t0, $zero, 30
  J(Jump)  fim
then
addi $t0, $zerom 20






