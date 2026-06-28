---
title: "TypeBubbleX Devlog #07: Criando Barra de Menu"
date: 2026-06-28
categories: [TypeBubbleX]
tags: [typebubblex, open-source, devlogs, godot-engine, scanlation, wireframe, gui, ux]
---

## Introdução e Contexto

Hoje vou criar a barra de menu. No post anterior, mencionei que faria o gerenciador de obras ou a barra de menu, e acabei decidindo pela barra de menu. O motivo é que quando eu for desenvolver o gerenciador de obras, já poderei integrá-lo diretamente com os menus.

Como o aplicativo terá diversas janelas — como o gerenciador de obras, o de capítulos, o editor de imagens, o editor de texto, entre outros —, é necessário que o menu se adapte dependendo da tela ativa. Portanto, o menu precisa ser dinâmico. Além disso, teremos seções permanentes, como as opções de Configurações e Ajuda.

Então, vamos começar!

### Estrutura de Nós na Janela Principal

Fiz algumas alterações na janela principal para acomodar o nó `MenuBar`. Adicionei um `MarginContainer` para garantir um espaçamento adequado (talvez eu mude para `PanelContainer` no futuro, mas por enquanto ele serve bem) e um `VBoxContainer` para organizar os elementos verticalmente, um abaixo do outro.

![Nodes](/assets/img/typebubblex/menubar/nodes.png)

## Arquitetura de Dados: Simplificando Itens e Atalhos

Agora, vamos criar o código para o `MenuBar`. Mas, antes disso, vou criar um `Resource` que guardará as informações de cada item do menu. A ideia é criar uma classe contendo o texto do item, um callback (que guarda a referência da função a ser executada quando ele for pressionado), o atalho de teclado e um ID. Esse ID garantirá que o gerenciador de menus, que faremos logo à frente, saiba exatamente qual item disparou o evento para acionar a ação correta.

Aqui está o código inicial da nossa classe `MenuItemConfig`:

```gdscript
class_name MenuItemConfig
extends Resource

var text : String
var callback : Callable
var shortcut : Shortcut
var id : int

func _init(_text : String, _callback : Callable, shortcut_text : String = "") -> void:
	text = _text
	callback = _callback
	
	if not shortcut_text.is_empty():
		shortcut = _create_shortcut_from_string(shortcut_text)
```

O grande ganho de usabilidade aqui foi definir o parâmetro `shortcut_text` como uma simples `String` em vez do tipo `Shortcut`. Fazer isso evita que eu precise ficar instanciando um objeto de atalho completo toda vez que for configurar um novo item de menu. Em vez disso, posso passar uma escrita simples e intuitiva, como "ctrl+q" ou "ctrl+shift+n". Bem melhor, não acha?

### Convertendo Strings em Atalhos da Godot

Para fazer isso acontecer, usei a seguinte função auxiliar:

```gdscript
func _create_shortcut_from_string(text_key : String) -> Shortcut:
	var input_event : InputEventKey = InputEventKey.new()
	var parts : PackedStringArray = text_key.to_lower().strip_edges().split("+")
	
	for part : String in parts:
		match part:
			"ctrl", "control":
				input_event.ctrl_pressed = true
			"shift":
				input_event.shift_pressed = true
			"alt":
				input_event.alt_pressed = true
			"meta", "command", "win":
				input_event.meta_pressed = true
			_:
				var keycode : Key = OS.find_keycode_from_string(part)
				if keycode != KEY_NONE:
					input_event.keycode = keycode
				else:
					printerr("Error: Invalid key found in shortcut: ", part)
					return null
	
	var new_shortcut : Shortcut = Shortcut.new()
	new_shortcut.events.append(input_event)
	return new_shortcut
```

Esta função é a responsável por destrinchar a nossa string e instanciar o `Shortcut` nativo da Godot corretamente. A lógica dela funciona em etapas:

1. **Tratamento da String:** Ela converte o texto para minúsculo (`.to_lower()`), limpa espaços extras (`.strip_edges()`) e quebra a string onde houver o símbolo `+` (`.split("+")`), gerando uma lista de palavras (ex: `["ctrl", "shift", "n"]`).
2. **Varredura dos Modificadores:** Através de um loop e um bloco `match`, ela identifica se a palavra se refere a uma tecla modificadora como **Ctrl**, **Shift**, **Alt** ou **Meta (Command/Win)** e ativa a respectiva propriedade booleana no `InputEventKey`.
3. **Identificação da Tecla Principal:** Caso a palavra não seja um modificador (caindo no bloco `_`), o código utiliza o método de sistema `OS.find_keycode_from_string(part)` para descobrir o código real da tecla (como a tecla **N**, por exemplo). Se a tecla for inválida, um erro é exibido no console.
4. **Construção do Atalho:** Por fim, um novo objeto `Shortcut` é criado, o evento de teclado configurado é anexado a ele e o atalho prontinho é retornado para o nosso item de menu.

