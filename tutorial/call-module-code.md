# Chame o seu código a partir de outro module

> Tradução não oficial de [go.dev/doc/tutorial/call-module-code](https://go.dev/doc/tutorial/call-module-code).

Na [seção anterior](create-module.md), você criou um module `greetings`. Nesta seção, você vai escrever código que chama a função `Hello` do module que acabou de criar. Você vai escrever um código que pode ser executado como aplicação e que chama o código do module `greetings`.

> **Nota:** este tópico faz parte de um tutorial em várias partes que começa em [Crie um module Go](create-module.md).

1. Crie uma pasta `hello` para o código-fonte do seu Go module. É aqui que você vai escrever o código que faz a chamada (o caller).

   Depois de criar essa pasta, você deve ter uma pasta `hello` e uma `greetings` no mesmo nível, assim:

   ```
   <home>/
    |-- greetings/
    |-- hello/
   ```

   Por exemplo, se o seu terminal estiver na pasta `greetings`, você pode usar os seguintes comandos:

   ```
   cd ..
   mkdir hello
   cd hello
   ```

2. Ative o dependency tracking para o código que você vai escrever.

   Para ativar o dependency tracking, rode o comando [`go mod init`](https://go.dev/ref/mod#go-mod-init), passando o nome do module em que o seu código vai ficar.

   Para este tutorial, use `example.com/hello` como module path.

   ```
   $ go mod init example.com/hello
   go: creating new go.mod: module example.com/hello
   ```

3. No seu editor, dentro da pasta `hello`, crie um arquivo para escrever o código e chame-o de `hello.go`.

4. Escreva o código que chama a função `Hello` e depois imprime o valor que ela retorna.

   Para isso, cole o código abaixo em `hello.go`.

   ```go
   package main

   import (
       "fmt"

       "example.com/greetings"
   )

   func main() {
       // Get a greeting message and print it.
       message := greetings.Hello("Gladys")
       fmt.Println(message)
   }
   ```

   Neste código, você:

   - Declara um package `main`. Em Go, código executado como aplicação precisa estar em um package `main`.
   - Importa dois packages: `example.com/greetings` e o [package `fmt`](https://pkg.go.dev/fmt/). Isso dá ao seu código acesso às funções desses packages. Importar `example.com/greetings` (o package que está no module que você criou antes) dá acesso à função `Hello`. Você também importa o `fmt`, que tem funções para lidar com texto de entrada e saída (como imprimir no console).
   - Obtém uma saudação chamando a função `Hello` do package `greetings`.

5. Edite o module `example.com/hello` para usar o seu module local `example.com/greetings`.

   Em produção, você publicaria o module `example.com/greetings` no repositório dele (com um module path que refletisse o local de publicação), onde as ferramentas do Go conseguiriam encontrá-lo e baixá-lo. Por enquanto, como você ainda não publicou o module, precisa adaptar o module `example.com/hello` para que ele encontre o código de `example.com/greetings` no seu sistema de arquivos local.

   Para isso, use o comando [`go mod edit`](https://go.dev/ref/mod#go-mod-edit) para editar o module `example.com/hello` e redirecionar as ferramentas do Go do module path (onde o module não está) para a pasta local (onde ele está).

   1. No terminal, dentro da pasta `hello`, rode o seguinte comando:

      ```
      $ go mod edit -replace example.com/greetings=../greetings
      ```

      O comando especifica que `example.com/greetings` deve ser substituído por `../greetings` na hora de localizar a dependência. Depois de rodar o comando, o arquivo `go.mod` da pasta `hello` deve ter uma [diretiva `replace`](https://go.dev/doc/modules/gomod-ref#replace) (a linha nova está marcada com `// novo`):

      ```
      module example.com/hello

      go 1.16

      replace example.com/greetings => ../greetings // novo
      ```

   2. No terminal, dentro da pasta `hello`, rode o comando [`go mod tidy`](https://go.dev/ref/mod#go-mod-tidy) para sincronizar as dependências do module `example.com/hello`, adicionando as que o código exige mas que ainda não estão registradas no module.

      ```
      $ go mod tidy
      go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
      ```

      Quando o comando terminar, o `go.mod` do module `example.com/hello` deve ficar assim:

      ```
      module example.com/hello

      go 1.16

      replace example.com/greetings => ../greetings

      require example.com/greetings v0.0.0-00010101000000-000000000000 // novo
      ```

      O comando encontrou o código local na pasta `greetings` e adicionou uma [diretiva `require`](https://go.dev/doc/modules/gomod-ref#require) para indicar que `example.com/hello` depende de `example.com/greetings`. Você criou essa dependência quando importou o package `greetings` em `hello.go`.

      O número depois do module path é um *pseudo-version number* (número de pseudoversão): um número gerado, usado no lugar de um número de versão semântica (que o module ainda não tem).

      Para referenciar um module *publicado*, um `go.mod` normalmente não teria a diretiva `replace` e usaria uma diretiva `require` com um número de versão (tag) no final.

      ```
      require example.com/greetings v1.1.0
      ```

      Para saber mais sobre números de versão, veja [Module version numbering](https://go.dev/doc/modules/version-numbers).

6. No terminal, dentro da pasta `hello`, rode o código para confirmar que funciona.

   ```
   $ go run .
   Hi, Gladys. Welcome!
   ```

Parabéns! Você escreveu dois modules que funcionam.

No próximo tópico, você vai adicionar tratamento de erros.

---

[< Crie um module Go](create-module.md) · [Retorne e trate um erro >](handle-errors.md)
