---
name: cardapio-interativo
description: Orienta a análise, planejamento, desenvolvimento, revisão e evolução do projeto de cardápio interativo de uma lanchonete. Use esta skill sempre que a tarefa envolver cardápio, produtos, categorias, carrinho, checkout, delivery, retirada, consumo antecipado no local, cálculo de valores, localStorage ou envio de pedidos para o WhatsApp.
---

# Skill — Cardápio Interativo

## 1. Papel do Claude Code

Você está trabalhando no desenvolvimento de um cardápio interativo para uma lanchonete.

Sua responsabilidade é:

- compreender o estado atual do código antes de realizar alterações;
- respeitar rigorosamente o escopo do MVP;
- desenvolver funcionalidades de maneira incremental;
- priorizar a experiência mobile;
- evitar complexidade desnecessária;
- manter o código organizado, tipado e reutilizável;
- não adicionar funcionalidades fora do escopo sem solicitação explícita;
- explicar decisões técnicas importantes;
- validar o fluxo completo após cada mudança relevante.

Antes de modificar arquivos:

1. Leia a estrutura do projeto.
2. Identifique as tecnologias instaladas.
3. Localize componentes, tipos, dados e configurações existentes.
4. Reutilize padrões já adotados.
5. Evite recriar funcionalidades que já existem.
6. Informe resumidamente o que será alterado.

Não substitua toda a arquitetura quando uma alteração localizada resolver o problema.

---

## 2. Visão geral do projeto

O projeto é um cardápio digital responsivo para uma lanchonete.

O cliente poderá:

1. Acessar o cardápio pelo celular.
2. Visualizar categorias.
3. Visualizar os produtos.
4. Abrir os detalhes de um produto.
5. Escolher quantidade.
6. Selecionar adicionais, quando disponíveis.
7. Escrever observações.
8. Adicionar produtos ao carrinho.
9. Revisar o pedido.
10. Escolher uma modalidade de atendimento.
11. Preencher seus dados.
12. Escolher a forma de pagamento.
13. Visualizar o resumo e o valor total.
14. Abrir o WhatsApp com a mensagem do pedido pronta.
15. Enviar manualmente a mensagem para a lanchonete.

O site monta o pedido, mas não confirma sua aceitação.

A confirmação será feita pela lanchonete diretamente no WhatsApp.

---

## 3. Objetivo principal

Criar um MVP simples, funcional e fácil de utilizar que permita ao cliente montar um pedido e enviá-lo ao WhatsApp da lanchonete.

A prioridade é validar o fluxo:

```text
Cardápio
→ Produto
→ Carrinho
→ Modalidade de atendimento
→ Dados do cliente
→ Pagamento
→ Resumo
→ WhatsApp
```

O MVP deve ser rápido, intuitivo e funcional principalmente em dispositivos móveis.

---

## 4. Escopo do MVP

O MVP deve conter:

- cardápio público;
- categorias;
- listagem de produtos;
- detalhes dos produtos;
- imagens, nomes, descrições e preços;
- disponibilidade de produtos;
- quantidade;
- adicionais simples;
- observação por produto;
- carrinho;
- persistência do carrinho no `localStorage`;
- cálculo de subtotal;
- seleção da modalidade de atendimento;
- formulário condicional para cada modalidade;
- seleção da forma de pagamento;
- cálculo da taxa de delivery;
- resumo completo do pedido;
- geração da mensagem;
- redirecionamento para o WhatsApp;
- layout responsivo;
- configurações da lanchonete armazenadas no código;
- produtos armazenados localmente no código.

---

## 5. Funcionalidades fora do MVP

Não implemente, salvo quando solicitado explicitamente:

