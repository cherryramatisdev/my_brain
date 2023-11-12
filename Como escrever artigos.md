---
title: Como escrever artigos
description: Compartilhar conhecimento escrito é uma ótima forma de dominar um assunto especifico alem de ser uma excelente forma de melhorar a organização das ideias, comunicação e obviamente se auto promover na comunidade. Nesse artigo vamos discutir como organizar as ideias e produzir conteúdos técnicos!
tags: beginners,tutorial,writing
cover_image: ""
canonical_url: 
published: false
---

Compartilhar conhecimento escrito é uma ótima forma de dominar um assunto especifico alem de ser uma excelente forma de melhorar a organização das ideias, comunicação e obviamente se auto promover na comunidade. Essa produção de artigos tanto técnicos quantos sociais são muito importantes para quem escreve para quem está lendo, a velha verdade sempre prevalece: `Você sabe mais hoje do que quem começou ontem`, portanto vamos discutir um pouco mais sobre como organizar as próprias ideias na produção de um artigo técnico para a plataforma dev.to!

## Table of contents

- [Coletando ideias e se motivando para escrever](#coletando-ideias-e-se-motivando-para-escrever)
- [Entendendo a plataforma e achando a propria linguagem](#entendendo-a-plataforma-e-achando-a-propria-linguagem)
- [Aprendendo markdown e dicas gerais para uma boa formatação](#aprendendo-markdown-e-dicas-gerais-para-uma-boa-formatacao)
- [A estrutura básica de um artigo técnico](#a-estrutura-basica-de-um-artigo-tecnico)
- [Como revisar e melhorar a escrita](#como-revisar-e-melhorar-a-escrita)
- [Conclusão e agradecimentos](#conclusao-e-agradecimentos)

## Coletando ideias e se motivando para escrever

Ter ideias é provavelmente uma das fases mais chatinhas antes de produzir um artigo online e, infelizmente, a solução também é bastante simples. **Consumir conteúdos**. A forma mais simples de encontrar ideias e de encontrar a sua própria linguagem é ler o que as pessoas já produzem sobre os temas que lhe interessam, quer se trate de uma linguagem de programação, de um tema específico em TI, etc. Leia o máximo possível!

Outra dica importante é manter um *segundo cérebro*, uma pasta de anotações onde você copia partes de artigos legais que leu na internet junto com observações próprias de forma super informal. Isto vai ajudá-lo a construir um repertório e a ganhar mais confiança ao escrever, uma vez que pode citar frases que leu em outros locais e abrir portas para muito mais contexto, afinal não podemos confiar no nosso próprio cérebro para lembrar de tudo o que consumimos não é mesmo?

## Entendendo a plataforma e achando a propria linguagem

É super importante de entender a plataforma e o público que vamos atingir escrevendo nossos conteúdos para podermos filtrar como vamos estruturar os artigos de forma geral certo?  O [dev.to](https://dev.to) é uma plataforma bem informal na *minha opinião* que valoriza bastante conteúdos conversacionais e diretos no formato de tutoriais. 

Isso significa que tudo o que você vai produzir são tutoriais diretos e informais?  De forma alguma! Só significa que você pode moldar o seu conteúdo para conter essa linguagem mais informal, conversacional e direta mesmo que o tema tratado seja super complexo, isso inclusive vira um desafio muito interessante de simplificar o complexo. 

*PS: Essa habilidade de simplificar o complexo vai te acompanhar para o resto da sua vida profissional, então pratique bastante*

## Aprendendo markdown e dicas gerais para uma boa formatação

A forma como produzimos nossos artigos para o dev.to é utilizando uma linguagem de marcação (exatamente, igual o HTML) para formatação de textos chamada [Markdown](https://www.markdownguide.org), apesar dela ser super simples é importante termos um domínio bem solido do que é possível fazer quando falamos sobre organizar e deixar nosso texto bonitinho.

Abaixo vou listar algumas partes bem importantes de saber quando vamos escrever artigos técnicos, mas recomendo fortemente o [4noobs](https://github.com/he4rt/4noobs) sobre markdown chamado [markdown4noobs](https://github.com/jpaulohe4rt/markdown4noobs)
### O básico sobre manipulação de texto e blocos de código

Coisas como aprender a deixar um texto em negrito, itálico, declarar níveis de titulo são habilidades básicas que é necessário para estruturar qualquer artigo, abaixo tem todas as formas mais simples de marcar textos:

```
# Primeiro titulo equivalente a um h1
## Segundo titulo equivalente a um h2
### Terceiro titulo equivalente a um h3
#### Quarto titulo equivalente a um h4

**Texto em negrito**
*Texto em itálico*
```

Outra coisa importante de se mencionar, esse bloco especifico acima é extremamente útil quando vamos escrever artigos técnicos, tanto pois ele permite um destaque maior a um bloco de texto quanto ele permite habilitar syntax highlighting caso esteja escrevendo um bloco de código, ele é utilizado da seguinte maneira:

> PS: Como markdown não permite um bloco dentro de um bloco, vou precisar mostrar com um screenshot.


![[Pasted image 20231112112559.png]]

Após os símbolos de apostrofo, podemos incluir o nome da linguagem (no meu caso ruby) para que o dev.to possa habilitar o syntax highlighting especifico para essa linguagem de programação.

### Tabela de conteúdo

Caso o seu artigo esteja ultrapassando a margem das duas mil palavras ou esteja com pelo menos 4 título eu recomendo fortemente que deixe definido um `Table of contents` ou `Tabela de conteúdo`. Essa tabela de conteúdo tem uns macetes que vamos aprender abaixo:

#### Na plataforma dev.to, use listas não ordenadas ao invés de listas numeradas

Listas em markdown são bem simples de serem usadas e elas possuem dois tipos **principais**: não ordenadas e numeradas.

```
- Uma lista
- Não
- Ordenada

1. Uma lista
2. Numerada
3. Aqui
```

O problema de usar as listas numeradas no dev.to é que elas ficam desalinhadas no momento de escrita desse artigo como podemos ver no exemplo abaixo:

![[Pasted image 20231112113823.png]]

Portanto de forma geral eu sempre assumo o uso de listas não ordenadas sempre que preciso utilizar uma lista no meu texto, facilita bastante a vida e deixa alinhado com o resto do paragrafo.

#### Como organizar os links para um título

Assumindo que você já descobriu como estruturar um link no markdown (ja que você leu o markdown4noobs né?) vamos aprender os macetes simples para indicar um link em um titulo e como estruturar nossa tabela de conteúdo.

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

Como você pode ver, a regra geral para definir a segunda parte do link é incluir uma hashtag `#` junto com o titulo em um formato especifico seguindo as regras:

- Trocar todos os espaços em branco por hífens `-`
- Deixar todo o titulo em minúsculo

E é isso! Títulos com acentos podem continuar igual sem nenhum problema, o markdown compreende como texto padrão igual mostrado abaixo:

```
- [Um título com muitos acentos e çedilha](#um-título-com-muitos-acentos-e-çedilha)
```

## A estrutura básica de um artigo técnico

Agora que temos uma noção interessante de como podemos marcar nosso texto para que fique legível e agradável de ler, vamos entender um pouco a estrutura de um artigo medio *na minha opinião*.

Primeiro é muito importante definir o paragrafo inicial de forma a chamar o leitor para o problema ou a situação que você vai dissecar ao longo do artigo, fazemos isso principalmente pois esse primeiro paragrafo é o que o dev.to vai usar para mandar por email em suas newsletter ou outras comunicações de marketing. Um exemplo de paragrafo inicial pode ser achado nesse mesmo artigo que você esta lendo ou em alguns outros que deixo abaixo:

![[Pasted image 20231112120801.png]]

![[Pasted image 20231112120819.png]]

A ideia é sempre utilizar essas perguntas e pausas no texto para que possamos alcançar uma comunicação direta e bem conversacional, sempre tentando apresentar a situação da maneira mais geral possível para que quem esteja lendo fique com bastante curiosidade e vontade de ler.

Após o primeiro paragrafo de apresentação, é muito importante definir a [Tabela de conteúdo](#tabela-de-conteúdo) para guiar o usuário ao longo dos títulos principais do seu artigo, sobre isso inclusive eu particularmente não recomendo listar os subtítulos junto dos títulos por conta de deixar a tabela de conteúdo muito grande e pouco util para quem estiver lendo, obviamente que se você considerar seus subtítulos muito importantes de serem listados é completamente valido incluir.

Passando para o corpo geral do artigo, entramos em uma área muito subjetiva pois depende muito do assunto que está sendo abordado para entender a estrutura dos seus títulos e parágrafos. Vou assumir um artigo no modelo de tutorial simples para que seja possível partir de algum lugar.

Recomendo sempre ter três *títulos satélites* que vão nortear o seu artigo e dar inspiração para aumentar o conteúdo com mais detalhes. Esses títulos satélites são os seguintes:

- `Introdução a tecnologia ou problema`: Esse parágrafo vai servir para que possamos detalhar o que foi falado no começo do artigo, respondendo as perguntas que subimos para atiçar a curiosidade e aprofundando mais no tema que vai ser discutido com os tópicos específicos.
- `Prós e contras`: Nesse momento vamos deixar claro os prós e contras da solução que vai ser apresentada no artigo seja ela uma arquitetura, um padrão de código, uma linguagem, framework, etc. A existência desse paragrafo pode ser bem situacional dependendo do seu tema, mas costuma ser bem util caso esteja apresentando uma solução em forma de tutorial!
- `Conclusão`: Aqui vai ser uma opinião pessoal, mas eu acho muito importante ter um paragrafo onde vai ser indicado o final do processo de leitura, assim podemos deixar argumentos finais, agradecimentos, contato e qualquer outra mensagem interessante.

Ao redor desses três títulos iniciais, fica mais fácil de organizar outros títulos ao redor detalhando e argumento o assunto escolhido.

## Como revisar e melhorar a escrita

Bom, agora temos uma boa ideia de como estruturar nosso artigo, como mante-lo bonito com markdown e como ponderar nossa linguagem para a plataforma especifica, o que falta? Agora o que falta é assumir que sempre erramos e precisamos de uma boa estratégia tanto para escrever nosso artigo quanto para revisar antes de postar.

Para auxiliar na escrita, eu recomendo fortemente utilizar um editor que oferece visualização do markdown em tempo real como [VSCode](https://code.visualstudio.com/Docs/languages/markdown) ou o queridinho da comunidade [Obsidian](https://obsidian.md). Inclusive esse artigo foi escrito utilizando o obsidian!

No que se entende por revisão temos algumas ferramentas bem interessantes que nos auxiliam em diversos aspectos da escrita:

- [Language tool](https://languagetool.org): Essa ferramenta é meu xodózinho e ela cuida de toda correção de ortografia, a parte legal é que nessa ferramenta você pode ter dicas de contexto que vão melhorar a frase e também correção de termos específicos de programação como nomes de linguagens visto que o banco de dados deles é super atualizado.
- [Deepl](https://www.deepl.com/translator): Entrando no mundo das inteligências artificiais o Deepl utiliza deep learning para oferecer uma interface incrível de tradução, mas não só isso! Com ele podemos ter uma segunda opinião para refrasear nosso paragrafo de forma muito simples, você só precisa traduzir o texto para inglês e traduzir o texto em inglês para português de novo; Normalmente no google translate isso destruiria o texto, mas nessa ferramenta funciona muito bem!
- [ChatGPT](http://chat.openai.com) ou [Bard](https://bard.google.com): Bom, aqui eu confesso que não tenho muito domínio e não uso tanto, mas essas interfaces de chat com IA ajudam muito a ter insights para refrasear um paragrafo já existente ou até mesmo começar a escrever algum. **Importante: Preciso ressaltar para que use essas ferramentas apenas para ter ideias ou ajudar a refrasear textos que você já escreveu, por favor não produza artigos inteiros utilizando inteligência artificial**
- [Comunidade](https://heartdevs.com): Na comunidade He4rt Developers tentamos providenciar o máximo de auxilio para a escrita de artigos técnicos na plataforma dev.to. Fazemos isso ao providenciar um fórum onde você pode postar seu artigo que ainda está em progresso e ganhar feedback da comunidade até a publicação, após a publicação a gente ainda faz um trabalho de divulgação para pessoas ativas! **Disclaimer: Obviamente estou citando a He4rt pois temos um projeto voltado para isso, mas a lição geral é compartilhar seus progressos com a comunidade de maneira geral.**

## Conclusão e agradecimentos

Esse é o ultimo artigo que estou postando seguindo o desafio [100 dias de codigo](https://www.100diasdecodigo.dev), foi um desafio muito intenso e com muito aprendizado onde eu descobri uma nova paixão: Escrever e compartilhar conhecimento! Não tenho nem palavras para agradecer a comunidade da He4rt por ter me apoiado 100% nessa jornada imensa. Espero que esse artigo seja útil para quem esteja lendo e que inspire qualquer um a compartilhar conhecimento online para que possamos criar uma internet mais segura e rica em informação. May the force be with you! 🍒
