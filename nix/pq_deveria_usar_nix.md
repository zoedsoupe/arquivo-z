# O que é Nix e por que você deveria experimentar?

Hoje em dia temos inúmeras opções para configurar um ambiente de desenvolvimento
que seja facilmente reproduzível e encapsulado. A escolha mais comum tem sido os
contêineres `Docker`.

Entretanto, nas minhas pesquisas aventureiras pela internet, descobri uma ferramenta bem
atrativa: o `Nix`.

Nesta breve série de artigos irei discorrer não só sobre o gerenciador de pacotes mas também mostrar
exemplos de ambientes para desenvolvimento que utilizam as vantagens do `Nix`!

## O que é Nix?

[Nix](https://nixos.org/learn.html) é um gerenciador de pacotes e uma linguagem de programação 
funcional-declarativa, preguiçosa (nem sempre), e dinamicamente tipada que usamos
para descrever as __derivações_ (resultado das expressões em `Nix`) para esse gerenciador.

### Como o gerenciador de pacotes funciona?

A partir de um arquivo que contenha as instruções para construção do pacote, o `Nix` computa
uma derivação. Para uma derivação ser considerada válida, o arquivo precisa ter as seguintes informações:

- referência para as dependências do pacote a ser construído;
- intruções de como construir o pacote;
- metainformações sobre o pacote, como os mantenedores;

Quando o pacote for construído com sucesso, um caminho será criada com o resultado da construção 
no formato: `/nix/store/<hash>-<nome>-<versão>`, onde o `hash` é criado a partir dos dados da derivação.

Veja o conjunto de expressões Nix que gera a derivação do software `GNU hello`, por exemplo:
```nix
{ lib, stdenv, fetchurl }:

stdenv.mkDerivation rec {
  pname = "hello";
  version = "2.10";

  src = fetchurl {
    url = "mirror://gnu/hello/${pname}-${version}.tar.gz";
    sha256 = "0ssi1wpaf7plaswqqjwigppsg5fyh99vdlb9kzl7c9lng89ndq1i";
  };

  doCheck = true;

  meta = with lib; {
    description = "A program that produces a familiar, friendly greeting";
    longDescription = ''
      GNU Hello is a program that prints "Hello, world!" when you run it.
      It is fully customizable.
    '';
    homepage = "https://www.gnu.org/software/hello/manual/";
    changelog = "https://git.savannah.gnu.org/cgit/hello.git/plain/NEWS?h=v${version}";
    license = licenses.gpl3Plus;
    maintainers = [ maintainers.eelco ];
    platforms = platforms.all;
  };
}
```

Dessa forma o `Nix` consegue assegurar que a mesma derivação vai sempre construir o mesmo pacote,
a não ser que alguma instrução de construção use informações dependentes de hardware.

## Vantagens de usar Nix

### Ambiente sempre reproduzível

Essa é a vantagem mais clara da utilização do `Nix` para gerir pacotes e dependências, pois como
dito anteriormente, se duas pessoas contruirem um pacote com a mesma derivação, as duas terão o mesmo
resultado final. Mesmo que haja uma diferença na versão de alguma dependência, o caminho gerado pela
derivação irá se alterar, explicitando que houve uma mudança.

### Múltiplas versões do mesmo pacote simultâneamente

Como cada pacote é instalado em seu próprio caminho, não há colissões. Isso se torna muito atrativo para
desenvolvedores, pois pode ser comparado com os ambientes virtuais de `Python`, só que para qualquer programa.

Além disso, como a cada versão construída do pacote gera um novo caminho, há sempre a possibilidade de retornar
à uma versão antiga, desde que ela não tenha sido apagada pelo _garbage-collector_.

### Construções seguras

Toda a contrução de cada derivação é isolada do sistema, o que torna seguro permitir que usuários
com menos privilégios executem suas derivações.

### Teste novas ferramentas temporariamente

Digamos que você precisa converter um arquivo escrito com `Markdown` para `HTML` com o pandoc! Porém, você
se lembra que não possui esse programa e precisa usá-lo apenas uma vez.

Naturalmente você possui algumas opções:
1. Baixar o programa dos repositórios oficiais, realizar a conversão e desinstalar;
2. Usar uma imagem `Docker` do programa, utilizá-la e deixar ela ocupando espaço na sua máquina;

Com o `Nix` é possível criar um ambiente sob demanda, mas sempre reprodutível, no qual um pacote não
existente no seu sistema passe a ser utilizável.

No caso do pandoc, com um simples `nix-shell -p pandoc` você tem acesso ao pacote de forma temporária! 

## Ecossistema

Essas vantagens são bem atrativas, mas os casos de uso para o `Nix` não param por aí. A seguir,
mostro exemplos do mundo real que usam `Nix`.

### Nixpkgs

`nixpkgs` é a maior coleção de derivações para o gerenciador de pacotes que podem ser alteradas e combinadas.
Além de derivações específicas para cada linguagem, como um mirror atualizado do `Hackage`, existem derivações de
diversos outros softwares.

### NixOS

Já o `NixOS` é uma distribuição GNU/Linux que é configurada a partir de expressões `Nix`, além de possuir nativamente
o gerenciador de pacotes. Todo o sistema é tratado como uma grande derivação que depende de todas as outras sub derivações
que você monta.

Neste trecho de exemplo, mostro como habilitar o uso do `pulseaudio` no `NixOS`:
```nix
sound.enable = true;
hardware.pulseaudio = {
  enable = true;
  package = pkgs.pulseaudioFull;
};
```

Esse trecho foi retirado da minha [configuração pessoal](https://github.com/Mdsp9070/dotfiles).

### Home Manager

Um projeto muito interessante no ecossistema `Nix` é o [home-manager](https://github.com/nix-community/home-manager), que permite
gerenciar um ambiente (dotfiles) completo a nível de usuário.

Neste trecho da minha configuração pessoal, declaro alguns pacotes a nível de usuário:
```nix
programs = {
  home-manager.enable = true;
  gpg.enable = true;
  command-not-found.enable = true;
  emacs.enable = true;
};

services.clipmenu.enable = true;
services.emacs.enable = true;

home.username = "matthew";
home.homeDirectory = "/home/matthew";

home.packages = with pkgs; [
  # bar
  haskellPackages.xmobar

  # chat
  tdesktop discord

  # browser
  google-chrome

  # theme
  lxappearance

  # tools
  docker-compose insomnia
  qbittorrent obs-studio
  screenkey gitAndTools.gh

  # audio
  spotify

  # images
  peek flameshot imagemagick

  # others
  any-nix-shell arandr
  bitwarden-cli
];
```

### Integração com diversas ferramentas

Também é possível usar o `Nix` para outras tarefas, como descrever a construção de uma imagem `Docker`:
```nix
{ pkgs ? import <nixpkgs> { system = "x86_64-linux" } }:

pkgs.dockerTools.buildLayeredImage { # função para facilitar a construção da imagem
  name = "nix-hello";                # nome da imagem a ser construída
  tag = "latest";                    # tag para a imagem
  contents = [ pkgs.hello ];         # pacotes presentes na imagem
}
```

## Conclusão

Em suma, o `Nix` e seu ecossistema é uma boa opção para quem quer ter novas experiências
no mundo Unix, permitindo que um ambiente de desenvolvimento ou até mesmo uma distribuição
GNU/Linux completa possa ser facilmente reproduzida, além de usufruir das vantagens da imutabilidade
de uma linguagem funcional.

Porém, é importante ter em mente que assim como qualquer ferramenta, o `Nix` não funciona como
"bala de prata". Existem alguns contras que você deve levar em consideração:
- O número de pacotes envelopados, apesar de serem inúmeros, não se comparam (ainda) com os repositórios
  oficiais de algumas distribuições, por exemplo.
- Alguns pacotes podem não ser atualizados com frequência. 

## Fontes e recursos adicionais
- [Site oficial para o Nix](https://nixos.org)
- [Repositório para nixpkgs](https://github.com/NixOS/nixpkgs)
- [Post usado como inspiração](https://serokell.io/blog/what-is-nix)
- [Grupo brasileiro de usuários do NixOS no Telegram](https://t.me/nixosbr)

Este post foi publicado com minha ferramenta de linha de comando `devit`!
Você pode encontrar instruções para instalação [neste link](https://github.com/Mdsp9070/devit).