- backend;
- API própria;
- banco de dados;
- painel administrativo;
- login;
- cadastro de clientes;
- histórico de pedidos;
- acompanhamento de status;
- confirmação automática do pedido;
- pagamento online;
- integração com gateway de pagamento;
- controle de estoque;
- controle de cozinha;
- gestão de entregadores;
- rastreamento de entrega;
- relatórios;
- cupons complexos;
- sistema de fidelidade;
- múltiplas lojas;
- notificações automáticas;
- integração oficial com a API do WhatsApp;
- reserva automática de mesas.

Não prepare estruturas excessivamente complexas para funcionalidades futuras que ainda não fazem parte do MVP.

---

## 6. Modalidades de atendimento

O cliente poderá escolher exatamente uma destas modalidades:

```ts
type OrderType = "delivery" | "pickup" | "dine-in"
```

Os nomes exibidos ao usuário devem ser:

- Delivery;
- Retirada no local;
- Comer no local.

A opção **Comer no local** representa um pedido antecipado feito antes de o cliente chegar à lanchonete.

Não representa pedido feito em uma mesa.

---

## 7. Modalidade: Delivery

### 7.1 Funcionamento

O cliente monta o pedido e solicita a entrega em seu endereço.

### 7.2 Dados obrigatórios

Solicitar:

- nome;
- telefone;
- rua;
- número;
- bairro;
- forma de pagamento.

Dados opcionais:

- complemento;
- ponto de referência;
- observação geral.

### 7.3 Taxa de entrega

A taxa será definida pela lanchonete no código.

O MVP poderá trabalhar com:

- taxa fixa; ou
- taxa diferente por bairro.

Priorize taxa por bairro quando houver uma lista previamente definida pela lanchonete.

Exemplo:

```ts
const deliveryFees = {
  Cohama: 7,
  Centro: 6,
  Renascença: 8,
  Turu: 10
}
```

O cliente não deve digitar manualmente o valor da taxa.

Ele seleciona o bairro, e o sistema localiza o valor correspondente.

### 7.4 Cálculo

```ts
const deliveryFee = deliveryFees[selectedNeighborhood]
const total = subtotal + deliveryFee
```

A taxa só pode ser adicionada quando:

```ts
orderType === "delivery"
```

Para retirada ou consumo no local:

```ts
deliveryFee = 0
```

### 7.5 Validações

Não permitir finalizar quando:

- a rua estiver vazia;
- o número estiver vazio;
- nenhum bairro tiver sido selecionado;
- o bairro não possuir taxa cadastrada;
- o telefone estiver inválido;
- o pedido não alcançar o valor mínimo, caso exista.

---

## 8. Modalidade: Retirada no local

### 8.1 Funcionamento

O cliente faz o pedido pelo site e vai buscá-lo na lanchonete.

### 8.2 Dados obrigatórios

Solicitar:

- nome;
- telefone;
- horário desejado para retirada;
- forma de pagamento.

Dados opcionais:

- observação geral.

### 8.3 Regras

- Não solicitar endereço.
- Não calcular taxa de entrega.
- Não afirmar que o horário está confirmado.
- A lanchonete deverá confirmar o horário pelo WhatsApp.

Exibir um aviso semelhante a:

```text
O horário de retirada será confirmado pela lanchonete no WhatsApp.
```

### 8.4 Cálculo

```ts
const deliveryFee = 0
const total = subtotal
```

---

## 9. Modalidade: Comer no local

### 9.1 Funcionamento

O cliente faz o pedido antes de chegar e pretende consumir os produtos no estabelecimento.

Não é um pedido feito na mesa.

Não solicitar número de mesa.

### 9.2 Dados obrigatórios

Solicitar:

- nome;
- telefone;
- quantidade de pessoas;
- horário previsto de chegada;
- forma de pagamento.

Dados opcionais:

- observação geral.

### 9.3 Regras

- Não calcular taxa de entrega.
- Não afirmar que existe uma mesa reservada.
- Não tratar o pedido como uma reserva confirmada.
- A disponibilidade de mesa deverá ser confirmada pelo WhatsApp.

