# Instruções de Segurança — Cardápio Interativo

## Objetivo

Estas instruções devem ser seguidas durante o desenvolvimento do cardápio interativo para reduzir riscos de segurança, proteger os dados do cliente e evitar comportamentos incorretos no fluxo do pedido.

O projeto é um MVP sem backend, sem banco de dados e sem painel administrativo. Mesmo assim, o front-end continua exposto a manipulações no navegador e deve ser tratado com cuidado.

---

## 1. Não confiar nos valores do navegador

O cliente consegue abrir o DevTools e alterar:

- preço do produto;
- taxa de entrega;
- subtotal;
- quantidade;
- endereço;
- mensagem do WhatsApp;
- dados do `localStorage`.

Por isso, a lanchonete deve conferir o pedido recebido no WhatsApp antes de confirmar.

Exibir uma mensagem semelhante a:

```text
Os preços e a disponibilidade serão confirmados pela lanchonete no WhatsApp.
```

A regra principal deve ser:

```text
O site calcula o pedido.
O WhatsApp envia a solicitação.
A lanchonete confere e confirma.
```

Como o MVP não possui backend, não existe uma forma realmente segura de impedir que o usuário manipule os valores no navegador.

---

## 2. Não colocar dados secretos no front-end

Tudo que estiver no código React poderá ser visualizado pelo usuário.

Não incluir:

- senhas;
- tokens;
- chaves privadas;
- credenciais;
- chaves de API secretas;
- acessos administrativos;
- dados financeiros internos;
- certificados;
- segredos de integração.

Variáveis do Vite que começam com:

```text
VITE_
```

também ficam visíveis no navegador depois do build.

Pode ficar público:

```env
VITE_RESTAURANT_WHATSAPP=5598999999999
```

Nunca colocar:

```env
VITE_SECRET_API_KEY=chave-secreta
```

O número comercial do WhatsApp pode ser público. Chaves e credenciais, não.

---

## 3. Validar todos os dados do formulário

Todos os dados devem ser validados antes de montar a mensagem do WhatsApp.

### Nome

- remover espaços no início e no final;
- não aceitar somente espaços;
- definir tamanho mínimo;
- definir tamanho máximo.

Exemplo:

```ts
const customerName = name.trim()
```

### Telefone

- remover caracteres que não sejam números;
- validar DDD;
- validar quantidade de dígitos;
- limitar o tamanho do campo.

Exemplo:

```ts
const normalizedPhone = phone.replace(/\D/g, "")
```

### Observações

Observações são campos de texto livre e devem possuir limite.

Sugestão:

```text
Máximo de 300 caracteres por observação de item.
Máximo de 500 caracteres para observação geral.
```

### Quantidade

A quantidade deve:

- ser um número inteiro;
- ser maior que zero;
- possuir um limite máximo razoável.

Sugestão:

```text
Quantidade mínima: 1
Quantidade máxima: 20
```

### Horários

- validar o formato;
- não permitir valores vazios;
- evitar horários inválidos;
- não permitir horário anterior ao atual quando não fizer sentido.

---

## 4. Evitar injeção de HTML

Nunca renderizar textos fornecidos pelo cliente como HTML.

Evitar:

```tsx
<div
  dangerouslySetInnerHTML={{
    __html: observation
  }}
/>
```

Usar:

```tsx
<p>{observation}</p>
```

O React escapa o texto automaticamente.

Também não usar `innerHTML` para mostrar:

- observações;
- nome do cliente;
- endereço;
- nome de produtos externos;
- conteúdo recebido de fontes não confiáveis.

---

## 5. Proteger a geração do link do WhatsApp

Sempre aplicar:

```ts
encodeURIComponent(message)
```

Exemplo:

```ts
const url = `https://wa.me/${phone}?text=${encodeURIComponent(message)}`
```

Também normalizar o número da lanchonete:

```ts
function normalizeWhatsAppNumber(phone: string): string {
  return phone.replace(/\D/g, "")
}
```

O número deve conter:

```text
DDI + DDD + número
```

Exemplo:

```text
5598999999999
```

Não permitir que o cliente escolha livremente o destino da mensagem.

O número deve vir da configuração fixa da lanchonete.

---

## 6. Abrir links externos com segurança

Ao abrir o WhatsApp em uma nova aba, utilizar:

```ts
window.open(
  whatsappUrl,
  "_blank",
  "noopener,noreferrer"
)
```

Em links HTML:

```tsx
<a
  href={whatsappUrl}
  target="_blank"
  rel="noopener noreferrer"
>
  Enviar pelo WhatsApp
