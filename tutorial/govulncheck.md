# Tutorial: Encontre e corrija dependências vulneráveis com o govulncheck

> Tradução não oficial de [go.dev/doc/tutorial/govulncheck](https://go.dev/doc/tutorial/govulncheck).

O govulncheck é uma ferramenta de pouco ruído que ajuda você a encontrar e corrigir dependências vulneráveis nos seus projetos Go. Ele faz isso analisando as dependências do projeto em busca de vulnerabilidades conhecidas e depois identificando qualquer chamada direta ou indireta a essas vulnerabilidades no seu código.

Neste tutorial, você vai aprender a usar o govulncheck para analisar um programa simples em busca de vulnerabilidades. Também vai aprender a priorizar e avaliar vulnerabilidades, para focar primeiro em corrigir as mais importantes.

Para saber mais sobre o govulncheck, veja a [documentação do govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck) e este [post no blog sobre gestão de vulnerabilidades](https://go.dev/blog/vuln) em Go. O time do Go também adoraria [receber o seu feedback](https://go.dev/s/govulncheck-feedback).

## Pré-requisitos

- **Go.** Recomendamos usar a versão mais recente do Go para seguir este tutorial. Para instruções de instalação, veja [Installing Go](https://go.dev/doc/install).
- **Um editor de código.** Qualquer editor serve.
- **Um terminal de comandos.** O Go funciona bem em qualquer terminal no Linux e no Mac, e no PowerShell ou cmd no Windows.

O tutorial vai guiar você pelos seguintes passos:

1. Criar um Go module de exemplo com uma dependência vulnerável.
2. Instalar e rodar o govulncheck.
3. Avaliar as vulnerabilidades.
4. Atualizar as dependências vulneráveis.

## Crie um Go module de exemplo com uma dependência vulnerável

**Passo 1.** Para começar, crie uma pasta nova chamada `vuln-tutorial` e inicialize um Go module. (Se você é novo em Go modules, veja [Crie um module Go](create-module.md).)

Por exemplo, a partir da sua pasta home, rode:

```
$ mkdir vuln-tutorial
$ cd vuln-tutorial
$ go mod init vuln.tutorial
```

**Passo 2.** Crie um arquivo chamado `main.go` dentro da pasta `vuln-tutorial` e copie o código abaixo para ele:

```go
package main

import (
        "fmt"
        "os"

        "golang.org/x/text/language"
)

func main() {
        for _, arg := range os.Args[1:] {
                tag, err := language.Parse(arg)
                if err != nil {
                        fmt.Printf("%s: error: %v\n", arg, err)
                } else if tag == language.Und {
                        fmt.Printf("%s: undefined\n", arg)
                } else {
                        fmt.Printf("%s: tag %s\n", arg, tag)
                }
        }
}
```

Este programa de exemplo recebe uma lista de language tags (etiquetas de idioma) como argumentos de linha de comando e imprime uma mensagem para cada tag, dizendo se ela foi interpretada (parse) com sucesso, se está indefinida ou se houve erro ao interpretá-la.

**Passo 3.** Rode `go mod tidy`, que vai preencher o arquivo `go.mod` com todas as dependências exigidas pelo código que você adicionou ao `main.go` no passo anterior.

Na pasta `vuln-tutorial`, rode:

```
$ go mod tidy
```

Você deve ver esta saída:

```
go: finding module for package golang.org/x/text/language
go: downloading golang.org/x/text v0.9.0
go: found golang.org/x/text/language in golang.org/x/text v0.9.0
```

**Passo 4.** Abra o seu arquivo `go.mod` para conferir que ele está assim:

```
module vuln.tutorial

go 1.20

require golang.org/x/text v0.9.0
```

**Passo 5.** Volte (downgrade) a versão de `golang.org/x/text` para a v0.3.5, que tem vulnerabilidades conhecidas. Rode:

```
$ go get golang.org/x/text@v0.3.5
```

Você deve ver esta saída:

```
go: downgraded golang.org/x/text v0.9.0 => v0.3.5
```

O arquivo `go.mod` agora deve ficar assim:

```
module vuln.tutorial

go 1.20

require golang.org/x/text v0.3.5
```

Agora, vamos ver o govulncheck em ação.

## Instale e rode o govulncheck

**Passo 6.** Instale o govulncheck com o comando `go install`:

```
$ go install golang.org/x/vuln/cmd/govulncheck@latest
```

**Passo 7.** Na pasta que você quer analisar (neste caso, `vuln-tutorial`), rode:

```
$ govulncheck ./...
```

Você deve ver esta saída:

```
govulncheck is an experimental tool. Share feedback at https://go.dev/s/govulncheck-feedback.

Using go1.20.3 and govulncheck@v0.0.0 with
vulnerability data from https://vuln.go.dev (last modified 2023-04-18 21:32:26 +0000 UTC).

Scanning your code and 46 packages across 1 dependent module for known vulnerabilities...
Your code is affected by 1 vulnerability from 1 module.

Vulnerability #1: GO-2021-0113
  Due to improper index calculation, an incorrectly formatted
  language tag can cause Parse to panic via an out of bounds read.
  If Parse is used to process untrusted user inputs, this may be
  used as a vector for a denial of service attack.

  More info: https://pkg.go.dev/vuln/GO-2021-0113

  Module: golang.org/x/text
    Found in: golang.org/x/text@v0.3.5
    Fixed in: golang.org/x/text@v0.3.7

    Call stacks in your code:
      main.go:12:29: vuln.tutorial.main calls golang.org/x/text/language.Parse

=== Informational ===

Found 1 vulnerability in packages that you import, but there are no call
stacks leading to the use of this vulnerability. You may not need to
take any action. See https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck
for details.

Vulnerability #1: GO-2022-1059
  An attacker may cause a denial of service by crafting an
  Accept-Language header which ParseAcceptLanguage will take
  significant time to parse.
  More info: https://pkg.go.dev/vuln/GO-2022-1059
  Found in: golang.org/x/text@v0.3.5
  Fixed in: golang.org/x/text@v0.3.8
```

### Interpretando a saída

*Nota: se você não estiver usando a versão mais recente do Go, pode ver vulnerabilidades adicionais da standard library.*

O nosso código é afetado por uma vulnerabilidade, a [GO-2021-0113](https://pkg.go.dev/vuln/GO-2021-0113), porque ele chama diretamente a função `Parse` de `golang.org/x/text/language` numa versão vulnerável (v0.3.5).

Existe outra vulnerabilidade, a [GO-2022-1059](https://pkg.go.dev/vuln/GO-2022-1059), no module `golang.org/x/text` na v0.3.5. Mas ela é reportada como "Informational" (informativa), porque o nosso código nunca chama (direta ou indiretamente) nenhuma das funções vulneráveis dela.

Agora, vamos avaliar as vulnerabilidades e decidir o que fazer.

### Avalie as vulnerabilidades

a. Avalie as vulnerabilidades.

Primeiro, leia a descrição da vulnerabilidade e veja se ela realmente se aplica ao seu código e ao seu caso de uso. Se precisar de mais informações, acesse o link "More info".

Pela descrição, a vulnerabilidade GO-2021-0113 pode causar um panic quando o `Parse` é usado para processar entradas de usuários não confiáveis. Vamos supor que queremos que o nosso programa aguente entradas não confiáveis e que estamos preocupados com negação de serviço; então a vulnerabilidade provavelmente se aplica.

A GO-2022-1059 provavelmente não afeta o nosso código, porque ele não chama nenhuma função vulnerável desse relatório.

b. Decida o que fazer.

Para mitigar a GO-2021-0113, temos algumas opções:

- **Opção 1: atualizar para uma versão corrigida.** Se houver uma correção disponível, podemos eliminar a dependência vulnerável atualizando para uma versão corrigida do module.
- **Opção 2: parar de usar o(s) símbolo(s) vulnerável(is).** Podemos remover todas as chamadas à função vulnerável no nosso código. Precisaríamos encontrar uma alternativa ou implementá-la nós mesmos.

Neste caso, há uma correção disponível, e a função `Parse` é essencial para o nosso programa. Vamos atualizar a nossa dependência para a versão "fixed in" (corrigida em), a v0.3.7.

Decidimos não priorizar a correção da vulnerabilidade informativa, GO-2022-1059. Mas, como ela está no mesmo module que a GO-2021-0113 e a versão que a corrige é a v0.3.8, podemos eliminar as duas de uma vez facilmente, atualizando para a v0.3.8.

## Atualize as dependências vulneráveis

Felizmente, atualizar dependências vulneráveis é bem simples.

**Passo 8.** Atualize `golang.org/x/text` para a v0.3.8:

```
$ go get golang.org/x/text@v0.3.8
```

Você deve ver esta saída:

```
go: upgraded golang.org/x/text v0.3.5 => v0.3.8
```

(Repare que também poderíamos ter escolhido atualizar para `latest`, ou qualquer outra versão depois da v0.3.8.)

**Passo 9.** Agora rode o govulncheck de novo:

```
$ govulncheck ./...
```

Agora você vai ver esta saída:

```
govulncheck is an experimental tool. Share feedback at https://go.dev/s/govulncheck-feedback.

Using go1.20.3 and govulncheck@v0.0.0 with
vulnerability data from https://vuln.go.dev (last modified 2023-04-06 19:19:26 +0000 UTC).

Scanning your code and 46 packages across 1 dependent module for known vulnerabilities...
No vulnerabilities found.
```

Por fim, o govulncheck confirma que nenhuma vulnerabilidade foi encontrada.

Analisando as suas dependências regularmente com o comando govulncheck, você protege o seu código identificando, priorizando e tratando as vulnerabilidades.
