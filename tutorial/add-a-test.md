# Adicione um teste

> Tradução não oficial de [go.dev/doc/tutorial/add-a-test](https://go.dev/doc/tutorial/add-a-test).

Agora que o seu código chegou a um ponto estável (bom trabalho, aliás), adicione um teste. Testar o código durante o desenvolvimento pode revelar bugs que aparecem conforme você faz mudanças. Neste tópico, você adiciona um teste para a função `Hello`.

> **Nota:** este tópico faz parte de um tutorial em várias partes que começa em [Crie um module Go](create-module.md).

O suporte embutido do Go a testes unitários facilita testar enquanto você desenvolve. Mais especificamente, usando convenções de nomes, o package `testing` do Go e o comando `go test`, você consegue escrever e rodar testes rapidamente.

1. Na pasta `greetings`, crie um arquivo chamado `greetings_test.go`.

   Terminar o nome do arquivo com `_test.go` avisa ao comando `go test` que esse arquivo contém funções de teste.

2. Em `greetings_test.go`, cole o código abaixo e salve o arquivo.

   ```go
   package greetings

   import (
       "testing"
       "regexp"
   )

   // TestHelloName calls greetings.Hello with a name, checking
   // for a valid return value.
   func TestHelloName(t *testing.T) {
       name := "Gladys"
       want := regexp.MustCompile(`\b`+name+`\b`)
       msg, err := Hello("Gladys")
       if !want.MatchString(msg) || err != nil {
           t.Errorf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
       }
   }

   // TestHelloEmpty calls greetings.Hello with an empty string,
   // checking for an error.
   func TestHelloEmpty(t *testing.T) {
       msg, err := Hello("")
       if msg != "" || err == nil {
           t.Errorf(`Hello("") = %q, %v, want "", error`, msg, err)
       }
   }
   ```

   Neste código, você:

   - Implementa as funções de teste no mesmo package do código que está sendo testado.
   - Cria duas funções de teste para testar a função `greetings.Hello`. Os nomes das funções de teste têm o formato `TestNome`, em que *Nome* diz algo sobre o teste específico. Além disso, as funções de teste recebem como parâmetro um pointer (ponteiro) para o [tipo `testing.T`](https://pkg.go.dev/testing/#T) do package `testing`. Você usa os métodos desse parâmetro para reportar e registrar (log) informações do teste.
   - Implementa dois testes:
     - `TestHelloName` chama a função `Hello` passando um valor de `name` com o qual a função deve conseguir retornar uma mensagem válida. Se a chamada retornar um erro ou uma mensagem inesperada (uma que não inclua o nome que você passou), você usa o [método `Errorf`](https://pkg.go.dev/testing/#T.Errorf) do parâmetro `t` para imprimir uma mensagem no console.
     - `TestHelloEmpty` chama a função `Hello` com uma string vazia. Esse teste serve para confirmar que o seu tratamento de erro funciona. Se a chamada retornar uma string não vazia ou nenhum erro, você usa o [método `Errorf`](https://pkg.go.dev/testing/#T.Errorf) do parâmetro `t` para imprimir uma mensagem no console.

3. No terminal, dentro da pasta `greetings`, rode o comando [`go test`](https://go.dev/cmd/go/#hdr-Test_packages) para executar o teste.

   O comando `go test` executa as funções de teste (cujos nomes começam com `Test`) nos arquivos de teste (cujos nomes terminam com `_test.go`). Você pode adicionar a flag `-v` para ter uma saída detalhada (verbose), que lista todos os testes e seus resultados.

   Os testes devem passar.

   ```
   $ go test
   PASS
   ok      example.com/greetings   0.364s

   $ go test -v
   === RUN   TestHelloName
   --- PASS: TestHelloName (0.00s)
   === RUN   TestHelloEmpty
   --- PASS: TestHelloEmpty (0.00s)
   PASS
   ok      example.com/greetings   0.372s
   ```

4. Quebre a função `greetings.Hello` para ver um teste falhando.

   A função de teste `TestHelloName` verifica o valor retornado para o nome que você passou como parâmetro para `Hello`. Para ver um teste falhando, altere a função `greetings.Hello` para que ela não inclua mais o nome.

   Em `greetings/greetings.go`, cole o código abaixo no lugar da função `Hello`. Repare que as linhas marcadas com `// novo` mudam o valor retornado pela função, como se o argumento `name` tivesse sido removido por acidente.

   ```go
   // Hello returns a greeting for the named person.
   func Hello(name string) (string, error) {
       // If no name was given, return an error with a message.
       if name == "" {
           return name, errors.New("empty name")
       }
       // Create a message using a random format.
       // message := fmt.Sprintf(randomFormat(), name) // novo (linha comentada)
       message := fmt.Sprint(randomFormat()) // novo
       return message, nil
   }
   ```

5. No terminal, dentro da pasta `greetings`, rode `go test` para executar o teste.

   Desta vez, rode `go test` sem a flag `-v`. A saída vai mostrar apenas os testes que falharam, o que é útil quando você tem muitos testes. O teste `TestHelloName` deve falhar, e o `TestHelloEmpty` continua passando.

   ```
   $ go test
   --- FAIL: TestHelloName (0.00s)
       greetings_test.go:15: Hello("Gladys") = "Hail, %v! Well met!", <nil>, want match for `\bGladys\b`, nil
   FAIL
   exit status 1
   FAIL    example.com/greetings   0.182s
   ```

No próximo (e último) tópico, você vai ver como compilar e instalar o seu código para rodá-lo localmente.
<!-- nav -->

---

[← Retorne saudações para várias pessoas](greetings-multiple-people.md) · [Índice](README.md) · [Compile e instale a aplicação →](compile-install.md)