</a>
```

Isso reduz o risco de a nova aba acessar a página original.

---

## 7. Tratar o localStorage como dado não confiável

O usuário pode editar o `localStorage` manualmente.

Ao recuperar o carrinho:

- usar `try/catch`;
- validar se o conteúdo é um array;
- validar cada item;
- descartar produtos desconhecidos;
- descartar quantidades inválidas;
- recalcular preços;
- não confiar em totais salvos.

Preferir salvar:

```ts
{
  productId: "x-bacon",
  quantity: 2,
  additionIds: ["extra-bacon"],
  observation: "Sem cebola"
}
```

Evitar confiar em:

```ts
{
  productId: "x-bacon",
  price: 0.01
}
```

Mesmo com validação, o usuário ainda pode manipular o JavaScript em execução. A confirmação final continua sendo responsabilidade da lanchonete.

---

## 8. Recalcular tudo antes de gerar a mensagem

Antes de abrir o WhatsApp, recalcular:

- preço base;
- adicionais;
- quantidade;
- subtotal;
- taxa de entrega;
- total.

Usar sempre os dados oficiais do projeto:

```text
src/data/products.ts
src/config/restaurant.ts
```

Fluxo recomendado:

```text
Ler os itens do carrinho
        ↓
Encontrar os produtos oficiais
        ↓
Validar disponibilidade
        ↓
Buscar preços atuais
        ↓
Recalcular subtotal
        ↓
Buscar taxa oficial do bairro
        ↓
Calcular total
        ↓
Montar a mensagem
```

Não utilizar diretamente um total armazenado ou digitado pelo cliente.

---

## 9. Não deixar o cliente definir a taxa de entrega

A taxa deve vir da configuração da lanchonete.

Exemplo:

```ts
export const deliveryFees = {
  Cohama: 7,
  Centro: 6,
  Turu: 10
}
```

O cliente apenas escolhe o bairro.

Não criar um campo como:

```text
Digite o valor da entrega
```

Também não confiar em uma taxa armazenada anteriormente no carrinho.

Buscar sempre a taxa atual antes da finalização.

---

## 10. Validar produtos e adicionais

Antes de finalizar, confirmar que:

- o produto existe;
- o produto está disponível;
- a quantidade é válida;
- os adicionais pertencem ao produto;
- os preços vêm do catálogo oficial;
- não existem adicionais inválidos;
- não existem adicionais duplicados indevidamente.

Exemplo de combinação inválida:

```text
Produto: Coca-Cola
Adicional: Bacon extra
```

O sistema deve impedir esse tipo de combinação.

---

## 11. Limitar tamanhos e quantidades

Definir limites para evitar travamentos e mensagens enormes.

Sugestão:

```text
Nome: até 80 caracteres
Telefone: até 15 dígitos
Rua: até 120 caracteres
Complemento: até 100 caracteres
Referência: até 150 caracteres
Observação do item: até 300 caracteres
Observação geral: até 500 caracteres
Quantidade por item: até 20
Itens diferentes no carrinho: até 50
```

Esses valores podem ser ajustados conforme a operação da lanchonete.

---

## 12. Evitar múltiplos cliques

O cliente pode clicar várias vezes em:

```text
Enviar pedido pelo WhatsApp
```

Usar um estado temporário:

```tsx
const [isOpeningWhatsApp, setIsOpeningWhatsApp] =
  useState(false)
```

Exemplo:

```tsx
<button
  disabled={isOpeningWhatsApp}
  onClick={handleSendOrder}
>
  {isOpeningWhatsApp
    ? "Abrindo WhatsApp..."
    : "Enviar pedido pelo WhatsApp"}
