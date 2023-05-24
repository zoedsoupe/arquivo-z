# Sintaxe e especificação da linguagem Nix


Com o `Nix` devidamente apresentado no post anterior, agora irei introduzir a
sintaxe da linguagem!

Este artigo foi feito com a intenção de traduzir, resumidamente, a especificação oficial
da linguagem que pode ser encontrada no link no final do post.

## Paradigmas da linguagem

### "Preguiçosa"

Isso significa que as expressões somente são executadas caso realmente sejam necessárias.

Neste exemplo, a função `abort` nunca será executada:
```nix
let
  a = abort "nunca irá acontecer";
  b = "olá";
  c = "mundo";
in b + c
```

### Funcional

Programação Funcional é um paradigma de linguagens de computação onde a estrutura e elementos de
um programa são tratados como expressões matemáticas, evitando a mutabilidade de estado. Linguagens
funcionais também tendem a fazer parte do paradigma declarativo, onde a construção do programa
é baseado em expressões.

### Pura

Definimos uma função como pura quando o retorno dela é unicamente definido pelos seus argumentos
de entrada, ou seja, não há efeitos colaterais externos ou "observáveis".

## Especificação da linguagem

### Expressões

Quando um tutorial referencia uma **expressão Nix**, significa que ele está descrevendo a
definição de uma função com várias entradas. Porém, uma expressão Nix pode ser definida como
um simples texto, até uma função ou um conjunto de outras expressões!

### Tipos primitivos

#### Texto

Pode ser definido com aspas duplas `"` ou duas aspas simples `''` para mais de uma linha:
```nix
# também suportam interpolações
"Diga ${pkgs.hello.name}"

''primeira linha
  segunda linha
''
```

#### Números

Existem os tipos `interios` e `ponto de fltuação` que possuem precisão limitada:
```nix
5

1.18
```

#### Caminhos

Caminhos relativos serão convertidos em absolutos quando a expressão for executada:
```nix
./ola/mundo
# /caminho/asbsoluto/para/ola/mundo

<nixpkgs/lib>
# /caminho/para/nixpkgs/lib
```

#### URIs

URIs podem ser representadas independentemente, como:
```nix
let
  uri = https://github.com/Mdsp9070/dotfiles 
in fetchTarball { src = uri }
```

#### Booleanos

Também existem os booleanos `true` e `false`.

#### Nulidade

A nulidade é representada pela expressão `null`.

#### Funções

Todas as funções são anônimas com o formato: `argumento: expressão-Nix`, como: `x: x * x`.

Caso queira nomear a função, você pode declarar: `quadrado = x: x * x`. E para executar essa
função com argumento: `quadrado 3`;

Múltiplos argumentos podem ser exigidos no formato: `arg1: arg2: expressão-Nix`.

É comum que Conjuntos sejam passados como argumentos para funções, e é possível referenciar apenas
os argumentos necessários:
```nix
a_e_b = conjunto: conjunto.a + conjunto.b

a_e_b { a = "ola"; b = "mundo"; }

# pode ser reescrita dessa maneira:
a_e_b = {a, b}: a + b
```

Também é possível definir argumentos padrões:
```nix
soma = { a ? 1, b ? 2 }: a + b

soma {}
# 3

soma { a = 3; }
# 5
```

Caso um argumento não declarado no cabeçalho da função seja fornecido, ocorrerá um erro, porém
isso pode ser evitado caso use reticências `...`:
```nix
soma = { a, b }: a + b
soma { a = 5; b = 2; c = 10; }
# error: anonymous function at (string):1:2 called with unexpected argument 'c', at (string):1:1

soma = { a, b, ... }: a + b
soma { a = 5; b = 2; c = 10; }
# 7
```

Você também pode referenciar todos os argumentos num Cojunto com a notação `@`:
```nix
soma = args@{ a, b, ... }: a + b + args.c
soma { a = 5; b = 2; c = 10; }
# 17
```

#### Listas

É um conjunto de diversos valores separados por espaços em branco, que podem ter vários tipos:
```nix
[ 123 ./exemplo.nix "abc" (f { x = y; }) ]
```

