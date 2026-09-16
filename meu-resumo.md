# Arquitetura de Computadores - Aula 5: MIPS

## resumo_focal

### A ideia que destrava tudo

Três fatos seguram o resto da matéria inteira:

1. A CPU só calcula sobre **registradores**. São 32, cada um com 32 bits. Pense neles como as únicas "variáveis" que a máquina consegue somar, comparar, etc.
2. A **memória** é um lugar separado e mais lento. Nada é calculado lá. Você move dado para o registrador (`lw`), calcula, e devolve (`sw`).
3. Toda instrução tem **exatamente 32 bits**. Isso força tudo a ser pequeno e regular, e explica várias limitações que parecem arbitrárias.

Se você segurar esses três, o resto é consequência.

### Como ler este resumo (os 4 níveis)

- **Nível 1 - o vocabulário:** o que cada instrução faz, uma por uma.
- **Nível 2 - frases simples:** aritmética só com registradores, C ao lado do MIPS, e as peças de hardware que fazem uma soma acontecer.
- **Nível 3 - textos de verdade:** memória, vetores com índice `i`, `if`, laços, `switch`, funções.
- **Nível 4 - o chão:** como as instruções viram binário, os endereçamentos, o mapa de memória, o caminho do código ao executável e o desempenho.

A ordem é proposital: você aprende a **ler e escrever** MIPS antes de aprender como ele é **codificado**. Dá para escrever em português sem saber fonética.

---

# Nível 1 - O vocabulário

Todas as instruções que a disciplina usa. Convenção de escrita: o **destino vem primeiro**.

## 1.1 Aritmética e lógica (operam entre registradores)

| Instrução | Sintaxe | Faz |
|---|---|---|
| `add` | `add rd, rs, rt` | `rd = rs + rt` |
| `sub` | `sub rd, rs, rt` | `rd = rs - rt` |
| `and` | `and rd, rs, rt` | `rd = rs & rt` (bit a bit) |
| `or` | `or rd, rs, rt` | `rd = rs \| rt` (bit a bit) |
| `slt` | `slt rd, rs, rt` | `rd = 1` se `rs < rt`, senão `rd = 0` |

`slt` é *set on less than*: não desvia, só deixa 0 ou 1 num registrador. É a peça que, junto com desvios, constrói `<`, `>`, `<=`, `>=` (Nível 3).

## 1.2 Versões imediatas (um operando é constante)

Constante embutida na instrução é mais rápida que buscar da memória.

| Instrução | Sintaxe | Faz |
|---|---|---|
| `addi` | `addi rt, rs, imm` | `rt = rs + imm` |
| `andi` | `andi rt, rs, imm` | `rt = rs & imm` |
| `ori` | `ori rt, rs, imm` | `rt = rs \| imm` |
| `slti` | `slti rt, rs, imm` | `rt = 1` se `rs < imm` |
| `lui` | `lui rt, imm` | carrega `imm` nos **16 bits altos** de `rt`, zera os de baixo |

Duas ausências que você precisa contornar:
- **Não existe `subi`.** Subtrair constante é `addi` com negativo: `addi $t0, $t0, -1`.
- **Não existe multiplicação no conjunto.** Multiplicar por 4 (o caso comum, para índice de vetor) são dois `add`: `x+x = 2x`, `2x+2x = 4x`.

## 1.3 Memória (movem dado entre registrador e RAM)

| Instrução | Sintaxe | Faz |
|---|---|---|
| `lw` | `lw rt, offset(rs)` | `rt = Memória[rs + offset]` (load word, 4 bytes) |
| `sw` | `sw rt, offset(rs)` | `Memória[rs + offset] = rt` (store word) |

O `rs` é o **endereço base**, o `offset` é somado a ele. Existem variantes por tamanho, já que a memória trabalha em blocos de 1, 2 ou 4 bytes: `lb`/`sb` (byte), `lh`/`sh` (meia palavra), `lw`/`sw` (palavra).

## 1.4 Desvio e controle (mudam o `pc`)

