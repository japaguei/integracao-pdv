# Manual de integrações a PDVs v1.0.0

## Informações da API:
- REST
- Envio e retorno por JSON.

## Como integrar seu PDV a JáPaguei

Basta implementar as duas funções descritas abaixo. Quando tiver concluído entrar em contato com parceria@japaguei.com para homologação e integração.

### Função 01: Detalhes do Pedido

#### Descrição:
Uma função que enviado o GUID do pedido, retorna todas informações relevantes e atualizadas daquele pedido.

A JáPaguei irá chamar essa função toda vez que um pedido de seu PDV for lido no celular de um cliente.

#### Parâmetros recebidos pela função:
- GUID do pedido, ex  (dd1e3e60-e0b1-4c7c-a90e-11287d60a000)

````JSON
{
  "order_id": "dd1e3e60-e0b1-4c7c-a90e-11287d60a000"
}
````

#### Retorno da função:
JSON com:
- Itens
- Informações do Pedido
- Informações de Pagamento
- Informações da Loja

JSON Básico:

````JSON
{
  "items": [{
    "soi_id": "guid do item do pedido",
    "description": "descrição do item",
    "price": "preço unitário",
    "ean_code": "código de barras",
    "code": "código do produto em seu erp",
    "unit_label": "unidade de medida",
    "qty": "quantidade", 
    "gross_price": "preço unitário + modificadores", 
    "final_price": "(preço unitário + modificadores) * quantidade - desconto-do-item; (não incluir desconto da conta)", 
    "modifiers": [{
      "soia_id": "guid do modificador do item",
      "extra_cost": "valor do modificador",
      "name": "descrição"
    }],
    "status": "active|canceled", 
  }],
  "sale_order": {
    "order_id": "guid do pedido",
    "currency": "BRL",
    "sub_total": "somatória dos final_price",
    "discount_price": "desconto dado no pedido, não tem relação com o desconto do item",
    "tips": "valor em real da gorjeta",
    "so_freight": "valor em real do frete",
    "so_total": "valor da compra (sub_total - discount_price + tips + so_freight)",
    "so_order_status": "active|canceled",
    "so_shop_id": "id do estabelecimento",
    "so_nfce_key": "chave da nfe ou nfce",
    "update_time": "data da ultima data de modificação (de qualquer item do pedido)",
  },
  "payment": [{
    "order_id": "guid do pedido",
    "payment_id": "guid do pagamento",
    "currency": "BRL",
    "amount": "valor",
    "status": "active|canceled",
    "gateway": "japaguei|other",
    "partner": "stone|cielo|rede|getnet|alelo|ticket|sodexo",
    "type": "credit|debit|wallet",
    "authcode": "cod de autorização",
    "update_time": "2018-04-24 14:21:32"
  }],
  "merchant": {
    "company_name": "razão social",
    "trade_name": "nome fantasia",
    "phone": "telefone",
    "cnpj": "cnpj se aplicavel",
    "cpf": "cpf se aplicavel",
    "street": "endereco",
    "number": "numero",
    "zip_code": "cep",
    "complementary": "complemento",
    "city": "cidade",
    "state": "estado (2 digitos)"
  }
}
````

Exemplo:

Imagine a seguinte compra:
- 1 - H20 (que custa R$ 6,00 cada). Este item, ganhou 20% de desconto
- 3 - Fanta Laranja (que custa R$ 5,00 cada).
- 1 - Coca-Cola (que custa R$ 5,00 cada), mas o cliente pediu limão e gelo (cobramos R$ 1,00 o limão).
- 1 - Guaraná (que custa R$ 5,00 cada), mas o cliente pediu laranja e gelo (cobramos R$ 2,00 a laranja). Este item ganhou 30% de desconto.
- 2 - Suco de Laranja (que custa R$ 6,00 cada).

Além disso:

- O cliente vai pagar 15% de serviço.
- O cliente vai ter 18% de desconto na conta toda.
- O cliente vai pagar R$ 8,00 de taxa de entrega.

