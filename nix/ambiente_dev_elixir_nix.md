# Ambiente de desevolvimento Elixir/Phoenix com Nix

Nos artigos anteriores, escrevi uma breve introdução ao `Nix`, apresentei algumas vantagens e exemplos,
 e a especificação da sintaxe da linaguagem além de apresentar alguns contras e limitações dessa ferramenta!

Neste artigo irei mostrar um exemplo de configuração para um ambiente de desenvolvimento de projetos na linguagem
[Elixir](https://elixir-lang.org) com o `mix` e também suporte ao framework [Phoenix](https://www.phoenixframework.org)

## Requisitos

Os únicos requisitos para seguir este tutorial é ter um conhecimento brévio sobre o ecossistema `Elixir` e
possuir o gerenciador de pacotes `Nix` instalado no seu sistema.

Caso esteja usando a distribuição `NixOS`, o gerenciador de pacotes já estará disponível.

## Projeto base criado com o mix

O `mix` é a ferramenta de contrução utilizada no ecossistema `Elixir` para compilar seu projeto,
realizar tarefas `mix`, rodar os testes de sua apliação dentre outras funcionalidades.

Para torná-lo acessível num ambiente isolado, crie um arquivo `shell.nix`, com esse conteúdo:
```nix
{ pkgs ? import <nixpkgs> {} }:

with pkgs;

let
  elixirDrv = elixir.override {
    version = "1.11.4";
    rev = "308255bda81e7f76f9bec838cef033e8e869981b";
    sha256 = "1y8fbhli29agf84ja0fwz6gf22a46738b50nwy26yvcl2n2zl9d8";
    minimumOTPVersion = "23";
  };
in mkShell {
  name = "exemplo_dev";

  buildInputs = [ readline elixirDrv ]
  
  shellHook = ''
    mkdir -p .nix-mix
    mkdir -p .nix-hex
    export MIX_HOME=$PWD/.nix-mix
    export HEX_HOME=$PWD/.nix-hex
    export PATH=$MIX_HOME/bin:$PATH
    export PATH=$HEX_HOME/bin:$PATH
    export LANG=en_US.UTF-8
    export ERL_AFLAGS="-kernel shell_history enabled"
    export ERL_LIBS=$HEX_HOME/lib/erlang/lib
  '';
}
```

Com isso, execute `nix-shell` ou `nix-shell <caminho/para/shell.nix>` e você poderá
utilizar o `mix`!

Vamos desconstruir esse arquivo:
```nix
{ pkgs ? import <nixpkgs> {} }:

with pkgs;
```

No começo do arquivo importamos os pacotes a aprtir do repositório `nixpkgs` e trazemos todas as opções desse
módulo para o escopo léxico atual.

Já dentro do bloco `let...in`, sobrescrevo a versão do pacote da linguagem `Elixir`, que atualmente está na versão `1.10.4`
no canal estável:
```nix
elixirDrv = elixir.override {
  version = "1.11.4";
  rev = "308255bda81e7f76f9bec838cef033e8e869981b";
  sha256 = "1y8fbhli29agf84ja0fwz6gf22a46738b50nwy26yvcl2n2zl9d8";
  minimumOTPVersion = "23";
};
```

Essa expressão pode ser usada para utilizar qualquer versão da linguagem. Existem versões anteriores já envelopadas
no reposiório `nixpkgs` como os pacotes `elixir_1_9` e `elixir_1_7`.

Dentro da derivção especial `mkShell`, definimos um nome para esse ambiente e depois definimos a opção `buildInputs`,
que presenta quais são as dependências desse ambiente a ser criado:
```nix
buildInputs = [ readline elixirDrv ]
```

E por último declaramos um pequeno script que será executado antes do ambiente ser criado:
```nix
shellHook = ''
  mkdir -p .nix-mix
  mkdir -p .nix-hex
  export MIX_HOME=$PWD/.nix-mix
  export HEX_HOME=$PWD/.nix-hex
  export PATH=$MIX_HOME/bin:$PATH
  export PATH=$HEX_HOME/bin:$PATH
  export LANG=en_US.UTF-8
  export ERL_AFLAGS="-kernel shell_history enabled"
  export ERL_LIBS=$HEX_HOME/lib/erlang/lib
'';
```

Neste exemplo, criamos dois diretórios para abrigar as instalações locais
do `mix` e `hex`, tornamos disponmíveis os seus executáveis, e definimos duas
variáveis ambiente referentes à `BEAM`:

1. `ERL_AFLAGS`: o conteúdo dessa variáveis é adicionado no começo do comando `erl`
2. `ERL_LIBS`: necessária caso você já possua uma instalação local de `elixir`, suprimindo os
  avisos que a biblioteca padrão está sendo redefinida durante a compilação

## Projetos Phoenix

Já para projetos `Phoenix`, devemos realizar algumas mudanças no arquivo `shell.nix`.

### Phoenix + PostgreSQL

A primeira mudança é na opção `buildInputs`, onde adicionamos algumas dependência para compilação de
algumas dependências do projeto dentre outros utilitários:
```nix
buildInputs = [
  gnumake
  gcc
  readline
  openssl
  zlib
  libxml2
  curl
  libiconv
  # minha versão do pacote `elixir`
  elixirDrv
  glibcLocales
  nodejs-12_x
  yarn
  postgresql
  gtk-engine-murrine
] ++ lib.optional stdenv.isLinux [ 
      inotify-tools 
      # dependência do GTK para usar o observer
      gtk-engine-murrine 
    ]
  ++ lib.optionals stdenv.isDarwin (with darwin.apple_sdk.frameworks; [
      CoreFoundation
      CoreServices
    ]);
```

A segunda alteração é no script inicial do ambiente, onde definimos algumas váriaveis ambiente
para configuração do servidor do `PostgreSQL` e criamos um usuário padrão.

Para isso, adicione os seguintes comandos na opção `shellHook`: 

```bash
export PGHOST=$PWD/.postgres
export PGDATA=$PGHOST/data
export PGLOG=$PGHOST/postgres.log
export PGPASSWORD=postgres

if [ ! -d $PGDATA ]; then
  echo 'Initializing postgresql database...'
  initdb --auth=trust --no-locale --encoding=UTF8 >/dev/null
fi

# inicializando o servidor
pg_ctl start -l $PGLOG -o "--unix_socket_directories='$PGHOST'"

# verifica se existe um usuário padrão, caso contrário ele é criado
psql -d postgres -tAc "SELECT 1 FROM pg_roles WHERE rolname='postgres'" | grep -q 1 \
  || createuser -s postgres

# assim que você sair do ambiente criado pelo `nix-shell`
# o servidor do postgres será desligado 
finish() {
  pg_ctl -D $PGDATA stop
}

trap finish EXIT
```

O código completo desta expressão Nix está disponível [neste gist](https://gist.github.com/Mdsp9070/e562e4caa7349ee26156fce8fd1a945e).

### Phoenix + MariaDB/MySQL

Caso você esteja usando o `MariaDB` ou `MySQL` como SGBD, apenas altere a útilma parte do `shellHook`, removendo
as opções relacionadas ao `PostgreSQL` e adicionando essas linhas:

```bash
export MYSQL_BASEDIR=${pkgs.mariadb}
export MYSQL_HOME=$PWD/mysql
export MYSQL_DATADIR=$MYSQL_HOME/data
export MYSQL_UNIX_PORT=$MYSQL_HOME/mysql.sock
export MYSQL_PID_FILE=$MYSQL_HOME/mysql.pid

mysql_install_db --datadir=$MYSQL_DATADIR --basedir=$MYSQL_BASEDIR --pid-file=$MYSQL_PID_FILE
mysqld --datadir=$MYSQL_DATADIR --pid-file=$MYSQL_PID_FILE --socket=$MYSQL_UNIX_PORT &
export MYSQL_PID=$!

finish() {
  mysqladmin --socket=$MYSQL_UNIX_PORT shutdown
  kill $MYSQL_PID
  wait $MYSQL_PID
}
trap finish EXIT
```

## Conclusão

Utilizando essas derivações, é possível configurar um ambiente de densenvolvimento totalmente reproduzível
e estável em poucos minutos!

Vale lembrar que esta derivação é plenamente editável e pode ser adaptada para outros tipo de ambientes, 
por exemplo aplicações `Rails`, `Django` ou mesmo `Express` ou `AdonisJS`.

Este post foi publicado com minha ferramenta de linha de comando `devit`!
Você pode encontrar instruções para instalação [neste link](https://github.com/Mdsp9070/devit).

