# Introdução a programação funcional com War game

Durante um freela, o cliente pediu para que fosse implementado  o jogo `War` em três linguagens de programação:
1. `Elixir`
2. `Rust`
3. `Clojure`

Essas são minhas linguagens favoritas, não necessariamente pela sintaxe mas pelo modelo diferente de se programar, pois todas pertencem ao paradigma funcional, mesmo que parcialmente como no caso de `Rust`, mas também possuem suas peculiaridades.

Durante essa série de artigos, será explicado minuciosamente as decisões de implementação, sempre tentando usar o máximo de cada linguagem de programação.

Porém antes, nos artigos iniciais, gostaria de introduzir as linguagens de programação utilizadas e o paradigma funcional, além de descrever como funciona o jogo de cartas War.

## Boas vindas ao Paradigma Funcional

Muitas pessoas se assustam quando o tópico é programação funcional (FP). De fato, tanto no mercado quanto no ensino superior, o paradigma padrão a ser usado é o imperativo/estrutural em conjunto com o Orientado a Objetos (OO), mas o que essa sopa de palavras quer dizer, na prática?

## Senta que lá vem história...

Vocês já devem ter ouvido que Alan Turing, pai da computação moderna, descreveu um modelo computacional no qual todos (bem, quase todos...) os sistemas computacionais se baseiam até hoje, certo? Esse modelo foi descrito conforme a `Máquina de Turing`. Mas como ela funciona?

### Máquina de Turing

A Máquina de Turing é um modelo matemático descrito na tese de Church-Turing.

> Falaremos sobre Alonzo Church logo a seguir!

Esse modelo matemático tem a capacidade de descrever qualquer algoritmo, apesar de ser um modelo relativamente simples.

Para entendê-la, imagine uma fita horizontal, de tamanho variável, separada por células que possuem uma marcação, que se chama "alfabeto" da máquina. Nesse exemplo vamos usar `0` e `1`.

Em conjunto existe uma máquina, imagine uma caixa, onde possui uma agulha, que é a "cabeça" da máquina, permitindo ler e modificar a marcação da célula atual. A mesma consegue se movimentar por essa fita e realizar comandos por ela, como:

- mova para a esquerda;
- mova para a direita;
- deixe o valor da célula como está;
- modifique a célula;
- apague a marcação desta célula;

A célula onde a caixa está seria seu estado atual e o propósito da máquina é chegar no "estado de espera", que representaria o final do algoritmo. Portanto, os valores das marcações nas células após a execução dos comandos direcionados à caixa representa a solução da computação/algoritmo.

### Cálculo Lambda

Lembra do Alonzo Church citado anteriormente? O mesmo foi orientador do Turing e também tinha proposto um modelo matemático para computações, diferente da máquina de Turing.

> Hoje em dia entendemos que os dois modelos são equivalentes. Logo, toda computação realizada por uma Máquina de Turing pode ser computada pelo Cálculo Lambda e vice-versa!