O `pc` (program counter) aponta a próxima instrução. Desvio é o que muda esse fluxo, e é o que separa um computador de uma calculadora: sem desvio não há `if`, `while`, nem função.

| Instrução | Sintaxe | Faz |
|---|---|---|
| `beq` | `beq rs, rt, L` | se `rs == rt`, `pc` vai para `L` (*branch if equal*) |
| `bne` | `bne rs, rt, L` | se `rs != rt`, `pc` vai para `L` (*branch if not equal*) |
| `j` | `j L` | vai para `L`, sempre (desvio incondicional) |
| `jal` | `jal L` | salva o endereço de volta em `$ra`, depois vai para `L` (chama função) |
| `jr` | `jr rs` | `pc = rs` (volta de função, com `jr $ra`) |

Esse é o alfabeto inteiro. O resto do resumo é combinar essas letras.

---

# Nível 2 - Frases simples: aritmética em registradores

Aqui só usamos registradores. Sem memória, sem desvio ainda.

## 2.1 Do C para o MIPS

O átomo, `a = b + c` (com `a=$s0, b=$s1, c=$s2`):

```mips
add $s0, $s1, $s2    # a = b + c
```

Agora uma expressão composta, `f = (g + h) - (i + j)`:

```c
f = (g + h) - (i + j);   // f=$s0, g=$s1, h=$s2, i=$s3, j=$s4
```

```mips
add $t0, $s1, $s2    # $t0 = g + h
add $t1, $s3, $s4    # $t1 = i + j
sub $s0, $t0, $t1    # f = $t0 - $t1
```

A lição central: **uma instrução tem no máximo 3 registradores.** Expressão maior quebra em passos, usando temporários (`$t`) para guardar resultados intermediários. É como resolver conta grande anotando os parciais.

## 2.2 O que faz uma soma acontecer (as peças)

Para executar até um simples `add $s0, $s1, $s2`, o hardware precisa de:

| Componente | Papel |
|---|---|
| Banco de registradores | 32 registradores de 32 bits. Lê 2 (rs, rt), escreve 1 (rd). |
| ALU | Faz a conta: soma, subtração, and, or, comparação. |
| Unidade de Controle | Lê o código da instrução e comanda quem faz o quê (libera a ALU, autoriza a escrita no destino). |
| PC | Aponta a instrução atual; anda +4 para a próxima. |
| Memória de instruções | Guarda o programa. |

O fluxo de uma instrução:

```
PC ─► Memória de instruções ─► instrução (32 bits)
                                   │
        código da operação ─────►  Unidade de Controle ─► sinais de controle
        rs, rt ────────────────►  Banco de Registradores ─► ALU ─► resultado ─► volta ao Banco (rd)
```

Um **multiplexador (MUX)** costuma aparecer nesse caminho: é um seletor que escolhe qual entrada passa para a saída conforme um sinal de controle (por exemplo, escolher se o segundo operando da ALU vem de um registrador ou de um imediato).

## 2.3 Por que 32 bits, e como eles se dividem

Toda instrução é uma palavra de 32 bits. O que muda é o corte desses bits. Três cortes (formatos):

**Tipo-R** (aritmética e lógica entre registradores):

| campo | op | rs | rt | rd | shamt | funct |
|---|---|---|---|---|---|---|
| bits | 6 | 5 | 5 | 5 | 5 | 6 |

**Tipo-I** (um imediato de 16 bits, ou um deslocamento de memória):

| campo | op | rs | rt | imediato |
|---|---|---|---|---|
| bits | 6 | 5 | 5 | 16 |

**Tipo-J** (salto, só precisa de endereço):

| campo | op | endereço |
|---|---|---|
| bits | 6 | 26 |

De onde saem esses números:
- **op = 6 bits** dá 2^6 = **64** operações distintas.
- **registrador = 5 bits** dá 2^5 = **32**, exatamente o número de registradores.
- **imediato = 16 bits** com sinal cobre de -32768 a +32767.

(Como os 32 bits viram um binário concreto e um endereço de 32 bits a partir de 26, fica para o Nível 4. Por enquanto basta saber que os campos existem e cabem.)

