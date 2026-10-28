# Retorne uma saudação aleatória

> Tradução não oficial de [go.dev/doc/tutorial/random-greeting](https://go.dev/doc/tutorial/random-greeting).

Nesta seção, você vai alterar o código para que, em vez de retornar sempre a mesma saudação, ele retorne uma entre várias mensagens predefinidas.

> **Nota:** este tópico faz parte de um tutorial em várias partes que começa em [Crie um module Go](create-module.md).

Para isso, você vai usar uma slice do Go. Uma slice é como um array, só que o tamanho dela muda dinamicamente conforme você adiciona e remove itens. A slice é um dos tipos mais úteis do Go.

Você vai adicionar uma pequena slice com três mensagens de saudação e fazer o código retornar uma delas aleatoriamente. Para saber mais sobre slices, veja [Go slices](https://go.dev/blog/slices-intro) no blog do Go.

1. Em `greetings/greetings.go`, altere o código para ficar como abaixo. O que mudou: o import de `"math/rand"`, a chamada `randomFormat()` dentro de `Hello` e a função `randomFormat` inteira, que é nova.

   ```go
   package greetings

   import (
       "errors"
       "fmt"
       "math/rand" // novo
   )

   // Hello returns a greeting for the named person.
   func Hello(name string) (string, error) {
       // If no name was given, return an error with a message.
       if name == "" {
           return name, errors.New("empty name")
       }
       // Create a message using a random format.
       message := fmt.Sprintf(randomFormat(), name) // novo: randomFormat()
       return message, nil
   }

   // randomFormat returns one of a set of greeting messages. The returned
   // message is selected at random.
   func randomFormat() string {
       // A slice of message formats.
       formats := []string{
           "Hi, %v. Welcome!",
           "Great to see you, %v!",
           "Hail, %v! Well met!",
       }

       // Return a randomly selected message format by specifying
       // a random index for the slice of formats.
       return formats[rand.Intn(len(formats))]
   }
   ```

   Neste código, você:

   - Adiciona uma função `randomFormat` que retorna um formato de saudação escolhido aleatoriamente. Repare que `randomFormat` começa com letra minúscula, o que a deixa acessível apenas para código do mesmo package (ou seja, ela não é exportada).
   - Em `randomFormat`, declara uma slice `formats` com três formatos de mensagem. Ao declarar uma slice, você omite o tamanho entre os colchetes, assim: `[]string`. Isso diz ao Go que o tamanho do array por trás da slice pode mudar dinamicamente.
   - Usa o [package `math/rand`](https://pkg.go.dev/math/rand/) para gerar um número aleatório e escolher um item da slice.
   - Em `Hello`, chama a função `randomFormat` para obter um formato para a mensagem que vai retornar, e então usa o formato junto com o valor de `name` para montar a mensagem.
   - Retorna a mensagem (ou um erro), como antes.

2. Em `hello/hello.go`, altere o código para ficar como abaixo.

   Você só está adicionando o nome da Gladys (ou outro nome, se preferir) como argumento na chamada da função `Hello` em `hello.go`.

   ```go
   package main

   import (
       "fmt"
       "log"

       "example.com/greetings"
   )

   func main() {
       // Set properties of the predefined Logger, including
       // the log entry prefix and a flag to disable printing
       // the time, source file, and line number.
       log.SetPrefix("greetings: ")
       log.SetFlags(0)

       // Request a greeting message.
       message, err := greetings.Hello("Gladys") // novo
       // If an error was returned, print it to the console and
       // exit the program.
       if err != nil {
           log.Fatal(err)
       }

       // If no error was returned, print the returned message
       // to the console.
       fmt.Println(message)
   }
   ```

3. No terminal, dentro da pasta `hello`, rode o `hello.go` para confirmar que o código funciona. Rode várias vezes e repare que a saudação muda.

   ```
   $ go run .
   Great to see you, Gladys!

   $ go run .
   Hi, Gladys. Welcome!

   $ go run .
   Hail, Gladys! Well met!
   ```

A seguir, você vai usar uma slice para cumprimentar várias pessoas.

---

[< Retorne e trate um erro](handle-errors.md) · [Retorne saudações para várias pessoas >](greetings-multiple-people.md)
