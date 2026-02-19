---
title: "TypeBubbleX: Banco de dados - Parte 1"
date: 2026-02-19
categories: [TypeBubbleX]
tags: [typebubblex, open-source, devlogs, godot-engine, scanlation, wireframe, database]
---

Certo, como nós terminamos de fazer a maioria dos wireframes, já temos a noção das informações que utilizaremos. Então, o próximo passo é fazer o banco de dados.

Como você pode ver no título, essa será a parte 1. Neste post, não vou codar nada ainda, só vou montar a estrutura. Para isso, utilizarei o [DB Diagram](https://dbdiagram.io/).

Agora, vamos começar a criar as tabelas.

---

## Tabela Works

Essa será a primeira tabela e o "pai" de todas. Sem ela, não será possível fazer nada no sistema.

![Works](/assets/img/typebubblex/database-1/works.jpg)

### ID

A nossa chave primária. Essencial para que todas as outras tabelas consigam se relacionar com uma obra específica.

### Original Title

O título da obra na sua língua materna (ex: o título em japonês ou coreano).

### Translated Title

O título traduzido para a língua alvo do projeto de scanlation.

### Description

A sinopse ou descrição da obra (pode ser a traduzida ou a original).

### Type

Define o formato da obra. Pode ser Mangá, Webtoon, Manhwa, Manhua ou Comics.

### Status

O estado atual da publicação: Em progresso, Hiato, Cancelado ou Finalizado.

### Author Artist

Os nomes dos autores e artistas. Não fiz uma tabela separada para isso porque, honestamente, não vi necessidade de complicar algo simples agora.

---

## Tabela Covers

Toda obra precisa de uma capa, então criamos essa tabela para gerenciar as imagens principais.

![Covers](/assets/img/typebubblex/database-1/covers.jpg)

### ID

Identificador único da capa. O próprio ID será o nome do arquivo de imagem no diretório para facilitar a busca.

### Work ID

Chave estrangeira para vincular a capa à obra correspondente na tabela **Works**.

### Selected

Um booleano para definir qual capa está ativa, já que uma obra pode ter várias capas diferentes (volumes, edições especiais, etc).

---

## Tabela Characters

Aqui é onde guardamos as informações dos personagens que aparecem na história.

![Characters](/assets/img/typebubblex/database-1/characters.jpg)

### ID

Identificador único do personagem.

### Work ID

Vincula o personagem a uma obra específica.

### Original Name / Translated Name

O nome original do personagem e como ele deve ser chamado na tradução oficial para manter o padrão.

### Description

Espaço para anotações sobre a personalidade ou aparência, ajudando o tradutor a não se perder no contexto.

---

## Tabelas Auxiliares de Personagens

Para dar mais profundidade aos personagens, criei essas três tabelas de suporte:

![Characters auxiliar](/assets/img/typebubblex/database-1/characters-auxiliar.jpg)

### Character Images

Funciona como a tabela de capas. Guarda referências visuais do personagem usando o `character_id` e um campo `selected`.

### Character Nicknames

Guarda apelidos `name` e o `context` (contexto) em que são usados. Essencial para saber quando um personagem usa um sufixo honorífico ou um apelido carinhoso.

### Character Chapters

Uma tabela de relação (Many-to-Many) para marcar em quais capítulos cada personagem aparece.

---

## Tabela Locations

Segue a mesma lógica dos personagens, mas voltada para os cenários da obra.

![Locations](/assets/img/typebubblex/database-1/locations.jpg)
![Locations auxiliar](/assets/img/typebubblex/database-1/locations-auxiliar.jpg)


Possui `work_id`, nomes (original/traduzido) e `description`. Assim como nos personagens, temos as tabelas auxiliares **LocationImages** (para referências visuais) e **LocationChapters** (para rastrear aparições).

---

## Tabela Glossary

O coração da consistência linguística do projeto. É o nosso dicionário interno.

![Glossary](/assets/img/typebubblex/database-1/glossary.jpg)
![Glossary chapters](/assets/img/typebubblex/database-1/glossary-chapters.jpg)

### Original / Translated Expression

O termo técnico ou gíria no idioma original e a tradução padronizada que a scan escolheu.

### Examples / Description

Exemplos de uso e uma explicação detalhada de quando aplicar esse termo específico.

---

## Tabela Chapters

Onde a organização dos lançamentos e arquivos realmente acontece.

![Chapters](/assets/img/typebubblex/database-1/chapters.jpg)

### Title

O título do capítulo.

### Number

O número do capítulo. Usei o tipo *float* porque capítulos fracionados (como "10.5") são muito comuns em mangás.

### Language Source / Target

Define os idiomas de origem e destino daquela tradução específica.

---

## Tabelas de Páginas (Raw, Clean e Done)

Dividi o fluxo de trabalho de edição em três estados fundamentais para qualquer scan:

![raw-pages](/assets/img/typebubblex/database-1/raw-pages.jpg)


![clean-done-pages](/assets/img/typebubblex/database-1/clean-done-pages.jpg)

### RawPages

As imagens brutas, do jeito que saíram da fonte original.

### CleanPages

Páginas que já passaram pelo processo de limpeza (limpeza de balões e reconstrução de arte/redraw).

### DonePages

O arquivo final, com texto inserido e pronto para publicação.

---

## Tabela Bubble

![bubble](/assets/img/typebubblex/database-1/bubble.jpg)

### Page ID

Vincula o bubble a uma raw pages específica.

### Original / Translated Text

O texto contido no balão antes e depois da tradução.

### Width / Height / X / Y

As coordenadas exatas e o tamanho do balão na página.

### Metadata

Campo extra para salvar configurações de fonte, tamanho da letra, cor ou estilos de balão e várias outras coisas. Ele será um json no formato de string.

Até pensei em definir várias colunas para cada coisa, mas o problema é que vou ficar modificando todas as vezes o banco de dados quando houver uma nova ou remoção de uma ferramenta.

---

E com isso, fechamos a nossa estrutura inicial!

![database](/assets/img/typebubblex/database-1/database.png)

Você pode acessar isso [aqui](https://dbdiagram.io/d/6970e209bd82f5fce22bcb25).

Pode parecer muita tabela agora, mas ter esses relacionamentos bem definidos vai facilitar absurdamente a vida na hora de codar as funcionalidades. No próximo post, vamos começar a falar sobre como integrar isso tudo dentro da Godot.

Então é isso. Até a próxima!