Exibir um aviso semelhante a:

```text
O pedido e a disponibilidade de mesa serão confirmados pela lanchonete no WhatsApp.
```

### 9.4 Cálculo

```ts
const deliveryFee = 0
const total = subtotal
```

---

## 10. Formas de pagamento

O MVP poderá disponibilizar:

```ts
type PaymentMethod =
  | "pix"
  | "cash"
  | "debit-card"
  | "credit-card"
```

Nomes exibidos:

- Pix;
- Dinheiro;
- Cartão de débito;
- Cartão de crédito.

O site apenas registra a preferência do cliente.

Não realizar pagamento dentro do site.

### Dinheiro

Quando o cliente selecionar dinheiro, perguntar:

```text
Precisa de troco?
```

Caso a resposta seja sim, solicitar:

```text
Troco para quanto?
```

Validar que o valor informado seja maior que o total do pedido.

---

## 11. Estrutura do cardápio

A página principal deverá conter, quando aplicável:

- cabeçalho da lanchonete;
- logo;
- nome da lanchonete;
- status de funcionamento;
- horário;
- tempo estimado;
- aviso importante;
- barra de pesquisa;
- categorias;
- produtos;
- botão flutuante ou fixo do carrinho.

Exemplo de categorias:

```text
Destaques
Hambúrgueres
Combos
Porções
Bebidas
Sobremesas
```

No celular, as categorias devem possuir rolagem horizontal.

---

## 12. Produto

Cada produto poderá possuir:

```ts
interface Product {
  id: string
  name: string
  description: string
  price: number
  image: string
  categoryId: string
  available: boolean
  featured?: boolean
  additions?: Addition[]
}
```

Exemplo:

```ts
{
  id: "x-bacon",
  name: "X-Bacon",
  description: "Pão, carne, queijo, bacon e salada",
  price: 24.9,
  image: "/images/x-bacon.webp",
  categoryId: "hamburgers",
  available: true,
  featured: true,
  additions: [
    {
      id: "extra-bacon",
      name: "Bacon extra",
      price: 4
    }
  ]
}
```

Produtos indisponíveis podem continuar visíveis, mas:

- devem possuir indicação visual;
- não podem ser adicionados ao carrinho;
- o botão deve estar desativado.

Exemplo:

```text
Indisponível no momento
```

---

## 13. Adicionais

Estrutura sugerida:

```ts
interface Addition {
  id: string
  name: string
  price: number
}
```

Exemplo:

```ts
{
  id: "extra-cheese",
  name: "Queijo extra",
  price: 3
}
```

O valor unitário configurado de um item deve considerar:

```text
Preço base do produto + total dos adicionais selecionados
```

O valor total do item deve considerar:

```text
Valor unitário configurado × quantidade
```

Evite criar regras complexas de grupos, mínimos, máximos e dependências no primeiro MVP, salvo se forem requisitos explícitos.

---

## 14. Detalhes do produto

Ao abrir um produto, mostrar:

- imagem;
- nome;
- descrição;
- preço base;
- adicionais disponíveis;
- controle de quantidade;
- campo de observação;
- total atualizado;
- botão de adicionar ao carrinho.

Exemplo:

```text
X-Bacon
R$ 24,90

Adicionais:
[ ] Bacon extra — R$ 4,00
[ ] Queijo extra — R$ 3,00

Quantidade:
[-] 1 [+]

Observação:
[ Sem cebola ]

Total: R$ 28,90

[ Adicionar ao carrinho ]
```

A observação não deve alterar o preço.

---

## 15. Carrinho

O carrinho deverá permitir:

- visualizar os itens;
- alterar a quantidade;
- remover um item;
- visualizar adicionais;
- visualizar observações;
- visualizar valor unitário;
- visualizar valor total do item;
- visualizar subtotal;
- continuar comprando;
- avançar para o checkout.

Estrutura sugerida:

```ts
interface CartItem {
  cartItemId: string
  productId: string
  name: string
  basePrice: number
  unitPrice: number
  quantity: number
  image?: string
  additions: Addition[]
  observation?: string
}
```

`cartItemId` deve identificar a configuração específica adicionada ao carrinho.

Dois produtos iguais com adicionais ou observações diferentes devem permanecer como itens separados.

Exemplo:

```text
1x X-Bacon sem cebola
1x X-Bacon com bacon extra
```

Não agrupar esses itens automaticamente.

---

## 16. Cálculos do carrinho

### Valor de um item

```ts
const additionsTotal = item.additions.reduce(
  (total, addition) => total + addition.price,
  0
)

const unitPrice = item.basePrice + additionsTotal
const itemTotal = unitPrice * item.quantity
```

### Subtotal

```ts
const subtotal = cart.reduce(
  (total, item) => total + item.unitPrice * item.quantity,
  0
)
```

### Taxa de entrega

```ts
const deliveryFee =
  orderType === "delivery"
    ? selectedDeliveryFee
    : 0
```

### Total

```ts
const total = subtotal + deliveryFee
```

Valores monetários devem permanecer como números durante os cálculos.

Formate como moeda brasileira apenas para exibição.

---

## 17. Persistência do carrinho

Como não existe backend, o carrinho será salvo no navegador usando `localStorage`.

Usar uma chave centralizada, por exemplo:

```ts
const CART_STORAGE_KEY = "interactive-menu-cart"
```

Salvar:

```ts
localStorage.setItem(
  CART_STORAGE_KEY,
  JSON.stringify(cart)
)
```

Recuperar:

```ts
const savedCart = JSON.parse(
  localStorage.getItem(CART_STORAGE_KEY) || "[]"
)
```

Regras:

- recuperar o carrinho ao iniciar a aplicação;
- atualizar o `localStorage` quando o carrinho mudar;
- tratar dados inválidos ou corrompidos;
- não acessar `window` sem proteção quando houver renderização no servidor;
- não salvar funções ou elementos React;
- salvar somente dados serializáveis.

Não confundir `localStorage` com histórico de pedidos.

O carrinho existe apenas no navegador do cliente.

---

## 18. Checkout

O checkout deve ser organizado em etapas claras.

### Etapa 1 — Modalidade

Exibir:

- Delivery;
- Retirada no local;
- Comer no local.

### Etapa 2 — Dados

Campos comuns:

- nome;
- telefone.

Campos condicionais conforme a modalidade.

### Etapa 3 — Pagamento

Exibir as formas habilitadas pela lanchonete.

### Etapa 4 — Resumo

Mostrar:

- modalidade;
- dados do cliente;
- itens;
- adicionais;
- observações;
- subtotal;
- taxa de entrega, quando houver;
- total;
- forma de pagamento;
- troco, quando houver.

### Etapa 5 — WhatsApp

Botão principal:

```text
Enviar pedido pelo WhatsApp
```

Não permita avançar com campos obrigatórios inválidos.

---

## 19. Estrutura do pedido

Estrutura sugerida:

```ts
interface Order {
  type: OrderType
  customerName: string
  phone: string
  paymentMethod: PaymentMethod
  needsChange?: boolean
  changeFor?: number
  generalObservation?: string

  delivery?: {
    street: string
    number: string
    neighborhood: string
    complement?: string
    reference?: string
    fee: number
  }

  pickup?: {
    requestedTime: string
  }

  dineIn?: {
    numberOfPeople: number
    arrivalTime: string
  }

  items: CartItem[]
  subtotal: number
  deliveryFee: number
  total: number
}
```

Dados de uma modalidade não devem ser enviados como se pertencessem a outra.

Exemplos:

- não incluir endereço em retirada;
- não incluir horário de retirada em delivery;
- não incluir número de pessoas em retirada;
- não incluir taxa de entrega em consumo no local.