---

# Nível 3 - Textos de verdade

Agora combinamos tudo: memória, índices, decisão, repetição, funções.

## 3.1 Vetores moram na memória

Registrador guarda um valor. Vetor é grande e mora na RAM. Regra de endereço: cada palavra tem 4 bytes, o offset é contado em bytes, então:

```
endereço de A[i] = endereço-base + 4*i
```

`g = h + A[8]` (base de A em `$s3`, `h=$s2`):

```mips
lw  $t0, 32($s3)     # $t0 = A[8]      (8 * 4 = 32)
add $s1, $s2, $t0    # g = h + A[8]
```

Quando o índice é **variável**, `A[i]`, primeiro calcula `4*i` (dois `add`, já que não há multiplicação):

```mips
add $t1, $s4, $s4    # 2i
add $t1, $t1, $t1    # 4i
add $t1, $t1, $s3    # base + 4i = endereço de A[i]
lw  $t0, 0($t1)      # $t0 = A[i]
```

Dois cuidados de baixo nível que aparecem aqui: endereços de palavra são múltiplos de 4 (**alinhamento**), e a disciplina usa **little endian** (o byte menos significativo fica no menor endereço).

## 3.2 Decisão: `if` e `if/else`

Regra de ouro: **inverta a condição e desvie para fora do bloco.**

`if (i == j) f = g + h;`

```mips
     bne $s3, $s4, Fim    # se i != j, pula o bloco
     add $s0, $s1, $s2    # f = g + h
Fim:
```

`if (i == j) f = g + h; else f = g - h;`

```mips
      bne $s3, $s4, Else   # se i != j, vai pro else
      add $s0, $s1, $s2    # then: f = g + h
      j   Fim              # sem este j, os dois blocos executam!
Else: sub $s0, $s1, $s2    # else: f = g - h
Fim:
```

O `j` no fim do `then` é o erro clássico de esquecer.

## 3.3 Comparar `<`, `>`, `<=`, `>=`

Não existem `blt`, `bgt`, `ble`, `bge`. Comparar igualdade é barato no hardware, menor/maior não. A solução é `slt` (deixa 0 ou 1) seguido de `beq`/`bne` contra `$zero` (que vale sempre 0):

| em C (a=$s0, b=$s1) | em MIPS |
|---|---|
| `if (a < b)` | `slt $t0, $s0, $s1` / `bne $t0, $zero, L` |
| `if (a >= b)` | `slt $t0, $s0, $s1` / `beq $t0, $zero, L` |
| `if (a > b)` | `slt $t0, $s1, $s0` / `bne $t0, $zero, L` |
| `if (a <= b)` | `slt $t0, $s1, $s0` / `beq $t0, $zero, L` |

Para `>`, é só trocar a ordem dos operandos: `a > b` é o mesmo que `b < a`.

**Truque:** `beq $zero, $zero, L` é sempre verdadeiro, então funciona como desvio incondicional quando você não pode usar `j`.

## 3.4 Repetição: laços

`while (save[i] == k) i++;` (`i=$s3, k=$s5, base=$s6`):

```mips
Loop: add  $t1, $s3, $s3   # 2i
      add  $t1, $t1, $t1   # 4i
      add  $t1, $t1, $s6   # endereço de save[i]
      lw   $t0, 0($t1)     # save[i]
      bne  $t0, $s5, Fim   # se save[i] != k, sai
      addi $s3, $s3, 1     # i++
      j    Loop            # repete
Fim:
```

O `for` clássico, somando um vetor, junta índice + comparação com `slt`:

```c
soma = 0;
for (i = 0; i < n; i++) soma += A[i];   // soma=$s0, base=$s1, n=$s2, i=$t0
```