## Gerenciamento de Estado Global: O MenuManager

Com a nossa classe de configuração de itens pronta, o próximo passo lógico seria criar o script para o `MenuBar`. No entanto, antes de fazermos isso, precisamos de um mecanismo que permita a qualquer tela do aplicativo avisar o menu de que o contexto mudou.
Para resolver isso de forma elegante e totalmente desacoplada, vamos criar um **Autoload** (Singleton) chamado `MenuManager`. Ele servirá como uma ponte de comunicação global.

```gdscript
extends Node

signal context_changed(menu_structure : Dictionary)

func set_menu_context(menu_structure : Dictionary) -> void:
	context_changed.emit(menu_structure)

```

O código do nosso gerenciador é simples:
* **O Sinal `context_changed`**: Este sinal carrega consigo um dicionário (`Dictionary`) que conterá toda a estrutura de menus que a tela atual deseja exibir.
* **A Função `set_menu_context`**: Sempre que uma nova janela ou tela for carregada na aplicação (seja o gerenciador de obras, o editor de textos, etc.), ela chamará essa função passando a sua própria lista de menus. A função, por sua vez, dispara o sinal para quem estiver ouvindo — que, no nosso caso, será o `MenuBar`.

## Implementação do MenuBar Dinâmico

Agora, vamos para script do `MenuBar`.

```gdscript
extends MenuBar

var _registered_callbacks : Dictionary[String, Dictionary] = {}

func _ready() -> void:
	MenuManager.context_changed.connect(rebuild_menu)
	
	_setup_permanent_menus()
	_clear_context_menus()
```

No topo do nosso script, declaramos um dicionário tipado chamado `_registered_callbacks`. A função dele é guardar temporariamente as referências das funções (*callables*) que serão chamadas quando um item for clicado. Como os menus são dinâmicos, organizamos esse dicionário usando o título do menu como chave principal e os IDs dos itens como subchaves.

Na função `_ready()`, conectamos o sinal `context_changed` do nosso singleton global (`MenuManager`) diretamente à função `rebuild_menu`. Isso garante que, sempre que o usuário mudar de tela, a barra de menu seja avisada. Logo em seguida, chamamos o `_setup_permanent_menus()` para criar os menus fixos e o `_clear_context_menus()` para garantir que a área dinâmica comece limpa.

### Configurando os Menus Fixos e Limpeza

```gdscript
func _setup_permanent_menus() -> void:
	var settings_items : Array[MenuItemConfig] = [
		MenuItemConfig.new("Preferences", _on_global_preferences_pressed)
	]
	
	var help_items : Array = [
		MenuItemConfig.new("Documentation", _on_documentation_pressed),
		MenuItemConfig.new("About TypeBubbleX", _on_about_pressed, 'f1')
	]
	
	_create_submenu("Settings", settings_items, false)
	_create_submenu("Help", help_items, false)
```

Esta função cria as seções que nunca devem sumir da tela, independentemente de onde o usuário esteja navegando no aplicativo. Criamos as listas para **Settings** (Configurações) e **Help** (Ajuda) e chamamos a função de criação passando o último parâmetro como `false`, indicando que esses menus **não** são dinâmicos.

```gdscript
func _clear_context_menus() -> void:
	for child in get_children():
		if child is PopupMenu and child.has_meta("is_dynamic"):
			_registered_callbacks.erase(child.name)
			child.queue_free()
```

Esta função limpa a barra para receber os novos menus da tela ativa. Ela varre todos os nós filhos do `MenuBar` e, se encontrar um `PopupMenu` que possua o metadado `"is_dynamic"`, ela remove o registro de callbacks dele do nosso dicionário e o deleta da memória com o `queue_free()`.

### Reconstrução e Renderização da Interface

```gdscript
func rebuild_menu(dynamic_menu_structure : Dictionary) -> void:
	_clear_context_menus()
	
	for menu_title in dynamic_menu_structure.keys():
		_create_submenu(menu_title, dynamic_menu_structure[menu_title], true)
		
	_sort_menu_order()
```

