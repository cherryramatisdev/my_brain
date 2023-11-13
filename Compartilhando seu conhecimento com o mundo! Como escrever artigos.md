---
title: Compartilhando seu conhecimento com o mundo! Como escrever artigos
description: Compartilhar conhecimento escrito é uma ótima forma de dominar um assunto especifico alem de ser uma excelente forma de melhorar a organização das ideias, comunicação e obviamente se auto promover na comunidade. Nesse artigo vamos discutir como organizar as ideias e produzir conteúdos técnicos!
tags: beginners,tutorial,writing
cover_image: ""
canonical_url: 
published: false
---

Compartilhar conhecimento escrito é uma ótima forma de dominar um assunto específico, além de ser uma excelente maneira de melhorar a organização das ideias, comunicação e obviamente se autopromover na comunidade. Essa produção de artigos tanto técnicos quanto sociais são muito importantes tanto para quem escreve quanto para quem está lendo, lembre-se sempre: `Você sabe mais hoje do que quem começou ontem`. Pensando nisso vamos discutir um pouco mais sobre como organizar as próprias ideias na produção de um artigo técnico para a plataforma dev.to!

## Table of contents

- [Coletando ideias e se motivando para escrever](#coletando-ideias-e-se-motivando-para-escrever)
- [Entendendo a plataforma e achando a propria linguagem](#entendendo-a-plataforma-e-achando-a-propria-linguagem)
- [Aprendendo markdown e dicas gerais para uma boa formatação](#aprendendo-markdown-e-dicas-gerais-para-uma-boa-formatacao)
- [A estrutura básica de um artigo técnico](#a-estrutura-basica-de-um-artigo-tecnico)
- [Como revisar e melhorar a escrita](#como-revisar-e-melhorar-a-escrita)
- [Conclusão e agradecimentos](#conclusao-e-agradecimentos)

## Coletando ideias e se motivando para escrever

O processo de inspiração é provavelmente uma das fases mais chatinhas antes de produzir um artigo online, muitas vezes ficamos presos em um loop infinito de técnicas mirabolantes para ter ideias incríveis quando, na verdade, a solução acaba sendo muito simples: aceite suas ideias e consuma o máximo de conteúdos possível. A forma mais simples de encontrar ideias e de construir sua própria linguagem é ler o que as pessoas já produzem sobre os temas que lhe interessam, quer se trate de uma linguagem de programação, de um tema específico em TI, etc. Seus primeiros artigos não vão ser iguais ao de uma pessoa que produz há muitos anos e aceitar isso vai fazer com que evolua muito mais rápido e com muito mais consistência.

Outra dica importante é manter um *segundo cérebro*, um local físico ou virtual onde você copia partes de artigos que leu na internet com observações próprias de forma super informal e separada. Isto vai ajudar a construir um repertório e a ganhar mais confiança ao escrever, uma vez que pode citar frases que leu em outros locais e abrir portas para muito mais contexto. Afinal não podemos confiar no nosso próprio cérebro para lembrar de tudo o que consumimos, não é mesmo?

## Entendendo a plataforma e achando a propria linguagem

Entender a plataforma e o público que vamos atingir escrevendo nossos conteúdos é muito importante para podermos filtrar como vamos estruturar os artigos de forma geral certo? Na *minha opinião*, o [dev.to](https://dev.to) é uma plataforma bem informal que valoriza muito conteúdos no formato de tutoriais com um estilo conversacional e direto ao ponto, com essa informação podemos deduzir algumas de estruturar nossos artigos para que possamos passar nossa ideia em um modelo conhecido pelos leitores.

Isso significa que tudo o que você vai produzir são tutoriais diretos e informais?  De forma alguma! Só significa que você pode moldar o seu conteúdo para conter essa linguagem mais informal, conversacional e direta mesmo que o tema tratado seja super complexo, isso inclusive vira um desafio muito interessante de simplificar o complexo. 

*PS: Essa habilidade de simplificar o complexo vai te acompanhar para o resto da sua vida profissional, então pratique bastante*

## Aprendendo markdown e dicas gerais para uma boa formatação

A forma para produzirmos nossos artigos no dev.to é utilizando uma linguagem de marcação (exatamente, igual o HTML) chamada [Markdown](https://www.markdownguide.org), apesar dela ser super simples é importante termos um domínio bem solido do que é possível fazer quando falamos sobre organizar e deixar nosso texto bonitinho, semelhante como vamos produzir uma estrutura complexa no Microsoft Word devemos ser capazes de produzir o mesmo com código Markdown.

É sempre importante ressaltar a importância de um material educacional bem estruturado (afinal você está lendo esse artigo justamente para isso certo?) e quando o assunto é excelência e qualidade não consigo recomendar o suficiente o projeto [4noobs](https://github.com/he4rt/4noobs) que junta em um único repositório diversos cursos grátis e no formato de texto sobre diversos assuntos em TI, para o tema desse artigo recomendo o uso do [markdown4noobs](https://github.com/jpaulohe4rt/markdown4noobs) no aprendizado da linguagem de marcação Markdown.

### O básico sobre manipulação de texto e blocos de código

Markdown nos permite marcar partes do nosso texto com estruturas super básicas e necessárias como negrito, itálico, níveis de título, etc. Abaixo vamos ver rapidamente como realizar cada uma das ações com a sintaxe correta.

```
# Primeiro titulo equivalente a um h1
## Segundo titulo equivalente a um h2
### Terceiro titulo equivalente a um h3
#### Quarto titulo equivalente a um h4

**Texto em negrito**
*Texto em itálico*
```

Outra coisa importante de se mencionar é o bloco especifico que usei acima, ele é extremamente útil quando vamos escrever artigos técnicos, tanto, pois ele permite um destaque maior a um bloco de texto quanto ele permite habilitar syntax highlighting caso esteja escrevendo um bloco de código, ele é utilizado da seguinte maneira:

> PS: Como markdown não permite um bloco dentro de um bloco optei por mostrar com um screenshot:

![Bloco de codigo](https://github.com/cherryramatisdev/public_zet/assets/86631177/61de98aa-e7bb-4baa-91c4-afca9db2991f)

Após os símbolos de apostrofo, podemos incluir o nome da linguagem (no meu caso ruby) para que o dev.to possa habilitar o syntax highlighting especifico para essa linguagem de programação.

### Tabela de conteúdo

Caso o seu artigo esteja ultrapassando a margem das duas mil palavras ou esteja com pelo menos 4 título, eu recomendo fortemente que deixe definido um `Table of contents`, ou `Tabela de conteúdo`. Essa tabela de conteúdo serve para guiar a leitura durante os pontos principais que serão lidados durante o artigo, para a criação de um existem uns macetes que vou demonstrar abaixo:

#### Na plataforma dev.to, use listas não ordenadas ao invés de listas numeradas

Listas em Markdown são bem simples de serem usadas e elas possuem dois tipos **principais**: não ordenadas e numeradas.

```
- Uma lista
- Não
- Ordenada

1. Uma lista
2. Numerada
3. Aqui
```

O problema de usar as listas numeradas no dev.to é que elas ficam desalinhadas como podemos ver no exemplo abaixo, portanto não recomendo o uso delas de maneira geral, sempre tento utilizar as listas não ordenadas e se for necessário aplicar alguma ordem utilizar um número após o símbolo da lista não ordenada manualmente.

![Listas no dev.to](https://github.com/cherryramatisdev/public_zet/assets/86631177/0ab1a9c1-efb3-40d5-b90f-7cacb7d20f77)

#### Como organizar os links para um título

Assumindo que você já descobriu como estruturar um link no Markdown (já que você leu o markdown4noobs né?) vamos aprender os macetes simples para indicar um link em um título e como estruturar nossa tabela de conteúdo.

Uma tabela de conteúdo de exemplo pode ser vista abaixo:

```
## Table of contents

- [What is metaprogramming anyway?](#what-is-metaprogramming-anyway)
- [In ruby everything is an object, what does that mean?](#in-ruby-everything-is-an-object-what-does-that-mean)
- [But what about rails? How this framework applies that concept for maximum developer experience](#but-what-about-rails-how-this-framework-applies-that-concept-for-maximum-developer-experience)
- [How to define methods dynamically](#how-to-define-methods-dynamically)
- [Using hooks to detect moments on the instantiation of the class](#using-hooks-to-detect-moments-on-the-instantiation-of-the-class)
- [Conclusion](#conclusion)
```

Como você pode ver, a ideia geral para definir a segunda parte do link é incluir uma hashtag `#` junto ao título em um formato especifico seguindo as regras:

- Trocar todos os espaços em branco por hifens `-` 
- Deixar todo o título em minúsculo

E é isso! Títulos com acentos podem continuar igual sem nenhum problema, o Markdown compreende como texto padrão igual mostrado abaixo:

```
- [Um título com muitos acentos e çedilha](#um-título-com-muitos-acentos-e-çedilha)
```

## A estrutura básica de um artigo técnico

Agora que temos uma noção interessante de como podemos marcar nosso texto para ficar legível e agradável de ler, vamos entender um pouco a estrutura de um artigo. É importante ressaltar que um modelo não vai servir para todo tipo de texto possível, a ideia é prover uma ideia geral que deve ser adaptada e mudada conforme o contexto.

Primeiro é muito importante definir o parágrafo inicial para chamar o leitor para o problema ou a situação que você vai dissecar ao longo do artigo, é importante fazer isso, pois esse primeiro paragrafo vai ser usado pelo dev.to para comunicações de marketing por e-mail ou por redes sociais. Um exemplo de parágrafo inicial pode ser achado nesse mesmo artigo que você esta lendo ou em alguns outros que deixo abaixo:

![Paragrafo inicial exemplo 1](https://github.com/cherryramatisdev/public_zet/assets/86631177/a76e0a72-60a7-4864-b2e9-f43922a8e0fb)

![Paragrafo inicial exemplo 2](https://github.com/cherryramatisdev/public_zet/assets/86631177/64fc0d55-eed4-4c81-b4cf-193cf0d594a6)

A ideia é sempre utilizar perguntas e pausas no texto para podermos alcançar uma comunicação direta e bem conversacional, sempre tentando apresentar a situação da maneira mais geral possível para que quem esteja lendo fique com bastante curiosidade e vontade de ler.

Após o primeiro parágrafo de apresentação, é muito importante definir a [Tabela de conteúdo](#tabela-de-conteúdo) para guiar o usuário ao longo dos títulos principais do seu artigo, sobre isso inclusive eu particularmente não recomendo listar os subtítulos junto dos títulos por conta de deixar a tabela de conteúdo muito grande e pouco útil para quem estiver lendo, obviamente que se você considerar seus subtítulos muito importantes de serem listados é completamente valido incluir.

Passando para o corpo geral do artigo, entramos em uma área muito subjetiva, pois depende muito do assunto que está sendo abordado para entender a estrutura dos seus títulos e parágrafos. Vou assumir um artigo no modelo de tutorial simples para ser possível partir de algum lugar.

Recomendo sempre ter três *títulos satélites* que vão nortear o seu artigo e dar inspiração para aumentar o conteúdo com mais detalhes. Esses títulos satélites são os seguintes:

- `Introdução a tecnologia ou problema`: Esse parágrafo vai servir para podermos detalhar o que foi falado no começo do artigo, respondendo às perguntas que criamos para atiçar a curiosidade e aprofundando mais no tema que vai ser discutido com os tópicos específicos.
- `Prós e contras`: Nesse momento vamos deixar claro os prós e contras da solução que vai ser apresentada no artigo, seja ela uma arquitetura, um padrão de código, uma linguagem, framework, etc. A existência desse paragrafo pode ser bem situacional dependendo do seu tema, mas costuma ser bem útil caso esteja apresentando uma solução em forma de tutorial!
- `Conclusão`: Esse ponto é mais uma opinião do que uma regra geral, mas eu acho muito importante ter um parágrafo onde vai ser indicado o final do processo de leitura, assim podemos deixar argumentos finais, agradecimentos, contato e qualquer outra mensagem interessante.

Ao redor desses três títulos iniciais, o resto do artigo vai ser definido tentando sempre aprofundar os temas e resolvê-los com conclusões simples e diretas.

## Como revisar e melhorar a escrita

Bom, agora que temos uma boa ideia de como estruturar nosso artigo, como mantê-lo bonito utilizando Markdown e como ponderar nossa linguagem para a plataforma especifica, o que falta? Bom, agora o que falta é entender que não somos perfeitos e erramos, portanto, precisamos de uma boa estratégia para revisar o artigo que acabamos de produzir com nossas técnicas aprendidas.

Para auxiliar na escrita, eu recomendo fortemente utilizar um editor que oferece visualização do Markdown em tempo real como [VSCode](https://code.visualstudio.com/Docs/languages/markdown) ou o queridinho da comunidade [Obsidian](https://obsidian.md). Inclusive esse artigo foi escrito utilizando o Obsidian!

No que se entende por revisão temos algumas ferramentas bem interessantes que nos auxiliam em diversos aspectos da escrita:

- [LanguageTool](https://languagetool.org): Essa ferramenta é meu xodózinho e ela cuida de toda correção de ortografia, a parte legal é que nessa ferramenta você pode ter dicas de contexto que vão melhorar a frase e também correção de termos específicos de programação como nomes de linguagens visto que o banco de dados deles é super atualizado.
- [Deepl](https://www.deepl.com/translator): Entrando no mundo das inteligências artificiais, o Deepl utiliza deep learning para oferecer uma interface incrível de tradução, mas não só isso! Com ele podemos ter uma segunda opinião para refrasear nosso parágrafo de forma muito simples, você só precisa traduzir o texto para inglês e traduzir o texto em inglês para português de novo; normalmente no Google Translate isso destruiria o contexto, mas essa ferramenta mantém o contexto e melhora as expressões para que você possua uma segundo percepção de um mesmo paragrafo.
- [ChatGPT](http://chat.openai.com) ou [Bard](https://bard.google.com): Bom, aqui eu confesso que não tenho muito domínio e não uso tanto, mas essas interfaces de chat com IA ajudam muito a ter visões diferentes para refrasear um parágrafo já existente ou até mesmo começar a escrever algum. **Importante: Preciso ressaltar para usar essas ferramentas apenas para ter ideias ou ajudar a refrasear textos que você já escreveu, por favor não produza artigos inteiros utilizando inteligência artificial**
- [Comunidade](https://heartdevs.com): Na comunidade He4rt Developers tentamos providenciar o máximo de auxílio para a escrita de artigos técnicos na plataforma dev.to. Fazemos isso providenciando um fórum onde você pode postar seu artigo ainda em progresso e ganhar feedback da comunidade até a publicação, após a publicação a gente ainda faz um trabalho de divulgação para pessoas ativas! **Disclaimer: Obviamente estou citando a He4rt, pois temos um projeto voltado para isso, mas a lição geral é compartilhar seus progressos com a comunidade de maneira geral.**

## Conclusão e agradecimentos

Esse é o último artigo que estou postando seguindo o desafio [100 dias de código](https://www.100diasdecodigo.dev), foi um desafio muito intenso e com muito aprendizado onde eu descobri uma nova paixão: escrever e compartilhar conhecimento! Não tenho nem palavras para agradecer a comunidade da He4rt por ter me apoiado 100% nessa jornada imensa. Espero que esse artigo seja útil para quem esteja lendo e que inspire qualquer um a compartilhar conhecimento online para podermos criar uma internet mais segura e rica em informação. May the force be with you! 🍒