```mips
      add  $s0, $zero, $zero   # soma = 0
      add  $t0, $zero, $zero   # i = 0
Loop: slt  $t1, $t0, $s2       # i < n ?
      beq  $t1, $zero, Fim     # se não, sai
      add  $t2, $t0, $t0       # 2i
      add  $t2, $t2, $t2       # 4i
      add  $t2, $t2, $s1       # endereço de A[i]
      lw   $t3, 0($t2)         # A[i]
      add  $s0, $s0, $t3       # soma += A[i]
      addi $t0, $t0, 1         # i++
      j    Loop
Fim:
```

## 3.5 `switch` / `case`: tabela de saltos

`if/else` encadeado funciona para poucos casos. Para muitos (`switch`), usa-se uma **tabela de saltos**: um vetor na memória que guarda o **endereço** de cada `case`. Você calcula o índice, lê o endereço e salta com `jr`.

`switch (k)` com casos 0 a 3 (supondo `$t2 = 4` e `$t4 = endereço-base da tabela`):

```mips
      slt $t3, $s5, $zero   # k < 0 ?
      bne $t3, $zero, Exit  # fora da faixa, sai
      slt $t3, $s5, $t2     # k < 4 ?
      beq $t3, $zero, Exit  # k >= 4, sai
      add $t1, $s5, $s5     # 2k
      add $t1, $t1, $t1     # 4k
      add $t1, $t1, $t4     # endereço de Tabela[k]
      lw  $t0, 0($t1)       # $t0 = endereço do case
      jr  $t0               # salta para lá
L0:   ...                   # case 0
      j Exit
L1:   ...                   # case 1
      j Exit
L2:   ...                   # case 2
      j Exit
L3:   ...                   # case 3
Exit:
```

A checagem de faixa (`k < 0` e `k >= 4`) evita saltar para um endereço inválido.

## 3.6 Funções (procedimentos)

Convenção de registradores para chamadas:

| Registradores | Uso | Preservado? |
|---|---|---|
| `$a0`-`$a3` | argumentos de entrada (máx. 4) | não |
| `$v0`-`$v1` | valores de retorno | não |
| `$ra` | endereço de volta | sim |
| `$t0`-`$t9` | temporários | não |
| `$s0`-`$s7` | salvos | sim |

Chamar e voltar:

```mips
jal soma      # $ra = endereço da instrução seguinte; salta para soma
# ...
jr  $ra       # volta para quem chamou
```

**Função folha** (não chama outra), `f = (g+h) - (i+j)`:

```mips
leaf:
    sub $sp, $sp, 4       # abre espaço na pilha
    sw  $s0, 0($sp)       # salva $s0 (vai usar e precisa devolver intacto)
    add $t0, $a0, $a1     # g + h
    add $t1, $a2, $a3     # i + j
    sub $s0, $t0, $t1     # (g+h) - (i+j)
    add $v0, $s0, $zero   # resultado em $v0
    lw  $s0, 0($sp)       # restaura $s0
    add $sp, $sp, 4       # fecha a pilha
    jr  $ra
```

A **pilha** entra quando os registradores não bastam ou quando você precisa preservar valores. Regras:
- `$sp` (stack pointer) aponta o topo; a pilha **cresce para baixo** (endereços menores).
- Empilhar: `addi $sp, $sp, -4` e `sw`. Desempilhar: `lw` e `addi $sp, $sp, 4`.
- Se a função usa um `$s`, ela **tem que** salvar e restaurar. `$t` pode ser sobrescrito à vontade.

**Funções aninhadas e recursão:** quando uma função chama outra, o `$ra` e os `$a` seriam sobrescritos pela chamada de dentro. Solução: salvar `$ra` (e os `$a` que ainda vai usar) na pilha antes do `jal`, e restaurar depois. É exatamente o que o fatorial recursivo faz: empilha `$ra` e `$a0`, chama `fact(n-1)`, desempilha, multiplica.

---

# Nível 4 - O chão: como isso é feito

Aqui descemos para o hardware e o sistema. Nada novo de comportamento, só o mecanismo por baixo.

## 4.1 Instrução vira número

Para montar o binário, você precisa de três tabelas.

Número de cada registrador:

