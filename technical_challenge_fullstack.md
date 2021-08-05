# Desafio técnico (desenvolvedor)

## Code Challenge: Validador de frete

Você deverá implementar uma aplicação em **Elixir** que valida métodos de frete para um CEP seguindo uma série de regras predefinidas.
Por favor leia as instruções abaixo e sinta-se à vontade para fazer perguntas caso ache necessário. 

## Preparando seu desafio para envio

Sua solução deve conter um arquivo de README com:

- Uma explicação sobre as decisões técnicas e arquiteturais do seu desafio;
- Uma justificativa para o uso de frameworks ou bibliotecas (caso sejam usadas);
- Instruções sobre como compilar e executar o projeto;
- Notas adicionais que você considere importantes para a avaliação.

Você deve entregar o código fonte de sua solução para nós em um arquivo comprimido (`zip`) ou um repositório público (`Github/Gitlab/Bitbucket`) contendo o código e toda a documentação possível em resposta ao email que foi enviado o desafio. Por favor não incluir arquivos desnecessários como binários, compilados, biblioteca, etc;

## Validador de Frete

### Como o programa deve funcionar?

Seu programa deve receber 3 entradas (`stdin`)

1. Um arquivo em formato json contendo uma lista de métodos de frete que devem ser considerados
2. Um CEP alvo que deve ser considerado para  a classificação dos métodos de envio
3. O preço do pedido, em centavos, que deve ser considerado para a classificação dos métodos de envio

### Arquivo de entrada

O arquivo de entrada contem uma lista de métodos com os seguintes campos

| Campo                | Tipo         | Descrição                                                                    |
|----------------------|--------------|-------------------------------------------------------------------------------|
| name                 | string       | Descrição do método de frete que é exibida para o usário                      |
| active               | boolean      | "true" caso o método seja ativo                                               |
| min_price_in_cents   | integer      | indica o preço minimo do pedido (em centavos) para que o método seja elegível |
| range_postcode_valid | list[string] | faixa de cep a qual o método deve ser elegível                                |

### Exemplo de arquivo de entrada

```json
[
	{"name": "Entrega normal SP", "active": true, "min_price_in_cents": 1, "range_postcode_valid": ["01000", "19999"]},
	{"name": "Entrega normal SP (frete grátis)", "active": true, "min_price_in_cents": 10000, "range_postcode_valid": ["01000", "19999"]},
	{"name": "Entrega normal RJ", "active": true, "min_price_in_cents": 4500, "range_postcode_valid": ["20000", "26600"]},
	{"name": "Entrega normal RJ (expressa)", "active": true, "min_price_in_cents": 29900, "range_postcode_valid": ["20000", "26600"]},
	{"name": "Entrega normal BR", "active": true, "min_price_in_cents": 5000, "range_postcode_valid": ["00000", "99999"]},
	{"name": "Entrega normal agendada SP", "active": true, "min_price_in_cents": 10000, "range_postcode_valid": ["01000", "19999"]},
	{"name": "Retirada em loja", "active": true, "min_price_in_cents": 1, "range_postcode_valid": ["0000000", "999999"]},
	{"name": "Retirada em loja agendada", "active": false, "min_price_in_cents": 1, "range_postcode_valid": ["0000000", "999999"]}
]
```

### Arquivo de saída

Considerando os dados de entrada o programa deve realizar a classificação dos métodos de frete, aplicando as regras de validações descritas abaixo, e deve gerar um arquivo de saída no formata json com uma lista de classificações com os seguintes campos

| Campo             | Tipo         | Descrição                                                                                                                                                            |
|-------------------|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name              | string       | Descrição do método de frete que é exibida para o usário, vinda do arquivo de entrada                                                                                |
| valid             | boolean      | indica se o método correpondente a linha é considerado válido perante as regras de validações                                                                        |
| incompatibilities | list[string] | lista de incompatibilidades do método referente aos dados de entrada (CEP e Preço)lista de incompatibilidades do método referente aos dados de entrada (CEP e Preço) |

### Exemplo de arquivo de saída

