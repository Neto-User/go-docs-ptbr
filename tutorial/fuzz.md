# Tutorial: Primeiros passos com fuzzing

> Tradução não oficial de [go.dev/doc/tutorial/fuzz](https://go.dev/doc/tutorial/fuzz).

Este tutorial apresenta o básico de fuzzing em Go. Com fuzzing, dados aleatórios são usados contra o seu teste, na tentativa de encontrar vulnerabilidades ou entradas que causam crash. Alguns exemplos de vulnerabilidades que o fuzzing pode encontrar são SQL injection, buffer overflow, negação de serviço (denial of service) e ataques de cross-site scripting.

Neste tutorial, você vai escrever um fuzz test para uma função simples, rodar o comando `go` e depurar e corrigir problemas no código.

Para ajuda com a terminologia usada no tutorial, veja o [glossário de Go Fuzzing](https://go.dev/security/fuzz/#glossary).

Você vai passar pelas seguintes seções:

1. [Criar uma pasta para o seu código.](#crie-uma-pasta-para-o-seu-código)
2. [Adicionar código para testar.](#adicione-código-para-testar)
3. [Adicionar um teste unitário.](#adicione-um-teste-unitário)
4. [Adicionar um fuzz test.](#adicione-um-fuzz-test)
5. [Corrigir dois bugs.](#corrija-o-erro-de-string-inválida)
6. [Explorar recursos adicionais.](#conclusão)

> **Nota:** para outros tutoriais, veja o [índice de Tutoriais](README.md).

> **Nota:** atualmente, o fuzzing do Go suporta um subconjunto dos tipos embutidos, listados na [documentação de Go Fuzzing](https://go.dev/security/fuzz/#requirements). O suporte a mais tipos embutidos deve ser adicionado no futuro.

## Pré-requisitos

- **Go.** Recomendamos usar a versão mais recente do Go para seguir este tutorial. Para instruções de instalação, veja [Installing Go](https://go.dev/doc/install).
- **Uma ferramenta para editar o código.** Qualquer editor de texto serve.
- **Um terminal de comandos.** O Go funciona bem em qualquer terminal no Linux e no Mac, e no PowerShell ou cmd no Windows.
- **Um ambiente que suporte fuzzing.** Atualmente, o fuzzing do Go com instrumentação de cobertura (coverage) só está disponível nas arquiteturas AMD64 e ARM64.

## Crie uma pasta para o seu código

Para começar, crie uma pasta para o código que você vai escrever.

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

2. No terminal, crie uma pasta para o seu código chamada `fuzz`.

   ```
   $ mkdir fuzz
   $ cd fuzz
   ```

3. Crie um module para guardar o seu código.

   Rode o comando `go mod init`, passando o module path do seu novo código.

   ```
   $ go mod init example/fuzz
   go: creating new go.mod: module example/fuzz
   ```

   **Nota:** em código de produção, você especificaria um module path mais adequado às suas necessidades. Para mais, veja [Managing dependencies](https://go.dev/doc/modules/managing-dependencies).

A seguir, você vai adicionar um código simples para inverter uma string, que depois vamos submeter ao fuzzing.

## Adicione código para testar

Neste passo, você vai adicionar uma função para inverter uma string.

### Escreva o código

1. No seu editor de texto, crie um arquivo chamado `main.go` na pasta `fuzz`.

2. No topo de `main.go`, cole a seguinte declaração de package.

   ```go
   package main
   ```

   Um programa independente (em oposição a uma library) fica sempre no package `main`.

3. Abaixo da declaração de package, cole a seguinte declaração de função.

   ```go
   func Reverse(s string) string {
       b := []byte(s)
       for i, j := 0, len(b)-1; i < len(b)/2; i, j = i+1, j-1 {
           b[i], b[j] = b[j], b[i]
       }
       return string(b)
   }
   ```

   Esta função recebe uma `string`, percorre a string um `byte` de cada vez e retorna a string invertida no final.

   *Nota:* este código é baseado na função `stringutil.Reverse`, de golang.org/x/example.

4. No topo de `main.go`, abaixo da declaração de package, cole a função `main` abaixo. Ela inicializa uma string, inverte a string, imprime o resultado e repete.

   ```go
   func main() {
       input := "The quick brown fox jumped over the lazy dog"
       rev := Reverse(input)
       doubleRev := Reverse(rev)
       fmt.Printf("original: %q\n", input)
       fmt.Printf("reversed: %q\n", rev)
       fmt.Printf("reversed again: %q\n", doubleRev)
   }
   ```

   Esta função executa algumas operações de `Reverse` e depois imprime o resultado na linha de comando. Isso ajuda a ver o código funcionando e, possivelmente, a depurar.

5. A função `main` usa o package `fmt`, então você precisa importá-lo.

   As primeiras linhas do código devem ficar assim:

   ```go
   package main

   import "fmt"
   ```

### Rode o código

No terminal, na pasta que contém o `main.go`, rode o código.

```
$ go run .
original: "The quick brown fox jumped over the lazy dog"
reversed: "god yzal eht revo depmuj xof nworb kciuq ehT"
reversed again: "The quick brown fox jumped over the lazy dog"
```

Você vê a string original, o resultado de invertê-la e depois o resultado de invertê-la de novo, que é igual à original.

Agora que o código está rodando, é hora de testá-lo.

## Adicione um teste unitário

Neste passo, você vai escrever um teste unitário básico para a função `Reverse`.

### Escreva o código

1. No seu editor de texto, crie um arquivo chamado `reverse_test.go` na pasta `fuzz`.

2. Cole o código abaixo em `reverse_test.go`.

   ```go
   package main

   import (
       "testing"
   )

   func TestReverse(t *testing.T) {
       testcases := []struct {
           in, want string
       }{
           {"Hello, world", "dlrow ,olleH"},
           {" ", " "},
           {"!12345", "54321!"},
       }
       for _, tc := range testcases {
           rev := Reverse(tc.in)
           if rev != tc.want {
                   t.Errorf("Reverse: %q, want %q", rev, tc.want)
           }
       }
   }
   ```

   Este teste simples verifica que as strings de entrada listadas são invertidas corretamente.

### Rode o código

Rode o teste unitário com `go test`.

```
$ go test
PASS
ok      example/fuzz  0.013s
```

A seguir, você vai transformar o teste unitário em um fuzz test.

## Adicione um fuzz test

O teste unitário tem limitações: principalmente, cada entrada precisa ser adicionada ao teste pela pessoa desenvolvedora. Uma vantagem do fuzzing é que ele cria entradas para o seu código e pode encontrar casos extremos (edge cases) que os casos de teste que você pensou não alcançaram.

Nesta seção, você vai converter o teste unitário em um fuzz test, para gerar mais entradas com menos trabalho!

Repare que você pode manter testes unitários, benchmarks e fuzz tests no mesmo arquivo `*_test.go`, mas neste exemplo você vai converter o teste unitário em um fuzz test.

### Escreva o código

No seu editor de texto, substitua o teste unitário em `reverse_test.go` pelo fuzz test abaixo.

```go
func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)  // Use f.Add to provide a seed corpus
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev := Reverse(orig)
        doubleRev := Reverse(rev)
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```

O fuzzing também tem algumas limitações. No seu teste unitário, você conseguia prever a saída esperada da função `Reverse` e verificar se a saída real atendia a essa expectativa.

Por exemplo, no caso de teste `Reverse("Hello, world")`, o teste unitário especifica que o retorno é `"dlrow ,olleH"`.

No fuzzing, você não consegue prever a saída esperada, porque não tem controle sobre as entradas.

Mas existem algumas propriedades da função `Reverse` que você pode verificar em um fuzz test. As duas propriedades verificadas neste fuzz test são:

1. Inverter uma string duas vezes preserva o valor original.
2. A string invertida continua sendo UTF-8 válido.

Repare nas diferenças de sintaxe entre o teste unitário e o fuzz test:

- A função começa com `FuzzXxx` em vez de `TestXxx` e recebe `*testing.F` em vez de `*testing.T`.
- Onde você esperaria ver uma execução `t.Run`, você vê `f.Fuzz`, que recebe uma função fuzz target cujos parâmetros são `*testing.T` e os tipos que vão passar pelo fuzzing. As entradas do seu teste unitário são fornecidas como entradas do seed corpus (o conjunto inicial de entradas) usando `f.Add`.

Confirme que o novo package, `unicode/utf8`, foi importado.

```go
package main

import (
    "testing"
    "unicode/utf8"
)
```

Com o teste unitário convertido em fuzz test, é hora de rodar o teste de novo.

### Rode o código

1. Rode o fuzz test sem fuzzing, para garantir que as entradas do seed passam.

   ```
   $ go test
   PASS
   ok      example/fuzz  0.013s
   ```

   Você também pode rodar `go test -run=FuzzReverse` se tiver outros testes no arquivo e quiser rodar só o fuzz test.

2. Rode o `FuzzReverse` com fuzzing, para ver se alguma string gerada aleatoriamente causa uma falha. Isso é feito com `go test` e uma flag nova, `-fuzz`, com o parâmetro `Fuzz`. Copie o comando abaixo.

   ```
   $ go test -fuzz=Fuzz
   ```

   Outra flag útil é a `-fuzztime`, que limita o tempo de fuzzing. Por exemplo, usar `-fuzztime 10s` no teste abaixo significaria que, se nenhuma falha acontecesse antes, o teste terminaria por padrão depois de 10 segundos. Veja [esta seção](https://pkg.go.dev/cmd/go#hdr-Testing_flags) da documentação do `cmd/go` para conhecer outras flags de teste.

   Agora, rode o comando que você copiou.

   ```
   $ go test -fuzz=Fuzz
   fuzz: elapsed: 0s, gathering baseline coverage: 0/3 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 3/3 completed, now fuzzing with 8 workers
   fuzz: minimizing 38-byte failing input file...
   --- FAIL: FuzzReverse (0.01s)
       --- FAIL: FuzzReverse (0.00s)
           reverse_test.go:20: Reverse produced invalid UTF-8 string "\x9c\xdd"

       Failing input written to testdata/fuzz/FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a
       To re-run:
       go test -run=FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a
   FAIL
   exit status 1
   FAIL    example/fuzz  0.030s
   ```

   Aconteceu uma falha durante o fuzzing, e a entrada que causou o problema foi gravada em um arquivo do seed corpus, que vai ser executado na próxima vez que `go test` for chamado, mesmo sem a flag `-fuzz`. Para ver a entrada que causou a falha, abra no editor de texto o arquivo do corpus gravado na pasta `testdata/fuzz/FuzzReverse`. O seu arquivo pode ter uma string diferente, mas o formato vai ser o mesmo.

   ```
   go test fuzz v1
   string("泃")
   ```

   A primeira linha do arquivo do corpus indica a versão da codificação. Cada linha seguinte representa o valor de cada tipo que compõe a entrada do corpus. Como o fuzz target recebe só 1 entrada, há só 1 valor depois da versão.

3. Rode `go test` de novo, sem a flag `-fuzz`; a nova entrada do seed corpus que falha vai ser usada:

   ```
   $ go test
   --- FAIL: FuzzReverse (0.00s)
       --- FAIL: FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a (0.00s)
           reverse_test.go:20: Reverse produced invalid string
   FAIL
   exit status 1
   FAIL    example/fuzz  0.016s
   ```

   Como o nosso teste falhou, é hora de depurar.

## Corrija o erro de string inválida

Nesta seção, você vai depurar a falha e corrigir o bug.

Fique à vontade para pensar um pouco e tentar corrigir o problema sozinho antes de seguir.

### Diagnostique o erro

Existem algumas formas diferentes de depurar esse erro. Se você usa o VS Code como editor, pode [configurar o debugger](https://github.com/golang/vscode-go/blob/master/docs/debugging.md) para investigar.

Neste tutorial, vamos registrar (log) informações úteis de depuração no terminal.

Primeiro, veja a documentação de [`utf8.ValidString`](https://pkg.go.dev/unicode/utf8).

```
ValidString reports whether s consists entirely of valid UTF-8-encoded runes.
```

(Em português: "ValidString informa se `s` é formada inteiramente por runes válidas codificadas em UTF-8".)

A função `Reverse` atual inverte a string byte a byte, e aí está o nosso problema. Para preservar as runes codificadas em UTF-8 da string original, precisamos inverter a string rune a rune.

Para entender por que a entrada (neste caso, o caractere chinês `泃`) faz o `Reverse` produzir uma string inválida ao ser invertida, você pode inspecionar o número de runes da string invertida.

#### Escreva o código

No seu editor de texto, substitua o fuzz target dentro de `FuzzReverse` pelo código abaixo.

```go
f.Fuzz(func(t *testing.T, orig string) {
    rev := Reverse(orig)
    doubleRev := Reverse(rev)
    t.Logf("Number of runes: orig=%d, rev=%d, doubleRev=%d", utf8.RuneCountInString(orig), utf8.RuneCountInString(rev), utf8.RuneCountInString(doubleRev))
    if orig != doubleRev {
        t.Errorf("Before: %q, after: %q", orig, doubleRev)
    }
    if utf8.ValidString(orig) && !utf8.ValidString(rev) {
        t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
    }
})
```

Essa linha `t.Logf` vai imprimir na linha de comando se acontecer um erro, ou se o teste for executado com `-v`, o que pode ajudar a depurar esse problema específico.

#### Rode o código

Rode o teste com `go test`.

```
$ go test
--- FAIL: FuzzReverse (0.00s)
    --- FAIL: FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6252e87f69863046a5e0 (0.00s)
        reverse_test.go:16: Number of runes: orig=1, rev=3, doubleRev=1
        reverse_test.go:21: Reverse produced invalid UTF-8 string "\x83\xb3\xe6"
FAIL
exit status 1
FAIL    example/fuzz    0.598s
```

Todo o seed corpus usava strings em que cada caractere tinha um único byte. Mas caracteres como `泃` podem precisar de vários bytes. Por isso, inverter a string byte a byte invalida os caracteres com vários bytes.

**Nota:** se você tem curiosidade sobre como o Go lida com strings, leia o post [Strings, bytes, runes and characters in Go](https://go.dev/blog/strings) para entender melhor.

Entendendo melhor o bug, corrija o erro na função `Reverse`.

### Corrija o erro

Para corrigir a função `Reverse`, vamos percorrer a string por runes, em vez de por bytes.

#### Escreva o código

No seu editor de texto, substitua a função `Reverse()` existente pelo código abaixo.

```go
func Reverse(s string) string {
    r := []rune(s)
    for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}
```

A diferença principal é que o `Reverse` agora percorre cada `rune` da string, em vez de cada `byte`. Repare que isto é só um exemplo e não trata corretamente os [caracteres combinantes (combining characters)](https://en.wikipedia.org/wiki/Combining_character).

#### Rode o código

1. Rode o teste com `go test`.

   ```
   $ go test
   PASS
   ok      example/fuzz  0.016s
   ```

   Agora o teste passa!

2. Faça o fuzzing de novo com `go test -fuzz`, para ver se aparecem bugs novos.

   ```
   $ go test -fuzz=Fuzz
   fuzz: elapsed: 0s, gathering baseline coverage: 0/37 completed
   fuzz: minimizing 506-byte failing input file...
   fuzz: elapsed: 0s, gathering baseline coverage: 5/37 completed
   --- FAIL: FuzzReverse (0.02s)
       --- FAIL: FuzzReverse (0.00s)
           reverse_test.go:33: Before: "\x91", after: "�"

       Failing input written to testdata/fuzz/FuzzReverse/1ffc28f7538e29d79fce69fef20ce5ea72648529a9ca10bea392bcff28cd015c
       To re-run:
       go test -run=FuzzReverse/1ffc28f7538e29d79fce69fef20ce5ea72648529a9ca10bea392bcff28cd015c
   FAIL
   exit status 1
   FAIL    example/fuzz  0.032s
   ```

   Dá para ver que a string ficou diferente da original depois de ser invertida duas vezes. Desta vez, a própria entrada é um unicode inválido. Como isso é possível, se estamos fazendo fuzzing com strings?

   Vamos depurar de novo.

## Corrija o erro da inversão dupla

Nesta seção, você vai depurar a falha da inversão dupla e corrigir o bug.

Fique à vontade para pensar um pouco e tentar corrigir o problema sozinho antes de seguir.

### Diagnostique o erro

Como antes, existem várias formas de depurar essa falha. Neste caso, usar um [debugger](https://github.com/golang/vscode-go/blob/master/docs/debugging.md) seria uma ótima abordagem.

Neste tutorial, vamos registrar informações úteis de depuração dentro da função `Reverse`.

Olhe com atenção para a string invertida para encontrar o erro. Em Go, [uma string é uma slice de bytes somente leitura](https://go.dev/blog/strings) e pode conter bytes que não são UTF-8 válido. A string original é uma slice de bytes com um byte, `'\x91'`. Quando a string de entrada é convertida para `[]rune`, o Go codifica a slice de bytes em UTF-8 e substitui o byte pelo caractere UTF-8 `�`. Quando comparamos esse caractere UTF-8 de substituição com a slice de bytes da entrada, claramente eles não são iguais.

#### Escreva o código

1. No seu editor de texto, substitua a função `Reverse` pelo código abaixo.

   ```go
   func Reverse(s string) string {
       fmt.Printf("input: %q\n", s)
       r := []rune(s)
       fmt.Printf("runes: %q\n", r)
       for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
           r[i], r[j] = r[j], r[i]
       }
       return string(r)
   }
   ```

   Isso vai nos ajudar a entender o que está dando errado ao converter a string para uma slice de runes.

#### Rode o código

Desta vez, queremos rodar só o teste que falha, para inspecionar os logs. Para isso, vamos usar `go test -run`.

Para rodar uma entrada específica do corpus dentro de `FuzzXxx/testdata`, você pode passar `{NomeDoFuzzTest}/{nomeDoArquivo}` para `-run`. Isso ajuda na depuração. Neste caso, defina a flag `-run` igual ao hash exato do teste que falha. Copie e cole o hash único do seu terminal; ele vai ser diferente do mostrado abaixo.

```
$ go test -run=FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6252e87f69863046a5e0
input: "\x91"
runes: ['�']
input: "�"
runes: ['�']
--- FAIL: FuzzReverse (0.00s)
    --- FAIL: FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6252e87f69863046a5e0 (0.00s)
        reverse_test.go:16: Number of runes: orig=1, rev=1, doubleRev=1
        reverse_test.go:18: Before: "\x91", after: "�"
FAIL
exit status 1
FAIL    example/fuzz    0.145s
```

Sabendo que a entrada é um unicode inválido, vamos corrigir o erro na nossa função `Reverse`.

### Corrija o erro

Para corrigir esse problema, vamos retornar um erro se a entrada de `Reverse` não for UTF-8 válido.

#### Escreva o código

1. No seu editor de texto, substitua a função `Reverse` existente pelo código abaixo.

   ```go
   func Reverse(s string) (string, error) {
       if !utf8.ValidString(s) {
           return s, errors.New("input is not valid UTF-8")
       }
       r := []rune(s)
       for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
           r[i], r[j] = r[j], r[i]
       }
       return string(r), nil
   }
   ```

   Essa mudança faz a função retornar um erro se a string de entrada tiver caracteres que não são UTF-8 válido.

2. Como a função `Reverse` agora retorna um erro, altere a função `main` para lidar com o valor de erro extra. Substitua a função `main` existente pelo código abaixo.

   ```go
   func main() {
       input := "The quick brown fox jumped over the lazy dog"
       rev, revErr := Reverse(input)
       doubleRev, doubleRevErr := Reverse(rev)
       fmt.Printf("original: %q\n", input)
       fmt.Printf("reversed: %q, err: %v\n", rev, revErr)
       fmt.Printf("reversed again: %q, err: %v\n", doubleRev, doubleRevErr)
   }
   ```

   Essas chamadas a `Reverse` devem retornar um erro `nil`, já que a string de entrada é UTF-8 válido.

3. Você vai precisar importar os packages `errors` e `unicode/utf8`. O `import` em `main.go` deve ficar assim:

   ```go
   import (
       "errors"
       "fmt"
       "unicode/utf8"
   )
   ```

4. Altere o arquivo `reverse_test.go` para verificar erros e pular o teste (com `return`) quando houver erro.

   ```go
   func FuzzReverse(f *testing.F) {
       testcases := []string {"Hello, world", " ", "!12345"}
       for _, tc := range testcases {
           f.Add(tc)  // Use f.Add to provide a seed corpus
       }
       f.Fuzz(func(t *testing.T, orig string) {
           rev, err1 := Reverse(orig)
           if err1 != nil {
               return
           }
           doubleRev, err2 := Reverse(rev)
           if err2 != nil {
                return
           }
           if orig != doubleRev {
               t.Errorf("Before: %q, after: %q", orig, doubleRev)
           }
           if utf8.ValidString(orig) && !utf8.ValidString(rev) {
               t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
           }
       })
   }
   ```

   Em vez de usar `return`, você também pode chamar `t.Skip()` para parar a execução daquela entrada de fuzzing.

#### Rode o código

1. Rode o teste com `go test`.

   ```
   $ go test
   PASS
   ok      example/fuzz  0.019s
   ```

2. Faça o fuzzing com `go test -fuzz=Fuzz` e, depois de alguns segundos, pare com `ctrl-C`. O fuzz test roda até encontrar uma entrada que falhe, a menos que você passe a flag `-fuzztime`. O padrão é rodar para sempre se nenhuma falha acontecer, e o processo pode ser interrompido com `ctrl-C`.

   ```
   $ go test -fuzz=Fuzz
   fuzz: elapsed: 0s, gathering baseline coverage: 0/38 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 38/38 completed, now fuzzing with 4 workers
   fuzz: elapsed: 3s, execs: 86342 (28778/sec), new interesting: 2 (total: 35)
   fuzz: elapsed: 6s, execs: 193490 (35714/sec), new interesting: 4 (total: 37)
   fuzz: elapsed: 9s, execs: 304390 (36961/sec), new interesting: 4 (total: 37)
   ...
   fuzz: elapsed: 3m45s, execs: 7246222 (32357/sec), new interesting: 8 (total: 41)
   ^Cfuzz: elapsed: 3m48s, execs: 7335316 (31648/sec), new interesting: 8 (total: 41)
   PASS
   ok      example/fuzz  228.000s
   ```

3. Faça o fuzzing com `go test -fuzz=Fuzz -fuzztime 30s`, que roda por 30 segundos e termina se nenhuma falha for encontrada.

   ```
   $ go test -fuzz=Fuzz -fuzztime 30s
   fuzz: elapsed: 0s, gathering baseline coverage: 0/5 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 5/5 completed, now fuzzing with 4 workers
   fuzz: elapsed: 3s, execs: 80290 (26763/sec), new interesting: 12 (total: 12)
   fuzz: elapsed: 6s, execs: 210803 (43501/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 9s, execs: 292882 (27360/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 12s, execs: 371872 (26329/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 15s, execs: 517169 (48433/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 18s, execs: 663276 (48699/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 21s, execs: 771698 (36143/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 24s, execs: 924768 (50990/sec), new interesting: 16 (total: 16)
   fuzz: elapsed: 27s, execs: 1082025 (52427/sec), new interesting: 17 (total: 17)
   fuzz: elapsed: 30s, execs: 1172817 (30281/sec), new interesting: 17 (total: 17)
   fuzz: elapsed: 31s, execs: 1172817 (0/sec), new interesting: 17 (total: 17)
   PASS
   ok      example/fuzz  31.025s
   ```

   O fuzzing passou!

   Além da flag `-fuzz`, várias flags novas foram adicionadas ao `go test`, e você pode vê-las na [documentação](https://go.dev/security/fuzz/#custom-settings).

   Veja [Go Fuzzing](https://go.dev/security/fuzz/#command-line-output) para mais informações sobre os termos usados na saída do fuzzing. Por exemplo, "new interesting" se refere a entradas que ampliam a cobertura de código do corpus do fuzz test existente. É esperado que o número de entradas "new interesting" cresça bastante quando o fuzzing começa, dê alguns picos conforme novos caminhos no código são descobertos e depois diminua com o tempo.

## Conclusão

Muito bem! Você acabou de conhecer o fuzzing em Go.

O próximo passo é escolher uma função do seu código para fazer fuzzing e experimentar! Se o fuzzing encontrar um bug no seu código, considere adicioná-lo à [vitrine de troféus (trophy case)](https://go.dev/wiki/Fuzzing-trophy-case).

Se tiver algum problema ou ideia de recurso, [abra uma issue](https://go.dev/issue/new/?&labels=fuzz).

Para discussão e feedback geral sobre o recurso, você também pode participar do [canal #fuzzing](https://gophers.slack.com/archives/CH5KV1AKE) no Slack dos Gophers.

Confira a documentação em [go.dev/security/fuzz](https://go.dev/security/fuzz/#requirements) para ler mais.

## Código completo

`main.go`:

```go
package main

import (
    "errors"
    "fmt"
    "unicode/utf8"
)

func main() {
    input := "The quick brown fox jumped over the lazy dog"
    rev, revErr := Reverse(input)
    doubleRev, doubleRevErr := Reverse(rev)
    fmt.Printf("original: %q\n", input)
    fmt.Printf("reversed: %q, err: %v\n", rev, revErr)
    fmt.Printf("reversed again: %q, err: %v\n", doubleRev, doubleRevErr)
}

func Reverse(s string) (string, error) {
    if !utf8.ValidString(s) {
        return s, errors.New("input is not valid UTF-8")
    }
    r := []rune(s)
    for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r), nil
}
```

`reverse_test.go`:

```go
package main

import (
    "testing"
    "unicode/utf8"
)

func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc) // Use f.Add to provide a seed corpus
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev, err1 := Reverse(orig)
        if err1 != nil {
            return
        }
        doubleRev, err2 := Reverse(rev)
        if err2 != nil {
            return
        }
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```

[Voltar ao topo](#tutorial-primeiros-passos-com-fuzzing)