---

## 20. Configurações da lanchonete

Como o MVP não possui painel administrativo, os dados devem ficar centralizados em um arquivo de configuração.

Exemplo de caminho:

```text
src/config/restaurant.ts
```

Estrutura sugerida:

```ts
export const restaurantConfig = {
  name: "Nome da Lanchonete",
  whatsapp: "5598999999999",
  minimumOrder: 0,

  serviceTypes: {
    delivery: true,
    pickup: true,
    dineIn: true
  },

  paymentMethods: [
    "pix",
    "cash",
    "debit-card",
    "credit-card"
  ],

  deliveryFees: {
    Cohama: 7,
    Centro: 6,
    Renascença: 8,
    Turu: 10
  },

  openingHours: {
    monday: null,
    tuesday: ["18:00", "23:00"],
    wednesday: ["18:00", "23:00"],
    thursday: ["18:00", "23:00"],
    friday: ["18:00", "00:00"],
    saturday: ["18:00", "00:00"],
    sunday: ["18:00", "23:00"]
  },

  messages: {
    pickupConfirmation:
      "O horário de retirada será confirmado pela lanchonete no WhatsApp.",

    dineInConfirmation:
      "O pedido e a disponibilidade de mesa serão confirmados pela lanchonete no WhatsApp."
  }
}
```

Não espalhe pelo código:

- número do WhatsApp;
- taxas;
- pedido mínimo;
- formas de pagamento;
- horários;
- nome da lanchonete.

Centralize essas informações.

---

## 21. Produtos e categorias locais

Como não existe backend, usar arquivos locais.

Exemplo:

```text
src/data/categories.ts
src/data/products.ts
```

Estrutura de categoria:

```ts
interface Category {
  id: string
  name: string
  order: number
  active: boolean
}
```

A aplicação deve:

- mostrar somente categorias ativas;
- mostrar somente produtos relacionados;
- manter uma ordenação previsível;
- permitir que produtos indisponíveis continuem visíveis;
- evitar valores e categorias repetidos diretamente nos componentes.

---

## 22. Mensagem do WhatsApp

A mensagem deve ser clara e fácil de ler pela lanchonete.

Ela deve conter:

1. título;
2. modalidade;
3. dados do cliente;
4. itens;
5. adicionais;
6. observações;
7. dados específicos da modalidade;
8. pagamento;
9. subtotal;
10. taxa de entrega, quando houver;
11. total;
12. aviso de que o pedido aguarda confirmação.

### Exemplo — Delivery

```text
*NOVO PEDIDO — DELIVERY*

*CLIENTE*
Nome: Manuel
Telefone: (98) 99999-9999

*ITENS*

2x X-Bacon
Adicionais: Bacon extra
Observação: Sem cebola
Valor: R$ 57,80

1x Coca-Cola 1L
Valor: R$ 9,00

*ENDEREÇO*
Rua das Flores, 120
Bairro: Cohama
Complemento: Apartamento 02
Referência: Próximo à farmácia

*PAGAMENTO*
Forma: Dinheiro
Troco para: R$ 100,00

*RESUMO*
Subtotal: R$ 66,80
Taxa de entrega: R$ 7,00
Total: R$ 73,80

Aguardando confirmação da lanchonete.
```

### Exemplo — Retirada

```text
*NOVO PEDIDO — RETIRADA NO LOCAL*

*CLIENTE*
Nome: Manuel
Telefone: (98) 99999-9999
Horário desejado: 20:30

*ITENS*

1x Combo Artesanal
Valor: R$ 34,90

1x Coca-Cola
Valor: R$ 5,00

*PAGAMENTO*
Forma: Pix

*RESUMO*
Subtotal: R$ 39,90
Total: R$ 39,90

Aguardando confirmação da lanchonete.
```

### Exemplo — Comer no local

