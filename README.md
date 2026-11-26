# 🌱 Horta tech

**Integrantes:** _ Beatriz Zorato Mendonça · Lucas Carvalho de Araújo · Lucas Perrone Borlin Alves· Luiz Filipe Ichiro Ogata · Yume Fernandes Lima

Projeto final de Compiladores (FMU, 2026): um compilador em Python para uma DSL de planejamento e cuidado de uma horta.
Todo o código está em [`horta_tech.ipynb`]

```
texto → léxico → sintático (AST) → semântico → TAC → otimização → VM
                                                          
```

## Como rodar

- **Google Colab:** envie `horta_tech.ipynb` e escolha *Ambiente de execução → Reiniciar e executar tudo*.
- **Local (Python 3.10+):** `pip install -r requirements.txt`, depois `jupyter notebook horta_tech.ipynb` e *Restart & Run All*.

## Ficha da linguagem

- **Nome:** HortaScript
- **Domínio:** planejamento e cuidado de uma horta
- **Tipos:** inteiro, decimal, texto, logico
- **Comandos do domínio:** `plantar <cultura>, <mudas>;`, `regar <cultura>, <litros>;`, `adubar <cultura>, <gramas>;`, `colher <cultura>;`, `esperar <dias>;` (além de `mostrar`)

**Regras de negócio**

- mudas entre 1 e 200, a capacidade da horta (erro)
- cada rega usa mais de 0 e no máximo 10 litros (erro)
- cada adubação usa mais de 0 e no máximo 500 g (erro)
- só é possível usar culturas do catálogo (erro)
- não regar, adubar ou colher o que não foi plantado (erro)
- rega acima de 5 litros pode encharcar o solo (aviso)
- esperar mais de 30 dias sem cuidar da horta (aviso)
- só colher depois do ciclo da cultura (erro de execução)

**Catálogo de culturas**

| Cultura | Ciclo (dias) | kg por muda |
|---|---|---|
| alface | 45 | 0.30 |
| rucula | 40 | 0.15 |
| manjericao | 60 | 0.10 |
| couve | 70 | 0.50 |
| cenoura | 80 | 0.12 |
| tomate | 90 | 2.00 |

Os valores do catálogo são didáticos, não uma referência agronômica.

## Exemplo

```
plantar "alface", 10;
regar "alface", 2.5;
esperar 45;
colher "alface";
```

```
⚠️  Aviso (linha 3): 45 dias sem cuidar da horta
[dia   0] 🌱 plantadas 10 mudas de alface
[dia   0] 💧 alface regada com 2.5 L
[dia  45] ⏳ passaram 45 dias
[dia  45] 🧺 colhidos 3.00 kg de alface
--- Resumo da horta ---
mudas plantadas  10
regas            1
água (L)         2.50
adubo (g)        0.00
colheitas        1
produção (kg)    3.00
dias             45
canteiros ativos nenhum
```

### Mensagens de erro

```
Erro léxico (linha 1, coluna 9): caractere inesperado "'"
   💡 Dica: a HortaScript usa aspas duplas: "alface".
Erro sintático (linha 1, coluna 18): encontrei '3'; esperava: '!=', '%', '*', '+', ',', '-', '/', '<', '<=', '==', '>', '>=', 'e', 'ou'
   💡 Dica: separe os dois argumentos com ','.
Erro semântico (linha 1): cultura 'alfaci' não está no catálogo
   💡 Dica: você quis dizer 'alface'?
Erro de execução (linha 1): colheita antes da hora: tomate tem 10 dias e precisa de 90
   💡 Dica: adicione 'esperar 80;' antes de colher.
```

## Tokens

| Token | Padrão |
|---|---|
| `INTEIRO` | `inteiro` |
| `DECIMAL` | `decimal` |
| `TEXTO` | `texto` |
| `LOGICO` | `logico` |
| `SE` | `se` |
| `SENAO` | `senao` |
| `ENQUANTO` | `enquanto` |
| `PLANTAR` | `plantar` |
| `REGAR` | `regar` |
| `ADUBAR` | `adubar` |
| `COLHER` | `colher` |
| `ESPERAR` | `esperar` |
| `MOSTRAR` | `mostrar` |
| `FUNCAO` | `funcao` |
| `RETORNAR` | `retornar` |
| `VERDADEIRO` | `verdadeiro` |
| `FALSO` | `falso` |
| `E` | `e` |
| `OU` | `ou` |
| `NAO` | `nao` |
| `NUM_DEC` | `\d+\.\d+` |
| `NUM_INT` | `\d+` |
| `CADEIA` | texto entre aspas duplas |
| `NOME` | `[a-zA-Z_]\w*` |
| `IGUAL` | `==` |
| `DIFERENTE` | `!=` |
| `MENOR_IGUAL` | `<=` |
| `MAIOR_IGUAL` | `>=` |
| `MENOR` | `<` |
| `MAIOR` | `>` |
| `ATRIB` | `=` |
| `MAIS` | `+` |
| `MENOS` | `-` |
| `VEZES` | `*` |
| `DIVIDIR` | `/` |
| `RESTO` | `%` |
| `ABRE_PAR` | `(` |
| `FECHA_PAR` | `)` |
| `ABRE_CHAVE` | `{` |
| `FECHA_CHAVE` | `}` |
| `VIRGULA` | `,` |
| `PONTO_VIRG` | `;` |

