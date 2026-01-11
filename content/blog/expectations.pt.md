+++
title = "Expectativas"
date = 2021-01-26

[taxonomies]
tags = ["guix", "outreachy"]
+++

# Como começou: esboço de um cronograma

Quando as inscrições para o Outreachy abrem, todos os inscritos em
potencial podem ver uma lista com todos os projetos disponíveis e
seus possíveis cronogramas. Após registrar uma primeira
contribuição - ou possivelmente mais -no projeto desejado, é hora de
começar a trabalhar na inscrição final. É necessário que o candidato
proveja um cronograma própriotambém, apontando os objetivos e
tarefas e quando eles poderão ser cumpridos. Em minha inscrição,
esse foi o cronograma que eu propus:

*01/12/2020 - Início do estágio*

1- Subcomando guix git log vai mostrar "olá mundo".

2- Aprender a usar Guile-Git.

*11/12/2020 - Feedback inicial*

2- Continuar aprendendo sobre como usar Guile-Git e também melhorar
a biblioteca.

*18/12/2020 - Semestre da faculdade acaba*

3- Percorrer o histórico de commits.

4- Lidar com vários canais.

*12/01/2021 - Feedback intermediário*

5- Adicionar opção para ordenar por data, canais, etc.

6- Adicionar suporte a expressões regulares: "guix git log --grep=".

7- Adicionar formatação de commits: "guix git log --format=".

8- Adicionar limitação de commits: "--after", "--before, "--author".

*02/03/2021 - Feedback final*

*02/03/2021 - Fim do estágio*

# Como está indo

Até agora, os objetivos e tarefas permaneceram quase iguais, mas o
cronograma que eu apresentei mudou consideravelmente. Veja como as
coisas estão indo por agora e tenha em mente que as minhas semanas
começam no domingo e terminam no sábado.

*Semana #1* 01/12 - 05/12

• Criar meu próprio repositório Guix no Gitlab.

• Escrever uma postagem de blog (tanto para o Guix quanto para o
meu blog pessoal).

• Mexer e modificar o código-fonte do Guix, a fim de aprender como
as coisas são feitas, assim como começar a usar o 'guix repl'.

• Criar o subcomando 'guix git log'.

• O subcomando acima deve mostrar o caminho para o checkout do
canal padrão do Guix.

*Semana #2* 06/12 - 12/12

• Melhorar o código da semana anterior.

• Adicionar opções fictícias no subcomando, e 'guix git log --help'
deve mostrar essas opções.

*Semana #3* 13/12 - 19/12

• Obter alguns commits que estão no caminho do checkout e mostrar
eles.

• Adicionar a opção '--online', e mostrar os commits de forma
análoga ao que é feito com 'git log --oneline'.

*Semana #4 and #5* 20/12 - 02/01

• Lidar com canais diferentes.

• Adicionar '~--format[FORMATO]~', e FORMATO pode ser ~oneline~,
~medium~ ou ~full~.

*Semana #6* 03/01 - 09/01

• '~guix git log --channel-cache-path~' deve mostrar o caminho para
todos os canais.

• O subcomando deve mostrar os commits de todos os canais.

• Adicionar '~--pretty=<string>~'

*Semana #7* 10/01 - 16/01

• Adicionar '~--grep=REGEXP~'.

*Semana #8* 17/01 - 23/01

• Comparar o tempo que 'git log' e 'guix git log' levam.

• Escrever testes.

*Semana #9* 24/01 - 30/01

• Aprender mais tentar utilizar a avalição preguiçosa.

Ao observar o cronograma planejado e o que foi implementado,
percebe-se que apenas a limitação de commits e a ordenação não
foram feitas. No momento, a melhoria e a otimização do código estão
sendo trabalhadas.
