# Tutorial: Primeiros passos com workspaces multi-module

> Tradução não oficial de [go.dev/doc/tutorial/workspaces](https://go.dev/doc/tutorial/workspaces).

Este tutorial apresenta o básico dos workspaces multi-module (com vários modules) em Go. Com eles, você pode avisar o comando `go` de que está escrevendo código em vários modules ao mesmo tempo, e fazer o build e rodar o código desses modules com facilidade.

Neste tutorial, você vai criar dois modules em um workspace multi-module compartilhado, fazer mudanças nesses modules e ver o resultado dessas mudanças em um build.

> **Nota:** para outros tutoriais, veja o [índice de Tutoriais](README.md).

## Pré-requisitos

- **Go.** Recomendamos usar a versão mais recente do Go para seguir este tutorial. Para instruções de instalação, veja [Installing Go](https://go.dev/doc/install).
- **Uma ferramenta para editar o código.** Qualquer editor de texto serve.
- **Um terminal de comandos.** O Go funciona bem em qualquer terminal no Linux e no Mac, e no PowerShell ou cmd no Windows.

## Crie um module para o seu código

Para começar, crie um module para o código que você vai escrever.

1. Abra um terminal e vá para a sua pasta pessoal (home).

   No Linux ou Mac:

   ```
   $ cd
   ```

   No Windows:

   ```
   C:\> cd %HOMEPATH%
   ```

   O resto do tutorial vai mostrar `$` como prompt. Os comandos que você usar também funcionam no Windows.

2. No terminal, crie uma pasta para o seu código chamada `workspace`.

   ```
   $ mkdir workspace
   $ cd workspace
   ```

3. Inicialize o module.

   O nosso exemplo vai criar um novo module `hello` que vai depender do module `golang.org/x/example`.

   Crie o module `hello`:

   ```
   $ mkdir hello
   $ cd hello
   $ go mod init example.com/hello
   go: creating new go.mod: module example.com/hello
   ```

   Adicione uma dependência do package `golang.org/x/example/hello/reverse` usando `go get`.

   ```
   $ go get golang.org/x/example/hello/reverse
   ```

   Crie um `hello.go` na pasta `hello` com o seguinte conteúdo:

   ```go
   package main

   import (
       "fmt"

       "golang.org/x/example/hello/reverse"
   )

   func main() {
       fmt.Println(reverse.String("Hello"))
   }
   ```

   Agora, rode o programa `hello`:

   ```
   $ go run .
   olleH
   ```

## Crie o workspace

Neste passo, vamos criar um arquivo `go.work` para definir um workspace com o module.

#### Inicialize o workspace

Na pasta `workspace`, rode:

```
$ go work init ./hello
```

O comando `go work init` diz ao `go` para criar um arquivo `go.work` para um workspace que contém os modules da pasta `./hello`.

O comando `go` gera um arquivo `go.work` assim:

```
go 1.18

use ./hello
```

O arquivo `go.work` tem uma sintaxe parecida com a do `go.mod`.

A diretiva `go` diz ao Go com qual versão do Go o arquivo deve ser interpretado. É parecida com a diretiva `go` do arquivo `go.mod`.

A diretiva `use` diz ao Go que o module da pasta `hello` deve ser um dos main modules (modules principais) ao fazer um build.

Então, em qualquer subpasta de `workspace`, o module vai estar ativo.

#### Rode o programa na pasta do workspace

Na pasta `workspace`, rode:

```
$ go run ./hello
olleH
```

O comando `go` inclui todos os modules do workspace como main modules. Isso nos permite referenciar um package do module mesmo fora dele. Rodar o comando `go run` fora do module ou do workspace daria erro, porque o comando `go` não saberia quais modules usar.

A seguir, vamos adicionar ao workspace uma cópia local do module `golang.org/x/example/hello`. Esse module fica numa subpasta do repositório Git `go.googlesource.com/example`. Depois, vamos adicionar uma nova função ao package `reverse`, que poderemos usar no lugar de `String`.

## Baixe e modifique o module `golang.org/x/example/hello`

Neste passo, vamos baixar uma cópia do repositório Git que contém o module `golang.org/x/example/hello`, adicioná-lo ao workspace e depois adicionar a ele uma nova função que vamos usar no programa `hello`.

1. Clone o repositório.

   Na pasta `workspace`, rode o comando `git` para clonar o repositório:

   ```
   $ git clone https://go.googlesource.com/example
   Cloning into 'example'...
   remote: Total 165 (delta 27), reused 165 (delta 27)
   Receiving objects: 100% (165/165), 434.18 KiB | 1022.00 KiB/s, done.
   Resolving deltas: 100% (27/27), done.
   ```

2. Adicione o module ao workspace.

   O repositório Git acabou de ser baixado em `./example`. O código-fonte do module `golang.org/x/example/hello` está em `./example/hello`. Adicione-o ao workspace:

   ```
   $ go work use ./example/hello
   ```

   O comando `go work use` adiciona um novo module ao arquivo `go.work`. Agora ele vai ficar assim:

   ```
   go 1.18

   use (
       ./hello
       ./example/hello
   )
   ```

   Agora o workspace inclui tanto o module `example.com/hello` quanto o module `golang.org/x/example/hello`, que fornece o package `golang.org/x/example/hello/reverse`.

   Isso vai nos permitir usar o código novo que vamos escrever na nossa cópia do package `reverse`, em vez da versão do package que está no module cache e que baixamos com o comando `go get`.

3. Adicione a nova função.

   Vamos adicionar ao package `golang.org/x/example/hello/reverse` uma nova função que inverte um número.

   Crie um arquivo novo chamado `int.go` na pasta `workspace/example/hello/reverse` com o seguinte conteúdo:

   ```go
   package reverse

   import "strconv"

   // Int returns the decimal reversal of the integer i.
   func Int(i int) int {
       i, _ = strconv.Atoi(String(strconv.Itoa(i)))
       return i
   }
   ```

4. Modifique o programa `hello` para usar a função.

   Altere o conteúdo de `workspace/hello/hello.go` para ficar assim:

   ```go
   package main

   import (
       "fmt"

       "golang.org/x/example/hello/reverse"
   )

   func main() {
       fmt.Println(reverse.String("Hello"), reverse.Int(24601))
   }
   ```

#### Rode o código no workspace

Na pasta `workspace`, rode:

```
$ go run ./hello
olleH 10642
```

O comando `go` encontra o module `example.com/hello`, passado na linha de comando, na pasta `hello` indicada pelo arquivo `go.work`, e da mesma forma resolve o import de `golang.org/x/example/hello/reverse` usando o `go.work`.

O `go.work` pode ser usado no lugar de diretivas [`replace`](https://go.dev/ref/mod#go-mod-file-replace) para trabalhar com vários modules.

Como os dois modules estão no mesmo workspace, é fácil fazer uma mudança em um module e usá-la no outro.

#### Próximo passo (futuro)

Para publicar esses modules do jeito certo, precisaríamos fazer um release do module `golang.org/x/example/hello`, por exemplo na versão `v0.1.0`. Normalmente isso é feito criando uma tag em um commit no repositório de controle de versão do module. Veja a [documentação do fluxo de release de modules](https://go.dev/doc/modules/release-workflow) para mais detalhes. Depois do release, podemos aumentar o requisito do module `golang.org/x/example/hello` em `hello/go.mod`:

```
cd hello
go get golang.org/x/example/hello@v0.1.0
```

Assim, o comando `go` consegue resolver corretamente os modules fora do workspace.

## Saiba mais sobre workspaces

Além do `go work init`, que vimos antes no tutorial, o comando `go` tem alguns subcomandos para trabalhar com workspaces:

- `go work use [-r] [dir]` adiciona uma diretiva `use` ao arquivo `go.work` para `dir`, se a pasta existir, e remove a diretiva `use` se a pasta passada não existir. A flag `-r` examina as subpastas de `dir` de forma recursiva.
- `go work edit` edita o arquivo `go.work` de forma parecida com o `go mod edit`.
- `go work sync` sincroniza as dependências da build list do workspace em cada um dos modules do workspace.

Veja [Workspaces](https://go.dev/ref/mod#workspaces) na Go Modules Reference para mais detalhes sobre workspaces e arquivos `go.work`.
<!-- nav -->

---

[← Conclusão da série de modules](module-conclusion.md) · [Índice](README.md) · [Trabalhando com JSON →](json.md)
