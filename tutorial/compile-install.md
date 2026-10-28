# Compile e instale a aplicação

> Tradução não oficial de [go.dev/doc/tutorial/compile-install](https://go.dev/doc/tutorial/compile-install).

Neste último tópico, você vai aprender alguns comandos `go` novos. O comando `go run` é um atalho útil para compilar e rodar um programa quando você está fazendo mudanças frequentes, mas ele não gera um executável binário.

Este tópico apresenta dois comandos a mais para fazer o build do código:

- O comando [`go build`](https://go.dev/cmd/go/#hdr-Compile_packages_and_dependencies) compila os packages junto com as dependências, mas não instala o resultado.
- O comando [`go install`](https://go.dev/ref/mod#go-install) compila e instala os packages.

> **Nota:** este tópico faz parte de um tutorial em várias partes que começa em [Crie um module Go](create-module.md).

1. No terminal, dentro da pasta `hello`, rode o comando `go build` para compilar o código em um executável.

   ```
   $ go build
   ```

2. No terminal, dentro da pasta `hello`, rode o novo executável `hello` para confirmar que o código funciona.

   O seu resultado pode ser diferente, dependendo de você ter alterado ou não o `greetings.go` depois de testá-lo.

   - No Linux ou Mac:

     ```
     $ ./hello
     map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
     ```

   - No Windows:

     ```
     $ hello.exe
     map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
     ```

   Você compilou a aplicação em um executável e consegue rodá-lo. Mas, por enquanto, para rodá-lo o terminal precisa estar na pasta do executável, ou você precisa informar o caminho dele.

   A seguir, você vai instalar o executável para poder rodá-lo sem informar o caminho.

3. Descubra o caminho de instalação do Go, onde o comando `go` vai instalar o package atual.

   Você descobre o caminho de instalação rodando o comando [`go list`](https://go.dev/cmd/go/#hdr-List_packages_or_modules), como neste exemplo:

   ```
   $ go list -f '{{.Target}}'
   ```

   Por exemplo, a saída do comando pode ser `/home/gopher/bin/hello`, o que significa que os binários são instalados em `/home/gopher/bin`. Você vai precisar dessa pasta de instalação no próximo passo.

4. Adicione a pasta de instalação do Go ao PATH do seu sistema.

   Assim, você vai conseguir rodar o executável do programa sem informar onde ele está.

   - No Linux ou Mac, rode o comando:

     ```
     $ export PATH=$PATH:/path/to/your/install/directory
     ```

   - No Windows, rode o comando:

     ```
     $ set PATH=%PATH%;C:\path\to\your\install\directory
     ```

   Como alternativa, se você já tem uma pasta como `$HOME/bin` no PATH e quer instalar seus programas Go lá, pode mudar o destino da instalação definindo a variável `GOBIN` com o comando [`go env`](https://go.dev/cmd/go/#hdr-Print_Go_environment_information):

   ```
   $ go env -w GOBIN=/path/to/your/bin
   ```

   ou

   ```
   $ go env -w GOBIN=C:\path\to\your\bin
   ```

5. Depois de atualizar o PATH, rode o comando `go install` para compilar e instalar o package.

   ```
   $ go install
   ```

6. Rode a aplicação digitando apenas o nome dela. Para ficar mais interessante, abra um novo terminal e rode o executável `hello` em outra pasta qualquer.

   ```
   $ hello
   map[Darrin:Hail, Darrin! Well met! Gladys:Great to see you, Gladys! Samantha:Hail, Samantha! Well met!]
   ```

E assim termina este tutorial de Go!

---

[< Adicione um teste](add-a-test.md) · [Conclusão e links para saber mais >](module-conclusion.md)