| reg | nº | reg | nº |
|---|---|---|---|
| $zero | 0 | $t8-$t9 | 24-25 |
| $at | 1 | $k0-$k1 | 26-27 |
| $v0-$v1 | 2-3 | $gp | 28 |
| $a0-$a3 | 4-7 | $sp | 29 |
| $t0-$t7 | 8-15 | $fp | 30 |
| $s0-$s7 | 16-23 | $ra | 31 |

`funct` das Tipo-R (todas têm `op = 0`):

| add | sub | and | or | slt | jr |
|---|---|---|---|---|---|
| 32 (0x20) | 34 (0x22) | 36 (0x24) | 37 (0x25) | 42 (0x2A) | 8 (0x08) |

`opcode` das outras:

| lw | sw | beq | bne | addi | andi | ori | slti | lui | j | jal |
|---|---|---|---|---|---|---|---|---|---|---|
| 35 | 43 | 4 | 5 | 8 | 12 | 13 | 10 | 15 | 2 | 3 |

**Traduzindo `add $t0, $s1, $s2`.** Atenção: escreve-se `rd, rs, rt`, mas no binário a ordem dos campos é `rs, rt, rd`.

```
op     rs($s1)  rt($s2)  rd($t0)  shamt  funct
0      17       18       8        0      32
000000 10001    10010    01000    00000  100000
```

Juntando de 4 em 4: `0000 0010 0011 0010 0100 0000 0010 0000` = **0x02324020**.

**Traduzindo `lw $t0, 32($s3)`** (`rs=$s3=19, rt=$t0=8, imediato=32`):

```
op(lw) rs($s3)  rt($t0)  imediato
35     19       8        32
100011 10011    01000    0000000000100000
```

= `1000 1110 0110 1000 0000 0000 0010 0000` = **0x8E680020**.

## 4.2 Por que 3 formatos, e detalhes de cada

Princípio "um bom projeto exige bons compromissos": queriam 32 bits fixos (busca e decodificação simples), mas instrução só de registradores não tem espaço para constante grande. Resposta: **tamanho fixo, formato variável.**

Detalhe elegante: `rs` e `rt` ficam **na mesma posição** nos tipos R e I. O hardware já começa a ler esses registradores antes de terminar de decodificar ("a simplicidade é favorecida pela regularidade").

No Tipo-I, o `rt` é o **destino** em `addi`, `lw`, etc., com duas exceções: em `sw` o `rt` é o valor a gravar, e em `beq`/`bne` é o segundo valor comparado.

**Extensão do imediato** (16 bits precisam virar 32):
- `addi`, `slti`, `lw`, `sw`, `beq`, `bne` estendem **com sinal** (copiam o bit mais alto; -1 vira 0xFFFFFFFF).
- `andi`, `ori` estendem **com zero** (0xFFFF vira 0x0000FFFF).

**Constante de 32 bits** não cabe em 16, então usa `lui` (parte alta) + `ori` (parte baixa):

```mips
lui $s0, 0x003D          # $s0 = 0x003D0000
ori $s0, $s0, 0x0900     # $s0 = 0x003D0900   (= 4.000.000)
```

Usa-se `ori` e não `addi` porque `addi` estende com sinal e estragaria a parte alta se a parte baixa passasse de 0x7FFF. O montador tem o pseudo `li` que faz esse par automaticamente.

## 4.3 Modos de endereçamento

Cinco maneiras de dizer onde está o operando:

1. **A registrador:** o valor está num registrador (`add`).
2. **Base + deslocamento:** endereço = registrador base + offset (`lw`, `sw`).
3. **Imediato:** o valor é uma constante na própria instrução (`addi`).
4. **Relativo ao PC:** usado por `beq`/`bne`.
5. **Pseudo-direto:** usado por `j`.

**Relativo ao PC (desvios condicionais):** o campo de 16 bits do `beq`/`bne` **não é o endereço** do label, é a **distância em instruções** a partir da instrução seguinte:

```
se a condição for verdadeira:  PC = (PC + 4) + (imediato * 4)
```

Faz sentido porque `if`/`while`/`for` quase sempre pulam para perto. Para longe, usa `j`.

