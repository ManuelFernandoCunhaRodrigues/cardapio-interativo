# Instrução — Manter o Cardápio Interativo Leve

## Objetivo

Esta instrução deve ser seguida durante o desenvolvimento do cardápio interativo para garantir:

- carregamento rápido;
- boa experiência em celulares;
- baixo consumo de dados;
- código simples;
- poucas dependências;
- build final pequeno;
- manutenção fácil.

O projeto é um MVP sem backend, banco de dados ou painel administrativo.

---

## 1. Imagens dos produtos

As imagens são o principal ponto de atenção para o desempenho.

### Formatos permitidos

Priorizar:

- WebP;
- AVIF, quando houver suporte e necessidade.

Evitar:

- PNG pesado;
- JPEG sem compressão;
- imagens em Base64;
- imagens muito maiores que o espaço exibido.

### Tamanhos visuais

No layout atual:

- card de produto: `104 × 104 px`;
- item do carrinho: `70 × 70 px`.

Não é obrigatório criar dois arquivos para cada produto.

Uma única imagem otimizada pode ser reutilizada nos dois contextos, sendo redimensionada visualmente pelo CSS.

### Dimensão recomendada do arquivo

Usar imagens quadradas, preferencialmente entre:

```text
208 × 208 px
e
416 × 416 px
```

Isso mantém boa nitidez em telas de maior densidade sem exagerar no peso.

### Peso recomendado

Buscar manter cada imagem entre:

```text
20 KB e 60 KB
```

Evitar imagens individuais acima de `100 KB`, salvo quando houver justificativa visual real.

### Organização

Armazenar em:

```text
public/images/products/
```

Exemplo:

```text
public/
└── images/
    └── products/
        ├── brasa-supreme.webp
        ├── x-bacon-crocante.webp
        ├── smash-duplo.webp
        ├── pizza-calabresa.webp
        └── suco-laranja.webp
```

### Uso no código

```tsx
<img
  src="/images/products/brasa-supreme.webp"
  alt="Brasa Supreme"
  width={104}
  height={104}
  loading="lazy"
  decoding="async"
/>
```

### Regras

- Não usar a colagem completa com todos os produtos dentro do site.
- Cada produto deve possuir sua própria imagem.
- Não recortar uma imagem grande usando CSS.
- Não salvar imagens dentro do `localStorage`.
- Não importar imagens como Base64.
- Manter proporção quadrada.
- Usar `object-fit: cover` para evitar deformação.

---

## 2. Lazy loading

Imagens fora da primeira área visível devem utilizar:

```tsx
loading="lazy"
```

Isso permite que sejam carregadas apenas quando o cliente se aproximar delas durante a rolagem.

Exemplo:

```tsx
<img
  src={product.image}
  alt={product.name}
  loading="lazy"
  decoding="async"
/>
```

As primeiras imagens da tela podem usar carregamento normal ou:

```tsx
loading="eager"
```

Use `eager` somente para poucas imagens realmente importantes.

---

## 3. Bibliotecas

Instalar apenas o necessário.

### Stack recomendada

- React;
- TypeScript;
- Vite;
- Tailwind CSS;
- React Router;
- React Hook Form;
- Zod;
- Lucide React.

### Evitar no MVP

Não instalar sem necessidade clara:

- Redux;
- Zustand, caso Context API já resolva;
- bibliotecas grandes de modal;
- bibliotecas grandes de carrossel;
- Framer Motion apenas para animações simples;
- Moment.js;
- bibliotecas para formatação de moeda;
- bibliotecas para operações simples de data;
- bibliotecas duplicadas com a mesma função.

Antes de instalar qualquer dependência:

1. Verifique se já existe solução equivalente no projeto.
2. Avalie se React, CSS ou JavaScript nativo resolvem.
3. Confirme que o benefício compensa o peso adicionado.
4. Evite dependências para pequenas tarefas.

---

## 4. Estado do carrinho

Para o MVP, priorizar:

```text
Context API + useReducer
```

O estado deve conter apenas os dados necessários.

Exemplo:

```ts
interface CartItem {
  cartItemId: string
  productId: string
  name: string
  unitPrice: number
  quantity: number
  additions: Addition[]
  observation?: string
}
```

Evitar salvar no estado:

- componentes React;
- funções;
- imagens em Base64;
- objetos duplicados;
- dados que podem ser calculados;
- informações de páginas não relacionadas ao carrinho.

---

## 5. localStorage

O `localStorage` será usado apenas para persistir o carrinho.

### Salvar apenas

- identificador do produto;
- nome;
- quantidade;
- preço unitário;
- adicionais;
- observação.

### Não salvar

- imagens;
- componentes;
- HTML;
- funções;
- dados duplicados;
- histórico de pedidos;
- estado completo da aplicação.

Exemplo:

```ts
const CART_STORAGE_KEY = "interactive-menu-cart"

localStorage.setItem(
  CART_STORAGE_KEY,
  JSON.stringify(cart)
)
```

Ao recuperar, tratar possíveis dados inválidos:

```ts
function loadCart(): CartItem[] {
  try {
    const savedCart = localStorage.getItem(CART_STORAGE_KEY)

    if (!savedCart) {
      return []
    }

    return JSON.parse(savedCart)
  } catch {
    return []
  }
}
```

---

## 6. Divisão de código

Separar páginas e funcionalidades para reduzir o JavaScript carregado inicialmente.

Usar `React.lazy` em páginas menos importantes para a primeira renderização.

Exemplo:

```tsx
import { lazy, Suspense } from "react"

const CartPage = lazy(() => import("./pages/CartPage"))
const CheckoutPage = lazy(() => import("./pages/CheckoutPage"))
```

