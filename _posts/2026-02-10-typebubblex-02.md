---
title: "TypeBubbleX: Wireframe"
date: 2026-02-10
categories: [TypeBubbleX]
tags: [typebubblex, open-source, devlogs, godot-engine, scanlation, wireframe]
---
Agora que o planejamento básico do novo TypeBubbleX está definido, o próximo passo é tirar as ideias do papel e criar o wireframe.

Para quem não está familiarizado, o wireframe é a primeira etapa do design de interface. O objetivo aqui não é a estética, mas sim a **estrutura**: definir quais informações são prioridade e como o usuário vai interagir com a ferramenta antes mesmo de começarmos a colocar cores, fontes ou imagens.

Para isso, nós temos que definir o que o usuário pode fazer.

Eu vou desenhar o wireframe no [Excalidraw](https://excalidraw.com/). Eu poderia utilizar outras ferramentas que são ideais para criação de wireframe, mas como já estou acostumado com ele, vou utilizá-lo.

Como eu já fiz a maioria dos wireframes, eu só vou mostrando as imagens e explicando cada um deles. E também não fiz todos, pois ainda estou em dúvida de como vou fazê-los, então decidi ignorá-los por enquanto, mas você saberá quais foram os que eu deixei de lado.

Então, a primeira coisa que será mostrada para o usuário quando ele iniciar o aplicativo pela primeira vez será essa janela abaixo:

## Primeira Janela

![Primeira janela](/assets/img/typebubblex/wireframe/primeira-janela.png)

Esta janela é crucial para que o usuário defina ou crie o diretório de trabalho do TypeBubbleX. Dentro dessa pasta estarão todas as informações, arquivos e imagens que o aplicativo precisa para funcionar.

Escolhi esse método porque facilita o gerenciamento de arquivos. A outra opção seria permitir que o usuário apontasse diretórios individuais para cada obra, mas isso daria muito trabalho para gerenciar via código (tratar erros de acesso, caminhos quebrados caso o usuário mova a pasta, etc.).

Após preencher o campo e clicar em "OK", o usuário verá esta tela:

## Gerenciador de Obras

![Obras layout 1](/assets/img/typebubblex/wireframe/obras-layout-1.png)
![Obras layout 2](/assets/img/typebubblex/wireframe/obras-layout-2.png)
![Obras layout 3](/assets/img/typebubblex/wireframe/obras-layout-3.png)

Coloquei três imagens para mostrar variações de layout, o padrão provavelmente será o grid. Sinceramente, eu ia explicar cada item, mas decidi não fazer isso porque as funções são intuitivas. Se não forem, então eu falhei na minha tarefa de design.

Na barra de menu, você verá "Configuração" e "Ajuda". Ainda não fiz os wireframes dessas seções, pois ainda não defini exatamente o que colocarei nelas.

Quando o usuário criar uma obra ou clicar em uma existente, será redirecionado para cá:

## Gerenciador de Obra

### Capítulos

![Obra Capítulos](/assets/img/typebubblex/wireframe/obra-capitulo.png)

Vou utilizar o fluxo de "criar primeiro e editar depois", pois acredito que facilita a organização. Na janela acima, ao clicar em um capítulo ou em "Add", surgirá um modal com os processos que o usuário deseja realizar. O objetivo é dar liberdade: o usuário pode querer apenas traduzir, ou fazer o typeset, ou apenas ler.

Além da aba de "Capítulos", temos as abas de "Personagens", "Glossário", "Locais" e "Artes". Confira os wireframes delas abaixo:

Agora vou mostrar das aba "Personagens", "Glossário", "Locais" e "Artes"

### Personagens

![Obra Personagens](/assets/img/typebubblex/wireframe/obra-personagem.png)

### Glossário

![Obra Glossário](/assets/img/typebubblex/wireframe/obra-glossario.png)

### Locais

![Obra Locais](/assets/img/typebubblex/wireframe/obra-local.png)

### Artes

![Obra Artes](/assets/img/typebubblex/wireframe/obra-arte.png)

## Processos

Certo, ao interagir com um capítulo, o usuário passará por diversas etapas. Vou mostrar quase todas as janelas desses processos. A primeira é a de importar imagens, que é obrigatória.

### Importar imagens

![Importar imagens](/assets/img/typebubblex/wireframe/import-pages.png)

### Crop e Merge

![Crop e Merge](/assets/img/typebubblex/wireframe/merge-crop.png)

Aqui temos duas variantes: a de cima para obras em formato de página e a de baixo para o formato scroll (como Manhwa, Manhua e Webtoons).

### Upscale

![Upscale](/assets/img/typebubblex/wireframe/upscale.png)

### Extrair as falas

![Extrair as falas](/assets/img/typebubblex/wireframe/extrair-texto.png)

### Tradução

![Tradução](/assets/img/typebubblex/wireframe/traducao.png)

### Leitura

![Leitura](/assets/img/typebubblex/wireframe/leitura.png)

Sinceramente, cansei.

Alguns wireframes, como o de typeset, ainda não foram feitos. Ele envolve muitas variáveis e ainda estou pensando na melhor forma de organizar as funcionalidades.

Sei que este post teve pouco texto e poucas explicações detalhadas, mas é difícil descrever cada clique dessa fase. Vou compensar isso no próximo post sobre Banco de Dados, onde tentarei explicar cada tabela e coluna detalhadamente.

Então é isso. Até a próxima!

