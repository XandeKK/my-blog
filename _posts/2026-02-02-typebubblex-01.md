---
title: O Planejamento do Novo TypeBubbleX
date: 2026-02-02
categories: [TypeBubbleX]
tags: [typebubblex, open-source, devlogs, godot-engine, scanlation]
---

Neste post, quero relatar meus planos e objetivos para entender melhor meu processo de pensamento. Sinto que, se eu apenas fizer as tarefas sem registrá-las, perco a noção do progresso e acabo esquecendo se algo foi difícil ou como resolvi um problema. Recentemente, ao revisar um código antigo, fiquei surpreso por não lembrar como cheguei àquela solução ou o nível de dificuldade que enfrentei.

Certo, antes de falar dos planos, deixe-me apresentar o TypeBubbleX.

## O que é TypeBubbleX?

O **TypeBubbleX** é um aplicativo focado em agilizar o *typesetting* (diagramação) de mangás, manhwas, manhuas e quadrinhos em geral.

Explicando de forma simples, é um editor de imagem especializado em  uma única missão: inserir textos em balões de fala com perfeição e rapidez.

Você pode se perguntar, "Por que não usar o Photoshop ou o GIMP?".

O problema é que esses softwares são... *generalistas*. Eles carregam centenas de recursos que um *typesetter* nunca vai usar, o que resulta em uma interface poluída, consumo excessivo de RAM e um fluxo de trabalho truncado.

O TypeBubbleX segue a filosofia da especialização, enquanto o Photoshop/GIMP é um "pau para toda obra", o TypeBubbleX é um especialista. Onde você levaria minutos configurando camadas e textos no PS/GIMP, aqui você faz em segundos.

Só para deixar claro que não estou desmerecendo o Photoshop ou o GIMP, a questão é que eu quero é **eficiência** e **foco**.

Certo, agora vou descrever as funcionalidades que o aplicativo tem.

### As principais funcionalidades

#### Lista de Texto

Uma das coisas que mais atrasam o fluxo de trabalho de um *typesetter* é ficar alternando entre janelas (o editor de texto com traduções e o PS/GIMP), além de ter que usar o tempo todo Ctrl+C e Ctrl+V e selecionar as falas com mouse.

Então, para resolver isso, acabei embutindo uma lista de texto para que você simplesmente clique no item para que ele insira o texto automaticamente o texto no balão ou então apertar Ctrl+Space sem a necessidade de utilizar o mouse.

Dava para melhorar ele, mas deixei o mais simples possível, pois queria ver ele funcionando. Mas prometo que na nova versão terá uma melhoria absurda.

Você pode ver o vídeo abaixo, a lista de texto está lá embaixo no canto esquerdo. Ignore o fato de eu não ter enquadrado o texto no balão, pois o objetivo é demostrar que inserir o texto é simples e rápido.

{% include embed/youtube.html id='hUcW_agdimk' %}

#### Gerenciador de Fontes

Não lembro exatamente qual foi o motivo de eu ter criado isso, e é por isso que quero relatar tudo o que estou pensando agora, para não esquecer de novo. Tenho certeza de que havia um plano envolvendo uma lista de textos. Bem, devo lembrar quando eu fizer a nova versão.

Uma das coisas que mais toma tempo na vida de um typesetter é ficar garimpando a fonte ideal em listas infinitas para descobrir, só na hora do teste, que ela não era tão boa quanto parecia.

Para resolver isso, essa ferramenta funciona como um verdadeiro centro de controle. Em vez de você sofrer para lembrar nomes técnicos complexos (quem diabos lembra o que é Brusch Casual Oblique Medium de cabeça?), você pode simplesmente dar um "apelido" para a fonte, como "Grito", e focar no sentimento que ela passa para a cena.

Além disso, a ferramenta conta com um preview instantâneo: clicou, viu. E se a fonte ideal não tiver uma versão Bold ou Italic nativa, dá para "forçar" esses estilos direto na interface por meio de transformações visuais, o que quebra um galho absurdo na hora da edição.