```text
*NOVO PEDIDO — COMER NO LOCAL*

Pedido antecipado para consumo no estabelecimento.

*CLIENTE*
Nome: Manuel
Telefone: (98) 99999-9999
Quantidade de pessoas: 3
Horário previsto de chegada: 20:00

*ITENS*

3x X-Bacon
Valor: R$ 74,70

2x Coca-Cola
Valor: R$ 10,00

*PAGAMENTO*
Forma: Cartão de crédito

*RESUMO*
Subtotal: R$ 84,70
Total: R$ 84,70

Aguardando confirmação do pedido e da disponibilidade de mesa.
```

---

## 23. Serviço do WhatsApp

Centralizar a geração da mensagem e do link.

Exemplo de caminho:

```text
src/services/whatsappService.ts
```

Responsabilidades:

- receber um pedido válido;
- criar a mensagem;
- formatar itens e valores;
- incluir somente os dados da modalidade escolhida;
- aplicar `encodeURIComponent`;
- gerar o link;
- abrir o WhatsApp.

Exemplo:

```ts
export function createWhatsAppUrl(
  phone: string,
  message: string
): string {
  return `https://wa.me/${phone}?text=${encodeURIComponent(message)}`
}
```

Para abrir:

```ts
const whatsappUrl = createWhatsAppUrl(
  restaurantConfig.whatsapp,
  message
)

window.open(
  whatsappUrl,
  "_blank",
  "noopener,noreferrer"
)
```

Validar o telefone da lanchonete antes de gerar o link.

O número deve conter:

```text
DDI + DDD + número
```

Sem:

- espaços;
- parênteses;
- hífens;
- sinal de mais.

---

## 24. Confirmação do envio

Abrir o WhatsApp não significa que a mensagem foi enviada.

Nunca mostrar automaticamente:

```text
Pedido enviado com sucesso.
```

Mostrar algo semelhante a:

```text
O WhatsApp foi aberto com seu pedido. Envie a mensagem para concluir.
```

Não limpar o carrinho imediatamente.

Após abrir o WhatsApp, oferecer:

```text
Você conseguiu enviar o pedido?

[Sim, limpar carrinho]
[Ainda não]
```

Somente limpar após confirmação explícita do cliente.

---

## 25. Validações gerais

Antes de gerar a mensagem:

### Todos os pedidos

Validar:

- carrinho não vazio;
- produtos ainda disponíveis;
- quantidades maiores que zero;
- modalidade selecionada;
- nome preenchido;
- telefone válido;
- pagamento selecionado;
- valores calculados corretamente.

### Delivery

Validar:

- rua;
- número;
- bairro;
- taxa do bairro;
- pedido mínimo, quando aplicável.

### Retirada

Validar:

- horário desejado.

### Comer no local

Validar:

- quantidade de pessoas maior que zero;
- horário previsto de chegada.

### Dinheiro

Quando houver troco:

- valor preenchido;
- valor numérico;
- valor maior que o total.

Erros devem ser exibidos próximos dos campos correspondentes.

Não utilizar apenas alertas genéricos.

---

## 26. Horário de funcionamento

O cardápio poderá continuar visível mesmo quando a lanchonete estiver fechada.

O comportamento da finalização deve seguir a configuração definida no projeto.

Para o MVP, priorizar:

- permitir visualizar o cardápio;
- impedir finalizar fora do horário;
- mostrar claramente quando a lanchonete abrirá.

Exemplo:

```text
A lanchonete está fechada no momento.