Aqui é onde a transformação acontece. Quando o `MenuManager` emite o sinal com a nova estrutura, esta função:

1. Limpa os menus dinâmicos da tela anterior.
2. Varre o dicionário criando os novos submenus específicos da tela atual (passando `true` no parâmetro dinâmico).
3. Organiza a casa chamando a função de ordenação para que as opções permanentes não fiquem perdidas no meio do menu.

```gdscript
func _create_submenu(menu_title : String, items : Array, is_dynamic : bool) -> void:
	var popup_menu : PopupMenu = PopupMenu.new()
	popup_menu.name = menu_title
	
	if is_dynamic:
		popup_menu.set_meta("is_dynamic", true)
	
	_registered_callbacks[menu_title] = {}
	
	for i : int in range(items.size()):
		var item : MenuItemConfig = items[i]
		item.id = i
		
		popup_menu.add_item(item.text, item.id)
		
		if item.shortcut:
			popup_menu.set_item_shortcut(popup_menu.get_item_index(item.id), item.shortcut)
		
		_registered_callbacks[menu_title][item.id] = item.callback
		
	popup_menu.id_pressed.connect(_on_item_pressed.bind(menu_title))
	add_child(popup_menu)
```

Essa função é responsável por gerar o menu visualmente em tempo real:

* Ela instancia um novo `PopupMenu`, define o nome com o título desejado e se ele for dinâmico, injetamos a meta `is_dynamic` nele.
* Ela faz um loop por todos os itens da lista, define seus IDs sequenciais, adiciona o texto e vincula o atalho de teclado (`shortcut`) caso ele exista.
* Salva a função de destino (`callback`) dentro do nosso dicionário usando o título do menu e o ID do item como coordenadas.
* Conecta o sinal `id_pressed` à nossa função centralizada `_on_item_pressed`, usando o método `.bind(menu_title)` para que o sistema saiba exatamente de qual menu aquele clique veio.

### Ajustando a Ordem Visual e Callbacks

```gdscript
func _sort_menu_order() -> void:
	var permanent_menus : Array[String] = ["Settings", "Help"]
	
	for menu_name in permanent_menus:
		var menu_node = get_node_or_null(menu_name)
		if menu_node:
			move_child(menu_node, -1)
```

Por padrão, a Godot joga os novos nós filhos criados sempre para o final da fila. Se não tratarmos isso, nossos menus dinâmicos ficariam depois de "Settings" e "Help". Para resolver isso, essa função pega os nós permanentes e usa o método `move_child(nodo, -1)` para empurrá-los de volta para a última posição, garantindo a consistência visual da interface.

```gdscript
func _on_item_pressed(id : int, menu_title : String) -> void:
	if _registered_callbacks.has(menu_title) and _registered_callbacks[menu_title].has(id):
		var callback : Callable = _registered_callbacks[menu_title][id]
		
		if callback.is_valid():
			callback.call()
```

Por fim, temos a função receptora que escuta os cliques. Graças ao `.bind()` feito anteriormente, ela recebe o ID do botão e o título do menu pai. Ela faz uma checagem rápida de segurança para garantir que o callback existe, está válido e, se tudo estiver certo, executa a ação correspondente usando o `.call()`.

## Resultado Final: O Sistema em Ação

Com tudo isso pronto, terminamos esta parte!

Para fechar com chave de ouro, veja um exemplo prático de como qualquer tela do **TypeBubbleX** pode definir e enviar sua própria estrutura de menus para o nosso sistema dinâmico de forma extremamente limpa:

```gdscript
var meu_menu: Dictionary = {
	"Arquivo": [
		MenuItemConfig.new("Nova Obra", _on_new_work_clicked, 'ctrl+n'),
		MenuItemConfig.new("Sair", _on_quit_clicked, 'ctrl+q')
	],
	"Visualizar": [
		MenuItemConfig.new("Layout", _on_layout_clicked),
		MenuItemConfig.new("Ordenar", _on_sort_clicked)
	]
}

MenuManager.set_menu_context(meu_menu)
```

Simples, curto e extremamente escalável, não acha?

## Próximos Passos

Bem, é isso! Essa etapa de infraestrutura foi rápida, mas essencial para dar a flexibilidade que o nosso ecossistema precisa. Agora que nossa barra de menus dinâmica e inteligente está pronta, no próximo post vamos finalmente entrar de cabeça na criação do **Gerenciador de Obras**!

Muito obrigado por acompanhar até aqui e nos vemos no próximo post!
