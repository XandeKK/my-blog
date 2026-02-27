---
title: "TypeBubbleX: Banco de dados - Parte 2"
date: 2026-02-24
categories: [TypeBubbleX]
tags: [typebubblex, open-source, devlogs, godot-engine, scanlation, wireframe, database, orm]
---

Vamos continuar de onde paramos na [parte 1](https://xandekk.com/posts/typebubblex-03/).

Neste post, vou colocar a mão na massa e codar o que planejei no diagrama passado. Já aviso logo: teremos uma Parte 3, porque vou dedicar um tempo para testar tudo. Para ser sincero, só fui escrevendo o código sem parar e ainda não sei se está tudo rodando 100%. Mas faz parte do processo, né? Além disso, quero manter o ritmo de postagem aqui no blog de pelo menos um post por semana.

Para começar, escolhi o plugin [godot-sqlite](https://github.com/2shady4u/godot-sqlite), que é um *Wrapper* para SQLite dentro da Godot.

Para organizar a comunicação com o banco, decidi utilizar o **Active Record Pattern**, muito famoso no framework **Ruby on Rails**.

A ideia aqui é simples: cada tabela no banco de dados vira uma classe no código, e cada linha dessa tabela vira um objeto. Assim, em vez de escrever queries SQL gigantes no meio da lógica do app, eu consigo fazer coisas como `capitulo.save()` ou `capitulo.delete()` de um jeito muito mais limpo e intuitivo.

Se você quiser se aprofundar, o próprio site do Ruby on Rails explica muito bem as [bases do Active Record](https://guides.rubyonrails.org/active_record_basics.html).

Além disso, como no Rails, utilizarei o Active Record como meu **ORM (Object-Relational Mapping)**. Mas olha, pé no chão: o ORM que estou construindo na Godot é uma versão "pocket". Ele resolve o meu problema, mas não espere todas as funcionalidades de um framework gigante. O foco aqui é eficiência, e não reinventar a roda além do necessário. É claro que, no decorrer do projeto, surgirão novas necessidades, mas, por enquanto, o "feijão com arroz" bem feito já me atende.

Aliás, eu estou mencionando a Godot o tempo todo, mas talvez você tenha caído aqui de paraquedas e não saiba do que estou falando. Erro meu! Deixe-me explicar: a Godot é uma engine de jogos open-source, leve e versátil. Sim, é para criar jogos. Talvez você esteja se perguntando: "Mas você não está criando um jogo, está?". É verdade que não estou criando um jogo. Embora o foco dela seja jogos, a facilidade de criar interfaces e a sua linguagem própria (GDScript) fazem dela uma ferramenta fantástica para criar softwares de produtividade, como é o caso do [Pixelorama](https://github.com/Orama-Interactive/Pixelorama) ou do [Material Maker](https://github.com/RodZill4/material-maker).

Então, tendo em mente o que é Godot, Active Record Pattern e ORM, vamos partir para os códigos.

## Primeiro Passo

Neste passo, temos que criar uma conexão com o banco de dados. Para isso, criaremos o arquivo `db_client.gd`.

Este arquivo será um **Singleton**. Caso você não saiba o que é, trata-se de um *pattern* que faz com que essa instância do código seja única e acessível por todo o projeto. Ou seja, eu não preciso ficar instanciando o objeto toda vez que quiser usar o banco.

Abaixo, o código inicial:

```gdscript
# ./src/database/db_client.gd
extends Node

var db : SQLite
var db_path : String = "user://database.db"
var verbosity_level : int = SQLite.QUIET

func _ready() -> void:
	_connect_to_db()

func _connect_to_db() -> void:
	db = SQLite.new()
	db.path = db_path
	db.verbosity_level = verbosity_level
	db.foreign_keys = true
	
	if db.open_db():
		var migrations : MigrationManager = MigrationManager.new()
		migrations.run_migrations(db)
	else:
		printerr("Critical error: Unable to open database.")

func insert(table_name : String, data : Dictionary) -> String:
	if db.insert_row(table_name, data):
		return data.get("id", "")
	return ""

func update(table_name : String, id : String, data : Dictionary) -> bool:
	var condition = 'id = "' + str(id) + '"'
	return db.update_rows(table_name, condition, data)

func select_all(table_name : String) -> Array:
	return db.select_rows(table_name, "", ["*"])

func select_where(table_name : String, condition : String) -> Array:
	return db.select_rows(table_name, condition, ["*"])

func select_where_bound(table_name : String, condition : String, bindings : Array) -> Array:
	var query : String = "SELECT * FROM %s WHERE %s;" % [table_name, condition]
	if db.query_with_bindings(query, bindings):
		return db.query_result
	return []

func select_like_bound(table_name : String, column : String, search_term : String) -> Array:
	var query : String = "SELECT * FROM %s WHERE %s LIKE ?;" % [table_name, column]
	var formatted_search : String = "%" + search_term + "%"
	
	if db.query_with_bindings(query, [formatted_search]):
		return db.query_result
	return []

func delete(table_name : String, condition : String) -> bool:
	return db.delete_rows(table_name, condition)

func _exit_tree() -> void:
	if db:
		db.close_db()

```

### Explicação

Este arquivo é o "coração" do nosso banco de dados. Como mencionei, ele funciona como um Singleton (ou **Autoload** na Godot), o que significa que ele é carregado assim que o app inicializa e fica disponível globalmente como `DBClient`.

Vamos dividir a explicação em três partes principais:

### 1. Configuração e Ciclo de Vida

No início, definimos onde o banco de dados será guardado. O prefixo `user://` é fundamental na Godot, pois garante que o arquivo será gravado na pasta de dados do usuário, independentemente do sistema operacional (Windows, Linux ou macOS). No futuro, pretendo implementar um gerenciador para que o usuário escolha o diretório de sua preferência.

Existem duas funções que a Godot executa automaticamente aqui:

* `_ready()`: Inicializa tudo chamando o método `_connect_to_db()`.

* `_exit_tree()`: Executada quando o app é fechado. Ela encerra a conexão com o SQLite para evitar corrupção de dados.
*
### 2. A Conexão e Migrações

A função `_connect_to_db` gera uma instância do plugin `godot-sqlite`. Note que ativei as `foreign_keys`, o SQLite não as ativa por padrão, então fazemos isso manualmente para manter a integridade referencial entre as tabelas.

Se o banco abrir com sucesso, chamamos o `MigrationManager`. Ele é o responsável por criar as tabelas e organizar a estrutura inicial (vou detalhar isso mais adiante). Caso ocorra um erro na abertura, um alerta é exibido no console, futuramente, isso será integrado a um sistema de logs mais robusto.

### 3. Métodos de Conveniência (A ponte para o ORM)

Em vez de espalhar funções do plugin `godot-sqlite` por todo o projeto, centralizei as operações de **CRUD** (Create, Read, Update, Delete) aqui.

* `insert` e `update`: Facilitam a inserção de novos dados e a atualização de registros existentes usando dicionários.

* `select_all` e `select_where`: Buscam todos os dados de uma tabela ou filtram registros baseados em uma condição simples.

* `select_where_bound` e `select_like_bound`: São métodos mais seguros que usam *bindings* para evitar **SQL Injection**, ideais para buscas por texto ou filtros dinâmicos.

* `delete`: Remove registros da tabela conforme a condição passada.

Com este código pronto, o nosso próximo passo é criar as tabelas, pois não há como trabalhar sem elas. Então, vamos para o `MigrationManager`.

## Migração

Diferente do [Active Record Migrations](https://guides.rubyonrails.org/active_record_migrations.html) do Ruby on Rails, eu fiz uma versão simplificada para que o aplicativo consiga "auto-construir" o próprio banco de dados.

Para isso, criei o `MigrationManager`. Ele é o responsável por olhar para uma pasta específica, identificar quais scripts de criação de tabela ainda não foram executados e rodá-los um por um, em ordem.

```gdscript
# ./src/database/migration_manager.gd
class_name MigrationManager
extends RefCounted

const MIGRATIONS_PATH : String = "res://src/database/migrations/"

func run_migrations(db: SQLite) -> void:
	db.query("CREATE TABLE IF NOT EXISTS _Migrations (version INTEGER PRIMARY KEY);")
	
	var current_version : int = 0
	var result = db.select_rows("_Migrations", "", ["MAX(version) AS version"])
	
	if result.size() > 0 and result[0]["version"] != null:
		current_version = result[0]["version"]
	
	var migrations : Dictionary = _get_migration_scripts()
	
	for version in migrations.keys():
		if version > current_version:
			var script = migrations[version].new()
			if script.up(db):
				db.insert_row("_Migrations", {"version": version})
			else:
				printerr("Migration failed ", version)
				return

func _get_migration_scripts() -> Dictionary:
	var migration_files = DirAccess.get_files_at(MIGRATIONS_PATH)
	var migrations : Dictionary = {}
	
	var gd_files : Array[String] = []
	for f in migration_files:
		if f.ends_with(".gd") or f.ends_with(".gdc"):
			gd_files.append(f)
	
	gd_files.sort()
	
	for file_name in gd_files:
		var version = file_name.get_slice("_", 0).to_int()
		if version > 0:
			var full_path = MIGRATIONS_PATH.path_join(file_name)
			migrations[version] = load(full_path)
	
	return migrations

```

**Por que fazer assim?**

A lógica é simples: eu salvo os arquivos com nomes padronizados, do tipo `001_initial_schema.gd`. O gerenciador lê esse número inicial, compara com a última versão gravada na tabela `_Migrations` e, se o número do arquivo for maior, ele executa o método `up()`.

Isso me dá uma liberdade enorme. Se daqui a dois meses eu precisar mudar o Schema do banco, basta criar um arquivo `002_adiciona_campo_x.gd`. Quando o usuário abrir o app, o sistema percebe a nova versão e atualiza a estrutura automaticamente, sem que ninguém perca os dados que já tem.

### Definindo o Schema Inicial

No arquivo `001_initial_schema.gd`, defini toda a estrutura que planejei no diagrama da Parte 1. Aqui estão as tabelas para **Works**, **Chapters**, **Characters**, **Bubbles** e tudo o que o app precisa para funcionar.

O `godot-sqlite` facilita muito a vida nesse ponto: ele permite definir a estrutura usando um dicionário do GDScript, o que é muito mais limpo do que concatenar strings SQL puras para o `CREATE TABLE`.

Abaixo está o código do `001_initial_schema.gd`:

```gdscript
extends RefCounted

func up(db: SQLite) -> bool:
	const tables = {
		"Works": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"original_title": {"data_type": "text"},
			"translated_title": {"data_type": "text"},
			"description": {"data_type": "text"},
			"type": {"data_type": "text"},
			"status": {"data_type": "text"},
			"author_artist": {"data_type": "text"}
		},
		"Covers": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"work_id": {"data_type": "text", "foreign_key": "Works.id", "on_delete": "CASCADE", "not_null": true},
			"selected": {"data_type": "int", "not_null": true}
		},
		"Characters": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"work_id": {"data_type": "text", "foreign_key": "Works.id", "on_delete": "CASCADE", "not_null": true},
			"original_name": {"data_type": "text"},
			"translated_name": {"data_type": "text"},
			"description": {"data_type": "text"}
		},
		"CharacterImages": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"character_id": {"data_type": "text", "foreign_key": "Characters.id", "on_delete": "CASCADE", "not_null": true},
			"selected": {"data_type": "int", "not_null": true}
		},
		"CharacterNicknames": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"character_id": {"data_type": "text", "foreign_key": "Characters.id", "on_delete": "CASCADE", "not_null": true},
			"nickname": {"data_type": "text"},
			"context": {"data_type": "text"}
		},
		"Chapters": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"work_id": {"data_type": "text", "foreign_key": "Works.id", "on_delete": "CASCADE", "not_null": true},
			"title": {"data_type": "text"},
			"number": {"data_type": "real"},
			"language_source": {"data_type": "text"},
			"language_target": {"data_type": "text"}
		},
		"CharacterChapters": {
			"character_id": {"data_type": "text", "primary_key": true, "foreign_key": "Characters.id", "on_delete": "CASCADE", "not_null": true},
			"chapter_id": {"data_type": "text", "primary_key": true, "foreign_key": "Chapters.id", "on_delete": "CASCADE", "not_null": true}
		},
		"RawPages": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"chapter_id": {"data_type": "text", "foreign_key": "Chapters.id", "on_delete": "CASCADE", "not_null": true},
			"number": {"data_type": "int"}
		},
		"CleanPages": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"raw_page_id": {"data_type": "text", "foreign_key": "RawPages.id", "on_delete": "CASCADE", "not_null": true},
			"chapter_id": {"data_type": "text", "foreign_key": "Chapters.id", "on_delete": "CASCADE"}
		},
		"DonePages": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"raw_page_id": {"data_type": "text", "foreign_key": "RawPages.id", "on_delete": "CASCADE", "not_null": true},
			"chapter_id": {"data_type": "text", "foreign_key": "Chapters.id", "on_delete": "CASCADE"}
		},
		"Locations": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"work_id": {"data_type": "text", "foreign_key": "Works.id", "on_delete": "CASCADE", "not_null": true},
			"original_name": {"data_type": "text"},
			"translated_name": {"data_type": "text"},
			"description": {"data_type": "text"}
		},
		"LocationImages": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"location_id": {"data_type": "text", "foreign_key": "Locations.id", "on_delete": "CASCADE", "not_null": true}
		},
		"LocationChapters": {
			"location_id": {"data_type": "text", "primary_key": true, "foreign_key": "Locations.id", "on_delete": "CASCADE", "not_null": true},
			"chapter_id": {"data_type": "text", "primary_key": true, "foreign_key": "Chapters.id", "on_delete": "CASCADE", "not_null": true}
		},
		"Glossaries": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"work_id": {"data_type": "text", "foreign_key": "Works.id", "on_delete": "CASCADE", "not_null": true},
			"original_expression": {"data_type": "text"},
			"translated_expression": {"data_type": "text"},
			"examples": {"data_type": "text"},
			"description": {"data_type": "text"}
		},
		"GlossaryChapters": {
			"glossary_id": {"data_type": "text", "primary_key": true, "foreign_key": "Glossaries.id", "on_delete": "CASCADE", "not_null": true},
			"chapter_id": {"data_type": "text", "primary_key": true, "foreign_key": "Chapters.id", "on_delete": "CASCADE", "not_null": true}
		},
		"Bubbles": {
			"id": {"data_type": "text", "primary_key": true, "not_null": true},
			"chapter_id": {"data_type": "text", "foreign_key": "Chapters.id", "on_delete": "CASCADE", "not_null": true},
			"page_id": {"data_type": "text", "foreign_key": "RawPages.id", "on_delete": "CASCADE", "not_null": true},
			"original_text": {"data_type": "text"},
			"translated_text": {"data_type": "text"},
			"width": {"data_type": "real"},
			"height": {"data_type": "real"},
			"x": {"data_type": "real"},
			"y": {"data_type": "real"},
			"metadata": {"data_type": "text"}
		}
	}
	
	for table_name in tables.keys():
		if not db.create_table(table_name, tables[table_name]):
			return false
	return true
```

Com isso, assim que o usuário inicializar o aplicativo, todas as tabelas acima serão criadas automaticamente.

Agora que a fundação está pronta, chegamos finalmente à parte do **Active Record**. Está na hora de criar os objetos que vão interagir de fato com esses dados.

## Modelos

Se o `DBClient` é o coração do banco de dados, o `BaseModel` é o cérebro da nossa lógica de dados. É aqui que o **Active Record Pattern** ganha vida.

A ideia é que eu não precise escrever código de banco de dados para cada tabela nova. Em vez disso, todas as minhas classes (como `Work`, `Chapter` ou `Character`) vão herdar do `BaseModel`. Ele utiliza um recurso chamado Introspecção (através do `get_property_list`) para ler as variáveis que eu defini no script e transformá-las automaticamente em colunas do banco.

```gdscript
class_name BaseModel
extends RefCounted

var table_name : String = ""
var id : String = ""

func get_db_properties() -> Dictionary:
	var props : Dictionary = {}
	
	for prop in get_property_list():
		if prop.usage & PROPERTY_USAGE_SCRIPT_VARIABLE and prop.name != "table_name":
			var prop_name = prop.name
			props[prop_name] = get(prop_name)
	
	return props

func has_many(table : String, class_to_instantiate : Object, foreign_key : String) -> Array:
	var results : Array = DBClient.select_where(table, foreign_key + " = '" + id + "'")
	var list : Array = []
	
	for dict in results:
		var obj = class_to_instantiate.new()
		if obj.has_method("from_dict"):
			obj.from_dict(dict)
		list.append(obj)
		
	return list

func has_many_through(target_class: Object, join_table: String, source_fk: String, target_fk: String) -> Array:
	var target_instance : BaseModel = target_class.new()
	var target_table : String = target_instance.table_name
	
	var query = """
		SELECT t.* FROM %s t
		INNER JOIN %s j ON t.id = j.%s
		WHERE j.%s = ?
	""" % [target_table, join_table, target_fk, source_fk]
	
	var list : Array = []
	if DBClient.db.query_with_bindings(query, [id]):
		for dict in DBClient.db.query_result:
			var obj = target_class.new()
			obj.from_dict(dict)
			list.append(obj)
	return list

func belongs_to(class_to_instantiate : Object, foreign_key_id : String) -> BaseModel:
	var obj = class_to_instantiate.new()
	if obj.load_by_id(foreign_key_id):
		return obj
	return null

func from_dict(data : Dictionary) -> void:
	for key in data.keys():
		if key in self:
			set(key, data[key])

func save() -> bool:
	var data = get_db_properties()
	
	if id == "":
		id = UUID.v4()
		data["id"] = id
		return not DBClient.insert(table_name, data).is_empty()
	else:
		return DBClient.update(table_name, id, data)

func all() -> Array:
	return find_all("")

func count(column: String = "", value: Variant = null) -> int:
	var query = "SELECT COUNT(*) as total FROM " + table_name
	var results = []
	
	if column != "":
		query += " WHERE %s = ?" % column
		DBClient.db.query_with_bindings(query, [value])
	else:
		DBClient.db.query(query)
	
	results = DBClient.db.query_result
	return results[0]["total"] if results.size() > 0 else 0

func find_by(column: String, value: Variant) -> bool:
	var condition = "%s = ?" % column
	var bindings = [value]
	
	var results = DBClient.select_where_bound(table_name, condition, bindings)
	if results.size() > 0:
		from_dict(results[0])
		return true
	return false

func delete() -> bool:
	if id == "":
		push_warning("Attempt to delete an object without an ID (not persisted).")
		return false
	
	var success : bool = DBClient.delete(table_name, "id = '" + id + "'")
	
	if success:
		id = ""
		
	return success

func find_all_like(column: String, search_term: String) -> Array:
	var results : Array = DBClient.select_like_bound(table_name, column, search_term)
	var list : Array = []
	
	for dict in results:
		var obj : BaseModel = get_script().new()
		obj.from_dict(dict)
		list.append(obj)
		
	return list

func find_all(where_clause: String = "") -> Array:
	var results: Array
	if where_clause == "":
		results = DBClient.select_all(table_name)
	else:
		results = DBClient.select_where(table_name, where_clause)
	
	var list = []
	
	for dict in results:
		var obj : BaseModel = get_script().new()
		obj.from_dict(dict)
		list.append(obj)
		
	return list

func load_by_id(target_id : String) -> bool:
	var results = DBClient.select_where(table_name, "id = '" + target_id + "'")
	
	if results.size() > 0:
		from_dict(results[0])
		return true
	
	return false

func _to_string() -> String:
	var props = get_db_properties()
	var output = "\n[ %s ]" % get_class()
	output += "\n" + "—".repeat(20)
	
	for key in props.keys():
		var value = props[key]
		output += "\n %-15s : %s" % [key, str(value)]
	
	output += "\n" + "—".repeat(20)
	return output
```

Com o `BaseModel` pronto, criar um modelo para qualquer tabela se torna ridiculamente fácil. Veja como fica a implementação da classe `Work.gd`. Note que eu não preciso escrever uma única linha de SQL aqui:

```gdscript
extends BaseModel
class_name Work

var original_title: String = ""
var translated_title: String = ""
var description: String = ""
var type: String = ""
var status: String = ""
var author_artist: String = ""

func _init():
	table_name = "Works"

func get_covers() -> Array:
	return has_many("Covers", Cover, "work_id")

func get_chapters() -> Array:
	return has_many("Chapters", Chapter, "work_id")

func get_characters() -> Array:
	return has_many("Characters", Character, "work_id")

func get_locations() -> Array:
	return has_many("Locations", Location, "work_id")

func get_glossaries() -> Array:
	return has_many("Glossaries", Glossary, "work_id")
```

Percebeu a simplicidade? O `BaseModel` cuida de descobrir que `original_title` deve ser salvo na coluna correspondente. Se eu quiser salvar uma obra nova, o código no resto do app seria apenas:

```gdscript
var nova_obra = Work.new()
nova_obra.original_title = "Cheolsu Saves the World"
nova_obra.save() # Pronto, persistido no SQLite!
```

### Relacionamentos Complexos

O `BaseModel` também resolve um dos maiores problemas de bancos relacionais: buscar dados vinculados. Através dos métodos `has_many`, `belongs_to` e `has_many_through`, conseguimos navegar pelos dados de forma intuitiva.

* **`has_many`**: Usado quando uma Obra tem vários Capítulos.
* **`belongs_to`**: Usado quando um Capítulo pertence a uma Obra.
* **`has_many_through`**: Esse é o "pulo do gato" para tabelas de junção (N:N). Por exemplo, se eu quiser saber em quais Capítulos um Personagem aparece, eu não preciso fazer um JOIN manual; o `BaseModel` resolve a query por baixo dos panos.

### Proteção e Segurança (SQL Injection)

Um detalhe importante que você deve ter notado no código do `BaseModel` são os métodos `_bound`. Ao usar `query_with_bindings`, garantimos que qualquer texto inserido pelo usuário (como o título de um mangá) seja tratado como dado, e não como parte do comando SQL. Isso protege o nosso app contra ataques de **SQL Injection**, mantendo a integridade do banco de dados do usuário.

---

## Conclusão da Parte 2

Chegamos ao fim da nossa implementação técnica. Agora temos:

1. Uma conexão sólida e global com o SQLite (**DBClient**).
2. Um sistema que cria e atualiza tabelas sozinho (**MigrationManager**).
3. Uma base inteligente que transforma objetos GDScript em linhas de banco de dados (**BaseModel**).

Como eu disse lá no começo, eu saí codando tudo isso de uma vez. A estrutura parece linda no papel, mas **será que funciona na prática?**

Na **Parte 3**, vamos colocar esse cara pra trabalhar! Vou utilizar o [GUT (Godot Unit Test)](https://github.com/bitwes/Gut) para criar testes automatizados. A ideia é validar se cada método do nosso ORM está se comportando como esperado. Se algo quebrar (e provavelmente vai!), vamos debugar juntos e garantir que a base do nosso app seja sólida antes de avançarmos no projeto.

Você pode acessar o repositório [aqui](https://github.com/XandeKK/TypeBubbleX), lembre-se de alterar a branch para `v2` se quer ver as alterações.

Então é isso. Até a próxima!