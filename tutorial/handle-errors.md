# Retorne e trate um erro

> Tradução não oficial de [go.dev/doc/tutorial/handle-errors](https://go.dev/doc/tutorial/handle-errors).

Tratar erros é uma característica essencial de um código sólido. Nesta seção, você vai adicionar um pouco de código para retornar um erro do module `greetings` e depois tratá-lo no caller.

> **Nota:** este tópico faz parte de um tutorial em várias partes que começa em [Crie um module Go](create-module.md).

1. Em `greetings/greetings.go`, adicione o código marcado abaixo (linhas novas ou alteradas têm `// novo`).

   Não faz sentido devolver uma saudação se você não sabe para quem é. Retorne um erro para o caller se o nome estiver vazio. Copie o código abaixo para `greetings.go` e salve o arquivo.

   ```go
   package greetings

   import (
       "errors" // novo
       "fmt"
   )

   // Hello returns a greeting for the named person.
   func Hello(name string) (string, error) { // novo: agora retorna (string, error)
       // If no name was given, return an error with a message. // novo
       if name == "" { // novo
           return "", errors.New("empty name") // novo
       } // novo

       // If a name was received, return a value that embeds the name
       // in a greeting message.
       message := fmt.Sprintf("Hi, %v. Welcome!", name)
       return message, nil // novo: ", nil"
   }
   ```

   Neste código, você:

   - Altera a função para que ela retorne dois valores: uma `string` e um `error`. O caller vai checar o segundo valor para ver se aconteceu um erro. (Qualquer função Go pode retornar vários valores. Para mais, veja [Effective Go](https://go.dev/doc/effective_go.html#multiple-returns).)
   - Importa o package `errors` da standard library para poder usar a [função `errors.New`](https://pkg.go.dev/errors/#example-New).
   - Adiciona um `if` para checar se a requisição é inválida (uma string vazia onde deveria estar o nome) e retorna um erro nesse caso. A função `errors.New` retorna um `error` com a sua mensagem dentro.
   - Adiciona `nil` (que significa "nenhum erro") como segundo valor no retorno de sucesso. Assim, o caller consegue ver que a função deu certo.

2. No arquivo `hello/hello.go`, trate o erro que agora é retornado pela função `Hello`, junto com o valor normal (não erro).

   Cole o código abaixo em `hello.go`.

   ```go
   package main

   import (
       "fmt"
       "log" // novo

       "example.com/greetings"
   )

   func main() {
       // Set properties of the predefined Logger, including // novo
       // the log entry prefix and a flag to disable printing
       // the time, source file, and line number.
       log.SetPrefix("greetings: ") // novo
       log.SetFlags(0)              // novo

       // Request a greeting message.
       message, err := greetings.Hello("") // novo
       // If an error was returned, print it to the console and // novo
       // exit the program.
       if err != nil { // novo
           log.Fatal(err) // novo
       } // novo

       // If no error was returned, print the returned message // novo
       // to the console.
       fmt.Println(message)
   }
   ```

   Neste código, você:

   - Configura o [package `log`](https://pkg.go.dev/log/) para imprimir o nome do comando ("greetings: ") no começo das mensagens de log, sem data/hora nem informação do arquivo-fonte.
   - Atribui os dois valores de retorno de `Hello`, incluindo o `error`, a variáveis.
   - Troca o argumento de `Hello` do nome da Gladys por uma string vazia, para você testar o código de tratamento de erro.
   - Verifica se o valor de `error` é diferente de `nil` (non-nil). Nesse caso, não faz sentido continuar.
   - Usa as funções do package `log` da standard library para mostrar a informação do erro. Se acontecer um erro, você usa a [função `Fatal`](https://pkg.go.dev/log?tab=doc#Fatal) do package `log` para imprimir o erro e parar o programa.

3. No terminal, dentro da pasta `hello`, rode o `hello.go` para confirmar que o código funciona.

   Como agora você está passando um nome vazio, vai receber um erro.

   ```
   $ go run .
   greetings: empty name
   exit status 1
   ```

Esse é o tratamento de erro comum em Go: retornar o erro como um valor, para que o caller possa checá-lo.

A seguir, você vai usar uma slice do Go para retornar uma saudação escolhida aleatoriamente.
<!-- nav -->

---

[← Chame o seu código a partir de outro module](call-module-code.md) · [Índice](README.md) · [Retorne uma saudação aleatória →](random-greeting.md)