Envolver as rotas com:

```tsx
<Suspense fallback={<p>Carregando...</p>}>
  {/* rotas */}
</Suspense>
```

Priorizar carregamento imediato da página principal do cardápio.

Carrinho, checkout e tela final podem ser carregados sob demanda.

---

## 7. Componentes

Não criar toda a aplicação em um único arquivo.

Separar, por exemplo:

```text
MenuPage
ProductCard
ProductModal
CartPage
CartItem
CheckoutPage
OrderSummary
```

Cada componente deve possuir uma responsabilidade clara.

Evitar:

- componentes enormes;
- lógica de WhatsApp dentro do card;
- cálculo de taxa dentro de vários componentes;
- repetição de estilos;
- repetição de regras de negócio.

---

## 8. Cálculos

Não salvar no estado valores que podem ser calculados.

Exemplo:

```ts
const subtotal = cart.reduce(
  (total, item) => total + item.unitPrice * item.quantity,
  0
)
```

Use `useMemo` apenas quando houver ganho real.

Exemplo:

```tsx
const subtotal = useMemo(() => {
  return cart.reduce(
    (total, item) => total + item.unitPrice * item.quantity,
    0
  )
}, [cart])
```

Não aplicar `useMemo` e `useCallback` em tudo sem necessidade.

---

## 9. Fontes

Usar no máximo uma família tipográfica principal.

Exemplo:

```text
Inter
```

Carregar somente os pesos realmente usados:

- 400;
- 600;
- 700.

Evitar carregar:

- muitos pesos;
- várias famílias;
- fontes decorativas desnecessárias.

Sempre definir fallback:

```css
font-family: Inter, system-ui, sans-serif;
```

---

## 10. Ícones

Usar apenas uma biblioteca de ícones.

Recomendação:

```text
Lucide React
```

Importar somente os ícones utilizados:

```tsx
import {
  Plus,
  Search,
  ShoppingCart
} from "lucide-react"
```

Nunca importar a biblioteca inteira.

---

## 11. Animações

Priorizar animações simples com CSS.

Exemplo:

```css
transition: transform 200ms ease, opacity 200ms ease;
```

Usar animações curtas, entre:

```text
150ms e 300ms
```

Evitar:

- Framer Motion apenas para fades simples;
- animações contínuas;
- efeitos pesados;
- blur excessivo;
- sombras exageradas;
- grandes transformações durante a rolagem.

---

## 12. Dados dos produtos

Como o projeto não terá backend, manter produtos e categorias em arquivos locais.

Exemplo:

```text
src/data/products.ts
src/data/categories.ts
```

As imagens devem ser referenciadas por caminho:

```ts
{
  id: "brasa-supreme",
  name: "Brasa Supreme",
  image: "/images/products/brasa-supreme.webp"
}
```

Nunca incluir a imagem diretamente no objeto como Base64.

---

## 13. CSS e Tailwind

Evitar classes ou estilos repetidos sem necessidade.

Criar componentes reutilizáveis para:

- botão principal;
- campo de formulário;
- card;
- badge;
- estado indisponível.

Evitar:

- arquivos CSS enormes;
- estilos duplicados;
- muitos efeitos visuais;
- sombras pesadas;
- filtros complexos;
- imagens de fundo grandes.

---

## 14. Carregamento inicial

A primeira tela deve carregar apenas o necessário para o cliente começar a navegar.

Priorizar na abertura:

- cabeçalho;
- categorias;
- primeiros produtos;
- botão do carrinho;
- estilos essenciais.

Carregar depois:

- checkout;
- tela de resumo;
- confirmação final;
- componentes menos usados.

---

## 15. Build de produção

Antes de publicar, executar:

```bash
npm run build
```

O Vite deverá:

- minificar o código;
- separar arquivos;
- remover código não utilizado;
- otimizar os assets.

Também executar, quando disponíveis:

```bash
npm run lint
npm run test
npm run build
```

Não considerar o projeto pronto apenas porque funciona com `npm run dev`.

---

## 16. Metas de desempenho

Buscar manter:

```text
Imagem individual: até 60 KB
Primeira tela: até 1 MB
JavaScript inicial comprimido: abaixo de 250 KB
```

Esses valores são metas, não regras absolutas.

Quando forem ultrapassados, investigar:

- imagens pesadas;
- bibliotecas desnecessárias;
- imports grandes;
- código duplicado;
- páginas carregadas antecipadamente.

---

## 17. Auditoria

Antes da publicação, verificar o site com:

- Lighthouse;
- aba Network do navegador;
- aba Performance;
- visualização mobile.

Avaliar:

- peso total carregado;
- imagens maiores;
- JavaScript inicial;
- tempo até a primeira renderização;
- mudanças de layout;
- carregamento em internet lenta.

---

## 18. Critérios de aceitação

O projeto estará adequado quando:

- imagens forem WebP ou AVIF;
- imagens não estiverem maiores que o necessário;
- cards usarem lazy loading;
- o carrinho persistir sem armazenar imagens;
- não houver dependências desnecessárias;
- páginas secundárias forem carregadas sob demanda;
- o build terminar sem erros;
- o site funcionar bem em celular;
- não houver lentidão perceptível ao navegar;
- a primeira tela carregar rapidamente;
- o WhatsApp abrir sem bloquear a interface.

---

## 19. Regra principal

Sempre priorizar:

```text
Imagens pequenas
+
poucas dependências
+
código dividido
+
estado enxuto
+
carregamento sob demanda
```

Não adicionar complexidade sem ganho real para o MVP.
