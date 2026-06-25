---
title: "TypeBubbleX Devlog #05: Banco de dados - Parte 3"
date: 2026-03-03
categories: [TypeBubbleX]
tags: [typebubblex, open-source, devlogs, godot-engine, scanlation, wireframe, database, orm]
---

Finalmente, vamos fechar esse ciclo! Na [parte 2](https://xandekk.com/posts/typebubblex-04/), eu mostrei toda a estrutura do nosso ORM  e como o **BaseModel** e o **DBClient** dão vida ao Active Record Pattern dentro da Godot. Mas, como eu mesmo confessei: eu saí escrevendo o código sem parar. E no desenvolvimento de software, código que não é testado é código que (provavelmente) está quebrado. **Spoiler: Realmente havia alguns erros, mas eram apenas erros de digitação.**

Neste post, vamos falar sobre **Confiança**. Para garantir que a base do TypeBubbleX seja sólida como uma rocha antes de partirmos para a interface, utilizei o [GUT (Godot Unit Test)](https://github.com/bitwes/Gut).

O objetivo aqui é simples: criar cenários controlados para "estressar" o banco de dados e garantir que cada engrenagem do nosso sistema de persistência está girando do jeito certo.

Dito isso, não vou me aprofundar muito, pois é algo chato de explicar, se já é maçante escrever os códigos de teste, imagine falar sobre eles.

---

## O Veredito dos Testes

Dividi os testes em quatro frentes principais, cada uma atacando um ponto crítico da nossa arquitetura. Vamos dar uma olhada no que foi validado.

### 1. Integridade do Modelo Base (`test_base_model.gd`)

Aqui é onde testamos o "cérebro" do sistema. Eu precisava garantir que o `BaseModel` conseguisse salvar, carregar, atualizar e deletar registros sem perder informações no caminho. Além disso, testei os relacionamentos que definimos no diagrama da [Parte 1](https://xandekk.com/posts/typebubblex-03).

**O que foi testado:**

* **CRUD Completo:** Salvar um novo mangá, atualizar o título de "One Piece" para "Naruto" e deletar registros.
* **Relacionamentos:** Validar se um `Work` consegue buscar seus `Chapters` (**has_many**) e se um `Character` consegue encontrar em quais capítulos ele aparece através de uma tabela de junção (**has_many_through**).
* **Busca Dinâmica:** Testar o `find_all_like` para garantir que a busca por "Dragon" retorne tanto "Dragon Ball" quanto "Dragon Ball Z".

```gdscript
extends GutTest

var data : Dictionary = {
	"id": '',
	"original_title": "One piece",
	"translated_title": "One piece",
	"description": "It follows the adventures of Monkey D. Luffy and his crew, the Straw Hat Pirates, as he explores the Grand Line in search of the mythical treasure known as the \"One Piece\" to become the next King of the Pirates.",
	"type": "Manga",
	"status": "Ongoing",
	"author_artist": "Eiichiro Oda"
}

func before_each() -> void:
	DBClient.open_db(":memory:")

func after_each() -> void:
	if DBClient.db:
		DBClient.db.close_db()

func test_get_db_properties_excludes_metadata():
	var work : Work = Work.new()
	work.original_title = "Jane"
	var props : Dictionary = work.get_db_properties()
	
	assert_has(props, "original_title")
	assert_has(props, "id")
	assert_false(props.has("table_name"), "table_name should not be included in database properties")

func test_has_many_relationship():
	var work : Work = Work.new()
	work.original_title = data.original_title
	work.save()
	
	var ch1_data = {"id": "1", "work_id": work.id, "title": "Romance dawn"}
	var ch2_data = {"id": "2", "work_id": work.id, "title": "That Guy'Straw Hat Luffy"}
	DBClient.insert("chapters", ch1_data)
	DBClient.insert("chapters", ch2_data)
	
	var chapters = work.has_many("chapters", Chapter, "work_id")
	
	assert_eq(chapters.size(), 2, "Should return 2 chapters.")
	assert_eq(chapters[0].title, "Romance dawn")

func test_has_many_through_relationship():
	var work = Work.new()
	work.original_title = "One Piece"
	work.save()
	
	var luffy : Character = Character.new()
	luffy.work_id = work.id
	luffy.original_name = "Monkey D. Luffy"
	luffy.save()

	var ch1 = Chapter.new()
	ch1.work_id = work.id
	ch1.title = "Romance Dawn"
	ch1.save()
	
	var ch2 : Chapter = Chapter.new()
	ch2.work_id = work.id
	ch2.title = "That Guy'Straw Hat Luffy"
	ch2.save()
	
	DBClient.insert("CharacterChapters", {"character_id": luffy.id, "chapter_id": ch1.id})
	DBClient.insert("CharacterChapters", {"character_id": luffy.id, "chapter_id": ch2.id})
	
	var chapters : Array = luffy.has_many_through(Chapter, "CharacterChapters", "character_id", "chapter_id")
	
	assert_eq(chapters.size(), 2, "Luffy should appear in 2 chapters.")
	
	var titles : Array = [chapters[0].title, chapters[1].title]
	assert_has(titles, "Romance Dawn")
	assert_has(titles, "That Guy'Straw Hat Luffy")

func test_belongs_to_relationship() -> void:
	var work = Work.new()
	work.original_title = data.original_title
	work.save()
	
	var chapter = Chapter.new()
	chapter.work_id = work.id
	
	var parent_work = chapter.belongs_to(Work, chapter.work_id)
	
	assert_not_null(parent_work, "Should find the parent.")
	assert_eq(parent_work.id, work.id)

func test_from_dict_fills_properties():
	var work : Work = Work.new()
	
	work.from_dict(data)
	
	assert_eq(work.id, data.id, "ID should be filled")
	assert_eq(work.original_title, data.original_title, "Name should be filled")

func test_save_new_record():
	var work : Work = Work.new()
	work.original_title = data.original_title
	work.description = data.description
	
	var success : bool = work.save()
	
	assert_true(success, "Record should be saved successfully.")
	assert_not_null(work.id, "ID should be generated by UUID after save.")
	
	var count : int = work.count()
	assert_eq(count, 1, "There should be exactly 1 record in the database.")

func test_find_by_column():
	var work : Work = Work.new()
	work.original_title = data.original_title
	work.save()
	
	var found_work : Work = Work.new()
	var found : bool = found_work.find_by("original_title", data.original_title)
	
	assert_true(found, "Should find the work by original title.")
	assert_eq(found_work.original_title, data.original_title)
	assert_eq(found_work.id, work.id)

func test_update_existing_record():
	var work : Work = Work.new()
	work.original_title = data.original_title
	work.save()
	
	var original_id : String = work.id
	work.original_title = "Naruto"
	var success : bool = work.save()
	
	assert_true(success, "Update should return success.")
	assert_eq(work.id, original_id, "ID should not change on update.")
	
	var checker : Work = Work.new()
	checker.load_by_id(original_id)
	assert_eq(checker.original_title, "Naruto", "Title in database should be updated.")

func test_delete_record():
	var work : Work = Work.new()
	work.save()
	
	var success : bool = work.delete()
	assert_true(success, "Deletion should be successful.")
	assert_eq(work.id, "", "Object ID should be cleared after deletion.")
	
	var count : int = work.count()
	assert_eq(count, 0, "Database should be empty after deletion.")

func test_find_all_like_search() -> void:
	var w1 : Work = Work.new()
	w1.original_title = "Dragon Ball"
	w1.save()
	
	var w2 : Work = Work.new()
	w2.original_title = "Dragon Ball Z"
	w2.save()
	
	var w3 : Work = Work.new()
	w3.original_title = "Naruto"
	w3.save()
	
	var results : Array = w1.find_all_like("original_title", "Dragon")
	
	assert_eq(results.size(), 2, "Should find both Dragon Balls")

func test_count_with_filter() -> void:
	var w1 : Work = Work.new()
	w1.status = "Ongoing"
	w1.save()
	
	var w2 : Work = Work.new()
	w2.status = "Finished"
	w2.save()
	
	var ongoing_count : int = w1.count("status", "Ongoing")
	assert_eq(ongoing_count, 1, "Should count only Ongoing records")

func test_delete_non_persisted_record() -> void:
	var work : Work = Work.new()
	
	var success : bool = work.delete()
	
	assert_false(success, "Should not be able to delete without an ID")

func test_load_by_id_not_found() -> void:
	var work : Work = Work.new()
	var exists : bool = work.load_by_id("id-que-nao-existe")
	
	assert_false(exists, "Should return false if ID does not exist in database")

func test_find_all_with_special_characters() -> void:
	var work : Work = Work.new()
	work.original_title = "Luffy's Adventure"
	work.save()
	
	var results : Array = work.find_all("original_title = 'Luffy''s Adventure'") 
	
	assert_gt(results.size(), 0, "Should handle strings containing single quotes")

func test_all_returns_empty_array_when_no_data() -> void:
	var work : Work = Work.new()
	var results : Array = work.all()
	
	assert_eq(typeof(results), TYPE_ARRAY, "Should return an Array")
	assert_eq(results.size(), 0, "The array should be empty")


```

### 2. Validação da Escala (`test_children_base_model.gd`)

Como o TypeBubbleX tem muitas tabelas (Works, Chapters, Bubbles, Glossaries, etc.), eu não queria escrever um teste manual para cada uma. Criei um teste automatizado que varre todos os meus modelos e verifica se eles respeitam o contrato do banco.

**O que foi testado:**

* **Mapeamento de Tabelas:** Se a classe `Chapter` aponta corretamente para a tabela "Chapters".
* **Existência de Campos:** Se todos os campos definidos no script existem fisicamente no objeto e no banco.
* **Segurança de Instanciação:** Garante que todas as classes conseguem nascer sem dar erro de memória.

```gdscript
extends GutTest

var models_to_test = [
	{
		"class": Work, 
		"table": "Works", 
		"fields": ["original_title", "translated_title", "description", "type", "status", "author_artist"]
	},
	{
		"class": Chapter, 
		"table": "Chapters", 
		"fields": ["work_id", "title", "number", "language_source", "language_target"]
	},
	{
		"class": Character, 
		"table": "Characters", 
		"fields": ["work_id", "original_name", "translated_name", "description"]
	},
	{
		"class": Bubble, 
		"table": "Bubbles", 
		"fields": ["chapter_id", "page_id", "original_text", "translated_text", "width", "height", "x", "y", "metadata"]
	},
	{
		"class": CharacterChapter, 
		"table": "CharacterChapters", 
		"fields": ["character_id", "chapter_id"]
	},
	{
		"class": CharacterImage, 
		"table": "CharacterImages", 
		"fields": ["character_id", "selected"]
	},
	{
		"class": CharacterNickname, 
		"table": "CharacterNicknames", 
		"fields": ["character_id", "nickname", "context"]
	},
	{
		"class": CleanPage, 
		"table": "CleanPages", 
		"fields": ["raw_page_id", "chapter_id", "number"]
	},
	{
		"class": Cover, 
		"table": "Covers", 
		"fields": ["work_id", "selected"]
	},
	{
		"class": DonePage, 
		"table": "DonePages", 
		"fields": ["raw_page_id", "chapter_id", "number"]
	},
	{
		"class": Glossary, 
		"table": "Glossaries", 
		"fields": ["work_id", "original_expression", "translated_expression", "examples", "description"]
	},
	{
		"class": GlossaryChapter, 
		"table": "GlossaryChapters", 
		"fields": ["glossary_id", "chapter_id"]
	},
	{
		"class": Location, 
		"table": "Locations", 
		"fields": ["work_id", "original_name", "translated_name", "description"]
	},
	{
		"class": LocationChapter, 
		"table": "LocationChapters", 
		"fields": ["location_id", "chapter_id"]
	},
	{
		"class": LocationImage, 
		"table": "LocationImages", 
		"fields": ["location_id"]
	},
	{
		"class": RawPage, 
		"table": "RawPages", 
		"fields": ["chapter_id", "number"]
	}
]

func test_models_fields_and_tables():
	for info in models_to_test:
		var model : BaseModel = info.class.new()
		var file_name = model.get_script().get_path().get_file()
		
		assert_eq(model.table_name, info.table, "Error in %s: incorrect table name." % file_name)
		
		for field in info.fields:
			assert_true(field in model, "Error in %s: field '%s' not found." % [file_name, field])
		
		assert_true("id" in model, "Error in %s: missing 'id' property from BaseModel." % file_name)
		assert_true("table_name" in model, "Error in %s: missing 'table_name' property." % file_name)

func test_instantiation_safety():
	for info in models_to_test:
		var obj = info.class.new()
		assert_not_null(obj, "Failed to instantiate %s" % info.table)
```

### 3. Estresse do Cliente de Banco (`test_db_client.gd`)

Aqui o buraco é mais embaixo. Testamos a comunicação direta com o motor SQLite.

**O que foi testado:**

* **Persistência Real:** Testei se, ao fechar e abrir o banco de dados no disco (`user://`), os dados continuavam lá.
* **Foreign Keys:** Tentei inserir uma capa (`Cover`) para uma obra que não existe. O sistema **precisa** barrar isso para não criar dados órfãos.
* **Unicode/Emojis:** Como trabalhamos com scanlation, testei se o banco aceita caracteres japoneses (ワンピース) e emojis (🏴‍☠️) sem corromper o texto.

```gdscript
extends GutTest

var test_table : String = "Works"
var data : Dictionary = {
	"id": UUID.v4(),
	"original_title": "One piece",
	"translated_title": "One piece",
	"description": "It follows the adventures of Monkey D. Luffy and his crew, the Straw Hat Pirates, as he explores the Grand Line in search of the mythical treasure known as the \"One Piece\" to become the next King of the Pirates.",
	"type": "Manga",
	"status": "Ongoing",
	"author_artist": "Eiichiro Oda"
}

var data_1 : Dictionary = {
	"id": UUID.v4(),
	"original_title": "Detective Conan",
	"translated_title": "Case Closed",
	"description": "The story follows the high school detective Shinichi Kudo, whose body was transformed into that of an elementary school-age child while investigating a mysterious organization.",
	"type": "Manga",
	"status": "Ongoing",
	"author_artist": "Gosho Aoyama"
}

func before_each() -> void:
	DBClient.open_db(":memory:")

func after_each() -> void:
	if DBClient.db:
		DBClient.db.close_db()

func test_open_db_connection() -> void:
	assert_not_null(DBClient.db, "Database instance should be initialized")
	assert_eq(DBClient.db_path, ":memory:", "Database path should be memory-resident")

func test_insert_data() -> void:
	var result_id : String = DBClient.insert(test_table, data)
	
	assert_eq(result_id, data.id, "Insert should return the ID of the record")
	
	var records : Array = DBClient.select_all(test_table)
	assert_eq(records.size(), 1, "There should be exactly one record in the table")
	assert_eq(records[0]["original_title"], "One piece", "Data integrity check failed")

func test_insert_failure_returns_empty_string():
	var result : String = DBClient.insert("invalid_table", {"name": "error"})
	assert_engine_error_count(1)
	assert_eq(result, "", "Should return an empty string on database failure")

func test_update_data() -> void:
	DBClient.insert(test_table, data)
	
	var update_data : Dictionary = {"translated_title": "Uma peça"}
	var success : bool = DBClient.update(test_table, data.id, update_data)
	
	assert_true(success, "Update operation should return true")
	var result : Array = DBClient.select_where(test_table, "id = '" + data.id + "'")
	assert_eq(result[0]["translated_title"], "Uma peça", "The name column was not updated correctly")

func test_select_where_bound() -> void:
	DBClient.insert(test_table, data)
	DBClient.insert(test_table, data_1)
	
	var results : Array = DBClient.select_where_bound(test_table, "author_artist = ?", ["Eiichiro Oda"])
	
	assert_eq(results.size(), 1, "Should find only one work")
	assert_eq(results[0]["author_artist"], "Eiichiro Oda", "Binding should have returned Oda")

func test_select_like_bound() -> void:
	DBClient.insert(test_table, data)
	DBClient.insert(test_table, data_1)
	
	var results : Array = DBClient.select_like_bound(test_table, "author_artist", "Oda")
	
	assert_eq(results.size(), 1, "Search should find 'Oda'")
	assert_true("Oda" in results[0]["author_artist"], "Result should contain the word Oda")

func test_select_no_results():
	var results = DBClient.select_where_bound(test_table, "author_artist = ?", ["Non-existent Author"])
	assert_true(results is Array, "Result should be an Array")
	assert_eq(results.size(), 0, "Array should be empty when no matches are found")

func test_delete_data() -> void:
	DBClient.insert(test_table, data)
	
	var success : bool = DBClient.delete(test_table, "id = '" + data.id + "'")
	
	assert_true(success, "Delete operation should return true")
	var size : int = DBClient.count(test_table)
	assert_eq(size, 0, "Table should be empty after deletion")

func test_sql_injection_resilience() -> void:
	DBClient.insert(test_table, data)
	
	var malicious_id = data.id + "' OR '1'='1"
	var malicious_update = {"translated_title": "Hacked"}
	
	DBClient.update(test_table, malicious_id, malicious_update)
	
	var all_data : Array = DBClient.select_all(test_table)
	assert_ne(all_data[0]["translated_title"], "Hacked", "Update should not be vulnerable to simple SQL injection")

func test_foreign_key_constraint():
	DBClient.insert(test_table, data)
	
	var result : String = DBClient.insert('Covers', {'id': UUID.v4(), 'work_id': "Not exist", 'selected': true})
	assert_engine_error_count(1)
	assert_eq(result, "", "Should fail to insert due to foreign key constraint")

func test_unicode_storage():
	var emoji_data = data.duplicate()
	emoji_data.id = UUID.v4()
	emoji_data.original_title = "One Piece 🏴‍☠️ 🔥"
	emoji_data.translated_title = "ワンピース"
	
	DBClient.insert(test_table, emoji_data)
	var result = DBClient.select_where(test_table, "id = '" + emoji_data.id + "'")
	
	assert_eq(result[0]["original_title"], "One Piece 🏴‍☠️ 🔥", "Falha ao salvar Emojis")
	assert_eq(result[0]["translated_title"], "ワンピース", "Falha ao salvar caracteres não-ASCII")

class TestFileDatabase:
	extends GutTest
	var test_table : String = "Works"
	var data : Dictionary = {"id": UUID.v4(),"original_title": "One piece"}
	var real_path : String = "user://test_persistent.db"
	
	func before_each() -> void:
		DBClient.open_db(real_path)

	func after_each() -> void:
		if DBClient.db:
			DBClient.db.close_db()
			var dir : DirAccess = DirAccess.open("user://")
			dir.remove("test_persistent.db")
	
	func test_persistence_between_sessions():
		DBClient.insert(test_table, data)
		DBClient.db.close_db()
		
		DBClient.open_db(real_path)
		var results = DBClient.select_all(test_table)
		assert_eq(results.size(), 1, "Data should persist after closing and reopening the file")


```

### 4. Ciclo de Vida das Migrações (`test_migration_manager.gd`)

Por fim, testei o nosso sistema de "auto-construção".

**O que foi testado:**

* **Criação Inicial:** Se o `MigrationManager` realmente cria as 17 tabelas que planejamos.
* **Idempotência:** Se eu rodar a migração duas vezes seguidas, ele não pode tentar criar o que já existe nem dar erro de "tabela duplicada".

```gdscript
extends GutTest

var _db = null
var _manager = null

func before_each():
	_db = SQLite.new()
	_db.path = ":memory:"
	_db.open_db()
	_manager = MigrationManager.new()

func after_each():
	if _db:
		_db.close_db()

func test_initial_scheme_tables_are_created():
	_manager.run_migrations(_db)
	
	var expected_tables = ["_Migrations", "Works", "Covers", "Characters", "CharacterImages",
		"CharacterNicknames", "Chapters", "CharacterChapters", "RawPages", "CleanPages", "DonePages",
		"Locations", "LocationImages", "LocationChapters", "Glossaries", "GlossaryChapters", "Bubbles"
	]
	
	for table in expected_tables:
		_db.query("SELECT name FROM sqlite_master WHERE type='table' AND name='%s';" % table)
		assert_eq(_db.query_result.size(), 1, "A tabela %s deve ter sido criada." % table)

func test_migration_version_is_updated():
	_manager.run_migrations(_db)
	
	var result = _db.select_rows("_Migrations", "", ["MAX(version) AS version"])
	assert_eq(result[0]["version"], 1, "A versão atual no banco deve ser 1.")

func test_rerunning_migrations_does_not_fail():
	_manager.run_migrations(_db)
	_manager.run_migrations(_db)
	
	assert_engine_error_count(0, 'no engine errors')
	assert_push_error_count(0, 'no push errors')
	assert_push_warning_count(0, 'no push warnings')
```

---

## Por que isso importa?

Depois de rodar todos esses testes, recebi a famosa "barra verde" do GUT. Isso significa que:

1. Meu sistema de **Migrações** funciona.
2. Meu **ORM** salva e relaciona os dados corretamente.
3. Meu app é **seguro** contra injeção de SQL e corrupção de dados.

Trabalhar com banco de dados pode ser frustrante se você não tiver certeza de que os alicerces estão firmes. Com esses testes, eu reduzi drasticamente o tempo que passaria debugando erros de lógica de dados no futuro.

---

Com o banco de dados devidamente "testado e aprovado", encerramos essa trilogia de back-end. Agora, o TypeBubbleX começa a ganhar cara! No próximo post, vamos começar a falar de **Interface de Usuário (UI)**.

Tenho uma notícia que pode ser boa ou má, dependendo do seu ponto de vista: vou dar uma pausa neste projeto para focar em outro. Preciso priorizar um trabalho da faculdade que consiste em desenvolver um jogo, mas não se preocupem, farei posts sobre o desenvolvimento.

Você pode conferir todos os testes na íntegra no [repositório do GitHub](https://github.com/XandeKK/TypeBubbleX), lembre-se de trocar a branch para v2.

Então é isso. Até a próxima!
