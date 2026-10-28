# Tutorial: Trabalhando com JSON

> Tradução não oficial de [go.dev/doc/tutorial/json](https://go.dev/doc/tutorial/json).

JSON (JavaScript Object Notation) é um formato simples de troca de dados. Na sintaxe, ele se parece com os objetos e listas do JavaScript. É muito usado na comunicação com serviços de API pela rede, mas também aparece em vários outros lugares. A página oficial, [json.org](http://json.org), traz uma definição do padrão maravilhosamente clara e concisa.

Com o [package encoding/json/v2](https://pkg.go.dev/encoding/json/v2), ler e escrever dados JSON nos seus programas Go é moleza. Esse package oferece uma API mais limpa e padrões (defaults) melhores que o package mais antigo [encoding/json](https://pkg.go.dev/encoding/json).

## Encoding (codificação)

Para codificar dados em JSON, usamos a função [`Marshal`](https://pkg.go.dev/encoding/json/v2#Marshal).

```go
func Marshal(in any, opts ...Options) (out []byte, err error)
```

Dada a estrutura de dados Go `Message`,

```go
type Message struct {
	Name string
	Body string
	Time time.Time
}
```

e uma instância de `Message`

```go
m := Message{"Alice", "Hello", time.Date(2011, 1, 25, 0, 0, 0, 0, time.UTC)}
```

podemos fazer o marshal de uma versão de `m` codificada em JSON usando `json.Marshal`:

```go
b, err := json.Marshal(m)
```

Se tudo der certo, `err` vai ser `nil` e `b` vai ser um `[]byte` contendo este JSON:

```go
b == []byte(`{"Name":"Alice","Body":"Hello","Time":"2011-01-25T00:00:00Z"}`)
```

Só estruturas de dados que podem ser representadas como JSON válido são codificadas:

- Pointers são codificados como os valores para os quais apontam (ou `null`, se o pointer for `nil`).
- Objetos JSON só aceitam strings como keys. Por isso, para codificar um map Go, ele precisa ter uma key que vire uma string JSON, como `map[string]T` (em que `T` é qualquer tipo Go suportado pelo package json).
- Tipos channel, complex e function não podem ser codificados.
- Estruturas de dados cíclicas não são suportadas.
- A documentação de [`Marshal`](https://pkg.go.dev/encoding/json/v2#Marshal) descreve o conjunto completo de regras de codificação.

O package json só acessa os campos exportados (exported fields) dos tipos struct, ou seja, os que começam com letra maiúscula. Portanto, só os campos exportados de uma struct aparecem na saída JSON.

## Decoding (decodificação)

Para decodificar dados JSON, usamos a função [`Unmarshal`](https://pkg.go.dev/encoding/json/v2#Unmarshal).

```go
func Unmarshal(in []byte, out any, opts ...Options) (err error)
```

Primeiro, precisamos criar um lugar onde os dados decodificados vão ficar

```go
var m Message
```

e chamar `json.Unmarshal`, passando um `[]byte` com os dados JSON e um pointer para `m`

```go
err := json.Unmarshal(b, &m)
```

Se `b` tiver um JSON válido que caiba em `m`, depois da chamada `err` vai ser `nil` e os dados de `b` vão estar guardados na struct `m`, como se fosse uma atribuição assim:

```go
m = Message{
	Name: "Alice",
	Body: "Hello",
	Time: time.Date(2011, 1, 25, 0, 0, 0, 0, time.UTC),
}
```

Como o `Unmarshal` identifica os campos onde guardar os dados decodificados? Para uma key JSON `"Foo"`, o `Unmarshal` procura entre os campos da struct de destino (nesta ordem de preferência):

- Um campo exportado com a tag `json:"Foo"` (veja a [especificação do Go](https://go.dev/ref/spec#Struct_types) para saber mais sobre struct tags), ou
- Um campo exportado chamado `Foo`.

O que acontece quando a estrutura dos dados JSON não bate exatamente com o tipo Go?

```go
b := []byte(`{"Name":"Bob","Food":"Pickle"}`)
var m Message
err := json.Unmarshal(b, &m)
```

O `Unmarshal` vai decodificar apenas os campos que encontrar no tipo de destino. Neste caso, só o campo `Name` de `m` vai ser preenchido, e o campo `Food` vai ser ignorado. Esse comportamento é bem útil quando você quer pegar só alguns campos específicos de um JSON grande. Também significa que campos não exportados da struct de destino não são afetados pelo `Unmarshal`.

Mas e se você não souber de antemão a estrutura dos seus dados JSON?

## JSON genérico com `any`

O tipo `any` descreve qualquer tipo Go. `any` é definido como `interface{}` (interface vazia), que descreve uma interface com zero métodos. Todo tipo Go implementa pelo menos zero métodos e, portanto, satisfaz a interface vazia.

`any` funciona como um tipo "recipiente" genérico:

```go
var a any
a = "a string"
a = 2011
a = 2.777
```

Uma type assertion (asserção de tipo) acessa o tipo concreto por trás:

```go
r := a.(float64)
fmt.Println("the circle's area", math.Pi*r*r)
```

Ou, se o tipo por trás for desconhecido, um type switch descobre o tipo:

```go
switch v := a.(type) {
case int:
    fmt.Println("twice a is", v*2)
case float64:
    fmt.Println("the reciprocal of a is", 1/v)
case string:
    h := len(v) / 2
    fmt.Println("a swapped by halves is", v[h:]+v[:h])
default:
    // a isn't one of the types above
}
```

O package json usa valores `map[string]any` e `[]any` para guardar objetos e arrays JSON arbitrários; ele faz o unmarshal de qualquer JSON válido num simples valor `any` sem problema. Os tipos Go concretos padrão são:

- `bool` para booleanos JSON,
- `float64` para números JSON,
- `string` para strings JSON, e
- `nil` para `null` JSON.

## Decodificando dados arbitrários

Considere estes dados JSON, guardados na variável `b`:

```go
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
```

Sem conhecer a estrutura desses dados, podemos decodificá-los num valor `any` com `Unmarshal`:

```go
var f any
err := json.Unmarshal(b, &f)
```

Nesse ponto, o valor Go em `f` seria um map cujas keys são strings e cujos valores também são guardados como valores `any`:

```go
f = map[string]any{
	"Name": "Wednesday",
	"Age":  6,
	"Parents": []any{
		"Gomez",
		"Morticia",
	},
}
```

Para acessar esses dados, podemos usar uma type assertion para acessar o `map[string]any` por trás de `f`:

```go
m := f.(map[string]any)
```

Depois, podemos percorrer o map com um `range` e usar um type switch para acessar os valores com seus tipos concretos:

```go
for k, v := range m {
	switch vv := v.(type) {
	case string:
		fmt.Println(k, "is string", vv)
	case float64:
		fmt.Println(k, "is float64", vv)
	case []any:
		fmt.Println(k, "is an array:")
		for i, u := range vv {
			fmt.Println(i, u)
		}
	default:
		fmt.Println(k, "is of a type I don't know how to handle")
	}
}
```

Assim, você consegue trabalhar com dados JSON desconhecidos e ainda aproveitar os benefícios da segurança de tipos (type safety).

## Tipos de referência (reference types)

Vamos definir um tipo Go para guardar os dados do exemplo anterior:

```go
type FamilyMember struct {
	Name    string
	Age     int
	Parents []string
}

var m FamilyMember
err := json.Unmarshal(b, &m)
```

Fazer o unmarshal desses dados num valor `FamilyMember` funciona como esperado, mas, olhando com atenção, dá para ver que aconteceu uma coisa notável. Com o `var`, alocamos uma struct `FamilyMember` e passamos um pointer para esse valor ao `Unmarshal`, mas naquele momento o campo `Parents` era uma slice `nil`. Para preencher o campo `Parents`, o `Unmarshal` alocou uma slice nova por baixo dos panos. Isso é típico de como o `Unmarshal` trabalha com os reference types suportados (pointers, slices e maps).

Considere fazer o unmarshal nesta estrutura de dados:

```go
type Foo struct {
	Bar *Bar
}
```

Se houvesse um campo `Bar` no objeto JSON, o `Unmarshal` alocaria um novo `Bar` e o preencheria. Se não houvesse, `Bar` ficaria como um pointer `nil`.

Daí surge um padrão útil: se você tem uma aplicação que recebe alguns tipos diferentes de mensagem, pode definir uma estrutura "receptora" assim:

```go
type IncomingMessage struct {
	Cmd *Command
	Msg *Message
}
```

e quem envia pode preencher o campo `Cmd` e/ou o campo `Msg` do objeto JSON principal, dependendo do tipo de mensagem que quer comunicar. O `Unmarshal`, ao decodificar o JSON numa struct `IncomingMessage`, só vai alocar as estruturas de dados presentes no JSON. Para saber quais mensagens processar, basta o programador testar se `Cmd` ou `Msg` não é `nil`.

## Marshal e Unmarshal em streaming

As interfaces [`io.Reader`](https://pkg.go.dev/io#Reader) e [`io.Writer`](https://pkg.go.dev/io#Writer) estão em todo lugar no Go e dão acesso em streaming a recursos como conexões HTTP, WebSockets ou arquivos. As funções `MarshalWrite` e `UnmarshalRead` permitem fazer marshal e unmarshal direto para e a partir desses streams, sem precisar de um `[]byte` intermediário com a mensagem inteira.

```go
func MarshalWrite(out io.Writer, in any, opts ...Options) (err error)
func UnmarshalRead(in io.Reader, out any, opts ...Options) (err error)
```

Por exemplo, podemos escrever a saída do marshal direto na saída padrão (standard out):

```go
err := json.MarshalWrite(os.Stdout, m)
```

## Marshal e Unmarshal personalizados

Às vezes, o comportamento padrão do marshal não faz sentido para o seu tipo.

Por exemplo, suponha que temos um tipo que descreve uma versão de software.

```go
type Version struct {
	Major, Minor, Patch int64
}

v := Version{1, 2, 3}
b, err := json.Marshal(s)
```

> **Nota da tradução:** no original está `json.Marshal(s)`, mas a variável declarada é `v`. Para rodar, use `json.Marshal(v)`.

Isso vira `{"Major":1,"Minor":2,"Patch":3}`, como esperado pela definição da struct, mas uma string de versão como "1.2.3" provavelmente seria uma representação melhor para quem consome o JSON.

Implementando a interface [`MarshalerTo`](https://pkg.go.dev/encoding/json/v2#MarshalerTo), podemos fornecer uma representação JSON personalizada. A [`UnmarshalerFrom`](https://pkg.go.dev/encoding/json/v2#UnmarshalerFrom) faz o unmarshal dessa representação personalizada.

```go
type MarshalerTo interface {
	MarshalJSONTo(*jsontext.Encoder) error
}

type UnmarshalerFrom interface {
	UnmarshalJSONFrom(*jsontext.Decoder) error
}
```

Podemos implementar esses métodos para converter de e para a representação em string.

```go
func (v Version) MarshalJSONTo(enc *jsontext.Encoder) error {
	return json.MarshalEncode(enc, fmt.Sprintf("%d.%d.%d", v.Major, v.Minor, v.Patch))
}
```

Com o marshal personalizado, o valor `Version{1, 2, 3}` agora vira `"1.2.3"`.

```go
func (v *Version) UnmarshalJSONFrom(dec *jsontext.Decoder) error {
	if k := dec.PeekKind(); k != jsontext.KindString {
		// Value must be a string.
		return &json.SemanticError{JSONKind: k}
	}

	var s string
	if err := json.UnmarshalDecode(dec, &s); err != nil {
		return err
	}

	_, err := fmt.Sscanf(s, "%d.%d.%d", &v.Major, &v.Minor, &v.Patch)
	return err
}
```

Com o unmarshal personalizado, `"1.2.3"` agora vira `Version{1, 2, 3}`. Este exemplo simples exige exatamente o formato `"major.minor.patch"`, mas um `UnmarshalJSONFrom` mais elaborado poderia ser mais flexível, por exemplo tornando opcionais as versões minor e patch.

## Referências

Para mais informações, veja a documentação dos packages [encoding/json/v2](https://pkg.go.dev/encoding/json/v2) e [encoding/json/jsontext](https://pkg.go.dev/encoding/json/jsontext).
<!-- nav -->

---

[← Workspaces multi-module](workspaces.md) · [Índice](README.md) · [Acessando um banco de dados relacional →](database-access.md)