```json
[
  {"method": "Entrega normal SP", "valid": true, "incompatibilities": []},
  {"method": "Entrega normal SP (frete grátis)", "valid": false, "incompatibilities": ["Minimum price not reached for this method"]},
  {"method": "Entrega normal agendada SP", "valid": false, "incompatibilities": ["Minimum price not reached for this method"]},
  {"method": "Entrega normal RJ", "valid": false, "incompatibilities": ["Zip code outside the delivery area for this method"]},
  {"method": "Entrega normal RJ (expressa)", "valid": false, "incompatibilities": ["Zip code outside the delivery area for this method", "Minimum price not reached for this method"]},
  {"method": "Entrega normal BR", "valid": false, "incompatibilities": ["Minimum price not reached for this method"]},
  {"method": "Retira em loja", "valid": true, "incompatibilities": []},
  {"method": "Retira em loja agendada", "valid": false, "incompatibilities": ["Disabled shipping"]}
]
```

## Regras de validações

### Método não compatível (`Disabled shipping`)

Quando o método possuir uma propriedade `active` com valor `false` ele sempre deverá retornar a incompatibilidade `Disabled shipping`

Exemplo:

```json
# Input
CEP: 03108010
Preço do pedido: 3000

{"name": "Retirada em loja agendada", "active": false, "min_price_in_cents": 1, "range_postcode_valid": ["0000000", "999999"]}

# Output
...
{"method": "Retira em loja agendada", "valid": false, "incompatibilities": ["Disabled shipping"]}
```

---

### Valor não compatível com o preço mínimo (`Minimum price not reached for this method`)

Quando o valor de entrada referente ao preço do pedido (em centavos) for menor do que o valor configurado pela propriedade `min_price_in_cents` deverá ser retornado a incompatibilidade `Minimum price not reached for this method`

Exemplo:

```json
# Input
CEP: 03108010
Preço do pedido: 3000

{"name": "Entrega normal SP (frete grátis)", "active": true, "min_price_in_cents": 10000, 
"range_postcode_valid": ["01000", "19999"]}

# Output
...
{"method": "Entrega normal SP (frete grátis)", "valid": false, 
"incompatibilities": ["Minimum price not reached for this method"]}
```

---

### CEP fora do intervalo do método de frete (`Zip code outside the delivery area for this method`)

Quando o CEP do parâmetro de entrada do programa não estiver contido dentro da faixa de CEP configurada no método na propriedade `range_postcode_valid`

Exemplo:

```json
# Input
CEP: 03108010
Preço do pedido: 4800

{"name": "Entrega normal RJ", "active": true, "min_price_in_cents": 4500, "range_postcode_valid": ["20000", "26600"]}

# Output
...
{"method": "Entrega normal RJ", "valid": false, "incompatibilities": ["Zip code outside the delivery area for this method"]}
```

---

### Método com mais de uma incompatibilidade

Quando mais de uma regra for quebrada para o mesmo método, todas as incompatibilidades precisam ser retornadas.

Exemplo:

```json
# Input
CEP: 03108010
Preço do pedido: 3000

{"name": "Entrega normal RJ", "active": false, "min_price_in_cents": 4500, "range_postcode_valid": ["20000", "26600"]}

# Output
...
{"method": "Entrega normal RJ", "valid": false, "incompatibilities": ["Disabled shipping",
"Minimum price not reached for this method", "Zip code outside the delivery area for this method"]}
```

## Nossas expectativas

Valorizamos as seguintes qualidades:

- Código legível e manutenível;
- Estrutura de código e abstrações para que seja simples de criar novas validações;
- Testes de unidade e integração de qualidade;
- Documentação onde for necessário;
- Instruções sobre como executar o código;

## Notas gerais

---

- Este código poderá ser extendido por você e por outras pessoas do desenvolvimento da Obramax durante outra etapa de processo;
- O validador deve receber as operações através da entrada padrão (`stdin`) e retornar o resultado da validação através da saída padrão (`stdout`), não em uma API REST;
- Uma solução incompleta não é critério de desclassificação, porém quanto mais regras implementar mais conseguiremos analisar sua capacidade de desenvolvimento.