Define uma lista com quatro elementos. Note que a função `f` está dentro parênteses, caso contrário,
haveria cinco elementos na lista, sendo o quarto uma função `f` previamente declarada e o quinto um
conjunto.

Os valores de uma lista são "preguiçosos", mas seu tamanho não.

#### Conjuntos

Esta estrutura é o coração da linguagem, já que o foco principal é criar derivações, que são apenas conjuntos
de atributos que são passados para scripts construtores.

Conjuntos são estrutas de nome/valor definidos dentro de chaves:
```nix
{ x = 123;
  texto = "hello";
  y = f { tudo = 42; };
}
```

A ordem dos atributos não é importante e é uma boa prática não haver repetições nos nomes das chaves.

Cada atributo pode ser acessado com o operador `.`:
```nix
{ a = "Foo"; b = "Bar" }.a
```

Retorna `"Foo"`. Também é possível definir um valor padrão caso um atributo não exista com a palavra-chave `or`:
```nix
{ a = "Foo"; b = "Bar" }.c or "xyz"
```

As chaves também podem ser definidas por textos entre aspas duplas `"`:
```nix
{ "exemplo de ${chave}" = "Foo"; }."exemplo de ${chave}"
```

Caso a chave de atributo tenha o valor nulo, o atributo não é executado:
```nix
{ ${if exemplo then "bar" else null} = true; }
```

Retorna `{}` caso `exemplo` seja falso.

Conjuntos podem ter chaves que referem-se umas às outras com o construtor `rec`:
```nix
rec {
  x = y;
  y = 123l
}.x
```

### Operadores

```nix
# acesso
e . atributo

# chamada de função
e1 e2

# negação numérica
-e

# testa a existência de um atributo
e ? atributo

# concatenação de listas
e1 ++ e2

# multiplicação numérica
e1 * e2

# divisão numérica
e1 / e2

# soma numérica
e1 + e2

# subtração
e1 - e2

# operadores relacionais
e1 > e2
e1 >= e2
e1 < e2
e1 <= e2
e1 == e2
e1 != e2

# operadores booleanos
!e
e1 && e2
e1 || e2
e1 -> e2 # equivalente a !e1 || e2

# concatenação de conjuntos
e1 // e2
```

### Importações

`import` carrega, valida e importa expressões Nix a partir de um caminho, a palavra-chave não faz parte da linguagem em si:
```nix
x = import <nixpkgs> {}
y = trace x.pkgs.hello.name x;
```

### Construtores

#### with

O construtor `with` introduz o escopo léxico da expressão referenciada na próxima expressão:
```nix
let
  conjunto = { a = 1; b = 2 };
in
  with conjunto; "Tenho acesso aos atributos ${toString a} e ${toString b}"
```

Pode ser muito útil nas expressões onde há necessidade de repetição de um atributo:
```nix
{ pkgs, ... }:

{
  buildInputs = with pkgs; [ curl rust bat ffmpeg ];
}

# ao invés de
{
  buildInputs = [ pkgs.curl pkgs.rust pkgs.bat pkgs.ffmpeg ];
}
```

#### let...in

Permite a crição de variáveis locais:
```nix
let
  a = 2;
  b = 3;
in a + b
# 5
```

#### inherit

Traz o escopo léxico externo para um interno:
```nix
buildPythonPackage rec {
  pname = "ola";
  version = "1.0";
  src = fetchPypi {
    inherit pname version;
   sha256 = "01ba..0";
  };
}
```

## Conclusão

Este artigo é uma tentativa de tornar a documentação do `Nix` acessível em português! Sempre irei
referenciar as fontes oficiais no final de cada artigo, junto com alguns recursos adicionais!

## Fontes e recursos adicionais
- [Manual oficial do Nix](https://nixos.org/manual/nix/stable/#ssec-values)
- [Aprenda Nix em Y minutos](https://learnxinyminutes.com/docs/nix/)
- [Especificação oficial das expressões Nix](https://nixos.wiki/wiki/Nix_Expression_Language)

Este post foi publicado com minha ferramenta de linha de comando `devit`!
Você pode encontrar instruções para instalação [neste link](https://github.com/Mdsp9070/devit).