O modelo consiste na definição de termos e na [redução](https://pt.wikipedia.org/wiki/C%C3%A1lculo_lambda#Redu%C3%A7%C3%A3o) dos mesmos. Na sua versão mais simples, descrita na década de 30, os termos são construídos seguindo as seguintes regras:

-  variável, que representa um parâmetro ou um valor lógico/matemático;
- abstração, que representa a definição de uma função, como: `λx. M`, onde `M` é um outro termo qualquer e `x` um parâmetro;
- aplicação de uma função, como `M N`, que aplica a função `M` (um termo) sobre outro termo `N`.

Esse modelo matemático é a base do paradigma funcional! A seguir, vamos implementar lógica booleana usando apenas as regras citadas acima:

```dark
true = λx.λ.y x
false = λx.λy y

not = λx.x false true
and = λx.λy.x y false
or = λx.λy.x true y
```

Para ficar mais entendível, vamos usar uma linguagem de programação de alto nível (`JavaScript`):

```js
const TRUE = x => y => x
const FALSE = x => y => y
```

Se utilizando das funções lambda ([arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)), que possuem esse nome de forma proposital, criamos duas funções que representam os valores lógicos "verdadeiro", que retorna o primeiro termo recebido, e "falso" , que retorna o segundo termo passado como parâmetro.

Vamos agora criar as operações de negação (`not`), conjunção (`and`) e disjunção inclusiva (`or`):

```js
const NOT = termo => termo(FALSE)(TRUE)

// NOT(TRUE) - [Function: FALSE]
// NOT(FALSE) - [Function: TRUE]
```

A negação é a definição de uma função lambda onde recebe um termo, e aplica esse termo aos termos `FALSE` e `TRUE`, respectivamente. 

Como `TRUE` retorna o primeiro termo recebido, caso o termo passado para `NOT` seja `TRUE`, o termo a ser resolvido é `FALSE`, e contrário acontece caso o parâmetro de `NOT` seja `FALSE`!

Agora, vamos para a conjunção lógica (`and`):

```js
const AND = termo => outro_termo => termo(outro_termo)(FALSE)

// AND(TRUE)(TRUE) - [Function: TRUE)
// AND(TRUE)(FALSE) - [Function: FALSE)
// AND(FALSE)(TRUE) - [Function: FALSE)
// AND(FALSE)(FALSE) - [Function: FALSE)
```

Se utilizando do que chamamos de "avaliação de curto-circuito", o termo `AND` recebe dois termos e aplica o primeiro sobre o segundo e o termo `FALSE`.

Por fim, vamos implementar a disjunção lógica (`or`), que é bem semelhante:

```js
const OR = termo => outro_termo => termo(TRUE)(outro_termo)

// OR(TRUE)(TRUE) - [Function: TRUE)
// OR(TRUE)(FALSE) - [Function: TRUE)
// OR(FALSE)(TRUE) - [Function: TRUE)
// OR(FALSE)(FALSE) - [Function: FALSE)
```

Com essa definições, podemos expandir ainda mais e definir a operação de controle de fluxo `if`:

```js
const IF_THEN_ELSE = predicado => then => else_branch => predicado(then)(else_branch)

// IF_THEN_ELSE(TRUE)(1)(2) - 1
// IF_THEN_ELSE(FALSE)(1)(2) - 2

// IF_THEN_ELSE(NOT(TRUE))(1)(2) - 2
// IF_THEN_ELSE(NOT(FALSE))(1)(2) - 1

// IF_THEN_ELSE(AND(TRUE)(TRUE))(1)(2) - 1
// IF_THEN_ELSE(AND(TRUE)(FALSE))(1)(2) - 2
// IF_THEN_ELSE(AND(FALSE)(TRUE))(1)(2) - 2
// IF_THEN_ELSE(AND(FALSE)(FALSE))(1)(2) - 2

// IF_THEN_ELSE(OR(TRUE)(TRUE))(1)(2) - 1
// IF_THEN_ELSE(OR(TRUE)(FALSE))(1)(2) - 1
// IF_THEN_ELSE(OR(FALSE)(TRUE))(1)(2) - 1
// IF_THEN_ELSE(OR(FALSE)(FALSE))(1)(2) - 2
```

A operação `IF` recebe um primeiro termo que chamamos de "predicado", que será avaliado em `TRUE` ou `FALSE`. Caso o termo avaliado seja `TRUE`, o termo `then` é retornado, caso contrário o termo `else_branch` é devolvida.

O que acabamos de implementar é equivalente à:

```js
const predicado = true;
const then = 1;
const else_branch = 2;

if (predicado) {
  return then;
} else {
  return else_branch;
}
```

## O que torna um linguagem "funcional"?

Para uma linguagem de programação ser considerada funcional, ela precisa atender alguns requisitos. É importante notar, que mesmo que uma linguagem não pertença ao paradigma funcional, a mesma pode implementar funcionalidades funcionais, como `Python` e `JavaScript`, que pertencem ao paradigma Orientado a Objetos porém possuem características do paradigma funcional!

Vamos entender esses requisitos!

### Imutabildiade

O conceito mais básico de uma linguagem funcional é ter seus valores imutáveis. Isso significa que não é possível modificar o valor de termo em memória, apenas criar novos valores a partir dele. Alguns exemplos usando `TypeSript` e `Elixir`:

```ts
type User = {
  id: string,
  name: string
};

const user: User = Object.freeze({id: "123", name: "Joe"});

user.id = "431"; // válido
user.name = null; // válido
user.x = true; // inválido
```

Uma das maneiras mais primitivas de exigir imutabilidade dentro do universo `JS/TS` é usando `Object.freeze`. Essa função força um objeto a ter um certo formato, porém permite que os valores de uma chave válida seja modificada em memória, o que não queremos! 

```ts
type User = ReadOnly<{id: string, name: string}>;

const user: User = {id: "um-id", name: "Dantas";

user.id = null // erro de compilação
user.x = true // erro de compilação
```

E caso quisermos atualizar nosso `user`?

```ts
function updateUser(user: User, params: Partial<User>): User {
  return Object.assign({}, user, params);
}

const newUser = updateUser(user, {id: "outro-id"})

console.log(newUser.name == user.name) // true
console.log(newUser.id == user.id) // false

newUser.id = "123" // erro de compilação
```

Agora sim temos imutabilidade! Mas me parece um tanto quanto verboso :thinking:. Vamos ver o mesmo exemplo em `Elixir`:

```elixir
defmodule User do
  @enforce_keys [:id, :name]
  defstruct [:id, :name]

  def update_user(user, params) do
    Map.merge(user, params)
  end
end

user = %User{id: "123", name: "Joe"}

user.id = "4321" # erro de compilação

new_user = User.update_user(user, %{id: "4321", name: "Dantas"})

new_user == user // false
user.id == "123" // true
```

Neste exemplo foi criado um `Struct` para forçar um formato, porém a mesma lógica se aplica a hashmaps planos! Todo termo em `Elixir` é imutável, ou seja, sempre que precisar atualizar um termo, um novo será gerado. As expressões na linguagem já foram implementadas para retornar novos termos, sendo muito menos verboso!

### Funções como valores de primeira classe

Outra característica básica de uma linguagem de programação funcional é que as funções são valores da mesma ordem dos tipos primitivos como `integer` ou `string`. Isso significa que funções podem ser passadas como parâmetro de outra função ou serem retornadas de outra função. Vamos acompanhar alguns exemplos:

```ts
const inc = x => x + 1;

const numbers = [1, 2, 3, 4, 5];

numbers.map(inc); // [2, 3, 4, 5, 6]
```

Passamos a função `inc`, que incrementa o valor de um termo, como parâmetro da função `map`, que itera sobre um array de termos e aplica a função recebida em cada termo, retornando um novo valor modificado. Vamos ver uma função retornando outra:

```ts
function soma_parcial(x) {
  return y => x + y;
}

const soma5 = soma_parcial(5);

soma5(10) // 15;
```

Neste exemplo criamos uma função `soma _parcial`, que recebe um número e devolve uma outra função, que também recebe um número, onde a avaliação dessa ultima retorna a soma dos dois números.

Chamamos essa técnica de `closure`, quando uma função interna utiliza o contexto do nível acima dentro do próprio escopo. O termo `x`, criado pelo contexto da função `soma_parcial` fica guardado na memória até a segunda função que é retornada, usá-lo.

Portanto, quando criamos o termo `soma5`, fazemos o que é chamado de "aplicação parcial", pois podemos entender que a função `soma_parcial` recebe dois parâmetros, porém apenas passamos o primeiro! Neste caso `x` terá o valor de `5`.

Como o retorno da aplicação parcial é uma outra função, `soma5` pode receber outro termo, que no exemplo passamos `10`, no que resulta na avaliação de `x = 5 + y = 10`, sendo retornado `15`.

### Funções puras e Transparência Referencial

Uma função pura é uma função que dado o mesmo termo como parâmetro, o mesmo resultado será retornado. Para ser pura, a função também não pode modificar o contexto externo dela. Vamos ver alguns exemplos:

```ts
let x = 10;

function somaX(y) {
  return x + y;
}
```

Essa função é impura, pois a mesma modifica/utiliza o contexto externo a si mesma!

Um exemplo claro de função pura são as operações matemáticas, como soma, multiplicação, subtração e divisão. Todas elas recebem dois parâmetros e aplica a operação sobre os dois. Dado os mesmos parâmetros, o mesmo termo será retornado:

```ts
const soma = (x, y) => x + y;

soma(1, 2) // 3
soma(1, 2) // 3
// ... assim infinitamente
```

Com isso, conseguimos introduzir outro conceito: Transparência Referencial, acontece quando podemos substituir a chamada de uma função pelo seu retorno, uma vez que dado o mesmo parâmetro, será retornado o mesmo resultado. Uma função só pode sofrer transparência referencial quando a mesma for pura!

### Recursão

Como linguagens de programação funcionais são imutáveis, iterar sobre uma coleção de termos usando `loops` perde seu significado. Por isso, o mais comum quando precisamos iterar sob uma coleção é usarmos recursão, que é o ato de uma função chamar a si mesma, criando uma pilha de execução! Vamos ver a diferença entre loops e recursão:

```ts
function fatorial(number) {
  let fact = 1;

  if (number == 0) return 1;

  for (let i = 1; i <= number; i++) {
    fact *= i;
  }

  return fact;
}
```

Com um `for` loop, criamos uma variável de estado `fact` que é modificada a cada iteração do loop com um novo valor do fatorial calculado. Vamos ver como seria a implementação com recursão:

```ts
function factorial(number) {
  if (number < 2) return 1;

  return number * factorial(number - 1);
}
```

Apesar de ter menos código, a lógica por trás da recursão pode ser um tanto quanto mais complexa. Toda recursão precisa de uma "condição de parada", que é uma condição base para que a recursão seja finalizada. No caso do cálculo do fatorial, o caso base é quando o número for menor que `2`, onde será retornado o valor `1`.

Depois, temos a definição da recursão, onde o resultado é o termo `number` multiplicado pelo resultado da chamada `factorial(number - 1)`. Veja na imagem a seguir a sequência de avaliação da recursão:


![Fatorial implementado com recursão](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4v91vbft07v2i3mflo3z.png)



Conforme a imagem, é possível notar que é criada uma pilha  de avaliação, onde a cada resultado a pilha diminui, fazendo o caminho "inverso" da avaliação.

## Conclusão

Neste artigo entendemos a história da programação funcional e as características que fazem com que uma linguagem de programação possa ser considerada "funcional". 

Vimos que a base da programação funcional é a definição e aplicação de funções, num modelo matemático, podendo ser funções puras ou não.

No próximo artigo, as linguagens `Elixir`, `Rust` e `Clojure` serão introduzidas, seguida pela introdução ao jogo de cartas War!

### Referências
- [Máquina de Turing Explicada - Computerphile (EN)](https://www.youtube.com/watch?v=dNRDvLACg5Q)
- [Calculus Lambda - Computerphile (EN)](https://www.youtube.com/watch?v=eis11j_iGMs)
- [Lambda Calculus com JavaScript (PTBR)](https://medium.com/@ahlechandre/lambda-calculus-com-javascript-5db88c3b45a)
- [Lógica matemática (PTBR)](https://mundoeducacao.uol.com.br/matematica/logica-matematica-1.htm)
- [O que é imutabildiade (PTBR)](https://segredo.dev/o-que-e-imutabilidade/)
- [Qual é a melhor maneira de se utilizar funções puras (PTBR)](https://natahouse.com/pt/qual-e-a-melhor-maneira-de-utilizar-as-funcoes-puras)
- [Programação Funcional e Transparência Referencial (PTBR)](https://pt.stackoverflow.com/questions/74992/programa%C3%A7%C3%A3o-funcional-e-transpar%C3%AAncia-referencial)
- [Algoritmos: entendendo recursividade (PTBR)](https://henriquebraga92.medium.com/algoritmos-recursividade-recurs%C3%A3o-c4aff7291bb4)
