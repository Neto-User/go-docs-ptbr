# Tutorial: Primeiros passos com Go

> Tradução não oficial de [go.dev/doc/tutorial/getting-started](https://go.dev/doc/tutorial/getting-started).

Neste tutorial, você vai ter uma breve introdução à programação em Go. Ao longo do caminho, você vai:

- Instalar o Go (se ainda não instalou).
- Escrever um código simples de "Hello, world".
- Usar o comando `go` para rodar o seu código.
- Usar a ferramenta de busca de packages do Go para encontrar packages que você pode usar no seu código.
- Chamar funções de um module externo.

> **Nota:** para outros tutoriais, veja o [índice de Tutoriais](README.md).

## Pré-requisitos

- **Um pouco de experiência com programação.** O código aqui é bem simples, mas ajuda saber algo sobre funções.
- **Uma ferramenta para editar o código.** Qualquer editor de texto serve. A maioria tem bom suporte a Go. Os mais populares são VSCode (gratuito), GoLand (pago) e Vim (gratuito).
- **Um terminal de comandos.** O Go funciona bem em qualquer terminal no Linux e no Mac, e no PowerShell ou cmd no Windows.

## Instale o Go

Basta seguir os passos de [Download e instalação](https://go.dev/doc/install).

## Escreva um pouco de código

Comece com o Hello, World.

1. Abra um terminal e vá (`cd`) para a sua pasta pessoal (home).

   No Linux ou Mac:

   ```
   cd
   ```

   No Windows:

   ```
   cd %HOMEPATH%
   ```

2. Crie uma pasta `hello` para o seu primeiro código-fonte em Go.

   Por exemplo, use os seguintes comandos:

   ```
   mkdir hello
   cd hello
   ```

3. Ative o rastreamento de dependências (dependency tracking) do seu código.

   Quando o seu código importa packages que estão em outros modules, você gerencia essas dependências pelo module do seu próprio código. Esse module é definido por um arquivo `go.mod`, que registra os modules que fornecem esses packages. O arquivo `go.mod` fica junto com o seu código, inclusive no repositório.

   Para ativar o dependency tracking criando um arquivo `go.mod`, rode o comando [`go mod init`](https://go.dev/ref/mod#go-mod-init), passando o nome do module em que o seu código vai ficar. Esse nome é o *module path* do module.

   No desenvolvimento real, o module path normalmente é o endereço do repositório onde o código-fonte vai ficar. Por exemplo, o module path pode ser `github.com/mymodule`. Se você pretende publicar o module para outras pessoas usarem, o module path *precisa* ser um endereço de onde as ferramentas do Go consigam baixar o module. Para saber mais sobre como nomear um module com um module path, veja [Managing dependencies](https://go.dev/doc/modules/managing-dependencies#naming_module).

   Para este tutorial, use apenas `example/hello`.

   ```
   $ go mod init example/hello
   go: creating new go.mod: module example/hello
   ```

4. No seu editor de texto, crie um arquivo `hello.go` para escrever o código.

5. Cole o código abaixo no arquivo `hello.go` e salve.

   ```go
   package main

   import "fmt"

   func main() {
       fmt.Println("Hello, World!")
   }
   ```

   Este é o seu código Go. Nele, você:

   - Declara um package `main` (um package é uma forma de agrupar funções, e é formado por todos os arquivos da mesma pasta).
   - Importa o popular [package `fmt`](https://pkg.go.dev/fmt/), que contém funções para formatar texto, incluindo imprimir no console. Esse package faz parte da [standard library](https://pkg.go.dev/std) (biblioteca padrão) que veio junto quando você instalou o Go.
   - Implementa uma função `main` para imprimir uma mensagem no console. Uma função `main` é executada por padrão quando você roda o package `main`.

6. Rode o código para ver a saudação.

   ```
   $ go run .
   Hello, World!
   ```

   O comando [`go run`](https://go.dev/cmd/go/#hdr-Compile_and_run_Go_program) é um dos vários comandos `go` que você vai usar no dia a dia com Go. Use o comando abaixo para ver a lista dos outros:

   ```
   $ go help
   ```

## Chame código de um package externo

Quando você precisa que o seu código faça algo que talvez outra pessoa já tenha implementado, pode procurar um package com funções que você possa usar.

1. Deixe a mensagem impressa um pouco mais interessante usando uma função de um module externo.

   1. Acesse o pkg.go.dev e [pesquise por um package "quote"](https://pkg.go.dev/search?q=quote).
   2. Nos resultados, encontre e clique na v1 do package `rsc.io/quote` (ela deve aparecer em "Other major versions" de `rsc.io/quote/v4`).
   3. Na seção **Documentation**, em **Index**, veja a lista de funções que você pode chamar no seu código. Você vai usar a função `Go`.
   4. No topo da página, repare que o package `quote` está incluído no module `rsc.io/quote`.

   Você pode usar o site pkg.go.dev para encontrar modules publicados cujos packages tenham funções úteis para o seu código. Packages são publicados dentro de modules (como `rsc.io/quote`), onde outras pessoas podem usá-los. Modules recebem novas versões com melhorias ao longo do tempo, e você pode atualizar o seu código para usar essas versões.

2. No seu código Go, importe o package `rsc.io/quote` e adicione uma chamada à função `Go` dele.

   Depois de adicionar as linhas marcadas com `// novo`, o seu código deve ficar assim:

   ```go
   package main

   import "fmt"

   import "rsc.io/quote" // novo

   func main() {
       fmt.Println(quote.Go()) // novo
   }
   ```

3. Adicione os novos requisitos de module e os checksums.

   O Go vai adicionar o module `quote` como requisito (requirement), além de um arquivo `go.sum`, usado para autenticar o module. Para mais detalhes, veja [Authenticating modules](https://go.dev/ref/mod#authenticating) na Go Modules Reference.

   ```
   $ go mod tidy
   go: finding module for package rsc.io/quote
   go: found rsc.io/quote in rsc.io/quote v1.5.2
   ```

4. Rode o código para ver a mensagem gerada pela função que você está chamando.

   ```
   $ go run .
   Don't communicate by sharing memory, share memory by communicating.
   ```

   Repare que o seu código chama a função `Go`, que imprime uma mensagem esperta sobre comunicação.

   Quando você rodou `go mod tidy`, ele encontrou e baixou o module `rsc.io/quote`, que contém o package que você importou. Por padrão, ele baixou a versão mais recente: v1.5.2.

## Escreva mais código

Com esta introdução rápida, você instalou o Go e aprendeu um pouco do básico. Para escrever mais código com outro tutorial, veja [Crie um module Go](create-module.md).