````JSON
{
  "items": [{
    "soi_id": "dd1e3e60-e0b1-4c7c-a90e-11287d60a5f1",
    "description": "H20",
    "price": 6.00,
    "ean_code": "3401575645851",
    "code": "2013",
    "unit_label": "UNIDADE",
    "qty": 1.0,
    "gross_price": 6.00,
    "final_price": 4.80,
    "status": "active"
  }, {
    "soi_id": "03378a51-0e7c-432e-8132-c17af691b010",
    "description": "Fanta Laranja",
    "price": 5.00,
    "ean_code": "7894900039849",
    "code": "632",
    "unit_label": "UNIDADE",
    "qty": 3.0,
    "gross_price": "5.00",
    "final_price": "15.00",
    "status": "active"
  }, {
    "soi_id": "6605f0bb-55a9-446a-9a8b-debd1b103770",
    "description": "Coca-Cola",
    "price": 5.00,
    "ean_code": "7894900700046",
    "code": 24,
    "unit_label": "UNIDADE",
    "qty": 1.0,
    "gross_price": "6.00",
    "final_price": "6.00",
    "modifiers": [{
      "soia_id": "aff40913-4bcd-48f9-a0de-52796546d1c3",
      "extra_cost": 1.00,
      "name": "gelo e limão "
    }],
    "status": "active"
  }, {
    "soi_id": "d40a4174-90cb-4519-aa03-f82371ffb2e4",
    "description": "Guaraná",
    "price": 5.00,
    "ean_code": "7891991002189",
    "code": "456",
    "unit_label": "UNIDADE",
    "qty": 1.0,
    "gross_price": 7.00,
    "final_price": 4.90,
    "modifiers": [{
      "soia_id": "c55ed8f6-1dcc-4787-aca7-cf87897e9b78",
      "extra_cost": "2.00",
      "name": "gelo e laranja"
    }],
    "status": "active"
  }, {
    "soi_id": "50abbd08-e8b2-42c7-8e1b-7f384f920bf8",
    "description": "Suco de Laranja",
    "price": 6.00,
    "ean_code": null,
    "code": "321",
    "unit_label": "UNIDADE",
    "qty": 2.0,
    "gross_price": "6.00",
    "final_price": "12.00",
    "status": "active"
  }],
  "sale_order": {
    "order_id": "b9961a7d-08e3-41a6-aa13-849adcfd162d",
    "currency": "BRL",
    "sub_total": 42.7,
    "discount_price": 7.69,
    "tips": 5.25,
    "so_freight": 8.00,
    "so_total": 48.26,
    "so_order_status": "active",
    "so_shop_id": 13321,
    "so_nfce_key": "NFCe35...",
    "update_time": "2018-05-01 13:06:00"
  },
  "payment": [{
    "order_id": "b9961a7d-08e3-41a6-aa13-849adcfd162d",
    "payment_id": "03338499-6aac-4e48-9115-3069fc27bec8",
    "currency": "BRL",
    "amount": 8.00,
    "status": "refund",
    "gateway": "japaguei",
    "partner": "stone",
    "type": "credit",
    "authcode": "T98328",
    "date": "2018-04-24 14:21:32"
  }, {
    "order_id": "b9961a7d-08e3-41a6-aa13-849adcfd162d",
    "payment_id": "d7787268-9305-4177-af94-159b99ab2d5a",
    "currency": "BRL",
    "amount": 8.00,
    "status": "success",
    "gateway": "japaguei",
    "partner": "stone",
    "type": "credit",
    "authcode": "T98328",
    "update_time": "2018-04-24 14:21:32"
  }],
  "merchant": {
    "company_name": "Loja Teste Felipe",
    "trade_name": "Rango do Felipe",
    "phone": "1111111111",
    "cnpj": null,
    "cpf": "12345678987",
    "street": "Rua Teste",
    "number": "1",
    "zip_code": "02366001",
    "complementary": null,
    "city": "são paulo",
    "state": "SP"
  }
}
````

### Função 02: Realizar Pagamento

#### Descrição:
Uma função que receba o GUID do pedido, ID de pagamento (gerado pela JáPaguei), valor, gateway, adquirente, tipo de pagamento, código de autorização, data, recibo. E registra que foi feito um pagamento via JáPaguei.

A JáPaguei irá realizar a transação e chamar essa função informando seu PDV que o pagamento foi realizado em um pedido para que possa dar baixa.

#### Parâmetros recebidos pela função:
- GUID do pedido, ex  (dd1e3e60-e0b1-4c7c-a90e-11287d60a000)
- ID de pagamento (gerado pela JáPaguei), ex: (dd1e3e60-e0b1-4c7c-a90e-11287d60a001)
- Valor em reais que foi pago
- Gateway (japaguei ou outro)
- Nome do adquirente utilizado
- Tipo do pagamento (débito, crédito, wallet, ...)
- Código de autorização
- Data do pagamento
- Texto do recibo a ser impresso junto ao cumpom fiscal.


````JSON
{
  "order_id": "dd1e3e60-e0b1-4c7c-a90e-11287d60a000",
  "payment_id": "dd1e3e60-e0b1-4c7c-a90e-11287d60a001",
  "currency": "BRL",
  "value": 20.36,
  "gateway": "japaguei",
  "partner": "stone|cielo|rede|getnet|alelo|ticket|sodexo",
  "type": "credito|debito|wallet",
  "authcode": "T98328",
  "date": "2018-04-24 14:21:32",
  "receipt": "JAPAGUEI\nLoja Do Felipe\n\nCompra a credito: R$ 20.36\nStone: T98328\nData: 24/04/2018 - 14:21:32"
}
````

#### Retorno da função:


````JSON
{
  "payment_status": true|false
}
````
