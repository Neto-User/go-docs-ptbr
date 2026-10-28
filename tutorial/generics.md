# Tutorial: Primeiros passos com generics

> Tradução não oficial de [go.dev/doc/tutorial/generics](https://go.dev/doc/tutorial/generics).

Este tutorial apresenta o básico de generics em Go. Com generics, você pode declarar e usar funções ou tipos escritos para funcionar com qualquer tipo de um conjunto de tipos fornecido pelo código que os chama.

Neste tutorial, você vai declarar duas funções simples não genéricas e depois juntar a mesma lógica em uma única função genérica.

Você vai passar pelas seguintes seções:

1. Criar uma pasta para o seu código.
2. Adicionar funções não genéricas.
3. Adicionar uma função genérica para lidar com vários tipos.
4. Remover os type arguments ao chamar a função genérica.
5. Declarar um type constraint.

> **Nota:** para outros tutoriais, veja o [índice de Tutoriais](README.md).

## Pré-requisitos

- **Go.** Recomendamos usar a versão mais recente do Go para seguir este tutorial. Para instruções de instalação, veja [Installing Go](https://go.dev/doc/install).
- **Uma ferramenta para editar o código.** Qualquer editor de texto serve.
- **Um terminal de comandos.** O Go funciona bem em qualquer terminal no Linux e no Mac, e no PowerShell ou cmd no Windows.

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

2. No terminal, crie uma pasta para o seu código chamada `generics`.

   ```
   $ mkdir generics
   $ cd generics
   ```

3. Crie um module para guardar o seu código.

   Rode o comando `go mod init`, passando o module path do seu novo código.

   ```
   $ go mod init example/generics
   go: creating new go.mod: module example/generics
   ```

   **Nota:** em código de produção, você especificaria um module path mais adequado às suas necessidades. Para saber mais, veja [Managing dependencies](https://go.dev/doc/modules/managing-dependencies).

A seguir, você vai adicionar um código simples para trabalhar com maps.

## Adicione funções não genéricas

Neste passo, você vai adicionar duas funções que somam os valores de um map e retornam o total.

Você está declarando duas funções em vez de uma porque está trabalhando com dois tipos diferentes de map: um que guarda valores `int64` e outro que guarda valores `float64`.

#### Escreva o código

1. No seu editor de texto, crie um arquivo chamado `main.go` na pasta `generics`. Você vai escrever o seu código Go nesse arquivo.

2. No topo de `main.go`, cole a seguinte declaração de package.

   ```go
   package main
   ```

   Um programa independente (em oposição a uma library) fica sempre no package `main`.

3. Abaixo da declaração de package, cole as duas declarações de função a seguir.

   ```go
   // SumInts adds together the values of m.
   func SumInts(m map[string]int64) int64 {
   	var s int64
   	for _, v := range m {
   		s += v
   	}
   	return s
   }

   // SumFloats adds together the values of m.
   func SumFloats(m map[string]float64) float64 {
   	var s float64
   	for _, v := range m {
   		s += v
   	}
   	return s
   }
   ```

   Neste código, você:

   - Declara duas funções que somam os valores de um map e retornam a soma.
     - `SumFloats` recebe um map de `string` para valores `float64`.
     - `SumInts` recebe um map de `string` para valores `int64`.

4. No topo de `main.go`, abaixo da declaração de package, cole a função `main` abaixo. Ela inicializa os dois maps e os usa como argumentos ao chamar as funções que você declarou no passo anterior.

   ```go
   func main() {
   	// Initialize a map for the integer values
   	ints := map[string]int64{
   		"first":  34,
   		"second": 12,
   	}

   	// Initialize a map for the float values
   	floats := map[string]float64{
   		"first":  35.98,
   		"second": 26.99,
   	}

   	fmt.Printf("Non-Generic Sums: %v and %v\n",
   		SumInts(ints),
   		SumFloats(floats))
   }
   ```

   Neste código, você:

   - Inicializa um map de valores `float64` e um map de valores `int64`, cada um com duas entradas.
   - Chama as duas funções que declarou antes para encontrar a soma dos valores de cada map.
   - Imprime o resultado.

5. Perto do topo de `main.go`, logo abaixo da declaração de package, importe o package de que você precisa para o código que acabou de escrever.

   As primeiras linhas do código devem ficar assim:

   ```go
   package main

   import "fmt"
   ```

6. Salve o `main.go`.

#### Rode o código

No terminal, na pasta que contém o `main.go`, rode o código.

```
$ go run .
Non-Generic Sums: 46 and 62.97
```

Com generics, você pode escrever uma função aqui em vez de duas. A seguir, você vai adicionar uma única função genérica para maps que contêm valores inteiros ou de ponto flutuante (float).

## Adicione uma função genérica para lidar com vários tipos

Nesta seção, você vai adicionar uma única função genérica que pode receber um map com valores inteiros ou float, substituindo as duas funções que você acabou de escrever por uma só.

Para aceitar valores de qualquer um dos dois tipos, essa função precisa de uma forma de declarar quais tipos suporta. Já o código que chama a função precisa de uma forma de dizer se está chamando com um map de inteiros ou de floats.

Para isso, você vai escrever uma função que declara *type parameters* (parâmetros de tipo), além dos parâmetros comuns da função. Esses type parameters tornam a função genérica, permitindo que ela trabalhe com argumentos de tipos diferentes. Você vai chamar a função com *type arguments* (argumentos de tipo) e argumentos comuns.

Cada type parameter tem um *type constraint* (restrição de tipo), que funciona como uma espécie de metatipo para o type parameter. Cada type constraint especifica quais type arguments o código que chama a função pode usar para aquele type parameter.

Embora o constraint de um type parameter normalmente represente um conjunto de tipos, em tempo de compilação o type parameter representa um único tipo: o tipo fornecido como type argument pelo código que chama. Se o tipo do type argument não for permitido pelo constraint do type parameter, o código não compila.

Lembre-se de que um type parameter precisa suportar todas as operações que o código genérico faz com ele. Por exemplo, se o código da sua função tentasse fazer operações de `string` (como indexação) com um type parameter cujo constraint inclui tipos numéricos, o código não compilaria.

No código que você vai escrever, vai usar um constraint que permite tipos inteiros ou float.

#### Escreva o código

1. Abaixo das duas funções que você adicionou antes, cole a seguinte função genérica.

   ```go
   // SumIntsOrFloats sums the values of map m. It supports both int64 and float64
   // as types for map values.
   func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
       var s V
       for _, v := range m {
           s += v
       }
       return s
   }
   ```

   Neste código, você:

   - Declara uma função `SumIntsOrFloats` com dois type parameters (dentro dos colchetes), `K` e `V`, e um argumento que usa esses type parameters: `m`, do tipo `map[K]V`. A função retorna um valor do tipo `V`.
   - Define para o type parameter `K` o type constraint `comparable`. Pensado justamente para casos assim, o constraint `comparable` já vem predeclarado no Go. Ele permite qualquer tipo cujos valores possam ser usados como operandos dos operadores de comparação `==` e `!=`. O Go exige que as keys de um map sejam comparáveis. Então declarar `K` como `comparable` é necessário para você poder usar `K` como key na variável de map. Isso também garante que o código que chama use um tipo permitido para keys de map.
   - Define para o type parameter `V` um constraint que é a união (union) de dois tipos: `int64` e `float64`. Usar `|` especifica uma união dos dois tipos, ou seja, o constraint permite qualquer um deles. O compilador vai aceitar qualquer um dos dois como argumento no código que chama.
   - Especifica que o argumento `m` é do tipo `map[K]V`, em que `K` e `V` são os tipos já definidos nos type parameters. Repare que sabemos que `map[K]V` é um tipo de map válido porque `K` é um tipo comparável. Se não tivéssemos declarado `K` como `comparable`, o compilador rejeitaria a referência a `map[K]V`.

2. Em `main.go`, abaixo do código que você já tem, cole o seguinte código.

   ```go
   fmt.Printf("Generic Sums: %v and %v\n",
   	SumIntsOrFloats[string, int64](ints),
   	SumIntsOrFloats[string, float64](floats))
   ```

   Neste código, você:

   - Chama a função genérica que acabou de declarar, passando cada um dos maps que criou.
   - Especifica os type arguments (os nomes de tipo entre colchetes) para deixar claro quais tipos devem substituir os type parameters na função que você está chamando.

     Como você vai ver na próxima seção, muitas vezes dá para omitir os type arguments na chamada. O Go frequentemente consegue inferi-los a partir do seu código.
   - Imprime as somas retornadas pela função.

#### Rode o código

No terminal, na pasta que contém o `main.go`, rode o código.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
```

Para rodar o seu código, em cada chamada o compilador substituiu os type parameters pelos tipos concretos especificados naquela chamada.

Ao chamar a função genérica que você escreveu, você especificou type arguments que disseram ao compilador quais tipos usar no lugar dos type parameters da função. Como vai ver na próxima seção, em muitos casos você pode omitir esses type arguments, porque o compilador consegue inferi-los.

## Remova os type arguments ao chamar a função genérica

Nesta seção, você vai adicionar uma versão modificada da chamada à função genérica, com uma pequena mudança para simplificar o código que chama. Você vai remover os type arguments, que não são necessários neste caso.

Você pode omitir os type arguments no código que chama quando o compilador do Go consegue inferir os tipos que você quer usar. O compilador infere os type arguments a partir dos tipos dos argumentos da função.

Repare que isso nem sempre é possível. Por exemplo, se você precisasse chamar uma função genérica sem argumentos, teria que incluir os type arguments na chamada.

#### Escreva o código

- Em `main.go`, abaixo do código que você já tem, cole o seguinte código.

  ```go
  fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
  	SumIntsOrFloats(ints),
  	SumIntsOrFloats(floats))
  ```

  Neste código, você:

  - Chama a função genérica omitindo os type arguments.

#### Rode o código

No terminal, na pasta que contém o `main.go`, rode o código.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
```

A seguir, você vai simplificar ainda mais a função, colocando a união de inteiros e floats em um type constraint que você pode reutilizar, por exemplo em outro código.

## Declare um type constraint

Nesta última seção, você vai mover o constraint que definiu antes para uma interface própria, para poder reutilizá-lo em vários lugares. Declarar constraints assim ajuda a deixar o código mais enxuto, por exemplo quando um constraint é mais complexo.

Você declara um *type constraint* como uma interface. O constraint permite qualquer tipo que implemente a interface. Por exemplo, se você declarar uma interface de type constraint com três métodos e usá-la com um type parameter numa função genérica, os type arguments usados para chamar a função precisam ter todos esses métodos.

Interfaces de constraint também podem se referir a tipos específicos, como você vai ver nesta seção.

#### Escreva o código

1. Logo acima de `main`, imediatamente depois dos imports, cole o código abaixo para declarar um type constraint.

   ```go
   type Number interface {
       int64 | float64
   }
   ```

   Neste código, você:

   - Declara o tipo de interface `Number` para usar como type constraint.
   - Declara uma união de `int64` e `float64` dentro da interface.

     Basicamente, você está movendo a união da declaração da função para um novo type constraint. Assim, quando quiser restringir um type parameter a `int64` ou `float64`, pode usar o type constraint `Number` em vez de escrever `int64 | float64`.

2. Abaixo das funções que você já tem, cole a seguinte função genérica `SumNumbers`.

   ```go
   // SumNumbers sums the values of map m. It supports both integers
   // and floats as map values.
   func SumNumbers[K comparable, V Number](m map[K]V) V {
       var s V
       for _, v := range m {
           s += v
       }
       return s
   }
   ```

   Neste código, você:

   - Declara uma função genérica com a mesma lógica da função genérica anterior, mas com o novo tipo de interface no lugar da união como type constraint. Como antes, você usa os type parameters para os tipos do argumento e do retorno.

3. Em `main.go`, abaixo do código que você já tem, cole o seguinte código.

   ```go
   fmt.Printf("Generic Sums with Constraint: %v and %v\n",
   	SumNumbers(ints),
   	SumNumbers(floats))
   ```

   Neste código, você:

   - Chama `SumNumbers` com cada map, imprimindo a soma dos valores de cada um.

     Como na seção anterior, você omite os type arguments (os nomes de tipo entre colchetes) nas chamadas à função genérica. O compilador do Go consegue inferir o type argument a partir dos outros argumentos.

#### Rode o código

No terminal, na pasta que contém o `main.go`, rode o código.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
Generic Sums with Constraint: 46 and 62.97
```

## Conclusão

Muito bem! Você acabou de conhecer os generics em Go.

Próximos tópicos sugeridos:

- O [Go Tour](https://go.dev/tour/) é uma ótima introdução passo a passo aos fundamentos do Go.
- Você encontra boas práticas úteis de Go descritas em [Effective Go](https://go.dev/doc/effective_go) e [How to write Go code](https://go.dev/doc/code).

## Código completo

Você pode rodar este programa no [Go playground](https://go.dev/play/p/apNmfVwogK0). No playground, basta clicar no botão **Run**.

```go
package main

import "fmt"

type Number interface {
	int64 | float64
}

func main() {
	// Initialize a map for the integer values
	ints := map[string]int64{
		"first": 34,
		"second": 12,
	}

	// Initialize a map for the float values
	floats := map[string]float64{
		"first": 35.98,
		"second": 26.99,
	}

	fmt.Printf("Non-Generic Sums: %v and %v\n",
		SumInts(ints),
		SumFloats(floats))

	fmt.Printf("Generic Sums: %v and %v\n",
		SumIntsOrFloats[string, int64](ints),
		SumIntsOrFloats[string, float64](floats))

	fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
		SumIntsOrFloats(ints),
		SumIntsOrFloats(floats))

	fmt.Printf("Generic Sums with Constraint: %v and %v\n",
		SumNumbers(ints),
		SumNumbers(floats))
}

// SumInts adds together the values of m.
func SumInts(m map[string]int64) int64 {
	var s int64
	for _, v := range m {
		s += v
	}
	return s
}

// SumFloats adds together the values of m.
func SumFloats(m map[string]float64) float64 {
	var s float64
	for _, v := range m {
		s += v
	}
	return s
}

// SumIntsOrFloats sums the values of map m. It supports both floats and integers
// as map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}

// SumNumbers sums the values of map m. It supports both integers
// and floats as map values.
func SumNumbers[K comparable, V Number](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}
```
