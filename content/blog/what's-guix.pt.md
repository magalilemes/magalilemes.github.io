+++
title = "O que é Guix"
date = 2021-01-26

[taxonomies]
tags = ["guix", "outreachy"]
+++


# Definição

Guix é tanto o nome de uma distribuição de sistema operacional
quanto de um gerenciador de pacotes. Dado que é desenvolvido pelo
projeto GNU, Guix segue os princípios de Software Livre e, portanto,
todos os pacotes fornecidos são livres. Uma coisa interessante é que
Guix é escrito principalmente em Guile Scheme, uma linguagem
funcional da família LISP e com a qual tem sido muito bacana
programar.

Se você usa outra distribuição GNU/Linux, Guix pode ser usado como
um gerenciador de pacotes, de forma que você pode ter acesso a todos
os pacotes disponíveis para o sistema operacional Guix. Eu, por
exemplo, uso Guix junto com o Ubuntu.

## Instalação

Eu instalei Guix usando o script que pode ser encontrado nesse
[link](https://guix.gnu.org/manual/en/html_node/Installation.html).

## Usando Guix

Como mencionado anteriormente, eu tenho utilizado o Guix como um
gerenciador de pacotes, o que significa que ele gerencia a
instalação, atualização e remoção de pacotes, entre outras
coisas. Abaixo está uma rápida demonstração de um uso simples do
Guix:

Suponha que você quer instalar um aplicativo de flash cards, você
pode procurar por algum usando:

```
$ guix search flash cards
```

Dessa forma, informações como versão, tipo de licença, site e uma
descrição de pacotes que correspondem ao que você está procurando
serão mostrados na sua tela.

Então, após ver quais são suas opções, você decide instalar o
[Anki](https://apps.ankiweb.net/). Isso pode ser feito de forma simples: 

```
$ guix install anki
```

Perceba como não há necessidade de ter acesso privilegiado - o que
significa que nenhum privilégio de root é necessário - para
instalar um pacote. Agora, após instalar o pacote, você pode se
perguntar como ele é definido no código-fonte do Guix e você pode
querer até mesmo **editar** essa definição:

```
$ guix edit anki
```

Por algum motivo você quer remover um pacote. Então, vá em frente e:

```
$ guix remove anki
```

E, finalmente, para rapidamente obter uma lista com todos os
pacotes instalados:

```
$ guix package -I
```

# O objetivo do meu estágio

Com sorte, após entender o básico sobre Guix, você pode se perguntar
qual o papel do meu estágio nisso tudo. Meu objetivo é implementar
um subcomando similiar ao 'git log', mas específico para o Guix.

Pacotes têm diferentes versões e, por muitos motivos, às vezes é
melhor ter uma versão do que outra. Guix torna possível ter versões
antigas de pacotes por meio do comando [time-machine](https://guix.gnu.org/manual/en/html_node/Invoking-guix-time_002dmachine.html). Um usuário
pode, por exemplo, prover uma ID hash de um commit como uma opção
para esse comando (`guix time-machine --commit=<commit-id> -- build
<package>`), e ter, então, um pacote como ele foi definido nesse
commit. O id do commit é necessário dado que as definições de
pacotes estão em um repositório Git.

Com `guix git log`, a história de commits do Guix vai estar
disponível facilmente e vai ser possível obter informações
relacionados a, por exemplo, quando um pacote teve sua última
atualização. O que foi feito até agora é descrito em minha [segunda postagem do blog](/pt/blog/present-and-future/).
