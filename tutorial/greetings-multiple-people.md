# Retorne saudações para várias pessoas

> Tradução não oficial de [go.dev/doc/tutorial/greetings-multiple-people](https://go.dev/doc/tutorial/greetings-multiple-people).

Nas últimas mudanças que você vai fazer no código do module, você vai adicionar suporte para obter saudações para várias pessoas em uma única requisição. Ou seja, você vai tratar uma entrada com vários valores e associar cada valor dessa entrada a uma saída com vários valores. Para isso, você precisa passar um conjunto de nomes para uma função que retorne uma saudação para cada um.

> **Nota:** este tópico faz parte de um tutorial em várias partes que começa em [Crie um module Go](create-module.md).

Mas tem um problema. Mudar o parâmetro da função `Hello` de um único nome para um conjunto de nomes mudaria a *signature* (assinatura) da função. Se você já tivesse publicado o module `example.com/greetings` e outras pessoas já tivessem escrito código chamando `Hello`, essa mudança quebraria os programas delas.

Nessa situação, a melhor escolha é escrever uma função nova, com outro nome. A nova função vai receber vários parâmetros. Isso preserva a função antiga, mantendo a compatibilidade com versões anteriores (backward compatibility).

1. Em `greetings/greetings.go`, altere o código para ficar como abaixo. A função `Hellos` inteira é nova.

   ```go
   package greetings

   import (
       "errors"
       "fmt"
       "math/rand"
   )

   // Hello returns a greeting for the named person.
   func Hello(name string) (string, error) {
       // If no name was given, return an error with a message.
       if name == "" {
           return name, errors.New("empty name")
       }
       // Create a message using a random format.
       message := fmt.Sprintf(randomFormat(), name)
       return message, nil
   }

   // Hellos returns a map that associates each of the named people
   // with a greeting message.
   func Hellos(names []string) (map[string]string, error) {
       // A map to associate names with messages.
       messages := make(map[string]string)
       // Loop through the received slice of names, calling
       // the Hello function to get a message for each name.
       for _, name := range names {
           message, err := Hello(name)
           if err != nil {
               return nil, err
           }
           // In the map, associate the retrieved message with
           // the name.
           messages[name] = message
       }
       return messages, nil
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

       // Return one of the message formats selected at random.
       return formats[rand.Intn(len(formats))]
   }
   ```

   Neste código, você:

   - Adiciona uma função `Hellos` cujo parâmetro é uma slice de nomes, em vez de um único nome. Também troca um dos tipos de retorno de `string` para `map`, para poder retornar os nomes associados às mensagens de saudação.
   - Faz a nova função `Hellos` chamar a função `Hello` que já existe. Isso ajuda a reduzir duplicação e mantém as duas funções.
   - Cria um map `messages` para associar cada nome recebido (como chave, key) a uma mensagem gerada (como valor, value). Em Go, você inicializa um map com a sintaxe `make(map[tipo-da-key]tipo-do-value)`. A função `Hellos` retorna esse map para o caller. Para saber mais sobre maps, veja [Go maps in action](https://go.dev/blog/maps) no blog do Go.
   - Percorre (loop) os nomes que a função recebeu, verificando se cada um tem valor não vazio, e associa uma mensagem a cada um. Neste loop `for`, o `range` retorna dois valores: o índice do item atual e uma cópia do valor do item. Você não precisa do índice, então usa o *blank identifier* do Go (um underline, `_`) para ignorá-lo. Para saber mais, veja [The blank identifier](https://go.dev/doc/effective_go.html#blank) no Effective Go.

2. No código que faz a chamada, em `hello/hello.go`, passe uma slice de nomes e depois imprima o conteúdo do map de nomes/mensagens que você recebe de volta.

   Em `hello.go`, altere o código para ficar como abaixo.

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

       // A slice of names. // novo
       names := []string{"Gladys", "Samantha", "Darrin"} // novo

       // Request greeting messages for the names. // novo
       messages, err := greetings.Hellos(names) // novo
       if err != nil {
           log.Fatal(err)
       }
       // If no error was returned, print the returned map of // novo
       // messages to the console.
       fmt.Println(messages) // novo
   }
   ```

   Com essas mudanças, você:

   - Cria uma variável `names` do tipo slice, com três nomes.
   - Passa a variável `names` como argumento para a função `Hellos`.

3. No terminal, vá para a pasta que contém `hello/hello.go` e use `go run` para confirmar que o código funciona.

   A saída deve ser a representação em texto do map que associa nomes a mensagens, algo assim:

   ```
   $ go run .
   map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hail, Samantha! Well met!]
   ```

Este tópico apresentou os maps para representar pares nome/valor. Também apresentou a ideia de preservar a backward compatibility implementando uma função nova para funcionalidades novas ou alteradas em um module. Para saber mais sobre backward compatibility, veja [Keeping your modules compatible](https://go.dev/blog/module-compatibility).

A seguir, você vai usar recursos embutidos do Go para criar um teste unitário para o seu código.

---

[< Retorne uma saudação aleatória](random-greeting.md) · [Adicione um teste >](add-a-test.md)
