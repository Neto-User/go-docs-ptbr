# Como contribuir

Obrigado por querer ajudar! Este projeto traduz a documentação oficial do Go ([go.dev](https://go.dev)) para português do Brasil, para facilitar a vida de quem está começando.

Toda ajuda conta: corrigir um erro de digitação, melhorar uma frase, revisar um tutorial inteiro ou traduzir uma página nova.

## Jeitos de ajudar

- **Revisar:** leia um tutorial e aponte (ou corrija) frases estranhas, erros ou termos inconsistentes.
- **Corrigir:** achou um erro? Abra uma issue usando o modelo "Erro de tradução" ou mande um PR direto.
- **Traduzir:** escolha uma página que ainda não foi traduzida (veja as issues com a label `help wanted`), comente na issue que vai pegar e mande o PR.
- **Atualizar:** o original muda de vez em quando. Se notar que uma página daqui ficou desatualizada, avise numa issue.

## Regras da tradução

1. **Termos técnicos ficam em inglês.** `package`, `module`, `slice`, `struct`, `map`, `pointer`, `interface`, `goroutine`, `channel`, `type parameter`, `handler`, `endpoint`... São os nomes que aparecem no código e nos erros do compilador. Na primeira vez que um termo aparecer, pode explicar entre parênteses: "type constraint (restrição de tipo)".
2. **Código não se traduz.** Blocos de código, comandos, saídas de terminal e comentários dentro do código ficam exatamente como no original.
3. **Português do Brasil, tom direto.** Trate o leitor por "você". Frases curtas, sem formalidade desnecessária.
4. **Fidelidade ao original.** Traduza o sentido, não palavra por palavra, mas sem acrescentar nem tirar conteúdo.
5. **Erros do original** viram uma nota marcada assim: `> **Nota da tradução:** ...`
6. **Links:** links para outras páginas já traduzidas apontam para o arquivo local (`create-module.md`); o resto aponta para `https://go.dev/...`.
7. **Cabeçalho:** toda página começa com o título e a linha `> Tradução não oficial de [go.dev/...](link do original).`

## Glossário rápido

| Inglês | Como usar aqui |
| --- | --- |
| package, module, slice, map, struct | mantém em inglês |
| caller | caller (o código que chama a função) |
| exported name | exported name (nome exportado) |
| dependency tracking | dependency tracking (rastreamento de dependências) |
| seed corpus | seed corpus (conjunto inicial de entradas) |
| standard library | standard library (biblioteca padrão) |
| build, deploy, release | mantém em inglês |

## Passo a passo do pull request

1. Faça um fork do repositório.
2. Crie uma branch: `git checkout -b traduz-effective-go` (ou `corrige-generics`, etc.).
3. Faça as mudanças seguindo as regras acima.
4. Confira o resultado no preview do GitHub ou do VS Code (`Ctrl+Shift+V`).
5. Abra o PR explicando o que mudou.

## Site

O site em GitHub Pages é gerado automaticamente a partir da branch `main`. Um detalhe: o Jekyll interpreta `{{ }}` e `{% %}` como template. Se um código tiver isso, escreva o bloco em HTML com entidades (`&#123;&#123;`), como em `tutorial/compile-install.md`.
