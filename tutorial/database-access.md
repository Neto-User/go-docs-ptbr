# Tutorial: Acessando um banco de dados relacional

> Tradução não oficial de [go.dev/doc/tutorial/database-access](https://go.dev/doc/tutorial/database-access).

Este tutorial apresenta o básico de como acessar um banco de dados relacional com Go e o package `database/sql` da standard library.

Você vai aproveitar melhor este tutorial se já tiver uma familiaridade básica com Go e suas ferramentas. Se este é o seu primeiro contato com Go, veja o [Tutorial: Primeiros passos com Go](getting-started.md) para uma introdução rápida.

O package [`database/sql`](https://pkg.go.dev/database/sql) que você vai usar tem tipos e funções para conectar a bancos de dados, executar transações, cancelar uma operação em andamento e mais. Para mais detalhes sobre como usar o package, veja [Accessing databases](https://go.dev/doc/database/index).

Neste tutorial, você vai criar um banco de dados e depois escrever código para acessá-lo. O seu projeto de exemplo vai ser um repositório de dados sobre discos antigos de jazz.

Neste tutorial, você vai passar pelas seguintes seções:

1. Criar uma pasta para o seu código.
2. Configurar um banco de dados.
3. Importar o driver do banco de dados.
4. Obter um database handle e conectar.
5. Consultar várias linhas.
6. Consultar uma única linha.
7. Adicionar dados.

> **Nota:** para outros tutoriais, veja o [índice de Tutoriais](README.md).

## Pré-requisitos

- **Uma instalação do sistema de gerenciamento de banco de dados (DBMS) relacional [MySQL](https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/).**
- **Uma instalação do Go.** Para instruções de instalação, veja [Installing Go](https://go.dev/doc/install).
- **Uma ferramenta para editar o código.** Qualquer editor de texto serve.
- **Um terminal de comandos.** O Go funciona bem em qualquer terminal no Linux e no Mac, e no PowerShell ou cmd no Windows.

## Crie uma pasta para o seu código

Para começar, crie uma pasta para o código que você vai escrever.

1. Abra um terminal e vá para a sua pasta pessoal (home).

   No Linux ou Mac:

   ```
   $ cd
   ```

   No Windows:

   ```
   C:\> cd %HOMEPATH%
   ```

   No resto do tutorial, vamos mostrar `$` como prompt. Os comandos que usamos também funcionam no Windows.

2. No terminal, crie uma pasta para o seu código chamada `data-access`.

   ```
   $ mkdir data-access
   $ cd data-access
   ```

3. Crie um module no qual você possa gerenciar as dependências que vai adicionar durante este tutorial.

   Rode o comando `go mod init`, passando o module path do seu novo código.

   ```
   $ go mod init example/data-access
   go: creating new go.mod: module example/data-access
   ```

   Esse comando cria um arquivo `go.mod`, onde as dependências que você adicionar vão ser listadas e rastreadas. Para mais, veja [Managing dependencies](https://go.dev/doc/modules/managing-dependencies).

   **Nota:** no desenvolvimento real, você especificaria um module path mais adequado às suas necessidades. Para mais, veja [Managing dependencies](https://go.dev/doc/modules/managing-dependencies#naming_module).

A seguir, você vai criar um banco de dados.

## Configure um banco de dados

Neste passo, você vai criar o banco de dados com que vai trabalhar. Você vai usar o CLI do próprio DBMS para criar o banco e a tabela, e também para adicionar dados.

Você vai criar um banco de dados com dados sobre gravações antigas de jazz em vinil.

O código aqui usa o [MySQL CLI](https://dev.mysql.com/doc/refman/8.0/en/mysql.html), mas a maioria dos DBMS tem o próprio CLI com recursos parecidos.

1. Abra um novo terminal.

2. Na linha de comando, entre no seu DBMS, como neste exemplo para o MySQL.

   ```
   $ mysql -u root -p
   Enter password:

   mysql>
   ```

3. No prompt do `mysql`, crie um banco de dados.

   ```
   mysql> create database recordings;
   ```

4. Mude para o banco que você acabou de criar, para poder adicionar tabelas.

   ```
   mysql> use recordings;
   Database changed
   ```

5. No seu editor de texto, na pasta `data-access`, crie um arquivo chamado `create-tables.sql` para guardar o script SQL que adiciona as tabelas.

6. Cole o código SQL abaixo no arquivo e salve.

   ```sql
   DROP TABLE IF EXISTS album;
   CREATE TABLE album (
     id         INT AUTO_INCREMENT NOT NULL,
     title      VARCHAR(128) NOT NULL,
     artist     VARCHAR(255) NOT NULL,
     price      DECIMAL(5,2) NOT NULL,
     PRIMARY KEY (`id`)
   );

   INSERT INTO album
     (title, artist, price)
   VALUES
     ('Blue Train', 'John Coltrane', 56.99),
     ('Giant Steps', 'John Coltrane', 63.99),
     ('Jeru', 'Gerry Mulligan', 17.99),
     ('Sarah Vaughan', 'Sarah Vaughan', 34.98);
   ```

   Neste código SQL, você:

   - Apaga (drop) uma tabela chamada `album`. Rodar esse comando primeiro facilita rodar o script de novo depois, se você quiser recomeçar a tabela do zero.
   - Cria uma tabela `album` com quatro colunas: `title`, `artist` e `price`. O valor de `id` de cada linha é criado automaticamente pelo DBMS. *(Nota da tradução: o original fala em quatro colunas e lista três; a quarta é o próprio `id`.)*
   - Adiciona quatro linhas com valores.

7. No prompt do `mysql`, rode o script que você acabou de criar.

   Você vai usar o comando `source` neste formato:

   ```
   mysql> source /path/to/create-tables.sql
   ```

8. No prompt do seu DBMS, use um `SELECT` para conferir que você criou a tabela com os dados.

   ```
   mysql> select * from album;
   +----+---------------+----------------+-------+
   | id | title         | artist         | price |
   +----+---------------+----------------+-------+
   |  1 | Blue Train    | John Coltrane  | 56.99 |
   |  2 | Giant Steps   | John Coltrane  | 63.99 |
   |  3 | Jeru          | Gerry Mulligan | 17.99 |
   |  4 | Sarah Vaughan | Sarah Vaughan  | 34.98 |
   +----+---------------+----------------+-------+
   4 rows in set (0.00 sec)
   ```

A seguir, você vai escrever código Go para conectar e poder fazer consultas.

## Encontre e importe um driver de banco de dados

Agora que você tem um banco de dados com alguns dados, comece o seu código Go.

Encontre e importe um driver de banco de dados, que vai traduzir as requisições feitas pelas funções do package `database/sql` em requisições que o banco entende.

1. No navegador, acesse a página da wiki [SQLDrivers](https://go.dev/wiki/SQLDrivers) para escolher um driver que você possa usar.

   Use a lista da página para escolher o driver. Para acessar o MySQL neste tutorial, você vai usar o [Go-MySQL-Driver](https://github.com/go-sql-driver/mysql/).

2. Anote o nome do package do driver: aqui, `github.com/go-sql-driver/mysql`.

3. No seu editor de texto, crie um arquivo para escrever o código Go e salve-o como `main.go` na pasta `data-access` que você criou antes.

4. Em `main.go`, cole o código abaixo para importar o package do driver.

   ```go
   package main

   import "github.com/go-sql-driver/mysql"
   ```

   Neste código, você:

   - Coloca o seu código em um package `main` para poder executá-lo de forma independente.
   - Importa o driver do MySQL `github.com/go-sql-driver/mysql`.

Com o driver importado, você vai começar a escrever o código para acessar o banco de dados.

## Obtenha um database handle e conecte

Agora escreva um código Go que dá acesso ao banco de dados por meio de um database handle.

Você vai usar um pointer para uma struct `sql.DB`, que representa o acesso a um banco de dados específico.

#### Escreva o código

1. Em `main.go`, abaixo do `import` que você acabou de adicionar, cole o código Go abaixo para criar um database handle.

   ```go
   var db *sql.DB

   func main() {
   	// Capture connection properties.
   	cfg := mysql.NewConfig()
   	cfg.User = os.Getenv("DBUSER")
   	cfg.Passwd = os.Getenv("DBPASS")
   	cfg.Net = "tcp"
   	cfg.Addr = "127.0.0.1:3306"
   	cfg.DBName = "recordings"

   	// Get a database handle.
   	var err error
   	db, err = sql.Open("mysql", cfg.FormatDSN())
   	if err != nil {
   		log.Fatal(err)
   	}

   	pingErr := db.Ping()
   	if pingErr != nil {
   		log.Fatal(pingErr)
   	}
   	fmt.Println("Connected!")
   }
   ```

   Neste código, você:

   - Declara uma variável `db` do tipo [`*sql.DB`](https://pkg.go.dev/database/sql#DB). Esse é o seu database handle.

     Deixar `db` como variável global simplifica este exemplo. Em produção, você evitaria a variável global, por exemplo passando a variável para as funções que precisam dela ou colocando-a dentro de uma struct.

   - Usa o [`Config`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config) do driver do MySQL (e o [`FormatDSN`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config.FormatDSN) desse tipo) para reunir as propriedades de conexão e formatá-las num DSN para a connection string.

     A struct `Config` deixa o código mais fácil de ler do que uma connection string.

   - Chama [`sql.Open`](https://pkg.go.dev/database/sql#Open) para inicializar a variável `db`, passando o valor retornado por `FormatDSN`.
   - Verifica se `sql.Open` retornou erro. Ele pode falhar se, por exemplo, os dados de conexão do banco não estiverem bem formados.

     Para simplificar o código, você chama `log.Fatal` para encerrar a execução e imprimir o erro no console. Em código de produção, você vai querer tratar os erros de forma mais elegante.

   - Chama [`DB.Ping`](https://pkg.go.dev/database/sql#DB.Ping) para confirmar que a conexão com o banco funciona. Em tempo de execução, o `sql.Open` pode não conectar imediatamente, dependendo do driver. Você está usando o `Ping` aqui para confirmar que o package `database/sql` consegue conectar quando precisar.
   - Verifica se o `Ping` retornou erro, caso a conexão tenha falhado.
   - Imprime uma mensagem se o `Ping` conectar com sucesso.

2. Perto do topo de `main.go`, logo abaixo da declaração de package, importe os packages de que você precisa para o código que acabou de escrever.

   O topo do arquivo agora deve ficar assim:

   ```go
   package main

   import (
   	"database/sql"
   	"fmt"
   	"log"
   	"os"

   	"github.com/go-sql-driver/mysql"
   )
   ```

3. Salve o `main.go`.

#### Rode o código

1. Comece a rastrear o module do driver do MySQL como dependência.

   Use o [`go get`](https://go.dev/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them) para adicionar o module `github.com/go-sql-driver/mysql` como dependência do seu module. Use um ponto como argumento, que significa "baixe as dependências do código na pasta atual".

   ```
   $ go get .
   go: added filippo.io/edwards25519 v1.1.0
   go: added github.com/go-sql-driver/mysql v1.8.1
   ```

   O Go baixou essa dependência porque você a adicionou na declaração `import` no passo anterior. Para saber mais sobre dependency tracking, veja [Adding a dependency](https://go.dev/doc/modules/managing-dependencies#adding_dependency).

2. No terminal, defina as variáveis de ambiente `DBUSER` e `DBPASS` para o programa Go usar.

   No Linux ou Mac:

   ```
   $ export DBUSER=username
   $ export DBPASS=password
   ```

   No Windows:

   ```
   C:\Users\you\data-access> set DBUSER=username
   C:\Users\you\data-access> set DBPASS=password
   ```

3. No terminal, na pasta que contém o `main.go`, rode o código digitando `go run` com um ponto como argumento, que significa "rode o package da pasta atual".

   ```
   $ go run .
   Connected!
   ```

Você consegue conectar! A seguir, você vai consultar alguns dados.

## Consulte várias linhas

Nesta seção, você vai usar Go para executar uma consulta SQL que retorna várias linhas.

Para comandos SQL que podem retornar várias linhas, você usa o método `Query` do package `database/sql` e depois percorre as linhas retornadas. (Você vai aprender a consultar uma única linha mais adiante, na seção [Consulte uma única linha](#consulte-uma-única-linha).)

#### Escreva o código

1. Em `main.go`, logo acima de `func main`, cole a definição abaixo de uma struct `Album`. Você vai usá-la para guardar os dados das linhas retornadas pela consulta.

   ```go
   type Album struct {
   	ID     int64
   	Title  string
   	Artist string
   	Price  float32
   }
   ```

2. Abaixo de `func main`, cole a função `albumsByArtist` abaixo para consultar o banco de dados.

   ```go
   // albumsByArtist queries for albums that have the specified artist name.
   func albumsByArtist(name string) ([]Album, error) {
   	// An albums slice to hold data from returned rows.
   	var albums []Album

   	rows, err := db.Query("SELECT * FROM album WHERE artist = ?", name)
   	if err != nil {
   		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
   	}
   	defer rows.Close()
   	// Loop through rows, using Scan to assign column data to struct fields.
   	for rows.Next() {
   		var alb Album
   		if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
   			return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
   		}
   		albums = append(albums, alb)
   	}
   	if err := rows.Err(); err != nil {
   		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
   	}
   	return albums, nil
   }
   ```

   Neste código, você:

   - Declara uma slice `albums` do tipo `Album` que você definiu. Ela vai guardar os dados das linhas retornadas. Os nomes e tipos dos campos da struct correspondem aos nomes e tipos das colunas do banco.
   - Usa [`DB.Query`](https://pkg.go.dev/database/sql#DB.Query) para executar um `SELECT` que consulta os álbuns do artista informado.

     O primeiro parâmetro de `Query` é o comando SQL. Depois dele, você pode passar zero ou mais parâmetros de qualquer tipo. Eles servem para você informar os valores dos parâmetros do comando SQL. Separando o comando SQL dos valores dos parâmetros (em vez de concatená-los com, por exemplo, `fmt.Sprintf`), você permite que o package `database/sql` envie os valores separados do texto SQL, eliminando qualquer risco de SQL injection.

   - Adia (`defer`) o fechamento de `rows`, para que os recursos que ele usa sejam liberados quando a função terminar.
   - Percorre as linhas retornadas, usando [`Rows.Scan`](https://pkg.go.dev/database/sql#Rows.Scan) para atribuir os valores das colunas de cada linha aos campos da struct `Album`.

     O `Scan` recebe uma lista de pointers para valores Go, onde os valores das colunas vão ser escritos. Aqui, você passa pointers para os campos da variável `alb`, criados com o operador `&`. O `Scan` escreve através dos pointers para atualizar os campos da struct.

   - Dentro do loop, verifica se houve erro ao ler (scan) os valores das colunas para os campos da struct.
   - Dentro do loop, adiciona (append) o novo `alb` à slice `albums`.
   - Depois do loop, verifica se houve erro na consulta como um todo, usando `rows.Err`. Repare que, se a própria consulta falhar, verificar o erro aqui é a única forma de descobrir que os resultados estão incompletos.

3. Atualize a sua função `main` para chamar `albumsByArtist`.

   No fim de `func main`, adicione o código abaixo.

   ```go
   albums, err := albumsByArtist("John Coltrane")
   if err != nil {
   	log.Fatal(err)
   }
   fmt.Printf("Albums found: %v\n", albums)
   ```

   No código novo, agora você:

   - Chama a função `albumsByArtist` que adicionou, atribuindo o valor retornado a uma nova variável `albums`.
   - Imprime o resultado.

#### Rode o código

No terminal, na pasta que contém o `main.go`, rode o código.

```
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
```

A seguir, você vai consultar uma única linha.

## Consulte uma única linha

Nesta seção, você vai usar Go para consultar uma única linha no banco de dados.

Para comandos SQL que você sabe que vão retornar no máximo uma linha, você pode usar o `QueryRow`, que é mais simples que um loop com `Query`.

#### Escreva o código

1. Abaixo de `albumsByArtist`, cole a função `albumByID` abaixo.

   ```go
   // albumByID queries for the album with the specified ID.
   func albumByID(id int64) (Album, error) {
   	// An album to hold data from the returned row.
   	var alb Album

   	row := db.QueryRow("SELECT * FROM album WHERE id = ?", id)
   	if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
   		if err == sql.ErrNoRows {
   			return alb, fmt.Errorf("albumsById %d: no such album", id)
   		}
   		return alb, fmt.Errorf("albumsById %d: %v", id, err)
   	}
   	return alb, nil
   }
   ```

   Neste código, você:

   - Usa [`DB.QueryRow`](https://pkg.go.dev/database/sql#DB.QueryRow) para executar um `SELECT` que consulta o álbum com o ID informado.

     Ele retorna um `sql.Row`. Para simplificar o código que chama (o seu código!), o `QueryRow` não retorna erro. Em vez disso, ele deixa qualquer erro da consulta (como `sql.ErrNoRows`) para ser retornado depois, pelo `Rows.Scan`.

   - Usa [`Row.Scan`](https://pkg.go.dev/database/sql#Row.Scan) para copiar os valores das colunas para os campos da struct.
   - Verifica se o `Scan` retornou erro.

     O erro especial `sql.ErrNoRows` indica que a consulta não retornou nenhuma linha. Normalmente vale a pena trocar esse erro por um texto mais específico, como "no such album" aqui.

2. Atualize a `main` para chamar `albumByID`.

   No fim de `func main`, adicione o código abaixo.

   ```go
   // Hard-code ID 2 here to test the query.
   alb, err := albumByID(2)
   if err != nil {
   	log.Fatal(err)
   }
   fmt.Printf("Album found: %v\n", alb)
   ```

   No código novo, agora você:

   - Chama a função `albumByID` que adicionou.
   - Imprime o álbum retornado.

#### Rode o código

No terminal, na pasta que contém o `main.go`, rode o código.

```
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
```

A seguir, você vai adicionar um álbum ao banco de dados.

## Adicione dados

Nesta seção, você vai usar Go para executar um `INSERT` SQL que adiciona uma linha nova ao banco de dados.

Você já viu como usar `Query` e `QueryRow` com comandos SQL que retornam dados. Para executar comandos SQL que *não* retornam dados, você usa o `Exec`.

#### Escreva o código

1. Abaixo de `albumByID`, cole a função `addAlbum` abaixo para inserir um novo álbum no banco de dados, e depois salve o `main.go`.

   ```go
   // addAlbum adds the specified album to the database,
   // returning the album ID of the new entry
   func addAlbum(alb Album) (int64, error) {
   	result, err := db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)", alb.Title, alb.Artist, alb.Price)
   	if err != nil {
   		return 0, fmt.Errorf("addAlbum: %v", err)
   	}
   	id, err := result.LastInsertId()
   	if err != nil {
   		return 0, fmt.Errorf("addAlbum: %v", err)
   	}
   	return id, nil
   }
   ```

   Neste código, você:

   - Usa [`DB.Exec`](https://pkg.go.dev/database/sql#DB.Exec) para executar um `INSERT`.

     Assim como o `Query`, o `Exec` recebe um comando SQL seguido dos valores dos parâmetros desse comando.

   - Verifica se a tentativa de `INSERT` retornou erro.
   - Obtém o ID da linha inserida no banco usando [`Result.LastInsertId`](https://pkg.go.dev/database/sql#Result.LastInsertId).
   - Verifica se a tentativa de obter o ID retornou erro.

2. Atualize a `main` para chamar a nova função `addAlbum`.

   No fim de `func main`, adicione o código abaixo.

   ```go
   albID, err := addAlbum(Album{
   	Title:  "The Modern Sound of Betty Carter",
   	Artist: "Betty Carter",
   	Price:  49.99,
   })
   if err != nil {
   	log.Fatal(err)
   }
   fmt.Printf("ID of added album: %v\n", albID)
   ```

   No código novo, agora você:

   - Chama `addAlbum` com um álbum novo, atribuindo o ID do álbum adicionado a uma variável `albID`.

#### Rode o código

No terminal, na pasta que contém o `main.go`, rode o código.

```
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
ID of added album: 5
```

## Conclusão

Parabéns! Você acabou de usar Go para fazer operações simples com um banco de dados relacional.

Próximos tópicos sugeridos:

- Dê uma olhada no guia de acesso a dados ([Accessing databases](https://go.dev/doc/database/index)), que tem mais informações sobre os assuntos apenas mencionados aqui.
- Se você é novo em Go, vai encontrar boas práticas úteis descritas em [Effective Go](https://go.dev/doc/effective_go) e [How to write Go code](https://go.dev/doc/code).
- O [Go Tour](https://go.dev/tour/) é uma ótima introdução passo a passo aos fundamentos do Go.

## Código completo

Esta seção traz o código da aplicação que você constrói neste tutorial.

```go
package main

import (
	"database/sql"
	"fmt"
	"log"
	"os"

	"github.com/go-sql-driver/mysql"
)

var db *sql.DB

type Album struct {
	ID     int64
	Title  string
	Artist string
	Price  float32
}

func main() {
	// Capture connection properties.
	cfg := mysql.NewConfig()
	cfg.User = os.Getenv("DBUSER")
	cfg.Passwd = os.Getenv("DBPASS")
	cfg.Net = "tcp"
	cfg.Addr = "127.0.0.1:3306"
	cfg.DBName = "recordings"

	// Get a database handle.
	var err error
	db, err = sql.Open("mysql", cfg.FormatDSN())
	if err != nil {
		log.Fatal(err)
	}

	pingErr := db.Ping()
	if pingErr != nil {
		log.Fatal(pingErr)
	}
	fmt.Println("Connected!")

	albums, err := albumsByArtist("John Coltrane")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Albums found: %v\n", albums)

	// Hard-code ID 2 here to test the query.
	alb, err := albumByID(2)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Album found: %v\n", alb)

	albID, err := addAlbum(Album{
		Title:  "The Modern Sound of Betty Carter",
		Artist: "Betty Carter",
		Price:  49.99,
	})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("ID of added album: %v\n", albID)
}

// albumsByArtist queries for albums that have the specified artist name.
func albumsByArtist(name string) ([]Album, error) {
	// An albums slice to hold data from returned rows.
	var albums []Album

	rows, err := db.Query("SELECT * FROM album WHERE artist = ?", name)
	if err != nil {
		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
	}
	defer rows.Close()
	// Loop through rows, using Scan to assign column data to struct fields.
	for rows.Next() {
		var alb Album
		if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
			return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
		}
		albums = append(albums, alb)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
	}
	return albums, nil
}

// albumByID queries for the album with the specified ID.
func albumByID(id int64) (Album, error) {
	// An album to hold data from the returned row.
	var alb Album

	row := db.QueryRow("SELECT * FROM album WHERE id = ?", id)
	if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
		if err == sql.ErrNoRows {
			return alb, fmt.Errorf("albumsById %d: no such album", id)
		}
		return alb, fmt.Errorf("albumsById %d: %v", id, err)
	}
	return alb, nil
}

// addAlbum adds the specified album to the database,
// returning the album ID of the new entry
func addAlbum(alb Album) (int64, error) {
	result, err := db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)", alb.Title, alb.Artist, alb.Price)
	if err != nil {
		return 0, fmt.Errorf("addAlbum: %v", err)
	}
	id, err := result.LastInsertId()
	if err != nil {
		return 0, fmt.Errorf("addAlbum: %v", err)
	}
	return id, nil
}
```
