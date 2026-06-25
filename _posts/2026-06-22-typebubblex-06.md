---
title: "TypeBubbleX Devlog #06: Voltando à ativa e criando a Janela de Configuração Inicial"
date: 2026-06-22
categories: [TypeBubbleX]
tags: [typebubblex, open-source, devlogs, godot-engine, scanlation, wireframe, gui, ux]
---

Bem, depois de 3 meses parado, vamos finalmente voltar a esse projeto! 

Como eu disse no final do meu [último post](https://xandekk.com/posts/typebubblex-05), o foco agora é a interface gráfica (GUI). Já aviso de antemão: a ideia inicial não é fazer algo visualmente lindo ou ultra estilizado, mas sim focar na **organização estrutural** para que eu possa aplicar temas facilmente depois.

Para começar, decidi tirar do papel o que planejei naquele post sobre [wireframes](https://xandekk.com/posts/typebubblex-02): a **Janela de Configuração Inicial (Initial Setup)**.

## O Objetivo da Janela (O Fluxo de Trabalho)

A ideia principal dessa janela é definir e configurar qual diretório do computador o TypeBubbleX terá controle total para gerenciar as obras de scanlation. 

Desenhei o comportamento dessa lógica para funcionar assim na inicialização do app:

* **Primeiro acesso:** O app verifica uma variável de preferência. Se não houver nenhum caminho salvo, a janela de setup se abre na tela.
* **Caminho válido encontrado:** Se o diretório já estiver configurado e o arquivo de banco de dados (`typebubblex.db`) estiver lá dentro, o setup é ignorado e o app vai direto para a tela de obras.
* **Recuperação de dados antigos:** Digamos que o usuário parou de usar o app, deletou o executável, mas manteve a pasta com as obras no computador. Se ele baixou o app de novo e apontar para essa mesma pasta, o sistema reconhece o banco de dados antigo e importa tudo automaticamente, sem que ele perca o progresso.
* **Tratamento de erros:** Se o usuário mover a pasta ou deletar o arquivo `.db` manualmente, o app detecta o problema e exibe uma notificação avisando que o workspace sumiu, forçando a abertura do setup para correção.

Com esses comportamentos definidos, fui para a engine estruturar a cena.

## Estruturando a árvore de nós (Scene Tree)

Abaixo, mostro de como ficou a árvore de nós:

![Nodes](/assets/img/typebubblex/initial_setup/nodes.png)

O nó raiz é uma `Window` chamada `InitialSetup`, que carrega o script responsável por toda a mágica da validação (que vou destrinchar daqui a pouco). 

Dentro da `Window`, usei um `PanelContainer` (que servirá para eu estilizar a interface mais para a frente) e, dentro dele, um `VBoxContainer` para fazer com que todos os elementos fiquem organizados verticalmente, um abaixo do outro.

Para a organização interna, usei três `HBoxContainer`, que são os nós responsáveis por alinhar os elementos na horizontal, deixando um ao lado do outro (como a linha do caminho e a linha dos botões).

Além dos containers, a estrutura conta com os seguintes nós principais:
* **`CreateFolderCheckButton`**: Um botão de alternância (checkbutton) para o usuário decidir se quer ou não criar automaticamente o diretório final.
* **`PathLineEdit`**: O campo de entrada de texto onde o caminho do diretório é exibido e pode ser digitado.
* **`FileDialogButton`**: O botão que abre o explorador de arquivos nativo para selecionar o diretório de forma muito mais fácil.
* **`MessageLabel`**: O texto de feedback que mostrará em tempo real se está tudo certo ou se há algo de errado com o caminho.
* **`ActionButton`**: O botão dinâmico ("Create" ou "Import") que conclui todo esse processo de configuração.

## Mostrando o código

Vou dividir o código em vários blocos para explicar cada um deles.

```gdscript
extends Window

signal setup_completed

@onready var path_line_edit : LineEdit = $PanelContainer/VBoxContainer/PathHBox/PathLineEdit
@onready var message_label : Label = $PanelContainer/VBoxContainer/MessageLabel
@onready var create_folder_check_button : CheckButton = $PanelContainer/VBoxContainer/HBoxContainer/CreateFolderCheckButton
@onready var file_dialog : FileDialog = $FileDialog
@onready var action_button : Button = $PanelContainer/VBoxContainer/ActionHBox/ActionButton
```

Começamos declarando um `signal` que será emitido para avisar à janela principal que deu tudo certo. Além desse sinal, temos algumas variáveis para que possamos manipular os elementos da interface via script.

Caso você nunca tenha trabalhado com **Godot**, o `@onready` indica ao script que ele deve esperar até que os nós da cena estejam totalmente carregados antes de tentar obter as referências deles. É uma lógica parecida com o **JavaScript**, quando usamos o `window.onload` (ou algo semelhante) para garantir que a página foi carregada completamente antes de executar uma ação.

```gdscript
const DEFAULT_DATABASE_FILE_NAME : String = "typebubblex.db"
const DEFAULT_DIR_NAME : String = "TypeBubbleX"

const MSG_WARNING_NOT_EMPTY = "The selected path is not empty. Choosing an empty folder is highly recommended."
const MSG_ERROR_PATH_NOT_FOUND = "The path specified doesn't exist."
const MSG_INFO_AUTO_CREATE_FOLDER = "The project folder will be automatically created."
const MSG_INFO_PATH_EMPTY = "The project folder exists and is empty."
const MSG_INFO_LOAD_EXISTING = "A valid project file was found. The project will be loaded from this directory."

enum Status { ERROR_PATH_NOT_FOUND, WARNING_NOT_EMPTY, INFO_AUTO_CREATE_FOLDER, INFO_PATH_EMPTY, INFO_LOAD_EXISTING }
```

Aqui declaramos as nossas constantes e um enum. Não tem muito segredo nessa parte, as primeiras constantes definem os nomes padrão de arquivos e pastas, enquanto as constantes que começam com `MSG_` centralizam as mensagens que exibiremos para o usuário, essa parte sofrerá mudanças no futuro para aplicar o **i18n** para traduzir em outras linguas.

Já o `enum` (`Status`) serve para organizar os estados possíveis da validação. Eu gosto de usar enums porque eles nos permitem aplicar tipagem estática em variáveis ou parâmetros de funções, garantindo que o código só aceite os valores que nós definimos aqui, evitando bugs.

```gdscript
var current_path : String = OS.get_system_dir(OS.SYSTEM_DIR_DOCUMENTS).path_join(DEFAULT_DIR_NAME)
var current_status : Status = Status.INFO_AUTO_CREATE_FOLDER
var should_create_folder : bool = true
```

Aqui temos as variáveis que guardarão o caminho escolhido pelo usuário, o status atual da validação e se o diretório deve ou não ser criado.

Você pode notar que a variável `current_status` está utilizando o `Status` (nosso enum) como tipo. Isso garante que ela receba apenas os valores que definimos anteriormente. 

Uma observação importante: por baixo dos panos, os enums na **Godot** são representados por números inteiros. Isso significa que, tecnicamente, a variável poderia receber um número, mas o motor da Godot não recomenda esse tipo de comportamento e o editor de código vai chamar a sua atenção (emitindo um warning ou erro de tipo) para garantir que você mantenha o código limpo e seguro.

```gdscript
func _ready() -> void:
	path_line_edit.text = current_path
	create_folder_check_button.set_pressed_no_signal(should_create_folder)
	_validate_and_update_path(current_path)
```

Nesta função, que é chamada automaticamente pela **Godot** assim que o nó e seus filhos estão prontos na cena, nós inicializamos os elementos da interface com os nossos valores padrão.

Um detalhe interessante aqui é o uso do método `set_pressed_no_signal()`. Nós o utilizamos para definir o estado inicial do botão de checagem (CheckButton) sem disparar o sinal de 'clique'. Isso é uma ótima prática para evitar que funções conectadas a esse botão sejam executadas antes da hora, garantindo que a inicialização da tela seja limpa. Por fim, chamamos a função de validação para checar o caminho padrão inicial.

```gdscript
func _validate_and_update_path(target_path : String) -> void:
	current_path = target_path
	
	if _is_valid_path(target_path):
		if _has_files_in_folder(target_path):
			if _has_database_project_file(target_path):
				_update_status(Status.INFO_LOAD_EXISTING)
			else:
				_update_status(Status.WARNING_NOT_EMPTY)
		else:
			if should_create_folder:
				_update_status(Status.INFO_AUTO_CREATE_FOLDER)
			else:
				_update_status(Status.INFO_PATH_EMPTY)
	else:
		if should_create_folder and _is_valid_path(target_path.get_base_dir()):
			_update_status(Status.INFO_AUTO_CREATE_FOLDER)
		else:
			_update_status(Status.ERROR_PATH_NOT_FOUND)
```

Esta função é o coração da nossa validação. Nela, recebemos o caminho definido pelo usuário e passamos por uma estrutura de condições (`if/else`) para atualizar o status da tela, permitindo que o usuário saiba exatamente se o diretório escolhido é adequado ou não.

A lógica segue uma tomada de decisão em cascata:

1. **Se o caminho for válido**: ela verifica se a pasta contém arquivos. Caso contenha, checa se o arquivo de banco de dados do projeto já existe ali (para carregá-lo) ou se é apenas uma pasta ocupada (gerando um aviso). Se estiver vazia, define o status com base na opção de criar ou não a pasta.
2. **Se o caminho NÃO for válido**: ela ainda faz uma última tentativa útil, checando se a pasta mãe (o diretório base) existe e se a opção de criar a pasta automaticamente está ativa. Se nada disso bater, ela exibe o erro de caminho não encontrado.

```gdscript
func _update_status(new_status : Status) -> void:
	current_status = new_status
	action_button.disabled = false
	
	match new_status:
		Status.ERROR_PATH_NOT_FOUND:
			message_label.text = MSG_ERROR_PATH_NOT_FOUND
			action_button.text = "Create"
			action_button.disabled = true
		Status.WARNING_NOT_EMPTY:
			message_label.text = MSG_WARNING_NOT_EMPTY
			message_label.self_modulate = Color.YELLOW
			action_button.text = "Create"
		Status.INFO_AUTO_CREATE_FOLDER:
			message_label.text = MSG_INFO_AUTO_CREATE_FOLDER
			message_label.self_modulate = Color.GREEN
			action_button.text = "Create"
		Status.INFO_PATH_EMPTY:
			message_label.text = MSG_INFO_PATH_EMPTY
			message_label.self_modulate = Color.GREEN
			action_button.text = "Create"
		Status.INFO_LOAD_EXISTING:
			message_label.text = MSG_INFO_LOAD_EXISTING
			message_label.self_modulate = Color.GREEN
			action_button.text = "Import"
```

Nesta função, nós atualizamos o estado atual do componente e alteramos a interface gráfica para refletir visualmente o que está acontecendo com a validação.

Utilizando a estrutura match da Godot, conseguimos mudar o comportamento da tela para cada estado do nosso enum:

* **Mensagens Dinâmicas**: O texto da label (`message_label.text`) assume a constante correspondente que criamos lá no início.
* **Feedback por Cores**: Usamos a propriedade `self_modulate` para colorir a mensagem, o que ajuda o usuário a entender a situação num piscar de olhos.
* **Mudança no Botão**: O botão de ação muda não apenas o seu comportamento (ficando desabilitado em caso de erro), mas também o seu texto. Se o projeto já existe, o botão vira **'Import'**; se for um caminho novo, ele exibe **'Create'**. Isso melhora drasticamente a experiência do usuário (UX)."

```gdscript
func _is_valid_path(target_path : String) -> bool:
	return DirAccess.dir_exists_absolute(target_path)

func _has_files_in_folder(target_path : String) -> bool:
	var dir : DirAccess = DirAccess.open(target_path)
	
	if dir:
		dir.list_dir_begin()
		var file_name : String = dir.get_next()
		if file_name != "":
			return true
	
	return false

func _has_database_project_file(target_path : String) -> bool:
	var project_file_path = target_path.path_join(DEFAULT_DATABASE_FILE_NAME)
	return FileAccess.file_exists(project_file_path)
```

Aqui temos o trio de funções auxiliares que realizam a checagem física no sistema de arquivos do computador, utilizando as classes `DirAccess` e `FileAccess` da Godot:

* `_is_valid_path`: Uma função simples que retorna um valor booleano (`true` ou `false`) indicando se o caminho de diretório informado realmente existe no sistema.
* `_has_files_in_folder`: Esta função abre o diretório e inicia uma leitura interna dele com o `list_dir_begin()`. Chamamos o `get_next()` para pegar o primeiro item encontrado. Se esse primeiro item for diferente de um texto vazio (""), sabemos imediatamente que a pasta possui algum conteúdo (seja um arquivo ou outra subpasta) e retornamos `true`.
* `_has_database_project_file`: Responsável por testar a existência do projeto em si. Ela combina o caminho atual com o nome padrão do banco de dados (`DEFAULT_DATABASE_FILE_NAME`) usando o método `path_join` e verifica se esse arquivo específico existe.

```gdscript
func _on_create_folder_check_button_toggled(toggled_on : bool) -> void:
	var path_text : String = path_line_edit.text.strip_edges()
	should_create_folder = toggled_on
	
	if toggled_on:
		var base_dir = path_text
		if base_dir.get_file() != DEFAULT_DIR_NAME:
			path_text = base_dir.path_join(DEFAULT_DIR_NAME)
	else:
		if path_text.get_file() != "":
			path_text = path_text.get_base_dir()
	
	path_line_edit.text = path_text
	_validate_and_update_path(path_text)
```

Esta função é acionada automaticamente sempre que o usuário ativa ou desativa o **CheckButton** (botão de checagem). Ela é responsável por atualizar dinamicamente o campo de texto, adicionando ou removendo o nome da pasta padrão do projeto.

A lógica funciona assim:

* **Limpeza inicial**: Primeiro, usamos o `.strip_edges()` para garantir que qualquer espaço em branco invisível digitado sem querer no início ou fim do caminho seja removido.
* **Se o botão for ativado (`toggled_on`)**: O código verifica se o caminho já não termina com o nome padrão da pasta (`DEFAULT_DIR_NAME`). Se não terminar, ele anexa esse nome ao final usando o `path_join()`.
* **Se o botão for desativado**: O código faz o inverso. Usando o `get_base_dir()`, ele 'sobe um nível', removendo o nome da pasta do final do caminho.

Por fim, o texto do campo de entrada (`path_line_edit.text`) é atualizado com o novo caminho e a função de validação é chamada novamente para recalcular o status da tela.

```gdscript
func _on_path_line_edit_text_changed(new_text : String) -> void:
	_validate_and_update_path(new_text)
```

Esta função é acionada automaticamente sempre que o usuário digita ou altera qualquer coisa no campo de texto, chamando imediatamente a nossa função de validação para atualizar o status em tempo real.

```gdscript
func _on_file_dialog_button_pressed() -> void:
	file_dialog.popup_centered()
```

Esta função é acionada quando o usuário clica no botão, abrindo a janela de seleção de diretórios (`FileDialog`). O método `popup_centered()` garante que essa janela apareça centralizada na tela do usuário.

```gdscript
func _on_file_dialog_dir_selected(dir : String) -> void:
	if should_create_folder:
		dir = dir.path_join(DEFAULT_DIR_NAME)
	
	path_line_edit.text = dir
	_validate_and_update_path(dir)
```

Esta função é acionada quando o usuário seleciona um diretório e confirma a escolha (clicando em 'OK') na janela de seleção.

Ao receber o caminho escolhido, ela verifica se a opção de criar a pasta automaticamente está ativa (`should_create_folder`). Se estiver, ela anexa o nome padrão da pasta (`DEFAULT_DIR_NAME`) ao final do caminho. Em seguida, atualiza o campo de texto na tela e dispara a validação para recalcular o status do novo diretório.

```gdscript
func _on_action_button_pressed() -> void:
	if current_status == Status.ERROR_PATH_NOT_FOUND:
		return
	
	var is_existing_project : bool = current_status == Status.INFO_LOAD_EXISTING
	
	if not is_existing_project:
		if not _is_valid_path(current_path):
			var err : Error = DirAccess.make_dir_absolute(current_path)
			if err != OK:
				_update_status(Status.ERROR_PATH_NOT_FOUND)
				return

	DBClient.open_db(current_path.path_join(DEFAULT_DATABASE_FILE_NAME))
	Preference.data.workspace_dir = current_path
	Preference.save_preferences()
	
	setup_completed.emit()
	
	queue_free()
```

Esta última função é a que finaliza todo o processo de configuração. Ela é acionada quando o usuário clica no botão de ação principal — que pode ser **'Create'** ou **'Import'**, dependendo do status do diretório.

O método executa uma sequência firme de passos para consolidar a ação:

1. **Criação física da pasta**: Se for um projeto novo e a pasta ainda não existir no computador, o código tenta criá-la usando `DirAccess.make_dir_absolute()`. Se algo falhar (como falta de permissão do sistema), ele interrompe a execução.
2. **Inicialização do Banco de Dados**: Ele abre (ou cria) o arquivo de banco de dados (`.db`) diretamente no caminho selecionado através de uma classe gerenciadora (`DBClient`).
3. **Salvamento de Preferências**: Guarda o caminho desse diretório nas configurações globais do aplicativo (`Preference`) para que o usuário não precise passar por essa tela toda vez que abrir o app.
4. **Finalização**: Emite o sinal `setup_completed` (aquele que declaramos lá no primeiro bloco) para avisar o restante do projeto que deu tudo certo e, por fim, usa o `queue_free()` para fechar e remover esta janela da memória.

Com isso, terminamos esta parte da configuração inicial.

Abaixo, você pode ver uma imagem de como ficou a interface gráfica. Lembre-se de que ela sofrerá alterações visuais mais para a frente, quando trabalharmos com os temas da Godot, mas a estrutura e a disposição dos elementos serão bem parecidas com estas.

![Nodes](/assets/img/typebubblex/initial_setup/window.png)

Bem, ainda falta um último detalhe para de fato concluirmos isso, integrar essa janela de configuração à janela principal do projeto. Então, vamos lá!

## Integrando com a janela principal

### Estrutura de Nós

Para começarmos a integração, precisamos primeiro entender como a nossa cena principal está estruturada no painel **Scene** da Godot. A organização dos nós é bastante simples e direta:

![Nodes](/assets/img/typebubblex/initial_setup/nodes-main.png)

Como você pode ver na imagem acima, temos a seguinte hierarquia:

* **`MainWindow` (Control):** O nó raiz da nossa tela principal, responsável por gerenciar o ciclo de vida inicial da aplicação. É nele que anexaremos o script de integração que veremos a seguir.
* **`InitialSetup` (Window):** Esta é a cena da nossa janela de configuração que criamos e detalhamos nos blocos anteriores, instanciada aqui como filha direta da janela principal.
* **`ErrorDialog` (AcceptDialog):** Um nó nativo da Godot perfeito para exibir mensagens de alerta simples com um botão de confirmação. Ele ficará responsável por avisar o usuário caso o banco de dados antigo tenha sumido.

Com essa estrutura visual em mente, vamos agora criar o script da `MainWindow` para controlar esse fluxo.

### Mostrando o código

```gdscript
extends Control

@onready var initial_setup_window : Window = $InitialSetup
@onready var error_dialog : AcceptDialog = $ErrorDialog

func _ready() -> void:
	initial_setup()

```

Neste início, o nosso script herda de `Control`, indicando que esta é a interface principal da aplicação. Usamos o `@onready` para capturar as referências da janela de configuração inicial (que acabamos de criar nos passos anteriores) e de uma janela de diálogo de erro (`AcceptDialog`), que usaremos caso algo dê errado com o diretório salvo.

Assim que a aplicação fica pronta, a função `_ready()` entra em ação e chama imediatamente a função `initial_setup()`, que cuidará de decidir o que deve ser exibido para o usuário logo de cara.


```gdscript
func initial_setup() -> void:
	if Preference.data.workspace_dir.is_empty():
		initial_setup_window.setup_completed.connect(_initialize)
		initial_setup_window.show()
		error_dialog.queue_free()
		return
	
	var db_path = Preference.data.workspace_dir.path_join("typebubblex.db")
	
	if DirAccess.dir_exists_absolute(Preference.data.workspace_dir) and FileAccess.file_exists(db_path):
		DBClient.open_db(db_path)
		_initialize()
		initial_setup_window.queue_free()
		error_dialog.queue_free()
	else:
		error_dialog.title = "Workspace Error"
		error_dialog.dialog_text = "The previously saved workspace directory or database file could not be found.\n\nPlease select or create a valid workspace to continue."
		error_dialog.popup_centered()

```

Esta função é o filtro de entrada da nossa aplicação. Ela precisa cobrir três cenários possíveis quando o usuário abre o programa:

1. **Primeira Inicialização (Sem caminho salvo):** Se o `workspace_dir` estiver vazio, significa que é a primeira vez que o app roda. Nós conectamos o sinal `setup_completed` da nossa janela de configuração à função `_initialize()`, exibimos a tela de setup com o `.show()` e liberamos a janela de erro da memória com o `queue_free()`, já que ela não será necessária.
2. **Caminho Salvo e Válido:** Se já existe um caminho nas preferências, o código verifica se a pasta e o banco de dados (`typebubblex.db`) realmente existem no computador. Se tudo estiver correto, o banco de dados é aberto pelo `DBClient`, a aplicação é inicializada e as janelas auxiliares de setup e erro são descartadas para poupar memória.
3. **Caminho Corrompido ou Apagado:** Se o usuário apagou ou moveu a pasta antiga manualmente pelo sistema operacional, a validação falha. Em vez de crashar o app, configuramos um texto explicativo no `error_dialog` e o exibimos centralizado, alertando que o espaço de trabalho anterior sumiu.


```gdscript
func _initialize() -> void:
	# To do
	pass

func _on_error_dialog_confirmed() -> void:
	initial_setup_window.setup_completed.connect(_initialize)
	initial_setup_window.show()
	
	error_dialog.queue_free()

```

Por fim, temos duas funções que resolvem o desfecho do fluxo:

* **`_initialize()`**: Esta é a função que carregará o restante da aplicação de fato. Por enquanto, ela está com o marcador `pass` (o nosso famoso *To Do*), pois faremos isso logo em seguida.
* **`_on_error_dialog_confirmed()`**: Caso o usuário tenha caído no cenário de erro (diretório sumiu ou arquivo sumiu) e clique no botão de "OK" da janela de alerta, este sinal é disparado. Ele remove a janela de erro da memória e traz de volta a nossa tela de `InitialSetup` para que o usuário possa escolher ou criar uma nova pasta válida, recomeçando o fluxo com segurança.

E agora, finalmente, terminamos!

No próximo post, vou abordar o gerenciador de obras ou talvez a barra de menu — mas acho que a barra de menu vai acabar vindo primeiro.

Então é isso, muito obrigado por acompanhar até aqui e nos vemos no próximo post!