</button>
```

Isso reduz a abertura de múltiplas abas e ações duplicadas.

---

## 13. Não afirmar que o pedido foi enviado

Abrir o WhatsApp não significa que a mensagem foi enviada.

Não mostrar:

```text
Pedido enviado com sucesso.
```

Mostrar:

```text
O WhatsApp foi aberto com seu pedido. Envie a mensagem para concluir.
```

Também não limpar o carrinho automaticamente.

---

## 14. Usar HTTPS

O site publicado deve utilizar:

```text
https://
```

A Vercel já fornece HTTPS automaticamente.

Não publicar em HTTP, principalmente porque o checkout pode conter:

- nome;
- telefone;
- endereço;
- ponto de referência.

Mesmo sem backend, a conexão do site deve ser segura.

---

## 15. Minimizar a coleta de dados pessoais

Coletar apenas os dados necessários para cada modalidade.

### Delivery

- nome;
- telefone;
- endereço;
- referência opcional.

### Retirada

- nome;
- telefone;
- horário.

### Comer no local

- nome;
- telefone;
- quantidade de pessoas;
- horário previsto.

Não solicitar no MVP:

- CPF;
- senha;
- data de nascimento;
- documentos;
- dados de cartão;
- localização precisa;
- e-mail, quando não for necessário.

---

## 16. Não armazenar dados pessoais sem necessidade

O carrinho pode ficar no `localStorage`, mas evitar guardar permanentemente:

- endereço;
- telefone;
- nome;
- forma de pagamento;
- observações pessoais.

Manter os dados do checkout apenas em memória durante a sessão sempre que possível.

Ao limpar o pedido:

```ts
localStorage.removeItem("interactive-menu-cart")
```

Se os dados de checkout forem armazenados temporariamente, usar uma chave separada e removê-la após a conclusão.

---

## 17. Informar o uso dos dados

Adicionar uma mensagem simples no checkout:

```text
Seus dados serão utilizados somente para organizar e confirmar este pedido pelo WhatsApp.
```

Evitar prometer segurança absoluta ou anonimato.

---

## 18. Manter dependências atualizadas

Executar periodicamente:

```bash
npm audit
```

Para ver pacotes desatualizados:

```bash
npm outdated
```

Evitar executar automaticamente:

```bash
npm audit fix --force
```

Esse comando pode introduzir incompatibilidades.

Processo recomendado:

1. analisar o alerta;
2. identificar a dependência;
3. confirmar se ela é utilizada;
4. atualizar com cuidado;
5. rodar build e testes.

---

## 19. Não expor arquivos desnecessários

Não publicar:

- `.env`;
- backups;
- planilhas internas;
- documentação privada;
- chaves;
- certificados;
- logs;
- dados reais usados em testes.

Adicionar ao `.gitignore`:

```gitignore
.env
.env.local
.env.*.local
node_modules/
dist/
*.log
```

---

## 20. Cabeçalhos de segurança

Na hospedagem, configurar cabeçalhos como:

```text
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
```

Exemplo inicial para Vercel:

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        },
        {
          "key": "Permissions-Policy",
          "value": "camera=(), microphone=(), geolocation=()"
        }
      ]
    }
  ]
}
```

A política `Content-Security-Policy` deve ser criada de acordo com os serviços realmente usados.

Não copiar uma política restritiva sem testar, pois ela pode bloquear fontes, imagens ou scripts necessários.

---

## 21. Imagens externas

Preferir armazenar as imagens localmente:

```text
public/images/products/
```

Caso use imagens externas:

- usar domínios confiáveis;
- evitar URLs fornecidas pelo cliente;
- não carregar conteúdo de fontes desconhecidas;
- configurar domínios permitidos;
- evitar SVGs externos não confiáveis.

Para produtos, preferir:

- WebP;
- AVIF.

---

## 22. Sanitizar textos sem destruir conteúdo válido

A mensagem deve ser criada como texto simples.

Exemplo:

```ts
function sanitizeText(value: string): string {
  return value
    .trim()
    .replace(/\s+/g, " ")
}
```

O foco deve ser:

- limitar tamanho;
- remover espaços excessivos;
- evitar conteúdo vazio;
- codificar corretamente a URL.

Não usar texto do cliente para construir:

- HTML;
- comandos;
- código;
- URLs de destino.

---

## 23. Manter resumo e mensagem consistentes

O resumo exibido no site deve usar os mesmos dados enviados ao WhatsApp.

Não permitir divergência entre:

- total exibido;
- total enviado;
- taxa exibida;
- taxa enviada;
- produtos exibidos;
- produtos enviados.

Usar uma única fonte de dados para montar:

- resumo;
- mensagem;
- total.

---

## 24. Limitações de segurança do MVP

Como não existe backend, o sistema não consegue garantir:

- integridade absoluta dos preços;
- autenticidade do pedido;
- numeração única;
- confirmação automática;
- proteção total contra manipulação;
- histórico seguro;
- bloqueio de pedidos falsos;
- disponibilidade em tempo real.

A mensagem pode terminar com:

```text
Pedido sujeito à confirmação de preço, disponibilidade e atendimento pela lanchonete.
```

---

## 25. Prioridades de segurança

As precauções mais importantes são:

1. Não colocar segredos no front-end.
2. Usar HTTPS.
3. Validar todos os campos.
4. Não usar `dangerouslySetInnerHTML`.
5. Recalcular preços usando dados oficiais.
6. Tratar o `localStorage` como manipulável.
7. Usar `encodeURIComponent` no WhatsApp.
8. Não armazenar dados pessoais desnecessariamente.
9. Manter dependências atualizadas.
10. Fazer a lanchonete conferir e confirmar cada pedido.

---

## Regra central

```text
O front-end pode ser manipulado.
O WhatsApp apenas transporta a mensagem.
A lanchonete deve conferir antes de confirmar e preparar o pedido.
```
