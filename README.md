# Mini Shell
Uma pequena implementação de interpretador shell. É capaz de executar comandos em segundo plano,
encadear comandos lógicamente ou executar múltiplos comandos em sequência. Não possui modo
standalone tampouco operadores complexos como `if` ou `for`.

## Executando
Executar o programa com a flag -d ativa o modo de debug: dado um comando, será exibido o
resultado da análise léxica e a fila de comandos resultante. No modo normal, os comandos são
interpretados e executados sequencialmente.. No modo normal, todos os comandos são executados
normalmente.

## Estrutura
O programa lida com três tipos de dados principais:
- Argumentos - são strings que formam um comando;
- Separadores - são entradas que separam um comando do outro (`&` ou `;`):
    - `&`: o programa anterior é executado em plano de fundo;
    - `;`: o próximo programa é executado independentemente do sucesso ou fracasso do anterior.
- Operadores - são uma variação dos separadores: encerram um comando, mas ao invés de iniciar um
  novo comando na fila de execução, o próximo comando é salvo na estrutura do comando atual
  dada a operação lógica utilizada:
    - `&&`: o próximo programa será executado se, e somente se, o presente for executado com
            sucesso;
    - `||`: o próximo programa será executado se, e somente se, o presente não for executado com
            sucesso;
  Em casos como `c1 || c2 && c3`, `c3` só será executado se ao menos 1 dos operandos de `||` for
  verdadeiro.

Todos os programas inseridos são executados em processos filhos.

## Implementação
O Mini Shell utiliza uma abordagem híbrida entre uma fila simples e uma AST (Abstract Syntax Tree).
Todo comando pode possuir ponteiros para comandos encadeados em caso de sucesso (&&) ou fracasso
(||). Estes comandos encadeados possuem prioridade de execução sobre a fila principal.

Diferente de shells tradicionais, o Mini Shell não utiliza subshells recursivos para resolver
agrupamentos. Os comandos são "achatados" (flattened), o que permite que o processo principal
mantenha a paternidade direta de todos os processos filhos, simplificando o controle de jobs
e a captura de status de saída. As relações lógicas são resolvidas de forma satisfatóriamente
equivalente à resolução recursiva com subshells.

## Limitações
Não há nenhum operador built-in além dos supracitados. Isso implica o não funcionamento de
comandos como `exit` ou `cd`. Observe que um built-in é um comando de um Shell, e não um
executável real.

Ironicamente, `false` e `true` não são operadores, mas executáveis binários reais. Dessarte,
o Mini Shell consegue lidar com eles nas expressões!

### Subshells
Em um shell totalmente funcional, comandos como `(c1 && c2)&`, agrupados por parênteses, são
interpretados como um subshell. Isso é, o processo filho criado executa, na verdade, o próprio
executável do shell em modo standalone — executa somente esse único comando e depois encerra.

Para as finalidades deste laboratório, o agrupamento por parênteses serve apenas para viabilizar
comandos como `(c1 &) && c2`. Dessa forma, o comando `(c1 && c2)&` é equivalente a
`(c1 &) && (c2 &)`. As portas lógicas funcionam até certo ponto: elas são todas distribuídas
e subcomandos são achatados.

Em implementações de interpretadores Shell reais, todo grupo de comando (envelopado por
parênteses) não é achatado, mas sim passado como argumento para o próprio executável do Shell.
Veja: ao invés de executar a cadeia de comando, o Shell irá executar, no processo filho,
ele mesmo e passará a cadeia de comandos como argumento, executando em modo standalone ao invés
do modo interativo ao qual temos acesso pelo terminal. Em suma, um Shell real resolve os grupos
recursivamente!
