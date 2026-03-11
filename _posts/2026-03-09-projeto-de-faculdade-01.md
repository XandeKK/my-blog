---
title: "Meu Primeiro Jogo"
date: 2026-03-09
categories: [Primeiro Jogo]
tags: [open-source, devlogs, godot-engine, game]
---

Eu tenho um novo projeto. Eu não queria interromper o [TypeBubbleX](/categories/typebubblex/), mas a faculdade vem em primeiro lugar. Basicamente, preciso criar um protótipo que aplique tudo o que aprendi nos cursos da graduação até agora.

### A Narrativa do Jogo

> Você assume o papel de um **Explorador Subaquático**, responsável por investigar setores específicos de uma cidade alagada. Utilizando equipamentos adequados e ferramentas básicas, sua missão é localizar e recuperar um **Módulo de Energia Azul**, capaz de reativar sistemas sustentáveis em parte da região.
>
> Durante a exploração, o personagem percorre ruas parcialmente submersas e estruturas danificadas, interage com mecanismos antigos, coleta informações e evita inimigos afetados pela contaminação. Todos os personagens e inimigos se movimentam no chão da cidade, sem flutuar ou pairar, reforçando a ideia de um deslocamento firme sobre as superfícies.

### Objetivos do Projeto

* **Exploração:** Permitir que o jogador navegue por uma área delimitada da cidade.
* **Missão Principal:** Coletar o Módulo de Energia Azul para restaurar o sistema de energia limpa.
* **Interatividade:** Incluir a possibilidade de interagir com objetos do cenário, ativar elementos simples e lidar com inimigos terrestres configurados com IA básica.

---

### O Dilema do Roteiro

Considerando o escopo, pensei em criar uma história simples. O problema é que não consigo ser básico; sempre levo as ideias ao extremo. Por falta de tempo, decidi não focar tanto na narrativa principal — talvez eu inclua apenas fragmentos do que criei. Ou então, simplifico tudo: ele pega o módulo e pronto.

Abaixo, apresento o conceito que desenvolvi (e que não pretendo concluir totalmente para este projeto):

## A Narrativa Estendida

### **I. Antecedentes: A Queda de BlueWave (Há 18 Anos)**

* **A Sabotagem:** A inundação da cidade não foi um desastre natural, mas uma manobra política para criar uma crise e gerar falsos heróis. O plano saiu do controle e o sistema de contenção colapsou.
* **O Êxodo:** 60% da população fugiu para a **Plataforma Oceânica**, o plano de reserva da cidade.
* **O Crime de Omissão:** Os políticos confiscaram os manuais do **Módulo de Energia Azul** (deixados pelo Engenheiro Chefe) para manter o controle. Por incompetência técnica, perderam os documentos e operaram o sistema errado, reduzindo a vida útil do Módulo de **100 para 20 anos**.

### **II. O Protagonista: O Peso do Sobrenome**

* **Perfil:** Homem de 35 anos, filho do Engenheiro Chefe de BlueWave.
* **A Falsa Narrativa:** Ele cresceu acreditando que o pai era um mentiroso incompetente cujas falhas condenaram a cidade. Carrega uma amargura profunda e o desejo de "limpar a sujeira" deixada pela família.
* **O Guardião Silencioso:** Durante 15 anos, ele trabalhou nos níveis baixos da plataforma, usando o conhecimento herdado para remendar o sistema e esticar sua durabilidade através de improvisos e soldas.

### **III. O Ponto de Ruptura**

* **O Diagnóstico Fatal:** Aos 35 anos, o protagonista percebeu que o Módulo Azul começou a pulsar irregularmente. Restavam apenas **alguns meses** antes de um apagão total e o naufrágio da plataforma.
* **A Opressão Política:** Os líderes abafaram os relatórios de risco para evitar revoltas, mas começaram a planejar uma missão de resgate egoísta para garantir sua própria sobrevivência.
* **O Gatilho:** O jogo se inicia no presente, 18 anos após a inundação, no exato momento em que o setor residencial sofre o **primeiro grande apagão de 48 horas**.

### **IV. O Incidente Inicial: A Missão Suicida**

Exausto e sujo de óleo após tentar conter uma explosão nos geradores, o protagonista é convocado à sala de comando climatizada.

* **O Ultimato:** O Governante (um dos sabotadores originais) utiliza a culpa do protagonista contra ele: *"O sistema que seu pai construiu está matando todos nós. Desça e traga um novo módulo ou seja o último a ver as luzes se apagarem."*
* **A Aceitação Amarga:** O protagonista aceita a missão, vendo nela a chance final de superar a suposta incompetência do pai e salvar o povo.

### **V. A Jornada de Descoberta (Gameplay e Conflito)**

Enquanto explora as ruínas submersas, a perspectiva do protagonista começa a mudar:

1. **Memórias e Evidências:** Flashbacks de conversas que ouviu aos 15 anos e documentos encontrados no lodo revelam que seu pai era inocente e que os políticos foram os verdadeiros arquitetos do caos.
2. **A Dualidade da Luta:** O objetivo deixa de ser apenas técnico. Agora, ele luta para recuperar a energia e para sobreviver a uma conspiração que pretende matá-lo.
3. **A Traição Final:** O plano dos políticos é executá-lo assim que ele retornar com o Módulo. Eles pretendem assumir o crédito pela salvação da humanidade e apagar o "filho do traidor" da história.

---

## Desenvolvimento Técnico

Eu planejava fazer o jogo em 2D, porém, para poupar tempo na criação de *assets* manuais, decidi migrar para o **3D**. Pode parecer mais complicado, mas farei **low-poly** com um **shader de pixel art** (como pode ver no vídeo abaixo). Isso me permite criar cenários sem a necessidade de detalhamento excessivo nas texturas.

{% include embed/youtube.html id='hDikuCci-3M' %}

Quanto à perspectiva, o jogo será **top-view**. Ainda estou decidindo se será uma visão totalmente vertical (estilo *Darkwood*) ou uma visão inclinada (estilo *Zelda: A Link to the Past*).

Por enquanto é isso! No próximo post, mostrarei o processo de **greyboxing**, definindo as formas básicas do mundo.

Até a próxima!
