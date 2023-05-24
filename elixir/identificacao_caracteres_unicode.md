# Identificação de caracteres Unicode em Elixir

*AVISO INICIAL*

Este artigo é uma tradução (não direta e com adaptações) de um artigo publicado no blog do [exercism.io](exercism.io) pelo _Percy Grunwald_. O artigo original pode ser encontrado por [esse link](https://exercism.org/blog/unicode-matching-in-elixir).

Uma das maiores vantagens de usar uma linguagem expressiva como Elixir é a possibilidade de solucionar problemas de forma clara, explícita e com um toque de elegância devido a sua sintaxe!

Hoje vou mostrar como podemos identificar ou "casar" (_it's a match!_) caracteres pertencentes ao padrão Unicode. Para quem não conhece ou não sabe em detalhes, [Unicode](https://unicode.org/standard/WhatIsUnicode.html) é uma solução criada para evitar o conflito entre diferentes codificadores de caracteres.
Isto é, cada um dos [codificadores de caracteres](https://en.wikipedia.org/wiki/Character_encoding#Common_character_encodings) implementava números diferentes para representar cada símbolo de uma ou mais línguas.

Porém, nenhum deles realmente conseguia abranger as extensas possibilidades de combinações, letras, símbolos e pontuações que centenas de línguas possuem. Por isso nasce o Unicode ou, código único (_unique code_), onde cada caractere recebe um identificador único, não sendo restingido por nenhuma plataforma, programa ou linguagem!

Ok, mas vamos falar de Elixir agora! Já de ínicio vou propor um desafio:

> "Implemente uma função `frequencia/1` que dada uma lista de strings devolva a frequência de cada caractere. Lembre que as strings podem estar descritas em qualquer linguagem!"

Exemplo:
```elixir
iex> Frequencia.frequencia(["ciências", "naturais"])
%{
  "a" => 3,
  "i" => 3,
  "ê" => 1,
  "c" => 2,
  "n" => 2,
  "r" => 1,
  "s" => 2,
  "t" => 1,
  "u" => 1
}
```

Aqui estão alguns casos de teste:
```elixir
["Eorum" "argumentum" "audiri" "posse" "per" "raedam" "multum"]
["When" "transplanting" "seedlings" "candied" "teapots" "will" "make" "the" "task" "easier"]
["El" "zorro" "del" "sombrero" "de" "copa" "le" "susurró" "al" "oído" "del" "conejo"]
["Estava" "escurecendo" "e" "ainda" "não" "estávamos" "lá"]
["E" "ʻike" "ʻoe" "i" "ke" "alahaka" "ānuenue" "ma" "hope" "o" "ka" "ua" "ʻana" "o" "nā" "pōpoki" "a" "me" "nā" "ʻīlio"]
["Σήμερα" "είναι" "η" "μέρα" "που" "θα" "ξέρω" "επιτέλους" "τι" "γεύση" "έχει" "το" "τούβλο"]
```

Vai lá, tenta implementar! Precisa de tempo? Tudo bem, eu espero...
![um calango na cadeira de balanço, apenas esperando...](https://media.giphy.com/media/QUmpqPoJ886Iw/giphy.gif)

## Identificação de letras em Elixir

5 minutos depois... 10 anos depois... Podemos fazer em conjunto! Vamos tentar com testes mais simples!

Partindo do básico, como podemos determinar que `"z"` é uma letra?

Obs: Existe uma funcionalidade na linguagem Elixir que torna a resolução desse exercício algo, digamos... único e intrigante! Nada prático, mas interessante! Deixarei uma demonstração no final deste artigo!

O caminho mais "comum" seria escolher o uso de [expressões regulares](https://www.regular-expressions.info/quickstart.html). Uma dica: o site [regexr.com](https://regexr.com/) será seu guia para entender cada modificador e sintaxes específicas.

```elixir
iex> String.match?("z", ~r|[a-z]|)
true
```

Realmente bem simples, mas e a letra `"Z"`?

Ah, fácil também:
```elixir
iex> String.match?("Z", ~r|[A-Za-z]|)
true

iex> String.match?("Z", ~r|[a-z]|i)
true
```

No segundo exemplo, é adicionado o nosso primeiro modificador: `i` [modificador caseless](https://hexdocs.pm/elixir/Regex.html#module-modifiers) que permite a não diferenciação entre letras maiúsculas ou minúsculas.

Tudo bem, por enquanto está bem simples, que tal testarmos com um caractere com acento, como `"ê"`. Bem, esse caractere também é uma letra, então...
```elixir
iex> String.match?("ê", ~r|[a-z]|i)
false
```

Opa, estranho, mas esperado, uma vez que `"ê"` é um caractere especial. Uma solução poderia ser usar uma expressão regular que falhe caso encontre um caractere especial. Algo como (de forma simplificada):

```elixir
iex> String.match? "É", ~r|[a-z0-9]|i
false
```

É uma solução válida, mas muito frágil. Como poderíamos cobrir todos os casos possíveis? Fora que seria necessário testar se cada caractere é especial ou não, identificá-lo manualmente e contabilizar sua frequência. Definitivamente tá longe de ser o caminho feliz e muito menos elegante.

### Expressões regulares com suporte a Unicode

Existe um modificador para expressões regulares que inclui e permite a identificação de caracteres Unicode numa expressão regular. Esse modificador é o [`u`](https://hexdocs.pm/elixir/Regex.html#module-modifiers)

> unicode (`u`) - habilita a identificação de padrões Unicode específicos como `\p` e modifica os já conhecidos operadores `\w`, `\W` e `\s` e similares para que possam detectar caracteres Unicode.

De fato o modificador `u` e em [especial o operador `\p`](https://www.regular-expressions.info/unicode.html), torna nossa solução bem elegante. O operador `\p` permite a identificação de um _grafema_.

Grafema é a unidade fundamental de um sistema de escrita que pode representar um fonema em escritas alfabéticas, uma ideia numa escrita ou até uma sílaba nas escritas silábicas. Pode ser entendido como um conjunto de caracteres.

Com este operador é possível detectar um grafema dentro de qualquer [categoria de caracteres Unicode](https://en.wikipedia.org/wiki/Unicode_character_property#General_Category). Não incluindo apenas as subcategorias `Ll` [(letra, caixa baixa)](https://www.compart.com/en/unicode/category/Ll) ou `Sc` [(símbolo, moeda)](https://www.compart.com/en/unicode/category/Sc), mas também as categorias mães `L` (letra) e `S` (símbolo).

Em outras palavras, você consegue identificar qualquer caractere, em qualquer tipografia em qualquer [língua humana suportada pelo Unicode](https://www.unicode.org/faq/basic_q.html) com o padrão `\p{L}`!

Vamos aos testes então!

Letras latinas comuns do português são facilmente detectadas:
```elixir
iex> String.match?("o", ~r|\p{L}|u)
true

iex> String.match?("O", ~r|\p{L}|u)
true
```

Caracteres latinos com acentos agudos ou trema também não causam nenhum problema:
```elixir
iex> String.match?("ü", ~r|\p{L}|u)
true

iex> String.match?("í", ~r|\p{L}|u)
true
```

Mas então esse padrão não acusaria verdadeiro para qualquer caractere? Veja a seguir:
```elixir
iex> String.match?("$", ~r|\p{L}|u)
false

iex> String.match?("&", ~r|\p{L}|u)
false
```

Entendido. Esses grafemas parecem letras, mas não são! Mas e quanto ao nosso problema com internacionalizações? Bem...
```elixir
# Grafema que representa "casa" ou "construção" em japônes
iex> String.match?("舎", ~r|\p{L}|u)
true

# Grafema que significa "eu" em cingalês
iex> String.match?("මට", ~r|\p{L}|u)
true
```

## Aplicando a identificação de caracteres Unicode ao nosso desafio

Agora temos todas as ferramentas necessárias para nos ajudar a decidir se um determinado grafema é ou não uma letra. A seguir, temos uma implementação inicial:
```elixir
defmodule Frequencia do
  def frequencia(sentencas) when is_list(sentencas) do
    sentencas
    |> separar_grafemas()
    |> contar_letras()
  end

  defp separar_grafemas(sentencas) do
    sentencas
    |> Enum.join()
    |> String.graphemes() # sim, esta função é nativa :P
  end
end
```

A função principal é a `contar_letras/1`, que aplicará a expressão regular `~r|\p{L}|u` em cada grafema, decidindo se é ou não uma letra! Aqui está uma implementação de exemplo:
```elixir
defmodule Frequencia do
  defp contar_letras(grafemas) do
    for grafema <- grafemas, letra?(grafema), reduce: %{} do
      acc -> Map.update(acc, grafema, 1, & &1 + 1)
    end
  end

  defp letra?(grafema), do: String.match?(grafema, ~r|\p{L}|u)
end
```

Nossa função aceita uma lista de grafemas e por meio de [compreensões de lista](https://hexdocs.pm/elixir/1.12/Kernel.SpecialForms.html#for/1), iteramos sobre cada elemento da lista, apenas mantendo os elementos que passarem no teste (a chamada da função auxiliar `letra?/1`) e reduzimos a lista em um mapa, onde cada chave é uma letra e seu valor é a frequência em que tal caractere aparece!

## Bônus: Sequências!

No começo deste artigo eu mencionei que Elixir possibilita uma forma bem mais "diferentona" de se resolver esse problema. Isso se deve ao fato de que a própria linguagem em si, suporta caracteres Unicode.

Uma das funcionalidades mais interessantes da linguagem são as sequências! Dado um valor inicial, um valor final e um "passo", é gerado um conjunto que representa aquela sequência (de forma inclusiva).

Exemplos!

```elixir
iex> Enum.to_list(1..10)
[1, 2, 3, 4, 5]

iex> Enum.to_list(1..5//-1)
[5, 4, 3, 2, 1]
```

Mas isso é muito comum, vamos incrementar essa funcionaldiade! Que tal criarmos sequências para letras? A expressão `?` antes dos caracteres permite que recupermos seus códigos em decimal:
```elixir
iex> Enum.to_list(?a..?c)
'abc'

iex> Enum.to_list(?c..?a)
'cba'
```

Uhumm... Mas onde quero chegar com isso? Lembra que Elixir suporta nativamente caracteres Unicode (infelizmente não para criação de variáveis haha)? Vamos usar isso então:
```elixir
iex> ?é
233

iex> ?ﬦ
64294

iex> ?😅
128517
```

### Possível solução para o problema

Com um algumas composições de funções, podemos solucionar nosso problema, não de uma maneira eficiente, não de uma maneira totalmente legível, mas certamente com toda a elegância possível!

```elixir
defmodule Frequencia do
  # [...]

  defp contar_letras(grafemas) do
    Enum.reduce(grafemas, %{}, fn grafema, acc ->
      codigo = 
        grafema 
        |> String.to_charlist() # charlists são literalmente listas de caracteres
        |> hd() # logo, precisamos apenas do cabeçalho da mesma, ou seja, um "codepoint"

      cond do
        caixa_alta?(codigo) -> Map.update(acc, grafema, 1, & &1 + 1)
        caixa_baixa?(codigo) -> Map.update(acc, grafema, 1, & &1 + 1)
        latin_A_e_B?(codigo) -> Map.update(acc, grafema, 1, & &1 + 1)
        true -> acc
      end
    end)
  end

  defp caixa_alta?(ch), do: ch in ?A..?Z # 65..90
  defp caixa_baixa?(ch), do: ch in ?a..?z # 97..122
  defp latin_A_e_B?(ch), do: ch in ?À..?ɏ # 192..591
end
```

## Conclusão

Descobrimos duas maneiras incríveis de detectar grafemas de múltiplas nacionalidades! Ok, talvez uma delas seja um tanto quanto.. exótica eu diria, mas ainda sim divertida de se implementar!

Essas funcuionalidades permitam que um leque de outras funcionalidades ou problemas sejam contruídos e solucionados com uma expressividade clara, sucinta e simples.

Vale lembrar que no exemplo usando sequências eu limitei os códigos Unicode levando emconta apenas os caracteres latinos básicos e as extensões [A](https://en.wikipedia.org/wiki/Latin_Extended-A) e [B](https://en.wikipedia.org/wiki/Latin_Extended-B), além do [suplemento 1](https://en.wikipedia.org/wiki/Latin-1_Supplement_(Unicode_block)) latino para fins de exemplificação, mas qualquer caractere Unicode é válido em Elixir!