Você pode conferir o funcionamento no vídeo abaixo.

{% include embed/youtube.html id='oAsmwjaVttE' %}

#### Perspectiva falsa

Eu lembro que sofri bastante para fazer essa funcionalidade. De vez em quando, reviso o código e fico pasmo de como diabos eu criei aquilo, nem eu entendo mais o que escrevi.

Bem, como o nome sugere, a perspectiva falsa simula uma profundidade ou tridimensionalidade em um ambiente que, tecnicamente, é estritamente 2D.

Ela é bastante útil, pois há muitas situações em que você precisa de um texto com perspectiva.

Ela também precisa de uma melhoria, mas isso só na nova versão.

{% include embed/youtube.html id='x1ZHDW5O9B0' %}

#### Contornos

Esta foi a primeira funcionalidade que desenvolvi, pois todas as obras precisam utilizar contornos.

Como o nome sugere, você pode adicionar contorno ao texto e ajustar sua posição, tamanho e cor, ou até aplicar um efeito de gradiente. Além disso, é possível adicionar quantos contornos você desejar.

{% include embed/youtube.html id='7D-WXEo7b3U' %}

#### Desfoque de movimento

Eu lembro que existia um bug específico do motor, relacionado ao SubViewport e SubViewportContainer do [Godot](https://godotengine.org/). Na época, sofri um pouco para encontrar uma gambiarra que funcionasse, mas acredito que isso tenha sido resolvido nas versões mais recentes.

Como o nome sugere, ao aplicar o desfoque, basta definir a força e a direção. Simples assim.

É um efeito bem útil para enfatizar sensações de velocidade ou até para criar um impacto visual em cenas de "grito".

{% include embed/youtube.html id='ny4EPrfhRSA' %}

#### Gradiente

Esse foi bem chato de desenvolver. Tudo o que eu tentava não funcionava, então a solução acabou não sendo do jeito que eu gostaria, mas fazer o quê, né? O que importa é que funciona.

Geralmente, esse efeito é usado apenas em manhwas ou manhuas, raramente você precisará utilizá-lo em mangás.

{% include embed/youtube.html id='Hxk6kbnPvaw' %}

#### Máscara

Meu amigo, este recurso foi o campeão de alterações. Eu lembro que fiz a primeira versão, mas ela travava muito, então decidi removê-la. Depois tive outra ideia e criei a segunda versão, ela não travava, mas era ruim de usar e nada intuitiva, então removi de novo. Até que cheguei na terceira versão, que é rápida, porém não tem cursor e o pincel tem um formato quadrado (como você pode ver no vídeo abaixo). Por isso, essa funcionalidade será alterada novamente na próxima versão.

Esta ferramenta permite criar uma máscara para ajustar a opacidade do texto. O objetivo é simular profundidade quando houver objetos sobrepostos ou aplicar um efeito de transparência para integrar melhor o texto ao cenário.

{% include embed/youtube.html id='foZP_kIvlIk' %}

---

Essas são as funcionalidades principais. Agora que você conhece um pouco do meu aplicativo, vou explicar quais serão as modificações.

## Um novo TypeBubbleX

Vai ser um desafio explicar tudo e, confesso, dá uma certa preguiça de escrever, mas vamos lá.

Para facilitar, fiz uma lista e vou explicar cada item resumidamente, pois as funcionalidades terão posts específicos no futuro.

* Gerenciador de Obras
  * Gerenciador de Personagens
  * Gerenciador de Locais
  * Gerenciador de Glossário
  * Gerenciador de Capítulos
    * Editor de Texto
      * OCR
      * Tradução automática
      * Dicionário
      * Monitoramento em Tempo Real
    * Editor de Imagem
      * Mesmas funcionalidades da versão antiga
      * Texto Curvado e Pattern Texto
      * Edição de Texto In-Canvas
      * Mesclagem de páginas


Certo, acho que listei tudo o que virá por aí.

### Gerenciador de Obras

Uma mudança drástica em relação à versão anterior do aplicativo é que ele não será mais apenas um editor de imagens. Ele se tornará um conjunto de ferramentas para auxiliar todo o processo de *scanlation* (tradução e *typesetting* de quadrinhos). O Gerenciador de Obras é o pilar dessa mudança.

Afinal, o meu foco é centralizar todas as ferramentas utilizadas no processo. Ou seja, o aplicativo não será voltado apenas ao *typesetter*, mas também ao tradutor.

**Mas o que seria o Gerenciador de Obras?**

Bem, como o nome sugere, ele servirá para organizar, em um só lugar, todos os projetos em que o usuário estiver trabalhando.

Ao selecionar uma obra, o usuário terá acesso aos gerenciadores de personagens, locais, glossário e capítulos.

Provavelmente, ao ver a lista acima, você já consiga ter uma ideia do que estou criando.

### Gerenciador de personagens, locais e glossários

Quando eu traduzia as obras, eu fazia uma planilha de personagens, locais e glossários, porém era cansativo ficar alternando as janelas para mexer nela. Comecei a ter esse hábito de registrar tudo só depois de alguns meses de tradução, pois me deparei com um capítulo de flashback de um personagem que tinha aparecido uma única vez em um capítulo específico.

Então, como eu gosto de manter a coerência da tradução, principalmente em expressões, poderes e habilidades, pretendo criar algo mais prático.

Um recurso que será bastante útil nesses gerenciadores é o monitoramento em tempo real. Durante a tradução, a ferramenta verifica o texto original e identifica se existe o nome de um personagem, local ou termo do glossário. Se encontrar, ele destaca a palavra com uma cor de fundo para notificar o tradutor de que aquele termo já existe no banco de dados.

Essa funcionalidade será essencial, principalmente para quem traduz línguas como japonês, mandarim ou coreano. Para quem não sabe, os nomes nessas línguas podem ser traduzidos de formas diferentes. No japonês, por exemplo, existe o **Gikun**, em que se faz uma leitura **"criativa"** em vez de seguir as regras padrões do kanji.

Por exemplo, na escrita está o kanji de Sora (Céu), mas o autor define que a leitura deve ser Aoi (Azul). Por que? Porque o céu é azul, logo, o autor faz essa associação lírica. Sem um gerenciador para fixar que aquele kanji específico deve ser lido como "Aoi" naquela obra, a chance de erro ou inconsistência é enorme.

Acho que agora deu para entender o porquê de ter um gerenciador, né?

### Gerenciador de capítulos

Ao contrário da versão anterior do TypeBubbleX, que gerenciava somente um capítulo por vez, a nova versão gerenciará todos os capítulos já lançados. Isso é ideal para fazer uma edição rápida, caso haja algum erro gramatical ou ortográfico, ou até mesmo para uma consulta rápida para refrescar a mente.

Ao selecionar um capítulo, o usuário terá as opções de tradução, edição de imagem ou leitura, caso o capítulo já tenha sido finalizado.

Então, vamos conferir quais funcionalidades o editor de texto terá.

#### Editor de texto

Eu tenho um editor de texto muito básico, o [text editor](https://github.com/XandeKK/text_editor). Eu sei que não tem nada no README, eu planejava refazê-lo, mas acabei desistindo, pois ele cumpria bem as tarefas e foi criado apenas para meu uso pessoal.

Ele possui um OCR que utiliza o [Tesseract](https://github.com/tesseract-ocr/tesseract) e contava com o Google Tradutor para auxiliar em algumas traduções.

Agora, como pretendo embutir um editor de texto no TypeBubblex, vou utilizá-lo como uma referência do que posso melhorar.

##### OCR

Uma coisa que não pode faltar é, obviamente, o OCR, pois ele ajuda muito na hora de transcrever, principalmente quando você está traduzindo línguas como japonês, mandarim ou coreano.

Eu planejava utilizar o Tesseract, mas vou tentar procurar outros OCRs, porque podem existir alguns que trabalham muito bem com línguas específicas.

##### Tradução Automática

Uma coisa que é bastante criticada, e, sinceramente, a inteligência artificial é sua assistente quase perfeita, ela vai te ajudar muito, é que você não pode simplesmente deixar ela traduzir sozinha. Você tem que conferir e ajustar. Tem muito desgraçado que simplesmente copia e cola a tradução e taca o foda-se se está correto contextualmente ou se está coerente com a história ou não.

No começo, eu utilizava o Google Tradutor para traduzir alguns termos e, de certo modo, ele faz isso bem. O problema é que traduzir não é simplesmente "traduzir", é transformar os sentimentos que o autor quer expressar em outra língua. Você tem que traduzir as emoções, e fazer isso não é tão fácil assim.

A IA ajuda muito quando você quer traduzir um termo, mas não sabe qual palavra usar exatamente, mesmo procurando sinônimos.

Por isso, deixarei o uso de IA como opcional, o usuário pode desativar caso queira.

##### Dicionário

Como já quero centralizar tudo, por que não centralizar o dicionário também?

Basicamente, o tradutor pode procurar o significado da palavra, simples assim.

Isso vai ser bastante útil. A questão é... onde vou arranjar isso? Não sei, mas esse é um problema do meu eu do futuro.

##### Monitoramento em Tempo Real

Já citei ele acima no tópico de [Gerenciador de personagens, locais e glossários](#gerenciador-de-personagens-locais-e-glossários), então não vou repetir.

---

Possivelmente haverá mais funcaionalidades no editor de texto, mas isso será mostrado em um post específico sobre o editor.

Agora, vamos para o editor de imagem.

#### Editor de imagem

Obviamente, a nova versão do editor de imagem terá as mesmas funcionalidades de antes, então não vou repetir tudo aqui de novo.

Mas o que queremos saber é: quais são as novas funcionalidades? Bem, você já viu a lista lá em cima, mas a questão é: o que elas fazem?

Bem, vamos começar com **Texto Curvado**.

##### Texto Curvado

Não sei se eu chamaria de "texto curvado", acho que seria mais correto chamar de Texto em Caminho (Path Text), mas "texto curvado" é mais fácil de entender.

É um recurso raramente usado, mas quando você precisa e não tem, faz muita falta. Então, vou criar só por causa disso.

E creio que não tem muito o que explicar sobre ele, então vamos partir para o próximo recurso.

##### Pattern Text

Não sei exatamente qual seria a tradução para esse recurso, mas seria algo como aplicar uma "textura" ao texto.

Por exemplo, como mostrado na imagem abaixo, é possível aplicar uma textura de tijolos diretamente nos caracteres.

![https://flyingmeat.com/acorn/docs/filling_text.html](/assets/img/typebubblex/pattern.png){: width="500"  }
_https://flyingmeat.com/acorn/docs/filling_text.html_

Esse efeito é amplamente utilizado em mangás para dar profundidade ou estilo às onomatopeias e títulos.

##### Edição de Texto In-Canvas

Na versão antiga, para escrever, você precisava usar um box que ficava no canto inferior esquerdo.

Mas agora, na nova versão, pretendo fazer com que você escreva diretamente no canvas, como no PS ou GIMP, pois assim o processo se torna muito mais intuitivo e rápido.

Vai ser trabalhoso fazer isso? Vai. Mas, sinceramente, como eu já disse, eu quero desafio.

##### Mesclagem de páginas

Bastante útil quando o mangá tem aquela parte em que o desenho ocupa duas páginas. É chato ter que abrir o PS ou o GIMP só para fazer a mesclagem e ainda ter que renomear o arquivo para manter a ordem certa das páginas.

Por exemplo, veja a imagem abaixo:

![Davre no Oukan](/assets/img/typebubblex/Davre-no-Oukan.jpg)
_Davre no Oukan_

---

Bem, tem mais funcionalidades, mas ainda estou pensando se vou colocar ou não e se serão úteis. Acho melhor pararmos por aqui, porque este post já está bem longo.

Então é isso, o próximo post será sobre wireframe. Até a próxima!