**Pseudo-direto (`j`):** como transformar 26 bits em um endereço de 32?
1. Toda instrução tem 4 bytes, logo todo endereço termina em `00`. Esses 2 bits não precisam ser guardados: multiplica por 4. 26 + 2 = 28 bits.
2. Os 4 bits que faltam vêm dos 4 bits mais altos do PC atual.

```
endereço final = [4 bits altos do PC] [26 bits do campo] [00]
```

## 4.4 O mapa de memória

Do endereço mais alto ao mais baixo:

| Segmento | Endereço | Cresce |
|---|---|---|
| Pilha (stack) | `$sp` = 0x7fff fffc | para baixo |
| Dados dinâmicos (heap) | entre a pilha e os dados estáticos | para cima |
| Dados estáticos | `$gp` = 0x1000 8000 (base 0x1000 0000) | fixo |
| Texto (código) | `pc` = 0x0040 0000 | fixo |
| Reservado | 0 | fixo |

Pilha e heap crescem **um em direção ao outro** para aproveitar o espaço livre no meio. Se um invade o outro, é estouro.

## 4.5 Do código ao executável

```
programa.c
   │ Compilador
programa.asm  (assembly)
   │ Montador (assembler)
programa.o  (linguagem de máquina)  +  bibliotecas (.o)
   │ Ligador (linker)
programa.exe  (executável)
   │ Carregador (loader)
memória  (executa)
```

- **Montador:** traduz assembly para binário. Oferece **pseudo-instruções** (ex.: `move $t0, $t1` vira `add $t0, $zero, $t1`; `li` vira `lui`+`ori`), usa o registrador reservado `$at` como rascunho, e monta a **tabela de símbolos** (endereço de cada label).
- **Ligador:** junta os módulos num só, resolve referências entre eles e ajusta endereços (relocação).
- **Carregador:** copia o executável para a memória, cria o espaço de código e dados, e inicializa registradores (o `$sp` no topo da pilha).

## 4.6 Desempenho (o fechamento)

**Throughput x tempo de resposta:** tempo de resposta (latência) é quanto demora **uma** tarefa; throughput (vazão) é quantas tarefas terminam **por segundo**. Trocar por um processador mais rápido melhora os dois; adicionar processadores melhora só o throughput.

```
Desempenho = 1 / Tempo de execução
Speedup    = Tempo_antigo / Tempo_novo
```

**Lei de Amdahl:** melhorar uma parte só acelera a fração de tempo em que ela é usada.

```
Tempo_novo = (Tempo afetado / fator) + Tempo não afetado
```

Programa de 100 s com 80 s de multiplicação: para ficar 4x mais rápido (25 s), a multiplicação precisa ser 16x mais rápida; para 5x (20 s) é impossível, porque os outros 20 s continuam lá. Moral: **torne o caso comum mais rápido.**

**Tempo de CPU:**

```
Tempo de CPU = Nº de instruções * CPI * Tempo de ciclo
             = Nº de instruções * CPI / Frequência do clock
```

Nº de instruções depende do programa e do compilador; CPI depende da arquitetura; o ciclo depende do hardware. É por isso que o RISC aposta em instruções simples: CPI baixo e ciclo curto, mesmo que precise de mais instruções. Instrução simples também significa hardware simples, mais barato e mais fácil de paralelizar (pipeline).

---

## Anexo: os tropeços que mais custam ponto

- Escreve `add rd, rs, rt`, mas o binário é `rs, rt, rd`.
- Esquecer o `j` no fim do bloco `then` (os dois blocos executam).
- Usar `i` como offset em vez de `4*i` (offset de `lw`/`sw` é em bytes).
- Tentar `subi` (não existe, use `addi` negativo) ou multiplicar direto (some duas vezes).
- Trocar extensão com sinal por extensão com zero (`addi`/`lw` com sinal; `andi`/`ori` com zero).
- Achar que o imediato do `beq` é o endereço do label (é a distância em instruções a partir do PC+4).
- Numa função, sobrescrever `$s` ou `$ra` sem salvar na pilha.
