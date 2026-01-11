+++
title = "Presente e futuro"
date = 2020-12-22

[taxonomies]
tags = ["guix", "outreachy"]
+++

# O que foi feito até agora

Eu não consigo acreditar que já se passaram três semanas. O tempo
passou muito rápido. Sinto que tenho aprendido muito: há muitas
maneiras novas de encarar um problema. A fim de manter uma certa
uniformidade no código, eu tenho passado algum tempo lendo e
analisando o código-fonte do Guix. Desta forma, eu também aprendo
alguns procedimentos novos.

## Motivação

A minha tarefa do estágio é implementar um subcomando de forma que
todo o histórico de commits do Guix possa ser acessado
facilmente. Usando este subcomando, um usuário poderá saber em qual
commit um pacote foi adicionado ou atualizado. Isso vai estar
disponível com 'guix git log'.

Para completar meu objetivo, eu tenho utilizado a [biblioteca Git em
Guile](https://gitlab.com/guile-git/guile-git), que provê bindings para a [biblioteca libgit2 em C](https://libgit2.org/libgit2/). Com esse
módulo, é fácil obter informações do repositório e dos commits.

## Resultados

Primeiramente, o código que estou trabalhando atualmente pode ser
encontrado neste [link](https://gitlab.com/magalilemes/guix). No momento em que escrevo isso, se você
executar o código com `~./pre-inst-env guix git log
--oneline`[^1], cinco commits serão mostrados na tela,
apresentados em um formato como se fosse o próprio `git log
--oneline`.

```
$ ./pre-inst-env guix git log --oneline | grep nano
eeb3db0 gnu: nano: Update to 5.4.
e6b2a3e gnu: nano: Update to 5.3.
5abbf43 gnu: nano: Update to 5.2.
0b4327f gnu: Add nanomsg.
5b893f4 gnu: nano: Update to 5.1.
c09f22a gnu: nano: Update to 5.0.
dfc4265 gnu: nano: Update to 4.9.3.
```

É facil ver em qual commit o editor de texto nano foi atualizado
por último, ou atualizado para uma versão específica, ou quando a
biblioteca nanomsg foi adicionada pela primeira vez no Guix.

## Uma pedra no meio do caminho

Embora essas três semanas tenham sido incríveis e eu tenha
aprendido muito, o caminho não foi sempre fácil. Eu encontrei
alguns problemas tentando fazer o subcomando funcionar como está
funcionando agora.

Claro, houve alguns problemas bobos e fáceis de resolver, que
exigiam apenas um pouco de atenção para encontrar a solução. Um
exemplo foi quando eu passei trinta minutos tentando entender por
que I estava recebendo uma mensagem de erro e a razão era que eu
não tinha importado o módulo necessário para usar uma determinada
função.

O problema mais difícil que eu tive foi um *segmentation fault*
ocorrendo quando eu rodava o código que escrevi. O que eu queria
fazer era bem simples: uma função que, dado o caminho para um
repositório Git, devolveria todos os commits encontrados lá. O
código que eu escrevi parecia bem simples, então receber
*segmentation fault* não era algo que eu esperava. Aqui está o que
eu estava tentando fazer, usando o Guix REPL, que tem se provado
ótimo para testar como as coisas funcionam - ou descobrir o porquê
delas não estarem funcionando.

```
    $ guix repl
    GNU Guile 3.0.4
    Copyright (C) 1995-2020 Free Software Foundation, Inc.

    Guile comes with ABSOLUTELY NO WARRANTY; for details type `,show w'.
    This program is free software, and you are welcome to redistribute it
    under certain conditions; type `,show c' for details.

    Enter `,help' for help.
    scheme@(guix-user)> (use-modules (guix git) (git) (guix channels) (ice-9 match) (srfi srfi-1))
    scheme@(guix-user)> (define cache (url-cache-directory (channel-url %default-guix-channel)))
    scheme@(guix-user)> cache
    $1 = "/home/magali/.cache/guix/checkouts/pjmkglp4t7znuugeurpurzikxq3tnlaywmisyr27shj7apsnalwq"
    scheme@(guix-user)> (let* ((repo (repository-open cache))
                            (latest-commit
                                (commit-lookup repo (reference-target (repository-head repo)))))
                        (let loop ((commit latest-commit)
                                    (res (list latest-commit)))
                            (match (commit-parents commit)
                                (() (reverse res))
                                ((head . tail)
                                    (loop head (cons head res))))))
    Segmentation fault (core dumped)
```

E a solução para isso, na verdade, é uma gambiarra[^2], dado que eu
esbarrei em um [bug](https://gitlab.com/guile-git/guile-git/-/issues/21) na biblioteca Git em Guile que [ainda não foi
consertado](https://lists.gnu.org/archive/html/guix-devel/2020-12/msg00226.html). Em vez de abrir o repositório dentro do let*, se o
repositório Git for definido em uma variável global, então eu posso
obter os commits, como eu desejava.

# Próximos passos

Até agora, o que `guix git log` faz é andar pelo histórico de
commits, visitando somente o primeiro pai do commit, ignorando
qualquer outro pai - no caso de commits com múltiplos pais - e
deixando alguns commits para trás. Então, na minha lista de coisas
para fazer, eu tenho que corrigir isso, encontrando uma solução
melhor para percorrer o histórico de commits, sem deixar nenhum sem
visitar.
Há também duas outras melhorias que eu devo fazer:

- Lidar com canais diferentes, visto que o Guix pode ser expandido
usando diferentes canais[^3]. Até agora, o subcomando só pode
lidar com o canal padrão.

- Adicionar outras opções de formatação. No momento em que escrevo,
há apenas a opção 'guix git log --oneline', mas o plano é expandir
isso, de forma que nós também possamos ter diferentes formtações
com `guix git log --format=<formato>`, e `formato` podendo ser
`short`, `medium` ou `full`.

[^1]:  https://guix.gnu.org/manual/devel/en/html_node/Running-Guix-Before-It-Is-Installed.html
[^2]: Bom, não é exatamente uma gambiarra, mas eu não consegui pensar
numa tradução mais apropriada para workaround.
[^3]: https://guix.gnu.org/manual/en/html_node/Channels.html