Comentários `// ...` e `/* ... */` e espaços são ignorados.

## Gramática (Lark, LALR)

```lark
programa: item*
?item: funcao | comando
funcao: FUNCAO tipo NOME ABRE_PAR [params] FECHA_PAR bloco
params: param (VIRGULA param)*
param: tipo NOME
!tipo: INTEIRO | DECIMAL | TEXTO | LOGICO

?comando: declaracao | atribuicao | se | enquanto | plantar | regar | adubar
        | colher | esperar | mostrar | retornar
declaracao: tipo NOME ATRIB expr PONTO_VIRG
atribuicao: NOME ATRIB expr PONTO_VIRG
se: SE ABRE_PAR expr FECHA_PAR bloco [SENAO bloco]
enquanto: ENQUANTO ABRE_PAR expr FECHA_PAR bloco
bloco: ABRE_CHAVE comando* FECHA_CHAVE
plantar: PLANTAR expr VIRGULA expr PONTO_VIRG
regar: REGAR expr VIRGULA expr PONTO_VIRG
adubar: ADUBAR expr VIRGULA expr PONTO_VIRG
colher: COLHER expr PONTO_VIRG
esperar: ESPERAR expr PONTO_VIRG
mostrar: MOSTRAR expr (VIRGULA expr)* PONTO_VIRG
retornar: RETORNAR expr PONTO_VIRG

?expr: ou_expr
?ou_expr: e_expr | ou_expr OU e_expr          -> binario
?e_expr: nao_expr | e_expr E nao_expr         -> binario
?nao_expr: comparacao | NAO nao_expr          -> unario
?comparacao: soma | soma op_comp soma         -> binario
!op_comp: IGUAL | DIFERENTE | MENOR | MAIOR | MENOR_IGUAL | MAIOR_IGUAL
?soma: produto | soma (MAIS|MENOS) produto    -> binario
?produto: fator | produto (VEZES|DIVIDIR|RESTO) fator -> binario
?fator: atomo | MENOS fator                   -> unario
?atomo: NUM_INT -> lit_int
      | NUM_DEC -> lit_dec
      | CADEIA -> lit_texto
      | VERDADEIRO -> lit_logico
      | FALSO -> lit_logico
      | NOME -> var
      | NOME ABRE_PAR [argumentos] FECHA_PAR -> chamada
      | ABRE_PAR expr FECHA_PAR -> parenteses
argumentos: expr (VIRGULA expr)*
```

Precedência, da menor para a maior: `ou`, `e`, `nao`, comparação, `+ -`, `* / %`, `-` unário, átomo.

## Fases

- **Semântico:** escopo de bloco com sombreamento; `inteiro → decimal` é a única coerção; avisos para variável não usada, rega acima de 5 L e espera acima de 30 dias; sugestões com `difflib`.
- **TAC:** `se` e `enquanto` com `ifFalse` (condição invertida); instruções `PLANTAR`, `REGAR`, `ADUBAR`, `COLHER`, `ESPERAR`, `MOSTRAR`.
- **Otimização:** dobramento de constantes, propagação de constantes e cópias, simplificação algébrica, eliminação de temporários e de código morto.
- **VM:** pilha de chamadas, estado da horta, estatísticas e erros de execução.

| Programa | Instruções sem otimizar | Otimizado |
|---|---|---|
| alface | 114 | 67 |
| horta mista | 230 | 195 |
| funções | 128 | 120 |

## Decisões e limitações

- Toda variável precisa de valor inicial.
- `/` sempre produz `decimal`; `%` só aceita inteiros.
- `e` e `ou` avaliam os dois lados (não há curto-circuito).
- Funções são declaradas no nível principal, enxergam só parâmetros e variáveis próprias e precisam retornar em todos os caminhos.
- A checagem "não foi plantada" no semântico segue a ordem do texto e só vale para nomes escritos direto no comando; os demais casos são verificados pela VM, inclusive regar depois de colher.
- Há um limite de 20.000 instruções para detectar laço infinito.

## Bônus implementados

Assembly com registradores e *spilling* (conferido por um simulador) · funções com recursão · "você quis dizer...?" · GitHub Actions · gráficos da execução.

## Testes

A célula do Passo 8 roda 21 casos e falha se algum não passar. O workflow em `.github/workflows/testes.yml` executa o notebook inteiro a cada push.

