# Tutorial: Desenvolvendo uma API RESTful com Go e Gin

> Tradução não oficial de [go.dev/doc/tutorial/web-service-gin](https://go.dev/doc/tutorial/web-service-gin).

Este tutorial apresenta o básico de como escrever uma API de web service RESTful com Go e o [Gin Web Framework](https://gin-gonic.com/en/docs/) (Gin).

Você vai aproveitar melhor este tutorial se já tiver uma familiaridade básica com Go e suas ferramentas. Se este é o seu primeiro contato com Go, veja o [Tutorial: Primeiros passos com Go](getting-started.md) para uma introdução rápida.

O Gin simplifica muitas tarefas de programação ligadas à construção de aplicações web, incluindo web services. Neste tutorial, você vai usar o Gin para rotear requisições (requests), obter detalhes das requisições e fazer o marshal de JSON para as respostas.

Neste tutorial, você vai construir um servidor de API RESTful com dois endpoints. O seu projeto de exemplo vai ser um repositório de dados sobre discos antigos de jazz.

O tutorial tem as seguintes seções:

1. Projete os endpoints da API.
2. Crie uma pasta para o seu código.
3. Crie os dados.
4. Escreva um handler para retornar todos os itens.
5. Escreva um handler para adicionar um item novo.
6. Escreva um handler para retornar um item específico.

> **Nota:** para outros tutoriais, veja o [índice de Tutoriais](README.md).

Para fazer este tutorial de forma interativa no Google Cloud Shell, clique no botão abaixo (o tutorial interativo é em inglês).

[![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.png)](https://ide.cloud.google.com/?cloudshell_workspace=~&walkthrough_tutorial_url=https://raw.githubusercontent.com/golang/tour/master/tutorial/web-service-gin.md)

## Pré-requisitos

- **Go.** Recomendamos usar a versão mais recente do Go para seguir este tutorial. Para instruções de instalação, veja [Installing Go](https://go.dev/doc/install).
- **Uma ferramenta para editar o código.** Qualquer editor de texto serve.
- **Um terminal de comandos.** O Go funciona bem em qualquer terminal no Linux e no Mac, e no PowerShell ou cmd no Windows.
- **A ferramenta curl.** No Linux e no Mac, ela já deve estar instalada. No Windows, ela vem incluída a partir do Windows 10 Insider build 17063. Em versões mais antigas do Windows, talvez você precise instalá-la. Para mais detalhes, veja [Tar and Curl Come to Windows](https://docs.microsoft.com/en-us/virtualization/community/team-blog/2017/20171219-tar-and-curl-come-to-windows).

## Projete os endpoints da API

Você vai construir uma API que dá acesso a uma loja que vende gravações antigas em vinil. Então vai precisar oferecer endpoints pelos quais um cliente possa obter e adicionar álbuns para os usuários.

Ao desenvolver uma API, normalmente você começa projetando os endpoints. Os usuários da sua API vão ter mais facilidade se os endpoints forem fáceis de entender.

Estes são os endpoints que você vai criar neste tutorial.

`/albums`
- `GET`: obtém a lista de todos os álbuns, retornada como JSON.
- `POST`: adiciona um álbum novo a partir dos dados da requisição, enviados como JSON.

`/albums/:id`
- `GET`: obtém um álbum pelo ID, retornando os dados do álbum como JSON.

A seguir, você vai criar uma pasta para o seu código.

## Crie uma pasta para o seu código

Para começar, crie um projeto para o código que você vai escrever.

1. Abra um terminal e vá para a sua pasta pessoal (home).

   No Linux ou Mac:

   ```
   $ cd
   ```

   No Windows:

   ```
   C:\> cd %HOMEPATH%
   ```

2. No terminal, crie uma pasta para o seu código chamada `web-service-gin`.

   ```
   $ mkdir web-service-gin
   $ cd web-service-gin
   ```

3. Crie um module no qual você possa gerenciar as dependências.

   Rode o comando `go mod init`, passando o path do module em que o seu código vai ficar.

   ```
   $ go mod init example/web-service-gin
   go: creating new go.mod: module example/web-service-gin
   ```

   Esse comando cria um arquivo `go.mod`, onde as dependências que você adicionar vão ser listadas e rastreadas. Para saber mais sobre como nomear um module com um module path, veja [Managing dependencies](https://go.dev/doc/modules/managing-dependencies#naming_module).

A seguir, você vai projetar as estruturas de dados.

## Crie os dados

Para manter o tutorial simples, você vai guardar os dados em memória. Uma API mais típica interagiria com um banco de dados.

Repare que guardar os dados em memória significa que o conjunto de álbuns vai se perder toda vez que você parar o servidor, e vai ser recriado quando você iniciá-lo.

#### Escreva o código

1. No seu editor de texto, crie um arquivo chamado `main.go` na pasta `web-service-gin`. Você vai escrever o seu código Go nesse arquivo.

2. No topo de `main.go`, cole a seguinte declaração de package.

   ```go
   package main
   ```

   Um programa independente (em oposição a uma library) fica sempre no package `main`.

3. Abaixo da declaração de package, cole a seguinte declaração de uma struct `album`. Você vai usá-la para guardar os dados dos álbuns em memória.

   Struct tags como `` `json:"artist"` `` definem qual deve ser o nome de um campo quando o conteúdo da struct é serializado em JSON. Sem elas, o JSON usaria os nomes dos campos da struct com inicial maiúscula, um estilo não tão comum em JSON.

   ```go
   // album represents data about a record album.
   type album struct {
   	ID     string  `json:"id"`
   	Title  string  `json:"title"`
   	Artist string  `json:"artist"`
   	Price  float64 `json:"price"`
   }
   ```

4. Abaixo da declaração da struct que você acabou de adicionar, cole a seguinte slice de structs `album`, com os dados que você vai usar para começar.

   ```go
   // albums slice to seed record album data.
   var albums = []album{
   	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
   	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
   	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
   }
   ```

A seguir, você vai escrever o código para implementar o seu primeiro endpoint.

## Escreva um handler para retornar todos os itens

Quando o cliente fizer uma requisição em `GET /albums`, você quer retornar todos os álbuns como JSON.

Para isso, você vai escrever:

- A lógica para preparar a resposta.
- O código para mapear o path da requisição para essa lógica.

Repare que essa é a ordem inversa de como eles vão ser executados em tempo de execução, mas você está adicionando primeiro as dependências e depois o código que depende delas.

#### Escreva o código

1. Abaixo do código da struct que você adicionou na seção anterior, cole o código abaixo para obter a lista de álbuns.

   Esta função `getAlbums` cria um JSON a partir da slice de structs `album` e escreve esse JSON na resposta.

   ```go
   // getAlbums responds with the list of all albums as JSON.
   func getAlbums(c *gin.Context) {
   	c.IndentedJSON(http.StatusOK, albums)
   }
   ```

   Neste código, você:

   - Escreve uma função `getAlbums` que recebe um parâmetro [`gin.Context`](https://pkg.go.dev/github.com/gin-gonic/gin#Context). Repare que você poderia ter dado qualquer nome a essa função: nem o Gin nem o Go exigem um formato específico de nome.

     O `gin.Context` é a parte mais importante do Gin. Ele carrega os detalhes da requisição, valida e serializa JSON, entre outras coisas. (Apesar do nome parecido, ele é diferente do package [`context`](https://pkg.go.dev/context/) embutido no Go.)

   - Chama [`Context.IndentedJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.IndentedJSON) para serializar a struct em JSON e adicioná-la à resposta.

     O primeiro argumento da função é o código de status HTTP que você quer enviar ao cliente. Aqui, você está passando a constante [`StatusOK`](https://pkg.go.dev/net/http#StatusOK) do package `net/http` para indicar `200 OK`.

     Repare que você pode trocar `Context.IndentedJSON` por uma chamada a [`Context.JSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.JSON) para enviar um JSON mais compacto. Na prática, a forma indentada é muito mais fácil de usar na hora de depurar, e a diferença de tamanho costuma ser pequena.

2. Perto do topo de `main.go`, logo abaixo da declaração da slice `albums`, cole o código abaixo para associar a função handler a um path de endpoint.

   Isso cria uma associação em que `getAlbums` trata as requisições para o path do endpoint `/albums`.

   ```go
   func main() {
   	router := gin.Default()
   	router.GET("/albums", getAlbums)

   	router.Run("localhost:8080")
   }
   ```

   Neste código, você:

   - Inicializa um router do Gin usando [`Default`](https://pkg.go.dev/github.com/gin-gonic/gin#Default).
   - Usa a função [`GET`](https://pkg.go.dev/github.com/gin-gonic/gin#RouterGroup.GET) para associar o método HTTP `GET` e o path `/albums` a uma função handler.

     Repare que você está passando o *nome* da função `getAlbums`. Isso é diferente de passar o *resultado* da função, o que você faria passando `getAlbums()` (repare nos parênteses).

   - Usa a função [`Run`](https://pkg.go.dev/github.com/gin-gonic/gin#Engine.Run) para ligar o router a um `http.Server` e iniciar o servidor.

3. Perto do topo de `main.go`, logo abaixo da declaração de package, importe os packages de que você precisa para o código que acabou de escrever.

   As primeiras linhas do código devem ficar assim:

   ```go
   package main

   import (
   	"net/http"

   	"github.com/gin-gonic/gin"
   )
   ```

4. Salve o `main.go`.

#### Rode o código

1. Comece a rastrear o module do Gin como dependência.

   No terminal, use [`go get`](https://go.dev/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them) para adicionar o module `github.com/gin-gonic/gin` como dependência do seu module. Use um ponto como argumento, que significa "baixe as dependências do código na pasta atual".

   ```
   $ go get .
   go get: added github.com/gin-gonic/gin v1.7.2
   ```

   O Go resolveu e baixou essa dependência para atender à declaração `import` que você adicionou no passo anterior.

2. No terminal, na pasta que contém o `main.go`, rode o código. Use um ponto como argumento, que significa "rode o código na pasta atual".

   ```
   $ go run .
   ```

   Com o código rodando, você tem um servidor HTTP funcionando, para o qual pode enviar requisições.

3. Em uma nova janela de terminal, use `curl` para fazer uma requisição ao seu web service em execução.

   ```
   $ curl http://localhost:8080/albums
   ```

   O comando deve mostrar os dados com que você inicializou o serviço.

   ```
   [
           {
                   "id": "1",
                   "title": "Blue Train",
                   "artist": "John Coltrane",
                   "price": 56.99
           },
           {
                   "id": "2",
                   "title": "Jeru",
                   "artist": "Gerry Mulligan",
                   "price": 17.99
           },
           {
                   "id": "3",
                   "title": "Sarah Vaughan and Clifford Brown",
                   "artist": "Sarah Vaughan",
                   "price": 39.99
           }
   ]
   ```

Você começou uma API! Na próxima seção, você vai criar outro endpoint, com código para tratar uma requisição `POST` que adiciona um item.

## Escreva um handler para adicionar um item novo

Quando o cliente fizer uma requisição `POST` em `/albums`, você quer adicionar o álbum descrito no corpo (body) da requisição aos dados dos álbuns existentes.

Para isso, você vai escrever:

- A lógica para adicionar o novo álbum à lista existente.
- Um pouco de código para rotear a requisição `POST` para essa lógica.

#### Escreva o código

1. Adicione o código que inclui os dados do álbum na lista de álbuns.

   Em algum lugar depois dos `import`, cole o código abaixo. (O fim do arquivo é um bom lugar para ele, mas o Go não impõe uma ordem para declarar as funções.)

   ```go
   // postAlbums adds an album from JSON received in the request body.
   func postAlbums(c *gin.Context) {
   	var newAlbum album

   	// Call BindJSON to bind the received JSON to
   	// newAlbum.
   	if err := c.BindJSON(&newAlbum); err != nil {
   		return
   	}

   	// Add the new album to the slice.
   	albums = append(albums, newAlbum)
   	c.IndentedJSON(http.StatusCreated, newAlbum)
   }
   ```

   Neste código, você:

   - Usa [`Context.BindJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.BindJSON) para vincular (bind) o corpo da requisição a `newAlbum`.
   - Adiciona (append) a struct `album` inicializada a partir do JSON à slice `albums`.
   - Adiciona um código de status `201` à resposta, junto com o JSON que representa o álbum que você adicionou.

2. Altere a sua função `main` para incluir a função `router.POST`, assim:

   ```go
   func main() {
   	router := gin.Default()
   	router.GET("/albums", getAlbums)
   	router.POST("/albums", postAlbums)

   	router.Run("localhost:8080")
   }
   ```

   Neste código, você:

   - Associa o método `POST` no path `/albums` à função `postAlbums`.

     Com o Gin, você pode associar um handler a uma combinação de método HTTP e path. Assim, você consegue rotear separadamente as requisições enviadas para um mesmo path, conforme o método que o cliente usar.

#### Rode o código

1. Se o servidor ainda estiver rodando desde a seção anterior, pare-o.

2. No terminal, na pasta que contém o `main.go`, rode o código.

   ```
   $ go run .
   ```

3. Em outra janela de terminal, use `curl` para fazer uma requisição ao seu web service em execução.

   ```
   $ curl http://localhost:8080/albums \
       --include \
       --header "Content-Type: application/json" \
       --request "POST" \
       --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
   ```

   O comando deve mostrar os headers e o JSON do álbum adicionado.

   ```
   HTTP/1.1 201 Created
   Content-Type: application/json; charset=utf-8
   Date: Wed, 02 Jun 2021 00:34:12 GMT
   Content-Length: 116

   {
       "id": "4",
       "title": "The Modern Sound of Betty Carter",
       "artist": "Betty Carter",
       "price": 49.99
   }
   ```

4. Como na seção anterior, use `curl` para obter a lista completa de álbuns e confirmar que o novo álbum foi adicionado.

   ```
   $ curl http://localhost:8080/albums \
       --header "Content-Type: application/json" \
       --request "GET"
   ```

   O comando deve mostrar a lista de álbuns.

   ```
   [
           {
                   "id": "1",
                   "title": "Blue Train",
                   "artist": "John Coltrane",
                   "price": 56.99
           },
           {
                   "id": "2",
                   "title": "Jeru",
                   "artist": "Gerry Mulligan",
                   "price": 17.99
           },
           {
                   "id": "3",
                   "title": "Sarah Vaughan and Clifford Brown",
                   "artist": "Sarah Vaughan",
                   "price": 39.99
           },
           {
                   "id": "4",
                   "title": "The Modern Sound of Betty Carter",
                   "artist": "Betty Carter",
                   "price": 49.99
           }
   ]
   ```

Na próxima seção, você vai adicionar código para tratar um `GET` de um item específico.

## Escreva um handler para retornar um item específico

Quando o cliente fizer uma requisição em `GET /albums/[id]`, você quer retornar o álbum cujo ID corresponde ao parâmetro de path `id`.

Para isso, você vai:

- Adicionar a lógica para obter o álbum pedido.
- Mapear o path para essa lógica.

#### Escreva o código

1. Abaixo da função `postAlbums` que você adicionou na seção anterior, cole o código abaixo para obter um álbum específico.

   Esta função `getAlbumByID` extrai o ID do path da requisição e depois procura um álbum correspondente.

   ```go
   // getAlbumByID locates the album whose ID value matches the id
   // parameter sent by the client, then returns that album as a response.
   func getAlbumByID(c *gin.Context) {
   	id := c.Param("id")

   	// Loop over the list of albums, looking for
   	// an album whose ID value matches the parameter.
   	for _, a := range albums {
   		if a.ID == id {
   			c.IndentedJSON(http.StatusOK, a)
   			return
   		}
   	}
   	c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
   }
   ```

   Neste código, você:

   - Usa [`Context.Param`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.Param) para obter o parâmetro de path `id` da URL. Quando você mapear esse handler para um path, vai incluir no path um marcador (placeholder) para o parâmetro.
   - Percorre as structs `album` da slice procurando uma cujo campo `ID` tenha o mesmo valor do parâmetro `id`. Se encontrar, serializa essa struct `album` em JSON e a retorna como resposta, com o código HTTP `200 OK`.

     Como mencionado antes, um serviço real provavelmente usaria uma consulta (query) ao banco de dados para fazer essa busca.

   - Retorna um erro HTTP `404` com [`http.StatusNotFound`](https://pkg.go.dev/net/http#StatusNotFound) se o álbum não for encontrado.

2. Por fim, altere a sua `main` para incluir uma nova chamada a `router.GET`, agora com o path `/albums/:id`, como no exemplo abaixo.

   ```go
   func main() {
   	router := gin.Default()
   	router.GET("/albums", getAlbums)
   	router.GET("/albums/:id", getAlbumByID)
   	router.POST("/albums", postAlbums)

   	router.Run("localhost:8080")
   }
   ```

   Neste código, você:

   - Associa o path `/albums/:id` à função `getAlbumByID`. No Gin, os dois-pontos antes de um item no path indicam que esse item é um parâmetro de path.

#### Rode o código

1. Se o servidor ainda estiver rodando desde a seção anterior, pare-o.

2. No terminal, na pasta que contém o `main.go`, rode o código para iniciar o servidor.

   ```
   $ go run .
   ```

3. Em outra janela de terminal, use `curl` para fazer uma requisição ao seu web service em execução.

   ```
   $ curl http://localhost:8080/albums/2
   ```

   O comando deve mostrar o JSON do álbum cujo ID você usou. Se o álbum não for encontrado, você vai receber um JSON com uma mensagem de erro.

   ```
   {
           "id": "2",
           "title": "Jeru",
           "artist": "Gerry Mulligan",
           "price": 17.99
   }
   ```

## Conclusão

Parabéns! Você acabou de usar Go e Gin para escrever um web service RESTful simples.

Próximos tópicos sugeridos:

- Se você é novo em Go, vai encontrar boas práticas úteis descritas em [Effective Go](https://go.dev/doc/effective_go) e [How to write Go code](https://go.dev/doc/code).
- O [Go Tour](https://go.dev/tour/) é uma ótima introdução passo a passo aos fundamentos do Go.
- Para saber mais sobre o Gin, veja a [documentação do package Gin Web Framework](https://pkg.go.dev/github.com/gin-gonic/gin) ou a [documentação do Gin Web Framework](https://gin-gonic.com/en/docs/).

## Código completo

Esta seção traz o código da aplicação que você constrói neste tutorial.

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

// album represents data about a record album.
type album struct {
	ID     string  `json:"id"`
	Title  string  `json:"title"`
	Artist string  `json:"artist"`
	Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

func main() {
	router := gin.Default()
	router.GET("/albums", getAlbums)
	router.GET("/albums/:id", getAlbumByID)
	router.POST("/albums", postAlbums)

	router.Run("localhost:8080")
}

// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
	c.IndentedJSON(http.StatusOK, albums)
}

// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
	var newAlbum album

	// Call BindJSON to bind the received JSON to
	// newAlbum.
	if err := c.BindJSON(&newAlbum); err != nil {
		return
	}

	// Add the new album to the slice.
	albums = append(albums, newAlbum)
	c.IndentedJSON(http.StatusCreated, newAlbum)
}

// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
	id := c.Param("id")

	// Loop through the list of albums, looking for
	// an album whose ID value matches the parameter.
	for _, a := range albums {
		if a.ID == id {
			c.IndentedJSON(http.StatusOK, a)
			return
		}
	}
	c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```
<!-- nav -->

---

[← Acessando um banco de dados relacional](database-access.md) · [Índice](README.md) · [Primeiros passos com generics →](generics.md)