Funcionamento hoje:
18h às 23h.
```

Observe corretamente:

- dia da semana;
- horários que passam da meia-noite;
- fuso horário;
- dias sem funcionamento.

Não criar uma lógica complexa de agendamento sem solicitação.

---

## 27. Experiência mobile

O cardápio será acessado principalmente pelo celular.

Priorizar:

- abordagem mobile-first;
- botões fáceis de tocar;
- campos com altura adequada;
- imagens otimizadas;
- textos legíveis;
- categorias com rolagem horizontal;
- carrinho acessível;
- checkout em uma coluna;
- resumo de valores fixo ou fácil de localizar;
- estados de carregamento claros;
- feedback visual após adicionar um produto;
- áreas clicáveis maiores que apenas ícones;
- respeito à área segura de celulares.

Exemplo de botão fixo:

```text
Ver carrinho · 3 itens · R$ 66,80
```

Evitar:

- excesso de modais;
- textos pequenos;
- elementos muito próximos;
- rolagem horizontal da página;
- botões escondidos atrás da barra do navegador;
- imagens pesadas;
- checkout confuso.

---

## 28. Acessibilidade

Sempre que possível:

- associe `label` aos campos;
- utilize botões reais;
- mantenha foco visível;
- permita navegação por teclado;
- use textos alternativos nas imagens;
- não dependa apenas de cores;
- use mensagens de erro legíveis;
- mantenha contraste suficiente;
- use atributos ARIA somente quando necessários;
- devolva o foco adequadamente ao fechar modais.

---

## 29. Tecnologias recomendadas

Caso o projeto ainda não tenha stack definida, priorize:

- React;
- TypeScript;
- Vite;
- Tailwind CSS;
- React Router, quando existirem páginas separadas;
- Context API ou Zustand para o carrinho;
- React Hook Form para formulários;
- Zod para validação;
- `localStorage` para persistência.

Não instale dependências sem necessidade.

Antes de instalar uma biblioteca:

1. Verifique se o projeto já possui solução equivalente.
2. Avalie se a funcionalidade pode ser feita de maneira simples.
3. Explique a necessidade.
4. Evite dependências grandes para pequenas tarefas.

Se o projeto já utilizar outras tecnologias, adapte-se ao código existente em vez de migrar sem autorização.

---

## 30. Estrutura sugerida

```text
src/
├── components/
│   ├── Header.tsx
│   ├── CategoryMenu.tsx
│   ├── ProductCard.tsx
│   ├── ProductModal.tsx
│   ├── CartItem.tsx
│   ├── CartSummary.tsx
│   ├── OrderTypeSelector.tsx
│   └── PaymentSelector.tsx
│
├── pages/
│   ├── MenuPage.tsx
│   ├── CartPage.tsx
│   ├── CheckoutPage.tsx
│   └── OrderResultPage.tsx
│
├── context/
│   └── CartContext.tsx
│
├── data/
│   ├── categories.ts
│   └── products.ts
│
├── config/
│   └── restaurant.ts
│
├── services/
│   └── whatsappService.ts
│
├── utils/
│   ├── calculateCart.ts
│   ├── formatCurrency.ts
│   ├── formatPhone.ts
│   ├── openingHours.ts
│   └── validateOrder.ts
│
├── types/
│   ├── product.ts
│   ├── cart.ts
│   └── order.ts
│
└── App.tsx
```

A estrutura é uma referência, não uma obrigação.

Respeite a organização já existente quando ela for coerente.

---

## 31. Organização e qualidade do código

Durante o desenvolvimento:

- use TypeScript corretamente;
- evite `any`;
- crie tipos compartilhados;
- mantenha componentes pequenos;
- separe regras de negócio da interface;
- centralize configurações;
- não duplique cálculos;
- não duplique mensagens;
- use nomes claros;
- trate valores monetários de maneira consistente;
- não coloque toda a aplicação em um único componente;
- evite estados derivados desnecessários;
- evite salvar no estado valores que podem ser calculados;
- trate erros;
- remova código morto;
- mantenha imports organizados;
- preserve os padrões já existentes no projeto.

A interface não deve montar diretamente toda a mensagem do WhatsApp.

Essa responsabilidade deve ficar em uma função ou serviço separado.

---

## 32. Testes mínimos

Quando houver estrutura de testes, cobrir principalmente:

### Carrinho

- adicionar produto;
- remover produto;
- alterar quantidade;
- calcular adicionais;
- calcular subtotal;
- manter itens configurados separadamente;
- persistir e recuperar `localStorage`.

### Checkout

- validar campos comuns;
- validar delivery;
- validar retirada;
- validar consumo no local;
- calcular taxa somente no delivery;
- validar troco.

### WhatsApp

- gerar a mensagem correta para delivery;
- gerar a mensagem correta para retirada;
- gerar a mensagem correta para consumo no local;
- não incluir campos de outras modalidades;
- codificar corretamente a URL;
- formatar valores corretamente.

Mesmo quando não houver testes automatizados, realizar uma verificação manual do fluxo completo.

---

## 33. Critérios de aceitação do MVP

O MVP estará funcional quando:

- o cliente conseguir visualizar as categorias;
- os produtos forem exibidos corretamente;
- produtos indisponíveis não puderem ser adicionados;
- o cliente puder configurar quantidade, adicionais e observação;
- os itens puderem ser adicionados ao carrinho;
- o carrinho permanecer após atualizar a página;
- o cliente puder alterar ou remover itens;
- os valores forem calculados corretamente;
- o cliente puder escolher uma modalidade;
- os campos mudarem conforme a modalidade;
- a taxa for aplicada somente no delivery;
- o cliente puder escolher o pagamento;
- os dados obrigatórios forem validados;
- o resumo mostrar informações corretas;
- o WhatsApp abrir com a mensagem organizada;
- o carrinho não for limpo antes da confirmação do cliente;
- o projeto funcionar corretamente em celular;
- não houver backend, painel ou histórico introduzidos indevidamente.

---

## 34. Processo para executar tarefas

Ao receber uma solicitação relacionada ao projeto:

### 1. Investigue

- Leia os arquivos relevantes.
- Identifique o fluxo atual.
- Localize tipos, componentes e serviços existentes.
- Procure possíveis conflitos.

### 2. Planeje

Apresente um plano curto com:

- arquivos que serão alterados;
- comportamento esperado;
- riscos ou decisões necessárias.

### 3. Implemente

- Faça alterações incrementais.
- Preserve o padrão do projeto.
- Não adicione escopo não solicitado.

### 4. Verifique

Execute, quando disponíveis:

```text
lint
typecheck
test
build
```

Além disso, valide manualmente o fluxo afetado.

### 5. Relate

Ao concluir, informe:

- o que foi alterado;
- os arquivos principais;
- como testar;
- limitações encontradas;
- pontos ainda pendentes.

Não declare que algo funciona sem ter verificado.

---

## 35. Conduta ao encontrar ambiguidades

Quando uma decisão pequena puder ser inferida com segurança:

- escolha a solução mais simples;
- siga o padrão existente;
- registre a decisão.

Quando a decisão alterar diretamente a operação da lanchonete, não invente regras.

Exemplos de regras que precisam ser fornecidas pelo responsável:

- bairros atendidos;
- valor das taxas;
- pedido mínimo;
- formas de pagamento;
- horário de funcionamento;
- tempo estimado;
- produtos;
- preços;
- adicionais;
- disponibilidade;
- número do WhatsApp.

Use valores de exemplo claramente identificados como demonstração, nunca como dados reais da lanchonete.

---

## 36. Regra central do projeto

Mantenha sempre esta separação:

```text
O site monta o pedido.
O WhatsApp transporta a mensagem.
A lanchonete confirma o pedido.
```

Não trate a abertura do WhatsApp como confirmação ou envio garantido.

Não apresente ao cliente um estado de “pedido aceito” sem uma resposta real da lanchonete.

---

## 37. Tarefa solicitada

Aplique todas as regras desta skill à seguinte solicitação:

```text
$ARGUMENTS
```

Antes de modificar o projeto, analise o código existente e confirme internamente que a solução permanece dentro do escopo definido.
