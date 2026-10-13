# Guia do frontend — Mulheres na Ciência

**Oficina Da Ideia à Aplicação · BVApp**

**Next.js + React + TypeScript + Tailwind CSS + Untitled UI + Dialog**

Vamos construir o frontend do catálogo de pesquisadoras, áreas acadêmicas e publicações, consumindo a API Flask da oficina. O backend valida e persiste os dados.

O guia está dividido em duas partes: configuração do projeto e acesso à API; depois, interface e integração. Adicione somente as famílias dos componentes utilizados e preserve seus auxiliares. A etapa 13 identifica os componentes extras que não são alcançados pelos imports da aplicação.

Se estiver usando a pasta `frontend` já entregue, execute `npm install` e configure `.env.local`. Os arquivos da aplicação já existem; não execute `init` nem `add` novamente. As etapas de criação abaixo servem para reproduzir o projeto do início.

## Sumário

- [1. Preparar o ambiente](#etapa-1)
- [2. Criar o projeto Next.js](#etapa-2)
- [3. Adicionar os componentes e configurar o tema](#etapa-3)
- [4. Organizar os arquivos e configurar o design](#etapa-4)
- [5. Configurar o acesso à API](#etapa-5)
- [6. Definir os tipos](#etapa-6)
- [7. Criar o cliente HTTP](#etapa-7)
- [8. Criar o formulário reutilizável](#etapa-8)
- [9. Definir os campos](#etapa-9)
- [10. Preparar os modais](#etapa-10)
- [11. Criar a tabela reutilizável e o perfil](#etapa-11)
- [12. Montar as abas, formulários e consultas](#etapa-12)
- [13. Exibir a página, limpar os componentes extras e executar](#etapa-13)
- [14. Executar o fluxo da oficina](#etapa-14)
- [15. Relacionar a interface aos requisitos](#etapa-15)
- [16. Fluxo da aplicação](#etapa-16)
- [Comandos necessários](#comandos-necessarios)
- [Resumo das responsabilidades](#responsabilidades)
- [Problemas comuns](#problemas-comuns)
- [Referências](#referencias)

## Etapas e entregas

| Etapa | Resultado esperado |
| --- | --- |
| Parte 1 — Etapas 1 a 7 | Ambiente, projeto Next.js, componentes, tema, estrutura, tipos e cliente HTTP. |
| Parte 2 — Etapas 8 a 15 | Formulários, campos, modais, tabelas, abas e integração com os requisitos. |

## Tecnologias

| Tecnologia | Finalidade |
| --- | --- |
| Node.js e npm | Executar o frontend e gerenciar as dependências. |
| Next.js | Organizar páginas com App Router e encaminhar chamadas à API Flask. |
| React | Construir a interface com componentes e estados. |
| TypeScript | Definir os tipos dos dados, formulários e respostas. |
| Tailwind CSS e PostCSS | Processar os estilos e aplicar o design das telas. |
| Untitled UI e React Aria | Fornecer campos, botões, Selects, abas e tabelas acessíveis. |
| Radix Dialog | Criar modais e confirmações de exclusão. |
| next-themes | Configurar o tema da interface. |
| API Flask | Validar os dados, executar as regras e persistir os registros no backend. |

<a id="estrutura-do-projeto"></a>

## Estrutura do Projeto

```text
src/
  app/
    layout.tsx
    page.tsx
  styles/
    globals.css
    theme.css
    typography.css
  providers/
    route-provider.tsx
    theme-provider.tsx
  utils/
    cx.ts
    is-react-component.ts
  hooks/
    use-resize-observer.ts
  lib/
    types.ts
    api.ts
    campos.ts
    dialog-portal.ts
  components/
    base/
    application/
    foundations/
    ui/
      dialog.tsx
      dialog-box.tsx
    oficina/
      formulario.tsx
      modal.tsx
      excluir-pesquisadora.tsx
      tabela.tsx
      perfil.tsx
      catalogo.tsx
scripts/
  limpar-componentes.cjs
```

## O que vamos construir

- Três abas principais: **Pesquisadoras**, **Publicações** e **Áreas acadêmicas**.
- Cadastro, visualização, edição e exclusão de pesquisadoras em modais.
- Cadastro e listagem de áreas acadêmicas.
- Painel de cadastro de publicação, vinculado a uma pesquisadora e uma área.
- Consulta de publicações no perfil, incluindo a área acadêmica.
- Cadastro de publicação pelo perfil, com a pesquisadora previamente selecionada.
- Formulários em uma coluna, tabelas reutilizáveis e estados de erro, carregamento e sucesso.
- Sem barras de pesquisa.

A aba Publicações é um painel de cadastro; a consulta permanece no perfil porque a API oferece listagem por pesquisadora. O cadastro da pesquisadora não possui Select de associação.

## Parte 1 — Projeto e acesso à API

<a id="etapa-1"></a>

### 1. Preparar o ambiente

Utilize Node.js **22.12 ou superior**, npm e um editor. Esse requisito acompanha o `package.json` do frontend entregue. Com Node 18, o build e o servidor do Next.js não executam. Confira também Docker Desktop e uv para o backend.

```powershell
node --version
npm --version
```

Use a branch **`pesquisadoras-api`** do backend da oficina, que já contém a configuração do banco e as migrações. Na integração local deste repositório, pesquisadoras usa frontend na porta **3000**, API na **5000** e PostgreSQL na **5432**. Eventos pode continuar em 3001, 5001 e 5433, em sua própria pasta.

Em uma máquina onde as dependências Python sejam permitidas, no terminal do backend fornecido:

```powershell
uv sync
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
docker compose up -d --wait db
uv run flask --app backend db upgrade
uv run flask --app backend run --debug --host 127.0.0.1 --port 5000
```

Esses comandos são para PowerShell.

Mantenha o terminal aberto. A documentação da API ficará em:

```text
http://localhost:5000/docs
```

As migrações já existem no projeto: não execute `db init` novamente.

#### Backend em Docker neste Windows

Neste computador, o Controle de Aplicativo bloqueia extensões nativas do Python. A versão do backend em `Oficina_Da-Ideia-A-Aplicacao/` agora contém `Dockerfile`, `.dockerignore` e `compose.override.yaml`, que adicionam um serviço Linux para a API e usam o banco já configurado no `compose.yaml`.

Com o Docker Desktop aberto, execute dentro dessa pasta:

```powershell
docker compose up -d --wait db
docker compose build backend
docker compose run --rm backend flask --app backend db upgrade
docker compose up -d --wait backend
```

A aplicação fica em `http://localhost:5000`, e a documentação em `http://localhost:5000/docs`. O frontend continua usando `API_URL=http://127.0.0.1:5000`.

Para acompanhar os logs:

```powershell
docker compose logs -f backend
```

Nesse modo, os comandos Flask devem executar no container. Para aplicar novas migrações com o backend ativo, utilize `docker compose exec backend flask --app backend db upgrade`. Não execute `uv run flask` no Windows enquanto as extensões nativas estiverem bloqueadas.

A configuração monta `src/` e `migrations/` no container e deixa a API em modo de desenvolvimento com recarga automática. Alterações nas dependências exigem uma nova execução de `docker compose build backend` e `docker compose up -d --wait backend`.

#### Contrato utilizado neste guia

Os caminhos e campos abaixo foram conferidos nas rotas e schemas da branch `pesquisadoras-api` disponível neste repositório.

| Método | Rota | Finalidade |
| --- | --- | --- |
| POST | `/pesquisadora` | Cadastrar pesquisadora |
| GET | `/pesquisadora` | Listar pesquisadoras |
| GET | `/pesquisadora/{id}` | Consultar perfil e publicações |
| PUT | `/pesquisadora/{id}` | Editar pesquisadora |
| DELETE | `/pesquisadora/{id}` | Excluir pesquisadora e publicações vinculadas |
| POST | `/area-academica` | Cadastrar área acadêmica |
| GET | `/area-academica` | Listar áreas acadêmicas |
| POST | `/publicacao` | Cadastrar publicação |
| GET | `/publicacao/pesquisadora/{id}` | Listar publicações de uma pesquisadora |

O frontend considera listagens como arrays, IDs numéricos, campos em `snake_case` e datas em `YYYY-MM-DD`. Cada publicação recebe `pesquisadora_id` e `area_academica_id`; a resposta também contém o objeto `area_academica`. Links opcionais são enviados como `null` quando vazios. O DELETE bem-sucedido retorna `204` sem corpo.

Não há rota GET para listar todas as publicações nem rotas de edição/exclusão de áreas e publicações no contrato utilizado. Por isso a aba Publicações contém um painel de cadastro, e a consulta ocorre no perfil. Ao utilizar outra versão do backend, confira o contrato em `/docs` antes de integrar.

#### Campos e limites confirmados

| Cadastro | Obrigatórios | Limites e opcionais |
| --- | --- | --- |
| Pesquisadora | nome, instituição, titularidade, e-mail e biografia | Nome: 70; instituição: 50; titularidade: 12; e-mail: 100 caracteres. Lattes é URL opcional. |
| Área acadêmica | nome e descrição | Nome: 30 caracteres. |
| Publicação | título, resumo, situação, data, pesquisadora e área | Título: 100; situação: 10 caracteres. Link é URL opcional; os dois IDs são inteiros positivos. |

Titularidade e situação são campos de texto: o backend não define opções fechadas. Use “publicada” como exemplo de situação. A validação da interface complementa a validação feita pelo backend.

<a id="etapa-2"></a>

### 2. Criar o projeto Next.js

Em outro terminal:

```powershell
npx create-next-app@latest frontend --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-npm
cd frontend
```

O comando instala Next.js, React e React DOM e configura o Tailwind CSS.

Instale as dependências da interface:

```powershell
npm install react-aria-components @untitledui/icons tailwind-merge tailwindcss-animate tailwindcss-react-aria-components next-themes lucide-react @radix-ui/react-dialog
```

A etapa 3 contém os códigos de tema, tipografia, PostCSS e auxiliares necessários. Não deixe esses arquivos vazios.

Na raiz do frontend, crie `components.json`:

```json
{
  "version": "8",
  "aliases": {
    "components": "@/components/",
    "utils": "@/utils/",
    "hooks": "@/hooks/",
    "styles": "@/styles/"
  }
}
```

<a id="etapa-3"></a>

### 3. Adicionar os componentes e configurar o tema

#### 3.1. Adicionar os componentes

Execute na raiz do frontend:

```powershell
npx untitledui@latest add input button textarea select --path src/components
npx untitledui@latest add tabs table --path src/components
```

Não utilize `--all`, `--include-all-components` ou `--yes`. Se a CLI perguntar sobre adicionar todos os componentes ou exemplos, recuse e mantenha apenas as famílias listadas.

Preserve os componentes auxiliares importados pelos componentes selecionados.

| Componente    | Referência                                                                               | Uso                             |
| ------------- | ---------------------------------------------------------------------------------------- | ------------------------------- |
| Input         | [Inputs](https://www.untitledui.com/react/components/inputs)                             | Campos curtos                   |
| Button        | [Buttons](https://www.untitledui.com/react/components/buttons)                           | Ações                           |
| TextArea      | [Textareas](https://www.untitledui.com/react/components/textareas)                       | Biografia, descrição e resumo              |
| Select        | [Select](https://www.untitledui.com/react/components/select)                             | Pesquisadora e área acadêmica            |
| Tabs          | [Tabs](https://www.untitledui.com/react/components/tabs)                                 | Abas principais                 |
| Table         | [Tables](https://www.untitledui.com/react/components/tables)                             | Listagens                       |
| Dialog Box    | [Dialog Box](https://21st.dev/@itsankitverma/components/dialog-box)                      | Cadastro, edição e visualização |
| Delete Dialog | [Delete Dialog](https://21st.dev/@felipemenezes098/components/the-dialog/delete-account) | Confirmação de exclusão         |

Utilize dois **Selects simples** no cadastro de publicação: uma pesquisadora e uma área acadêmica. O cadastro de pesquisadora não possui seleção de associações. MultiSelect só seria necessário se o contrato permitisse múltiplas associações no mesmo cadastro.

A CLI adiciona as famílias dos componentes selecionados, incluindo variações e auxiliares. Os arquivos extras que não forem utilizados pela aplicação serão identificados na etapa 13.

#### 3.2. Tema, tipografia e processamento do CSS

Crie `src/styles` e preencha os três arquivos abaixo. Esta versão inclui os tokens completos da base aberta do Untitled UI, mantendo as personalizações da oficina em `globals.css`.

Os estilos de tipografia incluem a classe `prose`, por isso esta configuração também utiliza o plugin `@tailwindcss/typography`. Ele é um plugin de CSS; não adiciona uma coleção de componentes ao projeto. Na aplicação enxuta já entregue, o plugin pode estar ausente porque as classes `prose` não são utilizadas. Ao reproduzir os estilos completos abaixo em um projeto novo, instale-o para compilar esta versão da folha de tipografia.

Confira as dependências:

```powershell
npm install -D tailwindcss @tailwindcss/postcss postcss @tailwindcss/typography
npm install tailwindcss-react-aria-components tailwindcss-animate
```

Mantenha somente um arquivo de configuração do PostCSS na raiz do frontend.

**`postcss.config.mjs`:**

```js
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

##### Tokens do tema

**`src/styles/theme.css`:**

```css
@theme {
  /* FONT FAMILY */
  --font-body:
    var(--font-inter, "Inter"), -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
  --font-display:
    var(--font-inter, "Inter"), -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
  --font-mono:
    ui-monospace, "Roboto Mono", SFMono-Regular, Menlo, Monaco, Consolas,
    "Liberation Mono", "Courier New", monospace;

  /* FONT SIZE */
  --text-xs: calc(var(--spacing) * 3);
  --text-xs--line-height: calc(var(--spacing) * 4.5);

  --text-sm: calc(var(--spacing) * 3.5);
  --text-sm--line-height: calc(var(--spacing) * 5);

  --text-md: calc(var(--spacing) * 4);
  --text-md--line-height: calc(var(--spacing) * 6);

  --text-lg: calc(var(--spacing) * 4.5);
  --text-lg--line-height: calc(var(--spacing) * 7);

  --text-xl: calc(var(--spacing) * 5);
  --text-xl--line-height: calc(var(--spacing) * 7.5);

  --text-display-xs: calc(var(--spacing) * 6);
  --text-display-xs--line-height: calc(var(--spacing) * 8);

  --text-display-sm: calc(var(--spacing) * 7.5);
  --text-display-sm--line-height: calc(var(--spacing) * 9.5);

  --text-display-md: calc(var(--spacing) * 9);
  --text-display-md--line-height: calc(var(--spacing) * 11);
  --text-display-md--letter-spacing: -0.72px;

  --text-display-lg: calc(var(--spacing) * 12);
  --text-display-lg--line-height: calc(var(--spacing) * 15);
  --text-display-lg--letter-spacing: -0.96px;

  --text-display-xl: calc(var(--spacing) * 15);
  --text-display-xl--line-height: calc(var(--spacing) * 18);
  --text-display-xl--letter-spacing: -1.2px;

  --text-display-2xl: calc(var(--spacing) * 18);
  --text-display-2xl--line-height: calc(var(--spacing) * 22.5);
  --text-display-2xl--letter-spacing: -1.44px;

  /* MAX WIDTH */
  --max-width-container: 1280px;

  /* BREAKPOINTS */
  --breakpoint-xxs: 320px;
  /* This must match the breakpoint in Sonner: https://github.com/emilkowalski/sonner/blob/main/src/styles.css */
  --breakpoint-xs: 600px;

  /* SHADOW */
  --shadow-xs: 0px 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-sm: 0px 1px 3px rgba(0, 0, 0, 0.1), 0px 1px 2px -1px rgba(0, 0, 0, 0.1);
  --shadow-md: 0px 4px 6px -1px rgba(0, 0, 0, 0.1), 0px 2px 4px -2px rgba(0, 0, 0, 0.06);
  --shadow-lg:
    0px 12px 16px -4px rgba(0, 0, 0, 0.08), 0px 4px 6px -2px rgba(0, 0, 0, 0.03),
    0px 2px 2px -1px rgba(0, 0, 0, 0.04);
  --shadow-xl:
    0px 20px 24px -4px rgba(0, 0, 0, 0.08), 0px 8px 8px -4px rgba(0, 0, 0, 0.03),
    0px 3px 3px -1.5px rgba(0, 0, 0, 0.04);
  --shadow-2xl:
    0px 24px 48px -12px rgba(0, 0, 0, 0.18), 0px 4px 4px -2px rgba(0, 0, 0, 0.04);
  --shadow-3xl:
    0px 32px 64px -12px rgba(0, 0, 0, 0.14), 0px 5px 5px -2.5px rgba(0, 0, 0, 0.04);

  --shadow-skeuomorphic:
    0px 0px 0px 1px rgba(0, 0, 0, 0.18) inset, 0px -2px 0px 0px rgba(0, 0, 0, 0.05) inset;
  --shadow-xs-skeuomorphic: var(--shadow-skeuomorphic), var(--shadow-xs);

  --shadow-modern-mockup-inner-lg:
    0px 0px 3.765px 1.255px rgba(10, 13, 18, 0.08) inset,
    0px 0px 2.51px 1.255px rgba(10, 13, 18, 0.03) inset;
  --shadow-modern-mockup-inner-md:
    0px 0px 1.692px 0.564px rgba(10, 13, 18, 0.08) inset,
    0px 0px 1.128px 0.564px rgba(10, 13, 18, 0.03) inset;
  --shadow-modern-mockup-inner-sm:
    0px 0px 4.48px 1.493px rgba(10, 13, 18, 0.08) inset,
    0px 0px 2.987px 1.493px rgba(10, 13, 18, 0.03) inset;

  --shadow-modern-mockup-outer-lg:
    0px 7.529px 10.039px -2.51px rgba(10, 13, 18, 0.08),
    0px 2.51px 3.765px -1.255px rgba(10, 13, 18, 0.03),
    0px 1.255px 1.255px -0.627px rgba(10, 13, 18, 0.04);
  --shadow-modern-mockup-outer-md:
    0px 3.385px 4.513px -1.128px rgba(10, 13, 18, 0.08),
    0px 1.128px 1.692px -0.564px rgba(10, 13, 18, 0.03),
    0px 0.564px 0.564px -0.282px rgba(10, 13, 18, 0.04);

  --drop-shadow-iphone-mockup: 20px 12px 18px rgba(16, 24, 40, 0.2);

  /* ANIMATIONS */
  --animate-marquee: marquee 60s linear infinite;
  --animate-caret-blink: caret-blink 1s infinite;

  @keyframes marquee {
    0% {
      transform: translateX(0);
    }
    100% {
      transform: translateX(-100%);
    }
  }

  @keyframes caret-blink {
    0%,
    50% {
      opacity: 1;
    }
    51%,
    100% {
      opacity: 0;
    }
  }

  /* BASE COLOR */
  --color-transparent: rgb(0 0 0 / 0);

  /* These will be inverted in dark mode. */
  --color-alpha-white: rgb(255 255 255);
  --color-alpha-black: rgb(0 0 0);

  --color-brand-50: rgb(249 245 255);
  --color-brand-100: rgb(244 235 255);
  --color-brand-200: rgb(233 215 254);
  --color-brand-300: rgb(214 187 251);
  --color-brand-400: rgb(182 146 246);
  --color-brand-500: rgb(158 119 237);
  --color-brand-600: rgb(127 86 217);
  --color-brand-700: rgb(105 65 198);
  --color-brand-800: rgb(83 56 158);
  --color-brand-900: rgb(66 48 125);
  --color-brand-950: rgb(44 28 95);

  /* LIGHT MODE VARIABLES */

  --color-alpha-white: rgb(255 255 255);
  --color-alpha-black: rgb(0 0 0);

  /* UTILITY COLORS */
  --color-utility-blue-50: var(--color-blue-50);
  --color-utility-blue-100: var(--color-blue-100);
  --color-utility-blue-200: var(--color-blue-200);
  --color-utility-blue-300: var(--color-blue-300);
  --color-utility-blue-400: var(--color-blue-400);
  --color-utility-blue-500: var(--color-blue-500);
  --color-utility-blue-600: var(--color-blue-600);
  --color-utility-blue-700: var(--color-blue-700);

  --color-utility-brand-50: var(--color-brand-50);
  --color-utility-brand-50_alt: var(--color-brand-50);
  --color-utility-brand-100: var(--color-brand-100);
  --color-utility-brand-100_alt: var(--color-brand-100);
  --color-utility-brand-200: var(--color-brand-200);
  --color-utility-brand-200_alt: var(--color-brand-200);
  --color-utility-brand-300: var(--color-brand-300);
  --color-utility-brand-300_alt: var(--color-brand-300);
  --color-utility-brand-400: var(--color-brand-400);
  --color-utility-brand-400_alt: var(--color-brand-400);
  --color-utility-brand-500: var(--color-brand-500);
  --color-utility-brand-500_alt: var(--color-brand-500);
  --color-utility-brand-600: var(--color-brand-600);
  --color-utility-brand-600_alt: var(--color-brand-600);
  --color-utility-brand-700: var(--color-brand-700);
  --color-utility-brand-700_alt: var(--color-brand-700);
  --color-utility-brand-800: var(--color-brand-800);
  --color-utility-brand-800_alt: var(--color-brand-800);
  --color-utility-brand-900: var(--color-brand-900);
  --color-utility-brand-900_alt: var(--color-brand-900);

  --color-utility-neutral-50: var(--color-neutral-50);
  --color-utility-neutral-100: var(--color-neutral-100);
  --color-utility-neutral-200: var(--color-neutral-200);
  --color-utility-neutral-300: var(--color-neutral-300);
  --color-utility-neutral-400: var(--color-neutral-400);
  --color-utility-neutral-500: var(--color-neutral-500);
  --color-utility-neutral-600: var(--color-neutral-600);
  --color-utility-neutral-700: var(--color-neutral-700);
  --color-utility-neutral-800: var(--color-neutral-800);
  --color-utility-neutral-900: var(--color-neutral-900);

  --color-utility-red-50: var(--color-red-50);
  --color-utility-red-100: var(--color-red-100);
  --color-utility-red-200: var(--color-red-200);
  --color-utility-red-300: var(--color-red-300);
  --color-utility-red-400: var(--color-red-400);
  --color-utility-red-500: var(--color-red-500);
  --color-utility-red-600: var(--color-red-600);
  --color-utility-red-700: var(--color-red-700);

  --color-utility-yellow-50: var(--color-yellow-50);
  --color-utility-yellow-100: var(--color-yellow-100);
  --color-utility-yellow-200: var(--color-yellow-200);
  --color-utility-yellow-300: var(--color-yellow-300);
  --color-utility-yellow-400: var(--color-yellow-400);
  --color-utility-yellow-500: var(--color-yellow-500);
  --color-utility-yellow-600: var(--color-yellow-600);
  --color-utility-yellow-700: var(--color-yellow-700);

  --color-utility-green-50: var(--color-green-50);
  --color-utility-green-100: var(--color-green-100);
  --color-utility-green-200: var(--color-green-200);
  --color-utility-green-300: var(--color-green-300);
  --color-utility-green-400: var(--color-green-400);
  --color-utility-green-500: var(--color-green-500);
  --color-utility-green-600: var(--color-green-600);
  --color-utility-green-700: var(--color-green-700);

  --color-utility-orange-50: var(--color-orange-50);
  --color-utility-orange-100: var(--color-orange-100);
  --color-utility-orange-200: var(--color-orange-200);
  --color-utility-orange-300: var(--color-orange-300);
  --color-utility-orange-400: var(--color-orange-400);
  --color-utility-orange-500: var(--color-orange-500);
  --color-utility-orange-600: var(--color-orange-600);
  --color-utility-orange-700: var(--color-orange-700);

  --color-utility-indigo-50: var(--color-indigo-50);
  --color-utility-indigo-100: var(--color-indigo-100);
  --color-utility-indigo-200: var(--color-indigo-200);
  --color-utility-indigo-300: var(--color-indigo-300);
  --color-utility-indigo-400: var(--color-indigo-400);
  --color-utility-indigo-500: var(--color-indigo-500);
  --color-utility-indigo-600: var(--color-indigo-600);
  --color-utility-indigo-700: var(--color-indigo-700);

  --color-utility-fuchsia-50: var(--color-fuchsia-50);
  --color-utility-fuchsia-100: var(--color-fuchsia-100);
  --color-utility-fuchsia-200: var(--color-fuchsia-200);
  --color-utility-fuchsia-300: var(--color-fuchsia-300);
  --color-utility-fuchsia-400: var(--color-fuchsia-400);
  --color-utility-fuchsia-500: var(--color-fuchsia-500);
  --color-utility-fuchsia-600: var(--color-fuchsia-600);
  --color-utility-fuchsia-700: var(--color-fuchsia-700);

  --color-utility-pink-50: var(--color-pink-50);
  --color-utility-pink-100: var(--color-pink-100);
  --color-utility-pink-200: var(--color-pink-200);
  --color-utility-pink-300: var(--color-pink-300);
  --color-utility-pink-400: var(--color-pink-400);
  --color-utility-pink-500: var(--color-pink-500);
  --color-utility-pink-600: var(--color-pink-600);
  --color-utility-pink-700: var(--color-pink-700);

  --color-utility-purple-50: var(--color-purple-50);
  --color-utility-purple-100: var(--color-purple-100);
  --color-utility-purple-200: var(--color-purple-200);
  --color-utility-purple-300: var(--color-purple-300);
  --color-utility-purple-400: var(--color-purple-400);
  --color-utility-purple-500: var(--color-purple-500);
  --color-utility-purple-600: var(--color-purple-600);
  --color-utility-purple-700: var(--color-purple-700);

  --color-utility-sky-50: var(--color-sky-50);
  --color-utility-sky-100: var(--color-sky-100);
  --color-utility-sky-200: var(--color-sky-200);
  --color-utility-sky-300: var(--color-sky-300);
  --color-utility-sky-400: var(--color-sky-400);
  --color-utility-sky-500: var(--color-sky-500);
  --color-utility-sky-600: var(--color-sky-600);
  --color-utility-sky-700: var(--color-sky-700);

  --color-utility-slate-50: var(--color-slate-50);
  --color-utility-slate-100: var(--color-slate-100);
  --color-utility-slate-200: var(--color-slate-200);
  --color-utility-slate-300: var(--color-slate-300);
  --color-utility-slate-400: var(--color-slate-400);
  --color-utility-slate-500: var(--color-slate-500);
  --color-utility-slate-600: var(--color-slate-600);
  --color-utility-slate-700: var(--color-slate-700);

  --color-utility-emerald-50: var(--color-emerald-50);
  --color-utility-emerald-100: var(--color-emerald-100);
  --color-utility-emerald-200: var(--color-emerald-200);
  --color-utility-emerald-300: var(--color-emerald-300);
  --color-utility-emerald-400: var(--color-emerald-400);
  --color-utility-emerald-500: var(--color-emerald-500);
  --color-utility-emerald-600: var(--color-emerald-600);
  --color-utility-emerald-700: var(--color-emerald-700);

  --color-utility-amber-50: var(--color-amber-50);
  --color-utility-amber-100: var(--color-amber-100);
  --color-utility-amber-200: var(--color-amber-200);
  --color-utility-amber-300: var(--color-amber-300);
  --color-utility-amber-400: var(--color-amber-400);
  --color-utility-amber-500: var(--color-amber-500);
  --color-utility-amber-600: var(--color-amber-600);
  --color-utility-amber-700: var(--color-amber-700);

  /* TEXT COLORS */
  --color-text-primary: var(--color-neutral-900);
  --color-text-tertiary: var(--color-neutral-600);
  --color-text-error-primary: var(--color-red-600);
  --color-text-warning-primary: var(--color-yellow-600);
  --color-text-success-primary: var(--color-green-600);
  --color-text-secondary: var(--color-neutral-700);
  --color-text-secondary_hover: var(--color-neutral-800);
  --color-text-tertiary_hover: var(--color-neutral-700);
  --color-text-brand-secondary: var(--color-brand-700);
  --color-text-placeholder: var(--color-neutral-500);
  --color-text-brand-tertiary: var(--color-brand-600);
  --color-text-editor-icon-fg: var(--color-neutral-400);
  --color-text-editor-icon-fg_active: var(--color-neutral-500);
  --color-text-quaternary: var(--color-neutral-500);
  --color-text-brand-primary: var(--color-brand-900);
  --color-text-primary_on-brand: var(--color-white);
  --color-text-secondary_on-brand: var(--color-brand-200);
  --color-text-tertiary_on-brand: var(--color-brand-200);
  --color-text-quaternary_on-brand: var(--color-brand-300);
  --color-text-brand-tertiary_alt: var(--color-brand-600);
  --color-text-error-primary_hover: var(--color-red-700);
  --color-text-brand-secondary_hover: var(--color-brand-800);

  /* BORDER COLORS */
  --color-border-primary: var(--color-neutral-300);
  --color-border-secondary: var(--color-neutral-200);
  --color-border-secondary_alt: rgb(0 0 0 / 0.1);
  --color-border-tertiary: var(--color-neutral-100);
  --color-border-error: var(--color-red-500);
  --color-border-error_subtle: var(--color-red-300);
  --color-border-brand: var(--color-brand-500);
  --color-border-brand_alt: var(--color-brand-600);

  /* FOREGROUND COLORS */
  --color-fg-secondary: var(--color-neutral-700);
  --color-fg-warning-primary: var(--color-yellow-600);
  --color-fg-success-primary: var(--color-green-600);
  --color-fg-white: var(--color-white);
  --color-fg-success-secondary: var(--color-green-500);
  --color-fg-secondary_hover: var(--color-neutral-800);
  --color-fg-primary: var(--color-neutral-900);
  --color-fg-brand-secondary: var(--color-brand-500);
  --color-fg-brand-primary: var(--color-brand-600);
  --color-fg-quaternary: var(--color-neutral-400);
  --color-fg-quaternary_hover: var(--color-neutral-500);
  --color-fg-error-primary: var(--color-red-600);
  --color-fg-warning-secondary: var(--color-yellow-500);
  --color-fg-error-secondary: var(--color-red-500);
  --color-fg-tertiary: var(--color-neutral-600);
  --color-fg-tertiary_hover: var(--color-neutral-700);
  --color-fg-brand-primary_alt: var(--color-fg-brand-primary);
  --color-fg-brand-secondary_alt: var(--color-fg-brand-secondary);
  --color-fg-brand-secondary_hover: var(--color-brand-600);

  /* BACKGROUND COLORS */
  --color-bg-primary: var(--color-white);
  --color-bg-tertiary: var(--color-neutral-100);
  --color-bg-brand-primary: var(--color-brand-50);
  --color-bg-error-secondary: var(--color-red-100);
  --color-bg-warning-primary: var(--color-yellow-50);
  --color-bg-warning-secondary: var(--color-yellow-100);
  --color-bg-success-primary: var(--color-green-50);
  --color-bg-success-secondary: var(--color-green-100);
  --color-bg-brand-solid: var(--color-brand-600);
  --color-bg-secondary-solid: var(--color-neutral-600);
  --color-bg-error-solid: var(--color-red-600);
  --color-bg-warning-solid: var(--color-yellow-600);
  --color-bg-success-solid: var(--color-green-600);
  --color-bg-secondary_hover: var(--color-neutral-100);
  --color-bg-primary_hover: var(--color-neutral-50);
  --color-bg-brand-solid_hover: var(--color-brand-700);
  --color-bg-error-primary: var(--color-red-50);
  --color-bg-brand-secondary: var(--color-brand-100);
  --color-bg-secondary: var(--color-neutral-50);
  --color-bg-quaternary: var(--color-neutral-200);
  --color-bg-primary_alt: var(--color-white);
  --color-bg-brand-primary_alt: var(--color-brand-50);
  --color-bg-secondary_alt: var(--color-neutral-50);
  --color-bg-overlay: var(--color-neutral-950);
  --color-bg-brand-section: var(--color-brand-800);
  --color-bg-brand-section_subtle: var(--color-brand-700);
  --color-bg-primary-solid: var(--color-neutral-950);
  --color-bg-error-solid_hover: var(--color-red-700);

  /* COMPONENT COLORS */
  --color-app-store-badge-border: 166 166 166 1;
  --color-avatar-styles-bg-neutral: 224 224 224 1;
  --color-featured-icon-light-fg-brand: var(--color-brand-600);
  --color-featured-icon-light-fg-error: var(--color-red-600);
  --color-featured-icon-light-fg-gray: var(--color-neutral-500);
  --color-featured-icon-light-fg-success: var(--color-green-600);
  --color-featured-icon-light-fg-warning: var(--color-yellow-600);
  --color-focus-ring-error: var(--color-red-500);
  --color-focus-ring: var(--color-brand-500);
  --color-footer-button-fg: var(--color-brand-200);
  --color-footer-button-fg_hover: var(--color-white);
  --color-icon-fg-brand: var(--color-brand-600);
  --color-icon-fg-brand_on-brand: var(--color-brand-200);
  --color-screen-mockup-border: var(--color-neutral-900);

  --color-slider-handle-bg: var(--color-white);
  --color-slider-handle-border: var(--color-brand-600);
  --color-toggle-border: var(--color-neutral-300);
  --color-toggle-slim-border_pressed-hover: var(--color-bg-brand-solid_hover);
  --color-toggle-slim-border_pressed: var(--color-bg-brand-solid);
  --color-tooltip-supporting-text: var(--color-neutral-300);

  /* BACKGROUND PROPERTY COLORS */
  --background-color-primary: var(--color-bg-primary);
  --background-color-tertiary: var(--color-bg-tertiary);
  --background-color-brand-primary: var(--color-bg-brand-primary);
  --background-color-error-secondary: var(--color-bg-error-secondary);
  --background-color-warning-primary: var(--color-bg-warning-primary);
  --background-color-warning-secondary: var(--color-bg-warning-secondary);
  --background-color-success-primary: var(--color-bg-success-primary);
  --background-color-success-secondary: var(--color-bg-success-secondary);
  --background-color-brand-solid: var(--color-bg-brand-solid);
  --background-color-secondary-solid: var(--color-bg-secondary-solid);
  --background-color-error-solid: var(--color-bg-error-solid);
  --background-color-warning-solid: var(--color-bg-warning-solid);
  --background-color-success-solid: var(--color-bg-success-solid);
  --background-color-secondary_hover: var(--color-bg-secondary_hover);
  --background-color-primary_hover: var(--color-bg-primary_hover);
  --background-color-brand-solid_hover: var(--color-bg-brand-solid_hover);
  --background-color-error-primary: var(--color-bg-error-primary);
  --background-color-brand-secondary: var(--color-bg-brand-secondary);
  --background-color-secondary: var(--color-bg-secondary);
  --background-color-quaternary: var(--color-bg-quaternary);
  --background-color-primary_alt: var(--color-bg-primary_alt);
  --background-color-brand-primary_alt: var(--color-bg-brand-primary_alt);
  --background-color-secondary_alt: var(--color-bg-secondary_alt);
  --background-color-overlay: var(--color-bg-overlay);
  --background-color-brand-section: var(--color-bg-brand-section);
  --background-color-brand-section_subtle: var(--color-bg-brand-section_subtle);
  --background-color-primary-solid: var(--color-bg-primary-solid);
  --background-color-error-solid_hover: var(--color-bg-error-solid_hover);
  --background-color-border-brand: var(--color-border-brand);
  --background-color-border-brand_alt: var(--color-border-brand_alt);

  /* TEXT PROPERTY COLORS */
  --color-text-white: var(--color-white);
  --text-color-primary: var(--color-text-primary);
  --text-color-secondary: var(--color-text-secondary);
  --text-color-secondary_hover: var(--color-text-secondary_hover);
  --text-color-tertiary: var(--color-text-tertiary);
  --text-color-tertiary_hover: var(--color-text-tertiary_hover);
  --text-color-error-primary: var(--color-text-error-primary);
  --text-color-warning-primary: var(--color-text-warning-primary);
  --text-color-success-primary: var(--color-text-success-primary);
  --text-color-brand-secondary: var(--color-text-brand-secondary);
  --text-color-placeholder: var(--color-text-placeholder);
  --text-color-brand-tertiary: var(--color-text-brand-tertiary);
  --text-color-editor-icon-fg: var(--color-text-editor-icon-fg);
  --text-color-editor-icon-fg_active: var(--color-text-editor-icon-fg_active);
  --text-color-quaternary: var(--color-text-quaternary);
  --text-color-brand-primary: var(--color-text-brand-primary);
  --text-color-primary_on-brand: var(--color-text-primary_on-brand);
  --text-color-secondary_on-brand: var(--color-text-secondary_on-brand);
  --text-color-tertiary_on-brand: var(--color-text-tertiary_on-brand);
  --text-color-quaternary_on-brand: var(--color-text-quaternary_on-brand);
  --text-color-brand-tertiary_alt: var(--color-text-brand-tertiary_alt);
  --text-color-error-primary_hover: var(--color-text-error-primary_hover);
  --text-color-brand-secondary_hover: var(--color-text-brand-secondary_hover);
  --text-color-tooltip-supporting-text: var(--color-tooltip-supporting-text);

  /* BORDER PROPERTY COLORS */
  --border-color-primary: var(--color-border-primary);
  --border-color-secondary: var(--color-border-secondary);
  --border-color-secondary_alt: var(--color-border-secondary_alt);
  --border-color-tertiary: var(--color-border-tertiary);
  --border-color-error: var(--color-border-error);
  --border-color-error_subtle: var(--color-border-error_subtle);
  --border-color-brand: var(--color-border-brand);
  --border-color-brand_alt: var(--color-border-brand_alt);
  --border-color-brand-solid: var(--color-bg-brand-solid);
  --border-color-brand-solid_hover: var(--color-bg-brand-solid_hover);

  /* RING PROPERTY COLORS */
  --ring-color-primary: var(--color-border-primary);
  --ring-color-secondary: var(--color-border-secondary);
  --ring-color-secondary_alt: var(--color-border-secondary_alt);
  --ring-color-tertiary: var(--color-border-tertiary);
  --ring-color-error: var(--color-border-error);
  --ring-color-error_subtle: var(--color-border-error_subtle);
  --ring-color-brand: var(--color-border-brand);
  --ring-color-brand_alt: var(--color-border-brand_alt);
  --ring-color-brand-solid: var(--color-bg-brand-solid);
  --ring-color-brand-solid_hover: var(--color-bg-brand-solid_hover);

  /* OUTLINE PROPERTY COLORS */
  --outline-color-primary: var(--color-border-primary);
  --outline-color-secondary: var(--color-border-secondary);
  --outline-color-secondary_alt: var(--color-border-secondary_alt);
  --outline-color-tertiary: var(--color-border-tertiary);
  --outline-color-error: var(--color-border-error);
  --outline-color-error_subtle: var(--color-border-error_subtle);
  --outline-color-brand: var(--color-border-brand);
  --outline-color-brand_alt: var(--color-border-brand_alt);
  --outline-color-brand-solid: var(--color-bg-brand-solid);
  --outline-color-brand-solid_hover: var(--color-bg-brand-solid_hover);
}

@layer base {
  /* DARK MODE VARIABLES */

  .dark-mode {
    --color-alpha-white: rgb(12 14 18);
    --color-alpha-black: rgb(255 255 255);

    /* UTILITY COLORS */

    --color-utility-blue-50: var(--color-blue-950);
    --color-utility-blue-100: var(--color-blue-900);
    --color-utility-blue-200: var(--color-blue-800);
    --color-utility-blue-300: var(--color-blue-700);
    --color-utility-blue-400: var(--color-blue-600);
    --color-utility-blue-500: var(--color-blue-500);
    --color-utility-blue-600: var(--color-blue-400);
    --color-utility-blue-700: var(--color-blue-300);

    --color-utility-brand-50: var(--color-brand-950);
    --color-utility-brand-50_alt: var(--color-utility-neutral-50);
    --color-utility-brand-100: var(--color-brand-900);
    --color-utility-brand-100_alt: var(--color-utility-neutral-100);
    --color-utility-brand-200: var(--color-brand-800);
    --color-utility-brand-200_alt: var(--color-utility-neutral-200);
    --color-utility-brand-300: var(--color-brand-700);
    --color-utility-brand-300_alt: var(--color-utility-neutral-300);
    --color-utility-brand-400: var(--color-brand-600);
    --color-utility-brand-400_alt: var(--color-utility-neutral-400);
    --color-utility-brand-500: var(--color-brand-500);
    --color-utility-brand-500_alt: var(--color-utility-neutral-500);
    --color-utility-brand-600: var(--color-brand-400);
    --color-utility-brand-600_alt: var(--color-utility-neutral-600);
    --color-utility-brand-700: var(--color-brand-300);
    --color-utility-brand-700_alt: var(--color-utility-neutral-700);
    --color-utility-brand-800: var(--color-brand-200);
    --color-utility-brand-800_alt: var(--color-utility-neutral-800);
    --color-utility-brand-900: var(--color-brand-100);
    --color-utility-brand-900_alt: var(--color-utility-neutral-900);

    --color-utility-neutral-50: var(--color-neutral-900);
    --color-utility-neutral-100: var(--color-neutral-800);
    --color-utility-neutral-200: var(--color-neutral-700);
    --color-utility-neutral-300: var(--color-neutral-700);
    --color-utility-neutral-400: var(--color-neutral-600);
    --color-utility-neutral-500: var(--color-neutral-500);
    --color-utility-neutral-600: var(--color-neutral-400);
    --color-utility-neutral-700: var(--color-neutral-300);
    --color-utility-neutral-800: var(--color-neutral-200);
    --color-utility-neutral-900: var(--color-neutral-100);

    --color-utility-red-50: var(--color-red-950);
    --color-utility-red-100: var(--color-red-900);
    --color-utility-red-200: var(--color-red-800);
    --color-utility-red-300: var(--color-red-700);
    --color-utility-red-400: var(--color-red-600);
    --color-utility-red-500: var(--color-red-500);
    --color-utility-red-600: var(--color-red-400);
    --color-utility-red-700: var(--color-red-300);

    --color-utility-yellow-50: var(--color-yellow-950);
    --color-utility-yellow-100: var(--color-yellow-900);
    --color-utility-yellow-200: var(--color-yellow-800);
    --color-utility-yellow-300: var(--color-yellow-700);
    --color-utility-yellow-400: var(--color-yellow-600);
    --color-utility-yellow-500: var(--color-yellow-500);
    --color-utility-yellow-600: var(--color-yellow-400);
    --color-utility-yellow-700: var(--color-yellow-300);

    --color-utility-green-50: var(--color-green-950);
    --color-utility-green-100: var(--color-green-900);
    --color-utility-green-200: var(--color-green-800);
    --color-utility-green-300: var(--color-green-700);
    --color-utility-green-400: var(--color-green-600);
    --color-utility-green-500: var(--color-green-500);
    --color-utility-green-600: var(--color-green-400);
    --color-utility-green-700: var(--color-green-300);

    --color-utility-orange-50: var(--color-orange-950);
    --color-utility-orange-100: var(--color-orange-900);
    --color-utility-orange-200: var(--color-orange-800);
    --color-utility-orange-300: var(--color-orange-700);
    --color-utility-orange-400: var(--color-orange-600);
    --color-utility-orange-500: var(--color-orange-500);
    --color-utility-orange-600: var(--color-orange-400);
    --color-utility-orange-700: var(--color-orange-300);

    --color-utility-indigo-50: var(--color-indigo-950);
    --color-utility-indigo-100: var(--color-indigo-900);
    --color-utility-indigo-200: var(--color-indigo-800);
    --color-utility-indigo-300: var(--color-indigo-700);
    --color-utility-indigo-400: var(--color-indigo-600);
    --color-utility-indigo-500: var(--color-indigo-500);
    --color-utility-indigo-600: var(--color-indigo-400);
    --color-utility-indigo-700: var(--color-indigo-300);

    --color-utility-fuchsia-50: var(--color-fuchsia-950);
    --color-utility-fuchsia-100: var(--color-fuchsia-900);
    --color-utility-fuchsia-200: var(--color-fuchsia-800);
    --color-utility-fuchsia-300: var(--color-fuchsia-700);
    --color-utility-fuchsia-400: var(--color-fuchsia-600);
    --color-utility-fuchsia-500: var(--color-fuchsia-500);
    --color-utility-fuchsia-600: var(--color-fuchsia-400);
    --color-utility-fuchsia-700: var(--color-fuchsia-300);

    --color-utility-pink-50: var(--color-pink-950);
    --color-utility-pink-100: var(--color-pink-900);
    --color-utility-pink-200: var(--color-pink-800);
    --color-utility-pink-300: var(--color-pink-700);
    --color-utility-pink-400: var(--color-pink-600);
    --color-utility-pink-500: var(--color-pink-500);
    --color-utility-pink-600: var(--color-pink-400);
    --color-utility-pink-700: var(--color-pink-300);

    --color-utility-purple-50: var(--color-purple-950);
    --color-utility-purple-100: var(--color-purple-900);
    --color-utility-purple-200: var(--color-purple-800);
    --color-utility-purple-300: var(--color-purple-700);
    --color-utility-purple-400: var(--color-purple-600);
    --color-utility-purple-500: var(--color-purple-500);
    --color-utility-purple-600: var(--color-purple-400);
    --color-utility-purple-700: var(--color-purple-300);

    --color-utility-sky-50: var(--color-sky-950);
    --color-utility-sky-100: var(--color-sky-900);
    --color-utility-sky-200: var(--color-sky-800);
    --color-utility-sky-300: var(--color-sky-700);
    --color-utility-sky-400: var(--color-sky-600);
    --color-utility-sky-500: var(--color-sky-500);
    --color-utility-sky-600: var(--color-sky-400);
    --color-utility-sky-700: var(--color-sky-300);

    --color-utility-slate-50: var(--color-slate-950);
    --color-utility-slate-100: var(--color-slate-900);
    --color-utility-slate-200: var(--color-slate-800);
    --color-utility-slate-300: var(--color-slate-700);
    --color-utility-slate-400: var(--color-slate-600);
    --color-utility-slate-500: var(--color-slate-500);
    --color-utility-slate-600: var(--color-slate-400);
    --color-utility-slate-700: var(--color-slate-300);

    --color-utility-emerald-50: var(--color-emerald-950);
    --color-utility-emerald-100: var(--color-emerald-900);
    --color-utility-emerald-200: var(--color-emerald-800);
    --color-utility-emerald-300: var(--color-emerald-700);
    --color-utility-emerald-400: var(--color-emerald-600);
    --color-utility-emerald-500: var(--color-emerald-500);
    --color-utility-emerald-600: var(--color-emerald-400);
    --color-utility-emerald-700: var(--color-emerald-300);

    --color-utility-amber-50: var(--color-amber-950);
    --color-utility-amber-100: var(--color-amber-900);
    --color-utility-amber-200: var(--color-amber-800);
    --color-utility-amber-300: var(--color-amber-700);
    --color-utility-amber-400: var(--color-amber-600);
    --color-utility-amber-500: var(--color-amber-500);
    --color-utility-amber-600: var(--color-amber-400);
    --color-utility-amber-700: var(--color-amber-300);

    --color-text-primary: var(--color-neutral-50);
    --color-text-tertiary: var(--color-neutral-400);
    --color-text-error-primary: var(--color-red-400);
    --color-text-warning-primary: var(--color-yellow-400);
    --color-text-success-primary: var(--color-green-400);
    --color-text-secondary: var(--color-neutral-300);
    --color-text-secondary_hover: var(--color-neutral-200);
    --color-text-tertiary_hover: var(--color-neutral-300);
    --color-text-brand-secondary: var(--color-neutral-300);
    --color-text-placeholder: var(--color-neutral-500);
    --color-text-brand-tertiary: var(--color-neutral-400);
    --color-text-editor-icon-fg: var(--color-neutral-400);
    --color-text-editor-icon-fg_active: var(--color-white);
    --color-text-quaternary: var(--color-neutral-400);
    --color-text-brand-primary: var(--color-neutral-50);
    --color-text-primary_on-brand: var(--color-neutral-50);
    --color-text-secondary_on-brand: var(--color-neutral-300);
    --color-text-tertiary_on-brand: var(--color-neutral-400);
    --color-text-quaternary_on-brand: var(--color-neutral-400);
    --color-text-brand-tertiary_alt: var(--color-neutral-50);
    --color-text-error-primary_hover: var(--color-red-300);
    --color-text-brand-secondary_hover: var(--color-neutral-200);

    --color-border-secondary: var(--color-neutral-800);
    --color-border-error_subtle: var(--color-red-500);
    --color-border-primary: var(--color-neutral-700);
    --color-border-brand: var(--color-brand-400);
    --color-border-error: var(--color-red-400);
    --color-border-tertiary: var(--color-neutral-800);
    --color-border-brand_alt: var(--color-neutral-700);
    --color-border-secondary_alt: var(--color-neutral-800);

    --color-fg-secondary: var(--color-neutral-300);
    --color-fg-warning-primary: var(--color-yellow-500);
    --color-fg-success-primary: var(--color-green-500);
    --color-fg-white: var(--color-white);
    --color-fg-success-secondary: var(--color-green-400);
    --color-fg-secondary_hover: var(--color-neutral-200);
    --color-fg-primary: var(--color-white);
    --color-fg-brand-secondary: var(--color-brand-500);
    --color-fg-brand-primary: var(--color-brand-500);
    --color-fg-quaternary: var(--color-neutral-600);
    --color-fg-quaternary_hover: var(--color-neutral-500);
    --color-fg-error-primary: var(--color-red-500);
    --color-fg-warning-secondary: var(--color-yellow-400);
    --color-fg-error-secondary: var(--color-red-400);
    --color-fg-tertiary: var(--color-neutral-400);
    --color-fg-tertiary_hover: var(--color-neutral-300);
    --color-fg-brand-primary_alt: var(--color-neutral-300);
    --color-fg-brand-secondary_alt: var(--color-neutral-600);
    --color-fg-brand-secondary_hover: var(--color-neutral-500);

    --color-bg-primary: var(--color-neutral-950);
    --color-bg-tertiary: var(--color-neutral-800);
    --color-bg-brand-primary: var(--color-brand-500);
    --color-bg-error-secondary: var(--color-red-600);
    --color-bg-warning-primary: var(--color-yellow-950);
    --color-bg-warning-secondary: var(--color-yellow-600);
    --color-bg-success-primary: var(--color-green-950);
    --color-bg-success-secondary: var(--color-green-600);
    --color-bg-brand-solid: var(--color-brand-600);
    --color-bg-secondary-solid: var(--color-neutral-600);
    --color-bg-error-solid: var(--color-red-600);
    --color-bg-warning-solid: var(--color-yellow-600);
    --color-bg-success-solid: var(--color-green-600);
    --color-bg-secondary_hover: var(--color-neutral-800);
    --color-bg-primary_hover: var(--color-neutral-900);
    --color-bg-brand-solid_hover: var(--color-brand-500);
    --color-bg-error-primary: var(--color-red-950);
    --color-bg-brand-secondary: var(--color-brand-600);
    --color-bg-secondary: var(--color-neutral-900);
    --color-bg-quaternary: var(--color-neutral-700);
    --color-bg-primary_alt: var(--color-bg-secondary);
    --color-bg-brand-primary_alt: var(--color-bg-secondary);
    --color-bg-secondary_alt: var(--color-bg-primary);
    --color-bg-overlay: var(--color-neutral-800);
    --color-bg-brand-section: var(--color-bg-secondary);
    --color-bg-brand-section_subtle: var(--color-bg-primary);
    --color-bg-primary-solid: var(--color-bg-secondary);
    --color-bg-error-solid_hover: var(--color-red-500);

    --color-app-store-badge-border: var(--color-white);
    --color-avatar-styles-bg-neutral: 224 224 224 1;
    --color-featured-icon-light-fg-brand: var(--color-brand-200);
    --color-featured-icon-light-fg-error: var(--color-red-200);
    --color-featured-icon-light-fg-gray: var(--color-neutral-200);
    --color-featured-icon-light-fg-success: var(--color-green-200);
    --color-featured-icon-light-fg-warning: var(--color-yellow-200);
    --color-focus-ring-error: var(--color-red-500);
    --color-focus-ring: var(--color-brand-500);
    --color-footer-button-fg: var(--color-neutral-300);
    --color-footer-button-fg_hover: var(--color-neutral-100);
    --color-icon-fg-brand: var(--color-neutral-400);
    --color-icon-fg-brand_on-brand: var(--color-neutral-400);
    --color-screen-mockup-border: var(--color-neutral-700);
    --color-slider-handle-bg: var(--color-fg-brand-primary);
    --color-slider-handle-border: var(--color-bg-primary);
    --color-toggle-border: var(--color-transparent);
    --color-toggle-slim-border_pressed-hover: var(--color-transparent);
    --color-toggle-slim-border_pressed: var(--color-transparent);
    --color-tooltip-supporting-text: var(--color-neutral-300);

    /* BACKGROUND PROPERTY COLORS */
    --background-color-primary: var(--color-bg-primary);
    --background-color-tertiary: var(--color-bg-tertiary);
    --background-color-brand-primary: var(--color-bg-brand-primary);
    --background-color-error-secondary: var(--color-bg-error-secondary);
    --background-color-warning-primary: var(--color-bg-warning-primary);
    --background-color-warning-secondary: var(--color-bg-warning-secondary);
    --background-color-success-primary: var(--color-bg-success-primary);
    --background-color-success-secondary: var(--color-bg-success-secondary);
    --background-color-brand-solid: var(--color-bg-brand-solid);
    --background-color-secondary-solid: var(--color-bg-secondary-solid);
    --background-color-error-solid: var(--color-bg-error-solid);
    --background-color-warning-solid: var(--color-bg-warning-solid);
    --background-color-success-solid: var(--color-bg-success-solid);
    --background-color-secondary_hover: var(--color-bg-secondary_hover);
    --background-color-primary_hover: var(--color-bg-primary_hover);
    --background-color-brand-solid_hover: var(--color-bg-brand-solid_hover);
    --background-color-error-primary: var(--color-bg-error-primary);
    --background-color-brand-secondary: var(--color-bg-brand-secondary);
    --background-color-secondary: var(--color-bg-secondary);
    --background-color-quaternary: var(--color-bg-quaternary);
    --background-color-primary_alt: var(--color-bg-primary_alt);
    --background-color-brand-primary_alt: var(--color-bg-brand-primary_alt);
    --background-color-secondary_alt: var(--color-bg-secondary_alt);
    --background-color-overlay: var(--color-bg-overlay);
    --background-color-brand-section: var(--color-bg-brand-section);
    --background-color-brand-section_subtle: var(--color-bg-brand-section_subtle);
    --background-color-primary-solid: var(--color-bg-primary-solid);
    --background-color-error-solid_hover: var(--color-bg-error-solid_hover);

    --background-color-border-brand: var(--color-border-brand);
    --background-color-border-tertiary: var(--color-border-tertiary);
    --background-color-border-brand_alt: var(--color-border-brand_alt);

    /* TEXT PROPERTY COLORS */
    --text-color-primary: var(--color-text-primary);
    --text-color-tertiary: var(--color-text-tertiary);
    --text-color-error-primary: var(--color-text-error-primary);
    --text-color-warning-primary: var(--color-text-warning-primary);
    --text-color-success-primary: var(--color-text-success-primary);
    --text-color-secondary: var(--color-text-secondary);
    --text-color-secondary_hover: var(--color-text-secondary_hover);
    --text-color-tertiary_hover: var(--color-text-tertiary_hover);
    --text-color-brand-secondary: var(--color-text-brand-secondary);
    --text-color-placeholder: var(--color-text-placeholder);
    --text-color-brand-tertiary: var(--color-text-brand-tertiary);
    --text-color-editor-icon-fg: var(--color-text-editor-icon-fg);
    --text-color-editor-icon-fg_active: var(--color-text-editor-icon-fg_active);
    --text-color-quaternary: var(--color-text-quaternary);
    --text-color-brand-primary: var(--color-text-brand-primary);
    --text-color-primary_on-brand: var(--color-text-primary_on-brand);
    --text-color-secondary_on-brand: var(--color-text-secondary_on-brand);
    --text-color-tertiary_on-brand: var(--color-text-tertiary_on-brand);
    --text-color-quaternary_on-brand: var(--color-text-quaternary_on-brand);
    --text-color-brand-tertiary_alt: var(--color-text-brand-tertiary_alt);
    --text-color-error-primary_hover: var(--color-text-error-primary_hover);
    --text-color-brand-secondary_hover: var(--color-text-brand-secondary_hover);
    --text-color-tooltip-supporting-text: var(--color-tooltip-supporting-text);

    /* BORDER PROPERTY COLORS */
    --border-color-primary: var(--color-border-primary);
    --border-color-secondary: var(--color-border-secondary);
    --border-color-secondary_alt: var(--color-border-secondary_alt);
    --border-color-tertiary: var(--color-border-tertiary);
    --border-color-error: var(--color-border-error);
    --border-color-error_subtle: var(--color-border-error_subtle);
    --border-color-brand: var(--color-border-brand);
    --border-color-brand_alt: var(--color-border-brand_alt);
    --border-color-brand-solid: var(--color-bg-brand-solid);
    --border-color-brand-solid_hover: var(--color-bg-brand-solid_hover);

    /* RING PROPERTY COLORS */
    --ring-color-primary: var(--color-border-primary);
    --ring-color-secondary: var(--color-border-secondary);
    --ring-color-secondary_alt: var(--color-border-secondary_alt);
    --ring-color-tertiary: var(--color-border-tertiary);
    --ring-color-error: var(--color-border-error);
    --ring-color-error_subtle: var(--color-border-error_subtle);
    --ring-color-brand: var(--color-border-brand);
    --ring-color-brand-solid: var(--color-bg-brand-solid);
    --ring-color-brand-solid_hover: var(--color-bg-brand-solid_hover);
    --ring-color-brand_alt: var(--color-border-brand_alt);

    /* OUTLINE PROPERTY COLORS */
    --outline-color-primary: var(--color-border-primary);
    --outline-color-secondary: var(--color-border-secondary);
    --outline-color-secondary_alt: var(--color-border-secondary_alt);
    --outline-color-tertiary: var(--color-border-tertiary);
    --outline-color-error: var(--color-border-error);
    --outline-color-error_subtle: var(--color-border-error_subtle);
    --outline-color-brand: var(--color-border-brand);
    --outline-color-brand-solid: var(--color-bg-brand-solid);
    --outline-color-brand-solid_hover: var(--color-bg-brand-solid_hover);
    --outline-color-brand_alt: var(--color-border-brand_alt);
  }
}
```

##### Tipografia

**`src/styles/typography.css`:**

```css
.prose:not(:where([class~="not-prose"], [class~="not-prose"] *)) {
  --tw-prose-body: var(--color-text-tertiary);
  --tw-prose-headings: var(--color-text-primary);
  --tw-prose-lead: var(--color-text-tertiary);
  --tw-prose-links: var(--color-text-tertiary);
  --tw-prose-bold: var(--color-text-primary);
  --tw-prose-counters: var(--color-text-tertiary);
  --tw-prose-bullets: var(--color-text-tertiary);
  --tw-prose-hr: var(--color-border-secondary);
  --tw-prose-quotes: var(--color-text-primary);
  --tw-prose-quote-borders: var(--color-fg-brand-primary_alt);
  --tw-prose-captions: var(--color-text-tertiary);
  --tw-prose-code: var(--color-text-tertiary);
  --tw-prose-pre-code: var(--color-text-tertiary);
  --tw-prose-pre-bg: var(--color-bg-primary);
  --tw-prose-th-borders: var(--color-border-primary);
  --tw-prose-td-borders: var(--color-border-secondary);

  /* Base */
  color: var(--tw-prose-body);
  font-size: var(--text-md);
  line-height: var(--text-md--line-height);
}

.prose :not(:where([class~="not-prose"], [class~="not-prose"] *)) {
  /* Text */
  &:where(p) {
    margin-top: calc(var(--spacing) * 4);
    margin-bottom: calc(var(--spacing) * 4);
  }

  &:where([class~="lead"]) {
    font-size: var(--text-md);
    line-height: var(--text-md--line-height);
    margin-top: 1.2em;
    margin-bottom: 1.2em;
  }

  /* Lists */
  &:where(ol) {
    list-style-type: decimal;

    margin-top: calc(var(--spacing) * 4);
    margin-bottom: calc(var(--spacing) * 4);
    padding-inline-start: calc(var(--spacing) * 5.75);
  }
  &:where(ul) {
    list-style-type: disc;

    margin-top: calc(var(--spacing) * 4);
    margin-bottom: calc(var(--spacing) * 4);
    padding-inline-start: calc(var(--spacing) * 5.75);
  }
  &:where(li) {
    margin-top: calc(var(--spacing) * 2);
    margin-bottom: calc(var(--spacing) * 2);
  }
  &:where(ol > li) {
    padding-inline-start: 1px;
    margin-top: 0;
    margin-bottom: 0;
  }
  &:where(ul > li) {
    padding-inline-start: 1px;
    margin-top: 0;
    margin-bottom: 0;
  }

  /* Horizontal rules */
  &:where(hr) {
    margin-top: calc(var(--spacing) * 8);
    margin-bottom: calc(var(--spacing) * 8);
  }

  /* Quotes */
  &:where(blockquote) {
    color: var(--tw-prose-quotes);

    border-left-width: 2px;
    border-left-color: var(--tw-prose-quote-borders);

    padding-inline-start: 0;
    margin-top: calc(var(--spacing) * 10);
    margin-bottom: calc(var(--spacing) * 10);
  }
  &:where(blockquote p) {
    margin: 0;
    font-weight: 500;
    font-style: italic;

    font-size: var(--text-xl);
    line-height: var(--text-xl--line-height);
  }
  &:where(blockquote p:first-of-type::before) {
    content: open-quote;
  }
  &:where(blockquote p:last-of-type::after) {
    content: close-quote;
  }

  /* Headings */
  &:where(h1) {
    color: var(--tw-prose-headings);
    font-weight: 600;

    font-size: var(--text-display-sm);
    line-height: var(--text-display-sm--line-height);
    margin-bottom: calc(var(--spacing) * 5);
    margin-top: calc(var(--spacing) * 10);
  }
  &:where(h2) {
    color: var(--tw-prose-headings);
    font-weight: 600;

    font-size: var(--text-display-xs);
    line-height: var(--text-display-xs--line-height);
    margin-bottom: calc(var(--spacing) * 4);
    margin-top: calc(var(--spacing) * 8);
  }
  &:where(h3) {
    color: var(--tw-prose-headings);
    font-weight: 600;

    font-size: var(--text-xl);
    line-height: var(--text-xl--line-height);
    margin-bottom: calc(var(--spacing) * 3);
    margin-top: calc(var(--spacing) * 8);
  }
  &:where(h4) {
    color: var(--tw-prose-headings);
    font-weight: 600;

    font-size: var(--text-lg);
    line-height: var(--text-lg--line-height);
    margin-bottom: calc(var(--spacing) * 2);
    margin-top: calc(var(--spacing) * 5);
  }

  &:where(h2 + *) {
    margin-top: 0;
  }
  &:where(h3 + *) {
    margin-top: 0;
  }
  &:where(h4 + *) {
    margin-top: 0;
  }

  &:where(h1 strong) {
    font-weight: 900;
    color: inherit;
  }
  &:where(h2 strong) {
    font-weight: 800;
    color: inherit;
  }
  &:where(h3 strong) {
    font-weight: 700;
    color: inherit;
  }
  &:where(h4 strong) {
    font-weight: 700;
    color: inherit;
  }

  /* Media */

  &:where(img) {
    border-radius: var(--radius-xl);
    width: 100%;
    object-fit: cover;

    margin-top: calc(var(--spacing) * 8);
    margin-bottom: calc(var(--spacing) * 8);
  }
  &:where(video) {
    margin-top: calc(var(--spacing) * 8);
    margin-bottom: calc(var(--spacing) * 8);
  }
  &:where(figure) {
    margin-top: calc(var(--spacing) * 10);
    margin-bottom: calc(var(--spacing) * 10);
  }
  &:where(figure > *) {
    margin-top: 0;
    margin-bottom: 0;
  }
  &:where(figure:has(> blockquote)) {
    border-left-width: 2px;
    border-left-color: var(--tw-prose-quote-borders);
    padding-top: calc(var(--spacing) * 2);
    padding-bottom: calc(var(--spacing) * 2);

    padding-inline-start: calc(var(--spacing) * 4);
  }
  &:where(figure:has(> blockquote) blockquote) {
    padding-inline-start: 0;
    border: none;
  }
  &:where(img + figcaption) {
    display: flex;
    align-items: center;
    gap: calc(var(--spacing) * 1.5);
  }
  &:where(figcaption) {
    color: var(--tw-prose-captions);

    font-size: var(--text-sm);
    line-height: var(--text-sm--line-height);
    margin-top: calc(var(--spacing) * 3);
  }
  &:where(cite) {
    font-style: normal;
  }

  /* Inline elements */
  &:where(a:not(h1 a, h2 a, h3 a, h4 a, h5 a, h6 a)) {
    font-weight: 400;
    text-decoration: underline;
    text-underline-offset: 3px;
  }

  &:where(a:focus-visible) {
    border-radius: var(--radius-sm);
    outline: 2px solid var(--color-focus-ring);
    outline-offset: 2px;
  }
  &:where(:is(h1, h2, h3) a) {
    color: var(--tw-prose-headings);
    font-weight: inherit;
    text-decoration: none;
  }

  /* Inline code element */
  &:where(code:not(pre code)) {
    font-size: 0.875rem;
    line-height: 1.25rem;
    font-weight: 700;
    border-radius: 6px;
    padding: calc(var(--spacing) * 0.5) calc(var(--spacing) * 1.5);
    margin: calc(var(--spacing) * -0.5) 0px;
    background: var(--color-bg-secondary);
    box-shadow: 0 0 0 1px var(--color-border-secondary);

    &::before,
    &::after {
      content: "";
    }
  }
}

.prose.prose-centered-quote :not(:where([class~="not-prose"], [class~="not-prose"] *)) {
  &:where(blockquote) {
    padding-inline-start: 0px !important;
    border-left: none;
    text-align: center;
  }
  &:where(figure:has(> blockquote)) {
    border-left: none;
    padding-inline-start: 0px !important;
    padding-top: 0px !important;
    padding-bottom: 0px !important;
    text-align: center;
  }
}

.prose.prose-minimal-quote :not(:where([class~="not-prose"], [class~="not-prose"] *)) {
  &:where(blockquote) {
    border-left: none;
    padding-inline-start: 0px !important;
  }
  &:where(figure:has(> blockquote)) {
    border-left: none;
    padding-inline-start: 0px !important;
  }
}

.prose.md\:prose-lg:not(:where([class~="not-prose"], [class~="not-prose"] *)) {
  @media (width >= 48rem /* 768px */) {
    /* Base */
    font-size: var(--text-lg);
    line-height: var(--text-lg--line-height);
  }
}

.prose.md\:prose-lg :not(:where([class~="not-prose"], [class~="not-prose"] *)) {
  @media (width >= 48rem /* 768px */) {
    /* Text */
    &:where(p) {
      margin-top: calc(var(--spacing) * 4.5);
      margin-bottom: calc(var(--spacing) * 4.5);
    }

    &:where([class~="lead"]) {
      font-size: var(--text-xl);
      line-height: var(--text-xl--line-height);
      margin-top: 1.09em;
      margin-bottom: 1.09em;
    }

    /* Lists */
    &:where(ol) {
      margin-top: calc(var(--spacing) * 4.5);
      margin-bottom: calc(var(--spacing) * 4.5);
      padding-inline-start: calc(var(--spacing) * 6.5);
    }
    &:where(ul) {
      margin-top: calc(var(--spacing) * 4.5);
      margin-bottom: calc(var(--spacing) * 4.5);
      padding-inline-start: calc(var(--spacing) * 6.5);
    }

    &:where(ol > li) {
      padding-inline-start: 1px;
      margin-top: 0;
      margin-bottom: 0;
    }
    &:where(ul > li) {
      padding-inline-start: 1px;
      margin-top: 0;
      margin-bottom: 0;
    }

    /* Horizontal rules */
    &:where(hr) {
      margin-top: calc(var(--spacing) * 8);
      margin-bottom: calc(var(--spacing) * 8);
    }

    /* Quotes */
    &:where(blockquote) {
      padding-inline-start: 0;
      margin-top: calc(var(--spacing) * 12);
      margin-bottom: calc(var(--spacing) * 12);
    }
    &:where(blockquote p) {
      margin: 0;

      font-size: var(--text-display-xs);
      line-height: var(--text-display-xs--line-height);
    }

    /* Headings */
    &:where(h1) {
      font-size: var(--text-display-md);
      line-height: var(--text-display-md--line-height);
      margin-bottom: calc(var(--spacing) * 6);
      margin-top: calc(var(--spacing) * 12);
    }
    &:where(h2) {
      font-size: var(--text-display-sm);
      line-height: var(--text-display-sm--line-height);
      margin-bottom: calc(var(--spacing) * 5);
      margin-top: calc(var(--spacing) * 10);
    }
    &:where(h3) {
      font-size: var(--text-display-xs);
      line-height: var(--text-display-xs--line-height);
      margin-bottom: calc(var(--spacing) * 4);
      margin-top: calc(var(--spacing) * 8);
    }
    &:where(h4) {
      font-size: var(--text-xl);
      line-height: var(--text-xl--line-height);
      margin-bottom: calc(var(--spacing) * 3);
      margin-top: calc(var(--spacing) * 8);
    }
    &:where(h2 + *) {
      margin-top: 0;
    }
    &:where(h3 + *) {
      margin-top: 0;
    }
    &:where(h4 + *) {
      margin-top: 0;
    }

    &:where(figure) {
      margin-top: calc(var(--spacing) * 12);
      margin-bottom: calc(var(--spacing) * 12);
    }
    &:where(figure > *) {
      margin-top: 0;
      margin-bottom: 0;
    }
    &:where(figure:has(> blockquote)) {
      padding-inline-start: calc(var(--spacing) * 5);
    }
    &:where(figure > blockquote + figcaption) {
      font-size: var(--text-md);
      line-height: var(--text-md--line-height);
    }

    &:where(figcaption) {
      margin-top: calc(var(--spacing) * 4);
    }

    /* Inline elements */
    &:where(a:not(h1 a, h2 a, h3 a, h4 a, h5 a, h6 a)) {
      font-weight: 400;
      text-decoration: underline;
      text-underline-offset: 3px;
    }
    &:where(:is(h1, h2, h3) a) {
      color: var(--tw-prose-headings);
      font-weight: inherit;
      text-decoration: none;
    }

    /* Inline code element */
    &:where(code:not(pre code)) {
      font-size: 0.875rem;
      line-height: 1.25rem;
      font-weight: 700;
      border-radius: 6px;
      padding: calc(var(--spacing) * 0.5) calc(var(--spacing) * 1.5);
      margin: calc(var(--spacing) * -0.5) 0px;
      background: var(--color-bg-secondary);
      box-shadow: 0 0 0 1px var(--color-border-secondary);
    }
  }
}

/* Remove top margin from first element */
.prose > :first-child:not(:where([class~="not-prose"], [class~="not-prose"] *)) {
  margin-top: 0;
}
.prose > :last-child:not(:where([class~="not-prose"], [class~="not-prose"] *)) {
  margin-bottom: 0;
}
```

##### CSS global e personalizações da oficina

**`src/styles/globals.css`:**

```css
@import "tailwindcss";
@import "./theme.css";
@import "./typography.css";

@plugin "tailwindcss-react-aria-components";
@plugin "tailwindcss-animate";
@plugin "@tailwindcss/typography";

@custom-variant dark (&:where(.dark-mode, .dark-mode *));
@custom-variant label (& [data-label]);
@custom-variant focus-input-within (&:has(input:focus));

@utility scrollbar-hide {
  &::-webkit-scrollbar {
    display: none;
    -webkit-appearance: none;
  }
  -ms-overflow-style: none;
  scrollbar-width: none;
}

@utility transition-inherit-all {
  transition-property: inherit;
  transition-duration: inherit;
  transition-timing-function: inherit;
}

@layer base {
  html,
  body {
    min-height: 100%;
    font-family: var(--font-body);
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
  body {
    margin: 0;
    background: #070f1b;
    color: var(--color-text-primary);
  }
  input[type="date"],
  input[type="datetime-local"] {
    color-scheme: dark;
  }
  textarea {
    resize: vertical;
  }
}

.dark-mode {
  color-scheme: dark;
  --color-bg-primary: #0d1a2b;
  --color-bg-primary_alt: #0d1a2b;
  --color-bg-primary_hover: #14243a;
  --color-bg-secondary: #101e31;
  --color-bg-secondary_alt: #14243a;
  --color-bg-secondary_hover: #1a2d46;
  --color-bg-tertiary: #182a41;
  --color-border-primary: #38506e;
  --color-border-secondary: #263b54;
  --color-border-tertiary: #1b2d44;
  --color-text-primary: #f4f5fb;
  --color-text-secondary: #b8c7de;
  --color-text-tertiary: #94a8c5;
}
```

Remova o CSS inicial do Next que aplica fontes Geist ou Arial ao `body`. O layout utiliza Inter e fornece a variável `--font-inter` usada pelo tema.

Importe somente `@/styles/globals.css` no layout. O próprio arquivo global importa `theme.css` e `typography.css`.

Reinicie o servidor após alterar o PostCSS.

#### 3.3. Criar os utilitários e hooks

Os componentes copiados pela CLI importam os arquivos abaixo. Crie-os caso ainda não existam.

**`src/utils/cx.ts`:**

```ts
import { extendTailwindMerge } from "tailwind-merge";

const twMerge = extendTailwindMerge({
  extend: {
    theme: {
      text: [
        "display-xs",
        "display-sm",
        "display-md",
        "display-lg",
        "display-xl",
        "display-2xl",
      ],
    },
  },
});

/**
 * This function is a wrapper around the twMerge function.
 * It is used to merge the classes inside style objects.
 */
export const cx = twMerge;

/**
 * This function does nothing besides helping us to be able to
 * sort the classes inside style objects which is not supported
 * by the Tailwind IntelliSense by default.
 */
export function sortCx<
  T extends Record<
    string,
    string | number | Record<string, string | number | Record<string, string | number>>
  >,
>(classes: T): T {
  return classes;
}
```

**`src/utils/is-react-component.ts`:**

```ts
import type React from "react";

// Below the import on purpose: the shadcn CLI drops comments that come before a file's first
// statement, which would drop this directive and fail lint in projects installed that way.
// `any` is required here: with `unknown`, the type guards below stop narrowing to component types.
/* eslint-disable @typescript-eslint/no-explicit-any */

type ReactComponent = React.FC<any> | React.ComponentClass<any, any>;

/**
 * Checks if a given value is a function component.
 */
export const isFunctionComponent = (component: any): component is React.FC<any> => {
  return typeof component === "function";
};

/**
 * Checks if a given value is a class component.
 */
export const isClassComponent = (
  component: any,
): component is React.ComponentClass<any, any> => {
  return (
    typeof component === "function" &&
    component.prototype &&
    (!!component.prototype.isReactComponent || !!component.prototype.render)
  );
};

/**
 * Checks if a given value is a forward ref component.
 */
export const isForwardRefComponent = (
  component: any,
): component is React.ForwardRefExoticComponent<any> => {
  return (
    typeof component === "object" &&
    component !== null &&
    component.$typeof?.toString() === "Symbol(react.forward_ref)"
  );
};

/**
 * Checks if a given value is a valid React component.
 */
export const isReactComponent = (component: any): component is ReactComponent => {
  return (
    isFunctionComponent(component) ||
    isForwardRefComponent(component) ||
    isClassComponent(component)
  );
};
```

**`src/hooks/use-resize-observer.ts`:**

```ts
"use client";
import { useEffect, type RefObject } from "react";
type Options<T extends Element> = {
  ref: RefObject<T | null>;
  box?: ResizeObserverBoxOptions;
  onResize: ResizeObserverCallback;
};
export function useResizeObserver<T extends Element>({
  ref,
  box = "content-box",
  onResize,
}: Options<T>) {
  useEffect(() => {
    const element = ref.current;
    if (!element || typeof ResizeObserver === "undefined") return;
    const observer = new ResizeObserver(onResize);
    observer.observe(element, { box });
    return () => observer.disconnect();
  }, [ref, box, onResize]);
}
```

O Select pode importar ComboBox, que utiliza o hook acima. Preserve as dependências indiretas até concluir a análise da etapa 13. O código da CLI é sensível à versão: conserve também qualquer outro auxiliar referenciado pelos componentes instalados.

#### 3.4. Providers e layout principal

Depois de criar os estilos e auxiliares acima, crie os providers abaixo.

**`src/providers/theme-provider.tsx`:**

```tsx
"use client";
import type { ReactNode } from "react";
import { ThemeProvider as NextThemeProvider } from "next-themes";
export function ThemeProvider({ children }: { children: ReactNode }) {
  return (
    <NextThemeProvider
      attribute="class"
      forcedTheme="dark"
      value={{
        light: "light-mode",
        dark: "dark-mode",
      }}
      disableTransitionOnChange
    >
      {children}
    </NextThemeProvider>
  );
}
```

**`src/providers/route-provider.tsx`:**

```tsx
"use client";
import type { ReactNode } from "react";
import { useRouter } from "next/navigation";
import { RouterProvider } from "react-aria-components";
export function RouteProvider({ children }: { children: ReactNode }) {
  const router = useRouter();
  return (
    <RouterProvider navigate={(href) => router.push(href)}>{children}</RouterProvider>
  );
}
```

Atualize **`src/app/layout.tsx`**:

```tsx
import type { Metadata } from "next";
import type { ReactNode } from "react";
import { Inter } from "next/font/google";
import { RouteProvider } from "@/providers/route-provider";
import { ThemeProvider } from "@/providers/theme-provider";
import "@/styles/globals.css";

const inter = Inter({
  subsets: ["latin"],
  display: "swap",
  variable: "--font-inter",
});

export const metadata: Metadata = {
  title: "Mulheres na Ciência | BVApp",
  description: "Catálogo de pesquisadoras, publicações e áreas acadêmicas.",
};

export default function RootLayout({
  children,
}: Readonly<{ children: ReactNode }>) {
  return (
    <html lang="pt-BR" suppressHydrationWarning>
      <body
        className={`${inter.variable} min-h-screen bg-primary text-primary antialiased`}
      >
        <RouteProvider>
          <ThemeProvider>{children}</ThemeProvider>
        </RouteProvider>
      </body>
    </html>
  );
}
```

O provider de tema aplica a classe `dark-mode`.

Os estilos completos e as personalizações do tema já foram definidos na etapa 3.2.

<a id="etapa-4"></a>

### 4. Organizar os arquivos e configurar o design

Organize os arquivos conforme a [Estrutura do Projeto](#estrutura-do-projeto) apresentada no início do guia.

Mantenha os auxiliares transitivos adicionados pela CLI. A árvore da seção Estrutura do Projeto indica os arquivos criados neste guia, além das pastas da biblioteca. Em um projeto novo, use os providers `route-provider.tsx` e `theme-provider.tsx` apresentados na etapa 3. A versão já entregue pode usar `router-provider.tsx` e `theme.tsx`: conserve seus nomes e imports juntos, sem criar providers duplicados.

#### Padrão compartilhado com Eventos

- Cabeçalho BVApp, título separado por borda e descrição abaixo.
- Três abas sublinhadas; a troca de aba limpa a mensagem de sucesso.
- Pesquisadoras e Áreas acadêmicas: título, botão de cadastro e tabela dentro do mesmo card.
- Publicações: formulário em uma coluna com largura máxima `max-w-2xl`.
- Formulários: validação após `trim`, limites do backend, valores preservados em erros e rodapé com borda.
- Modais: Dialog Box com largura `sm:max-w-xl`, rolagem e fechamento externo bloqueado.
- Exclusão: diálogo dedicado com nome da pesquisadora, aviso sobre publicações e indicador de carregamento.

#### Design das telas

| Tela | Conteúdo | Ações |
| --- | --- | --- |
| Pesquisadoras | Nome, instituição e titularidade | Cadastrar, visualizar, editar e excluir |
| Publicações | Painel de cadastro | Cadastrar e limpar |
| Áreas acadêmicas | Nome e descrição | Cadastrar |
| Perfil | Dados pessoais e aba de publicações | Editar pesquisadora e cadastrar publicação |
| Exclusão | Confirmação e aviso sobre os registros vinculados | Cancelar e excluir |

#### Design dos formulários

| Formulário | Organização |
| --- | --- |
| Pesquisadora | Nome, instituição, titularidade, e-mail, biografia e Lattes |
| Área acadêmica | Nome e descrição |
| Publicação | Título, resumo, situação, data, link, pesquisadora e área acadêmica |

Use cards com borda discreta, cantos arredondados, fundo azul escuro e ação principal roxa. Todos os formulários têm um campo por linha, inclusive os dois Selects. O rodapé alinha os botões à direita e permite quebra de linha em telas menores. A data usa um input nativo `date`.

#### Design dos modais

- Cadastro e edição: título, descrição, campos, Cancelar e Salvar.
- Perfil: nome, instituição, titularidade, dados pessoais e publicações.
- Exclusão: ícone de lixeira, aviso, Cancelar e Excluir.
- Erros: mensagem próxima ao formulário e opção de nova tentativa nas consultas.
- Durante o envio: desabilitar ações que possam repetir a operação ou fechar o formulário.

Os popovers dos Selects compartilham o portal do Dialog, conforme a etapa 10. Isso é necessário no cadastro de publicação aberto pelo perfil.

<a id="etapa-5"></a>

### 5. Configurar o acesso à API

Crie `.env.local` na raiz do frontend:

```dotenv
API_URL=http://127.0.0.1:5000
```

Essa variável é usada pelo servidor Next.js. O navegador acessa o proxy `/api`, e não a porta do Flask diretamente.

**`next.config.ts`:**

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  async rewrites() {
    const apiUrl = (process.env.API_URL || "http://127.0.0.1:5000").replace(/\/$/, "");
    return [
      {
        source: "/api/:path*",
        destination: `${apiUrl}/:path*`,
      },
    ];
  },
};

export default nextConfig;
```

Mantenha somente um arquivo de configuração do Next.js. Se o projeto já possuir `next.config.mjs`, incorpore `rewrites` nele e preserve as demais opções. Não mantenha um `next.config.ts` e um `next.config.mjs` concorrentes.

O navegador chama `/api/pesquisadora`; o Next encaminha para `http://127.0.0.1:5000/pesquisadora`. Reinicie o servidor após alterar `.env.local` ou a configuração.

<a id="etapa-6"></a>

### 6. Definir os tipos

**`src/lib/types.ts`:**

```typescript
export type Area = {
  id: number;
  nome: string;
  descricao: string;
};

export type Pesquisadora = {
  id: number;
  nome: string;
  instituicao: string;
  titularidade: string;
  email: string;
  biografia: string;
  lattes: string | null;
};

export type Publicacao = {
  id: number;
  titulo: string;
  resumo: string;
  situacao: string;
  data_publicacao: string;
  link: string | null;
  pesquisadora_id: number;
  area_academica_id: number;
  area_academica: Area;
};

export type Detalhe = Pesquisadora & {
  publicacoes: Publicacao[];
};

export type DadosPesquisadora = Omit<Pesquisadora, "id">;

export type DadosPublicacao = Omit<
  Publicacao,
  "id" | "area_academica"
>;
```

`Omit` remove os campos que não pertencem ao corpo do cadastro.

Os IDs são gerados pelo banco. No formulário de publicação, as associações serão enviadas como IDs numéricos.

<a id="etapa-7"></a>

### 7. Criar o cliente HTTP

**`src/lib/api.ts`:**

```ts
import type {
  Area,
  Pesquisadora,
  Detalhe,
  Publicacao,
  DadosPesquisadora,
  DadosPublicacao,
} from "./types";

async function request<T>(
  path: string,
  method = "GET",
  data?: unknown,
): Promise<T> {
  let response: Response;

  try {
    response = await fetch(`/api${path}`, {
      method,
      cache: "no-store",
      headers: data ? { "Content-Type": "application/json" } : undefined,
      body: data ? JSON.stringify(data) : undefined,
    });
  } catch {
    throw new Error("Não foi possível conectar à API. Verifique o backend e tente novamente.");
  }

  if (response.status === 204) return undefined as T;

  const text = await response.text();
  let body: unknown = null;

  if (text) {
    try {
      body = JSON.parse(text);
    } catch {
      if (response.ok) {
        throw new Error("A API retornou uma resposta que não é JSON.");
      }
    }
  }

  if (!response.ok) {
    const details = body && typeof body === "object"
      ? body as Record<string, unknown>
      : null;
    const message = typeof details?.message === "string"
      ? details.message
      : typeof details?.error === "string"
        ? details.error
        : response.status >= 500
          ? "Não foi possível acessar a API. Verifique se o backend está ativo."
          : `Não foi possível concluir a operação (${response.status}).`;
    const errors = details?.errors ? JSON.stringify(details.errors) : "";
    throw new Error([message, errors].filter(Boolean).join(" "));
  }

  return body as T;
}

export const api = {
  listarPesquisadoras: () =>
    request<Pesquisadora[]>("/pesquisadora"),

  buscarPesquisadora: (id: number) =>
    request<Detalhe>(`/pesquisadora/${id}`),

  criarPesquisadora: (data: DadosPesquisadora) =>
    request<Pesquisadora>("/pesquisadora", "POST", data),

  editarPesquisadora: (
    id: number,
    data: DadosPesquisadora,
  ) =>
    request<Pesquisadora>(
      `/pesquisadora/${id}`,
      "PUT",
      data,
    ),

  excluirPesquisadora: (id: number) =>
    request<void>(`/pesquisadora/${id}`, "DELETE"),

  listarAreas: () =>
    request<Area[]>("/area-academica"),

  criarArea: (data: Omit<Area, "id">) =>
    request<Area>("/area-academica", "POST", data),

  listarPublicacoes: (id: number) =>
    request<Publicacao[]>(
      `/publicacao/pesquisadora/${id}`,
    ),

  criarPublicacao: (data: DadosPublicacao) =>
    request<Publicacao>("/publicacao", "POST", data),
};
```

O cliente centraliza as requisições, preserva os detalhes de validação devolvidos pela API e trata o DELETE `204` antes de tentar ler JSON. Falhas de conexão e respostas inválidas são convertidas em mensagens para a interface. Não adicione uma rota de listagem geral de publicações: ela não existe no backend utilizado.

## Parte 2 — Interface e integração

<a id="etapa-8"></a>

### 8. Criar o formulário reutilizável

O componente apresenta um campo por linha, com largura completa. Ele será utilizado pelos cadastros de pesquisadora, área acadêmica e publicação.

**`src/components/oficina/formulario.tsx`:**

```tsx
"use client";

import { useState, type FormEvent } from "react";
import { Input } from "@/components/base/input/input";
import { TextArea } from "@/components/base/textarea/textarea";
import { Select } from "@/components/base/select/select";
import { Button } from "@/components/base/buttons/button";

export type Campo = {
  name: string;
  label: string;
  required?: boolean;
  maxLength?: number;
  type?: "text" | "email" | "url" | "date" | "textarea";
  options?: {
    id: string;
    label: string;
  }[];
};

type Props = {
  fields: Campo[];
  initial?: Record<string, string>;
  label?: string;
  cancelLabel?: string;
  onSave: (data: Record<string, string>) => Promise<void>;
  onCancel: () => void;
  onBusy: (value: boolean) => void;
};

export function Formulario({
  fields,
  initial = {},
  label = "Cadastrar",
  cancelLabel = "Cancelar",
  onSave,
  onCancel,
  onBusy,
}: Props) {
  const [values, setValues] =
    useState<Record<string, string>>(initial);

  const [busy, setBusy] = useState(false);
  const [error, setError] = useState("");
  const [invalid, setInvalid] = useState<string[]>([]);

  function change(name: string, value: string) {
    setValues((old) => ({
      ...old,
      [name]: value,
    }));

    setInvalid((old) =>
      old.filter((item) => item !== name),
    );
  }

  async function submit(
    event: FormEvent<HTMLFormElement>,
  ) {
    event.preventDefault();

    const data = Object.fromEntries(
      fields.map((field) => [
        field.name,
        (values[field.name] || "").trim(),
      ]),
    );

    const missing = fields
      .filter(
        (field) =>
          field.required && !data[field.name],
      )
      .map((field) => field.name);

    setInvalid(missing);

    if (missing.length) {
      setError("Preencha os campos obrigatórios indicados.");
      return;
    }

    setBusy(true);
    onBusy(true);
    setError("");

    try {
      await onSave(data);
    } catch (err) {
      setError(
        err instanceof Error
          ? err.message
          : "Não foi possível salvar.",
      );
    } finally {
      setBusy(false);
      onBusy(false);
    }
  }

  return (
    <form onSubmit={submit} className="space-y-4">
      {fields.map((field) => {
        const common = {
          label: field.label,
          name: field.name,
          isRequired: field.required,
          isDisabled: busy,
          isInvalid: invalid.includes(field.name),
        };

        if (field.options) {
          return (
            <Select
              key={field.name}
              {...common}
              placeholder="Selecione"
              items={field.options}
              selectedKey={values[field.name] || null}
              onSelectionChange={(key) =>
                change(
                  field.name,
                  key == null ? "" : String(key),
                )
              }
            >
              {(item) => (
                <Select.Item id={item.id}>
                  {item.label}
                </Select.Item>
              )}
            </Select>
          );
        }

        if (field.type === "textarea") {
          return (
            <TextArea
              key={field.name}
              {...common}
              rows={3}
              value={values[field.name] || ""}
              maxLength={field.maxLength}
              onChange={(value) =>
                change(field.name, value)
              }
            />
          );
        }

        return (
          <Input
            key={field.name}
            {...common}
            type={field.type || "text"}
            maxLength={field.maxLength}
            value={values[field.name] || ""}
            onChange={(value) =>
              change(field.name, value)
            }
          />
        );
      })}

      {error && (
        <p
          role="alert"
          className="break-words text-sm text-red-300"
        >
          {error}
        </p>
      )}

      <div className="flex flex-wrap justify-end gap-3 border-t border-secondary pt-5">
        <Button
          type="button"
          color="secondary"
          isDisabled={busy}
          onPress={onCancel}
        >
          {cancelLabel}
        </Button>

        <Button type="submit" isLoading={busy}>
          {label}
        </Button>
      </div>
    </form>
  );
}
```

Os valores ficam no estado e são preservados quando a API rejeita a operação. Campos obrigatórios compostos apenas por espaços são recusados. Os Selects trabalham com IDs em string; a conversão para número ocorre antes do POST. `cancelLabel` permite “Cancelar” nos modais e “Limpar” no painel; `label` personaliza o botão de envio.

<a id="etapa-9"></a>

### 9. Definir os campos

**`src/lib/campos.ts`:**

```typescript
import type { Campo } from "@/components/oficina/formulario";
import type { Area, Pesquisadora } from "./types";

export const camposPesquisadora: Campo[] = [
  {
    name: "nome",
    label: "Nome",
    required: true,
    maxLength: 70,
  },
  {
    name: "instituicao",
    label: "Instituição",
    required: true,
    maxLength: 50,
  },
  {
    name: "titularidade",
    label: "Titularidade",
    required: true,
    maxLength: 12,
  },
  {
    name: "email",
    label: "E-mail",
    type: "email",
    required: true,
    maxLength: 100,
  },
  {
    name: "biografia",
    label: "Biografia",
    type: "textarea",
    required: true,
  },
  {
    name: "lattes",
    label: "Currículo Lattes (opcional)",
    type: "url",
  },
];

export const camposArea: Campo[] = [
  {
    name: "nome",
    label: "Nome",
    required: true,
    maxLength: 30,
  },
  {
    name: "descricao",
    label: "Descrição",
    type: "textarea",
    required: true,
  },
];

export function camposPublicacao(
  pesquisadoras: Pesquisadora[],
  areas: Area[],
): Campo[] {
  return [
    {
      name: "titulo",
      label: "Título",
      required: true,
      maxLength: 100,
    },
    {
      name: "resumo",
      label: "Resumo",
      type: "textarea",
      required: true,
    },
    {
      name: "situacao",
      label: "Situação",
      required: true,
      maxLength: 10,
    },
    {
      name: "data_publicacao",
      label: "Data da publicação",
      type: "date",
      required: true,
    },
    {
      name: "link",
      label: "Link de acesso (opcional)",
      type: "url",
    },
    {
      name: "pesquisadora_id",
      label: "Pesquisadora",
      required: true,
      options: pesquisadoras.map((p) => ({
        id: String(p.id),
        label: p.nome,
      })),
    },
    {
      name: "area_academica_id",
      label: "Área acadêmica",
      required: true,
      options: areas.map((a) => ({
        id: String(a.id),
        label: a.nome,
      })),
    },
  ];
}
```

Os limites correspondem aos schemas do backend.

Titularidade e situação são campos de texto porque a API não define uma lista fechada de valores.

Use “publicada” como exemplo de situação. O campo aceita até 10 caracteres.

Lattes e link são opcionais. A pesquisadora não possui Select de associação em seu cadastro.

<a id="etapa-10"></a>

### 10. Preparar os modais

A base combina Radix Dialog, Dialog Box para formulários/perfil e um diálogo dedicado de exclusão. O Radix controla foco e teclado. Clique fora não fecha o modal; Escape e os botões fecham apenas quando não há envio em andamento.

#### 10.1. Compartilhar o portal do Dialog com os Selects

Os Selects aparecem no painel e no modal de cadastro pelo perfil. Crie o contexto e a base antes dos demais componentes:

**`src/lib/dialog-portal.ts`:**

```ts
"use client";

import { createContext } from "react";

// Keep Select popovers inside the Radix dialog focus and pointer boundary.
export const DialogPortalContext = createContext<HTMLElement | null>(null);
```

**`src/components/ui/dialog.tsx`:**

```tsx
"use client";

import * as Primitive from "@radix-ui/react-dialog";
import {
  forwardRef,
  useState,
  useCallback,
  type ComponentPropsWithoutRef,
  type ElementRef,
  type HTMLAttributes,
} from "react";

import { DialogPortalContext } from "@/lib/dialog-portal";

export const Dialog = Primitive.Root;
export const DialogTrigger = Primitive.Trigger;
export const DialogClose = Primitive.Close;
export const DialogTitle = Primitive.Title;
export const DialogDescription = Primitive.Description;

export const DialogContent = forwardRef<
  ElementRef<typeof Primitive.Content>,
  ComponentPropsWithoutRef<typeof Primitive.Content>
>(({ className = "", children, ...props }, ref) => {
  const [container, setContainer] = useState<HTMLDivElement | null>(null);
  const attachRef = useCallback((node: HTMLDivElement | null) => {
    setContainer(node);
    if (typeof ref === "function") ref(node);
    else if (ref) ref.current = node;
  }, [ref]);
  return (
  <Primitive.Portal>
    <Primitive.Overlay className="fixed inset-0 z-40 bg-black/40 backdrop-blur-sm" />

    <Primitive.Content
      ref={attachRef}
      {...props}
      className={`fixed left-1/2 top-1/2 z-50 grid w-[calc(100%-2rem)] max-w-md -translate-x-1/2 -translate-y-1/2 gap-4 rounded-2xl border border-secondary bg-primary p-8 text-primary shadow-2xl outline-none ${className}`}
    >
      <DialogPortalContext.Provider value={container}>{children}</DialogPortalContext.Provider>
    </Primitive.Content>
  </Primitive.Portal>
  );
});

DialogContent.displayName = "DialogContent";

export function DialogHeader({
  className = "",
  ...props
}: HTMLAttributes<HTMLDivElement>) {
  return (
    <div
      className={`flex flex-col gap-2 ${className}`}
      {...props}
    />
  );
}

export function DialogFooter({
  className = "",
  ...props
}: HTMLAttributes<HTMLDivElement>) {
  return (
    <div
      className={`flex flex-wrap justify-end gap-3 ${className}`}
      {...props}
    />
  );
}
```

#### 10.2. Criar o Dialog Box

**`src/components/ui/dialog-box.tsx`:**

```tsx
"use client";

import type { ReactNode } from "react";
import { Button } from "@/components/base/buttons/button";
import {
  Dialog,
  DialogContent,
  DialogTitle,
  DialogDescription,
} from "./dialog";

type Props = {
  open: boolean;
  title: string;
  description?: string;
  busy?: boolean;
  onClose: () => void;
  children: ReactNode;
};

export function Component({
  open,
  title,
  description = "Mulheres na Ciência",
  busy = false,
  onClose,
  children,
}: Props) {
  return (
    <Dialog
      open={open}
      onOpenChange={(nextOpen) => {
        if (!nextOpen && !busy) {
          onClose();
        }
      }}
    >
      <DialogContent
        className="max-h-[90dvh] overflow-y-auto sm:max-w-xl"
        onInteractOutside={(event) =>
          event.preventDefault()
        }
        onEscapeKeyDown={(event) => {
          if (busy) event.preventDefault();
        }}
      >
        <div className="absolute right-4 top-4">
          <Button
            color="tertiary"
            size="sm"
            isDisabled={busy}
            onPress={onClose}
            aria-label="Fechar modal"
          >
            ×
          </Button>
        </div>

        <div className="pr-10">
          <DialogTitle className="mb-1 text-2xl font-bold tracking-tight">
            {title}
          </DialogTitle>

          <DialogDescription className="mb-3 text-sm text-secondary">
            {description}
          </DialogDescription>
        </div>

        {children}
      </DialogContent>
    </Dialog>
  );
}
```

#### 10.3. Adaptar o popover do Select

Substitua o conteúdo do arquivo gerado pela CLI pelo código abaixo. Ele usa o contexto quando está dentro de um modal e mantém o comportamento normal fora dele:

**`src/components/base/select/popover.tsx`:**

```tsx
"use client";

import type { RefAttributes } from "react";
import { useContext } from "react";
import { DialogPortalContext } from "@/lib/dialog-portal";
import type { PopoverProps as AriaPopoverProps } from "react-aria-components";
import { Popover as AriaPopover } from "react-aria-components";
import { cx } from "@/utils/cx";

interface PopoverProps extends AriaPopoverProps, RefAttributes<HTMLElement> {
    /** Caps how tall the popover can grow before its list starts scrolling: 14rem for `sm`, 16rem for `md`, and 20rem for `lg`. */
    size: "sm" | "md" | "lg";
}

export const Popover = (props: PopoverProps) => {
    const dialogContainer = useContext(DialogPortalContext);
    return (
        <AriaPopover
            placement="bottom"
            containerPadding={0}
            offset={4}
            {...props}
            UNSTABLE_portalContainer={props.UNSTABLE_portalContainer ?? dialogContainer ?? undefined}
            className={(state) =>
                cx(
                    "z-[60] w-(--trigger-width) origin-(--trigger-anchor-point) overflow-x-hidden overflow-y-auto rounded-lg bg-primary py-1 shadow-lg ring-1 ring-secondary_alt outline-hidden will-change-transform",

                    state.isEntering &&
                        "duration-150 ease-out animate-in fade-in placement-right:slide-in-from-left-0.5 placement-top:slide-in-from-bottom-0.5 placement-bottom:slide-in-from-top-0.5",
                    state.isExiting &&
                        "duration-100 ease-in animate-out fade-out placement-right:slide-out-to-left-0.5 placement-top:slide-out-to-bottom-0.5 placement-bottom:slide-out-to-top-0.5",

                    props.size === "sm" && "max-h-56!",
                    props.size === "md" && "max-h-64!",
                    props.size === "lg" && "max-h-80!",

                    typeof props.className === "function" ? props.className(state) : props.className,
                )
            }
        />
    );
};
```

A propriedade de portal corresponde à versão de `react-aria-components` usada nesta implementação. Se outra versão alterar seus tipos, confira a API do Popover antes de adaptar esse trecho.

#### 10.4. Criar o wrapper da oficina

**`src/components/oficina/modal.tsx`:**

```tsx
"use client";

import type { ReactNode } from "react";
import { Component as DialogBox } from "@/components/ui/dialog-box";

type Props = {
  title: string;
  description?: string;
  children: ReactNode;
  busy?: boolean;
  onClose: () => void;
};

export function Modal({
  title,
  description = "Mulheres na Ciência",
  children,
  busy = false,
  onClose,
}: Props) {
  return (
    <DialogBox
      open
      title={title}
      description={description}
      busy={busy}
      onClose={onClose}
    >
      {children}
    </DialogBox>
  );
}
```

#### 10.5. Confirmar a exclusão

**`src/components/oficina/excluir-pesquisadora.tsx`:**

```tsx
"use client";

import { Trash2Icon } from "lucide-react";
import { Button } from "@/components/base/buttons/button";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";

type Props = {
  nome: string;
  busy: boolean;
  error: string;
  onClose: () => void;
  onConfirm: () => void;
};

export function ExcluirPesquisadora({
  nome,
  busy,
  error,
  onClose,
  onConfirm,
}: Props) {
  return (
    <Dialog
      open
      onOpenChange={(nextOpen) => {
        if (!nextOpen && !busy) onClose();
      }}
    >
      <DialogContent
        className="sm:max-w-md"
        onInteractOutside={(event) =>
          event.preventDefault()
        }
        onEscapeKeyDown={(event) => {
          if (busy) event.preventDefault();
        }}
      >
        <DialogHeader>
          <div className="flex flex-col gap-3">
            <div className="flex size-10 items-center justify-center rounded-full bg-red-500/10 text-red-400">
              <Trash2Icon className="size-5" />
            </div>

            <div className="flex flex-col gap-2">
              <DialogTitle className="text-xl font-semibold">
                Excluir pesquisadora?
              </DialogTitle>

              <DialogDescription className="text-sm text-secondary">
                Deseja excluir <strong>{nome}</strong>?
                As publicações vinculadas também serão
                removidas. As áreas acadêmicas serão
                mantidas.{" "}
                <span className="font-medium text-primary">
                  Esta ação não pode ser desfeita.
                </span>
              </DialogDescription>
            </div>
          </div>
        </DialogHeader>

        {error && (
          <p role="alert" className="text-sm text-red-300">
            {error}
          </p>
        )}

        <DialogFooter>
          <Button
            color="secondary"
            isDisabled={busy}
            onPress={onClose}
          >
            Cancelar
          </Button>

          <Button
            color="primary-destructive"
            isLoading={busy}
            onPress={onConfirm}
          >
            Excluir pesquisadora
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

O modal fecha após sucesso. Se a API recusar, a mensagem permanece visível. A exclusão remove a pesquisadora e suas publicações vinculadas; as áreas acadêmicas permanecem cadastradas.

<a id="etapa-11"></a>

### 11. Criar a tabela reutilizável e o perfil

#### 11.1. Tabela reutilizável

**`src/components/oficina/tabela.tsx`:**

```tsx
"use client";
import type { ReactNode } from "react";
import { Table } from "@/components/application/table/table";
export type Coluna<T> = {
  id: string;
  label: string;
  render: (item: T) => ReactNode;
};
type Props<T extends { id: number }> = {
  label: string;
  items: T[];
  columns: Coluna<T>[];
};
export function Tabela<T extends { id: number }>({ label, items, columns }: Props<T>) {
  if (!items.length) {
    return <p className="p-6 text-sm text-secondary">Nenhum registro cadastrado.</p>;
  }
  return (
    <div className="overflow-x-auto">
      <Table aria-label={label} selectionMode="none">
        <Table.Header>
          {columns.map((column, index) => (
            <Table.Head key={column.id} id={column.id} isRowHeader={index === 0}>
              {column.label}
            </Table.Head>
          ))}
        </Table.Header>
        <Table.Body items={items}>
          {(item) => (
            <Table.Row id={item.id}>
              {columns.map((column) => (
                <Table.Cell key={column.id}>{column.render(item)}</Table.Cell>
              ))}
            </Table.Row>
          )}
        </Table.Body>
      </Table>
    </div>
  );
}
```

Cada coluna informa um ID, um título e como renderizar o valor. O primeiro campo identifica a linha para acessibilidade. O componente também apresenta uma mensagem quando a listagem está vazia.

#### 11.2. Perfil da pesquisadora

**`src/components/oficina/perfil.tsx`:**

```tsx
"use client";

import { useEffect, useState } from "react";
import { Button } from "@/components/base/buttons/button";
import { Tabs } from "@/components/application/tabs/tabs";
import { api } from "@/lib/api";
import type {
  Detalhe,
  Pesquisadora,
  Publicacao,
} from "@/lib/types";

type Props = {
  id: number;
  onEdit: (p: Pesquisadora) => void;
  onPublicacao: (p: Pesquisadora) => void;
};

export function Perfil({
  id,
  onEdit,
  onPublicacao,
}: Props) {
  const [profile, setProfile] =
    useState<Detalhe | null>(null);

  const [items, setItems] = useState<Publicacao[]>([]);
  const [error, setError] = useState("");
  const [attempt, setAttempt] = useState(0);

  useEffect(() => {
    let active = true;

    setProfile(null);
    setError("");

    Promise.all([
      api.buscarPesquisadora(id),
      api.listarPublicacoes(id),
    ])
      .then(([p, list]) => {
        if (active) {
          setProfile(p);
          setItems(list);
        }
      })
      .catch((err: unknown) => {
        if (active) {
          setError(
            err instanceof Error
              ? err.message
              : "Erro ao carregar.",
          );
        }
      });

    return () => {
      active = false;
    };
  }, [id, attempt]);

  if (error) {
    return (
      <>
        <p role="alert">{error}</p>
        <Button
          onPress={() => setAttempt((value) => value + 1)}
        >
          Tentar novamente
        </Button>
      </>
    );
  }

  if (!profile) {
    return <p role="status">Carregando perfil...</p>;
  }

  return (
    <div className="space-y-5">
      <div>
        <h3 className="text-xl font-semibold">
          {profile.nome}
        </h3>
        <p className="text-secondary">
          {profile.titularidade} · {profile.instituicao}
        </p>
      </div>

      <Tabs defaultSelectedKey="dados">
        <Tabs.List
          type="underline"
          aria-label="Seções do perfil"
        >
          <Tabs.Item id="dados">Dados pessoais</Tabs.Item>
          <Tabs.Item id="publicacoes">
            Publicações ({items.length})
          </Tabs.Item>
        </Tabs.List>

        <Tabs.Panel id="dados" className="space-y-3 pt-4">
          <p className="whitespace-pre-wrap">
            {profile.biografia}
          </p>

          <p>
            <a
              className="text-purple-300"
              href={`mailto:${profile.email}`}
            >
              {profile.email}
            </a>
          </p>

          {profile.lattes && (
            <a
              className="text-purple-300"
              href={profile.lattes}
              target="_blank"
              rel="noreferrer"
            >
              Currículo Lattes ↗
            </a>
          )}
        </Tabs.Panel>

        <Tabs.Panel
          id="publicacoes"
          className="space-y-3 pt-4"
        >
          {!items.length && (
            <p>Nenhuma publicação cadastrada.</p>
          )}

          {items.map((p) => (
            <article
              className="space-y-2 rounded-lg border border-secondary p-4"
              key={p.id}
            >
              <h4 className="font-semibold">{p.titulo}</h4>
              <p className="text-purple-300">
                {p.area_academica.nome}
              </p>
              <p>
                {p.situacao} ·{" "}
                {p.data_publicacao
                  .split("-")
                  .reverse()
                  .join("/")}
              </p>
              <p className="whitespace-pre-wrap">
                {p.resumo}
              </p>

              {p.link && (
                <a
                  className="text-purple-300"
                  href={p.link}
                  target="_blank"
                  rel="noreferrer"
                >
                  Acessar publicação ↗
                </a>
              )}
            </article>
          ))}
        </Tabs.Panel>
      </Tabs>

      <div className="flex flex-wrap justify-end gap-3">
        <Button
          color="secondary"
          onPress={() => onEdit(profile)}
        >
          Editar pesquisadora
        </Button>

        <Button onPress={() => onPublicacao(profile)}>
          Cadastrar publicação
        </Button>
      </div>
    </div>
  );
}
```

O perfil recebe o ID selecionado na tabela. Ele consulta os dados e a rota específica de publicações em paralelo, oferece nova tentativa em caso de erro e separa dados pessoais e publicações em duas abas.

A consulta `GET /pesquisadora/{id}` também possui a coleção de publicações, mas este guia usa `GET /publicacao/pesquisadora/{id}` para demonstrar a operação específica. Ao cadastrar pelo perfil, a pesquisadora vem previamente selecionada.

<a id="etapa-12"></a>

### 12. Montar as abas, formulários e consultas

**`src/components/oficina/catalogo.tsx`:**

```tsx
"use client";

import { useEffect, useState } from "react";
import { Button } from "@/components/base/buttons/button";
import { Tabs } from "@/components/application/tabs/tabs";
import { Tabela, type Coluna } from "./tabela";
import { api } from "@/lib/api";
import type { Area, Pesquisadora } from "@/lib/types";
import {
  camposArea,
  camposPesquisadora,
  camposPublicacao,
} from "@/lib/campos";
import { Formulario } from "./formulario";
import { Modal } from "./modal";
import { Perfil } from "./perfil";
import { ExcluirPesquisadora } from "./excluir-pesquisadora";

type ModalState =
  | null
  | { type: "criar" | "area" }
  | {
      type: "editar" | "perfil" | "excluir" | "publicacao";
      item: Pesquisadora;
    };

function mensagemErro(error: unknown) {
  return error instanceof Error
    ? error.message
    : "Não foi possível concluir a operação.";
}

export function Catalogo() {
  const [pesquisadoras, setPesquisadoras] =
    useState<Pesquisadora[]>([]);

  const [areas, setAreas] = useState<Area[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");
  const [notice, setNotice] = useState("");
  const [modal, setModal] = useState<ModalState>(null);
  const [busy, setBusy] = useState(false);
  const [deleteError, setDeleteError] = useState("");
  const [aba, setAba] = useState("pesquisadoras");
  const [versaoFormulario, setVersaoFormulario] =
    useState(0);

  async function carregar() {
    setLoading(true);
    setError("");

    try {
      const [p, a] = await Promise.all([
        api.listarPesquisadoras(),
        api.listarAreas(),
      ]);

      setPesquisadoras(p);
      setAreas(a);
    } catch (err) {
      setError(mensagemErro(err));
    } finally {
      setLoading(false);
    }
  }

  useEffect(() => {
    void carregar();
  }, []);

  function abrir(value: ModalState) {
    if (busy) return;

    setNotice("");
    setDeleteError("");
    setModal(value);
  }

  function fechar() {
    if (!busy) setModal(null);
  }

  async function salvarPesquisadora(
    data: Record<string, string>,
  ) {
    const payload = {
      nome: data.nome,
      instituicao: data.instituicao,
      titularidade: data.titularidade,
      email: data.email,
      biografia: data.biografia,
      lattes: data.lattes || null,
    };

    if (modal?.type === "editar") {
      await api.editarPesquisadora(modal.item.id, payload);
    } else {
      await api.criarPesquisadora(payload);
    }

    setModal(null);
    setNotice("Pesquisadora salva.");
    await carregar();
  }

  async function salvarArea(
    data: Record<string, string>,
  ) {
    await api.criarArea({
      nome: data.nome,
      descricao: data.descricao,
    });

    setModal(null);
    setNotice("Área acadêmica cadastrada.");
    await carregar();
  }

  async function cadastrarPublicacao(
    data: Record<string, string>,
  ) {
    const pesquisadoraId = Number(data.pesquisadora_id);
    const areaId = Number(data.area_academica_id);

    if (
      !Number.isInteger(pesquisadoraId) ||
      pesquisadoraId <= 0
    ) {
      throw new Error("Selecione uma pesquisadora válida.");
    }

    if (!Number.isInteger(areaId) || areaId <= 0) {
      throw new Error(
        "Selecione uma área acadêmica válida.",
      );
    }

    await api.criarPublicacao({
      titulo: data.titulo,
      resumo: data.resumo,
      situacao: data.situacao,
      data_publicacao: data.data_publicacao,
      link: data.link || null,
      pesquisadora_id: pesquisadoraId,
      area_academica_id: areaId,
    });

    return pesquisadoraId;
  }

  async function salvarPublicacaoPainel(
    data: Record<string, string>,
  ) {
    setNotice("");
    await cadastrarPublicacao(data);

    setNotice(
      "Publicação cadastrada. Consulte-a no perfil da pesquisadora.",
    );

    setVersaoFormulario((value) => value + 1);
  }

  async function salvarPublicacaoPerfil(
    data: Record<string, string>,
  ) {
    const id = await cadastrarPublicacao(data);
    const p = pesquisadoras.find((item) => item.id === id);

    setNotice("Publicação cadastrada.");
    setModal(p ? { type: "perfil", item: p } : null);
  }

  async function excluir() {
    if (modal?.type !== "excluir") return;

    setBusy(true);
    setDeleteError("");

    try {
      await api.excluirPesquisadora(modal.item.id);
      setModal(null);
      setNotice("Pesquisadora e publicações removidas.");
      await carregar();
    } catch (err) {
      setDeleteError(mensagemErro(err));
    } finally {
      setBusy(false);
    }
  }

  const colunasPesquisadoras: Coluna<Pesquisadora>[] = [
    { id: "nome", label: "Nome", render: (p) => p.nome },
    { id: "instituicao", label: "Instituição", render: (p) => p.instituicao },
    { id: "titularidade", label: "Titularidade", render: (p) => p.titularidade },
    {
      id: "acoes",
      label: "Ações",
      render: (p) => (
        <div className="flex flex-wrap gap-2">
          <Button color="secondary" aria-label={`Visualizar ${p.nome}`} onPress={() => abrir({ type: "perfil", item: p })}>Visualizar</Button>
          <Button color="secondary" aria-label={`Editar ${p.nome}`} onPress={() => abrir({ type: "editar", item: p })}>Editar</Button>
          <Button color="secondary-destructive" aria-label={`Excluir ${p.nome}`} onPress={() => abrir({ type: "excluir", item: p })}>Excluir</Button>
        </div>
      ),
    },
  ];

  const colunasAreas: Coluna<Area>[] = [
    { id: "nome", label: "Nome", render: (a) => a.nome },
    { id: "descricao", label: "Descrição", render: (a) => a.descricao },
  ];

  const initial =
    modal?.type === "editar"
      ? Object.fromEntries(
          camposPesquisadora.map((field) => [
            field.name,
            String(
              modal.item[field.name as keyof Pesquisadora] ??
                "",
            ),
          ]),
        )
      : {};

  const podeCadastrarPublicacao =
    pesquisadoras.length > 0 && areas.length > 0;

  return (
    <main className="mx-auto max-w-7xl space-y-6 px-4 py-10 sm:px-8">
      <h1 className="flex flex-wrap items-center gap-5 text-2xl font-semibold">
        <span className="text-purple-400">BVApp</span>
        <span className="border-l border-secondary pl-5">
          Mulheres na Ciência
        </span>
      </h1>

      <p className="text-secondary">Pesquisadoras, trajetórias e produção científica em um só lugar.</p>

      {notice && <p role="status" className="rounded-lg border border-purple-400/30 bg-purple-400/10 p-4 text-purple-200">{notice}</p>}

      {error && (
        <div role="alert" className="space-y-3 text-red-300">
          <p>{error}</p>
          <Button color="secondary" onPress={carregar}>
            Tentar novamente
          </Button>
        </div>
      )}

      {loading ? (
        <p role="status">Carregando dados...</p>
      ) : (
        !error && (
          <Tabs
            selectedKey={aba}
            onSelectionChange={(key) => {
              if (!busy) {
                setAba(String(key));
                setNotice("");
              }
            }}
          >
            <Tabs.List
              type="underline"
              aria-label="Seções principais"
            >
              <Tabs.Item id="pesquisadoras" isDisabled={busy}>
                Pesquisadoras
              </Tabs.Item>
              <Tabs.Item id="publicacoes" isDisabled={busy}>
                Publicações
              </Tabs.Item>
              <Tabs.Item id="areas" isDisabled={busy}>
                Áreas acadêmicas
              </Tabs.Item>
            </Tabs.List>

            <Tabs.Panel
              id="pesquisadoras"
              className="mt-6 overflow-hidden rounded-xl border border-secondary bg-primary"
            >
              <div className="flex flex-wrap items-center justify-between gap-4 p-5">
                <h2 className="text-xl font-semibold">
                  Pesquisadoras
                </h2>
                <Button
                  onPress={() => abrir({ type: "criar" })}
                >
                  Cadastrar pesquisadora
                </Button>
              </div>

              <Tabela label="Pesquisadoras cadastradas" items={pesquisadoras} columns={colunasPesquisadoras} />

            </Tabs.Panel>

            <Tabs.Panel
              id="publicacoes"
              className="mt-6 rounded-xl border border-secondary bg-primary p-5 sm:p-8"
            >
              <div className="mb-6 space-y-2">
                <h2 className="text-xl font-semibold">
                  Cadastrar publicação
                </h2>
                <p className="text-sm text-secondary">
                  Informe os dados e selecione a pesquisadora
                  responsável e a área acadêmica.
                </p>
              </div>

              {podeCadastrarPublicacao ? (
                <div className="max-w-2xl">
                  <Formulario
                    key={`publicacao-painel-${versaoFormulario}`}
                    fields={camposPublicacao(
                      pesquisadoras,
                      areas,
                    )}
                    onSave={salvarPublicacaoPainel}
                    onBusy={setBusy}
                    onCancel={() => {
                      setNotice("");
                      setVersaoFormulario((value) => value + 1);
                    }}
                    label="Cadastrar publicação"
                    cancelLabel="Limpar"
                  />
                </div>
              ) : (
                <div className="space-y-4">
                  <p className="text-secondary">
                    Cadastre uma pesquisadora e uma área
                    acadêmica antes de cadastrar a publicação.
                  </p>

                  <div className="flex flex-wrap gap-3">
                    {!pesquisadoras.length && (
                      <Button
                        onPress={() => abrir({ type: "criar" })}
                      >
                        Cadastrar pesquisadora
                      </Button>
                    )}

                    {!areas.length && (
                      <Button
                        color="secondary"
                        onPress={() => abrir({ type: "area" })}
                      >
                        Cadastrar área acadêmica
                      </Button>
                    )}
                  </div>
                </div>
              )}
            </Tabs.Panel>

            <Tabs.Panel
              id="areas"
              className="mt-6 overflow-hidden rounded-xl border border-secondary bg-primary"
            >
              <div className="flex flex-wrap items-center justify-between gap-4 p-5">
                <h2 className="text-xl font-semibold">
                  Áreas acadêmicas
                </h2>
                <Button
                  onPress={() => abrir({ type: "area" })}
                >
                  Cadastrar área
                </Button>
              </div>

              <Tabela label="Áreas acadêmicas cadastradas" items={areas} columns={colunasAreas} />

            </Tabs.Panel>
          </Tabs>
        )
      )}

      {modal?.type === "excluir" && (
        <ExcluirPesquisadora
          nome={modal.item.nome}
          busy={busy}
          error={deleteError}
          onClose={fechar}
          onConfirm={excluir}
        />
      )}

      {(modal?.type === "criar" ||
        modal?.type === "editar") && (
        <Modal
          key={
            modal.type === "editar"
              ? `editar-${modal.item.id}`
              : "criar"
          }
          title={
            modal.type === "editar"
              ? "Editar pesquisadora"
              : "Cadastrar pesquisadora"
          }
          busy={busy}
          onClose={fechar}
        >
          <Formulario
            fields={camposPesquisadora}
            initial={initial}
            onBusy={setBusy}
            onSave={salvarPesquisadora}
            onCancel={fechar}
            label={
              modal.type === "editar"
                ? "Salvar alterações"
                : "Cadastrar"
            }
          />
        </Modal>
      )}

      {modal?.type === "area" && (
        <Modal
          key="area"
          title="Cadastrar área acadêmica"
          busy={busy}
          onClose={fechar}
        >
          <Formulario
            fields={camposArea}
            onBusy={setBusy}
            onSave={salvarArea}
            onCancel={fechar}
          />
        </Modal>
      )}

      {modal?.type === "perfil" && (
        <Modal
          key={`perfil-${modal.item.id}`}
          title="Visualizar pesquisadora"
          onClose={fechar}
        >
          <Perfil
            id={modal.item.id}
            onEdit={(p) =>
              abrir({ type: "editar", item: p })
            }
            onPublicacao={(p) =>
              abrir({ type: "publicacao", item: p })
            }
          />
        </Modal>
      )}

      {modal?.type === "publicacao" && (
        <Modal
          key={`publicacao-${modal.item.id}`}
          title="Cadastrar publicação"
          busy={busy}
          onClose={fechar}
        >
          {areas.length ? (
            <Formulario
              fields={camposPublicacao(pesquisadoras, areas)}
              initial={{
                pesquisadora_id: String(modal.item.id),
              }}
              onBusy={setBusy}
              onSave={salvarPublicacaoPerfil}
              onCancel={() =>
                abrir({ type: "perfil", item: modal.item })
              }
              label="Cadastrar publicação"
            />
          ) : (
            <p>
              Cadastre uma área acadêmica antes de cadastrar
              a publicação.
            </p>
          )}
        </Modal>
      )}
    </main>
  );
}
```

#### Como funciona o painel de publicação

Na aba Publicações, o formulário aparece diretamente no painel. A pesquisadora e a área precisam existir antes do cadastro. Os IDs dos Selects são convertidos em inteiros positivos e enviados no POST. Após o sucesso, a mensagem aparece e o formulário é limpo. O botão Limpar apaga os valores sem enviar uma requisição.

#### Cadastro pelo perfil

Ao abrir Cadastrar publicação no perfil, `pesquisadora_id` inicia com a pesquisadora selecionada. Após salvar, o perfil é aberto novamente e consulta a lista atualizada. A associação corresponde à opção selecionada no formulário.

#### Consulta das publicações

As publicações são consultadas dentro do perfil, com área acadêmica, resumo, situação, data e link. Não há barra de pesquisa nem listagem geral de publicações.

#### Exclusão da pesquisadora

A confirmação informa que as publicações vinculadas também serão removidas e que as áreas permanecerão. Enquanto a requisição está em andamento, o diálogo bloqueia operações repetidas. Em caso de erro, mantém a mensagem e permite tentar novamente.

<a id="etapa-13"></a>

### 13. Exibir a página, limpar os componentes extras e executar

**`src/app/page.tsx`:**

```tsx
import { Catalogo } from "@/components/oficina/catalogo";

export default function Page() {
  return <Catalogo />;
}
```

Mantenha os providers e os estilos configurados no `layout.tsx`. Na aplicação pronta, essa página já está ligada ao catálogo; não reinstale todas as famílias da biblioteca.

#### 13.1. Criar o script de limpeza

Execute a limpeza somente depois de criar os arquivos da oficina e conectar o catálogo à página. O script identifica a raiz do frontend pela localização da pasta `scripts`, sem depender da pasta atual do terminal.

Crie **`scripts/limpar-componentes.cjs`** na raiz do frontend:

```js
const fs = require("node:fs");
const path = require("node:path");
const ts = require("typescript");

const project = path.resolve(__dirname, "..");
const root = path.join(project, "src");
const components = path.join(root, "components");
const catalog = path.join(components, "oficina", "catalogo.tsx");
const apply = process.argv.includes("--aplicar");

if (
  !fs.existsSync(path.join(root, "app", "page.tsx")) ||
  !fs.existsSync(catalog)
) {
  throw new Error(
    "Execute na raiz do frontend, depois de criar a aplicação da oficina.",
  );
}

const configPath = ts.findConfigFile(
  project,
  ts.sys.fileExists,
  "tsconfig.json",
);

if (!configPath) {
  throw new Error("tsconfig.json não encontrado.");
}

const config = ts.readConfigFile(configPath, ts.sys.readFile);

if (config.error) {
  throw new Error("Não foi possível ler tsconfig.json.");
}

const parsedConfig = ts.parseJsonConfigFileContent(
  config.config,
  ts.sys,
  path.dirname(configPath),
);

if (parsedConfig.errors.length) {
  throw new Error("Corrija os erros do tsconfig.json antes da limpeza.");
}

function walk(dir) {
  return fs.readdirSync(dir, { withFileTypes: true }).flatMap((entry) => {
    if (entry.isSymbolicLink()) {
      throw new Error(`Link simbólico não permitido: ${entry.name}`);
    }

    const file = path.join(dir, entry.name);
    return entry.isDirectory() ? walk(file) : [file];
  });
}

const source = walk(root).filter((file) =>
  /\.[cm]?[jt]sx?$/.test(file),
);

const sourceSet = new Set(source);
const kept = new Set();
const unresolved = [];

const entries = source.filter((file) => {
  const relative = path.relative(root, file);
  const inApp = relative.split(path.sep)[0] === "app";

  return (
    /\.d\.[cm]?ts$/.test(file) ||
    (inApp &&
      /^(page|layout|not-found|error|global-error|loading|template|default|route|sitemap|robots|manifest|icon|apple-icon|opengraph-image|twitter-image)\.[jt]sx?$/.test(
        path.basename(file),
      ))
  );
});

function visit(file) {
  if (kept.has(file)) return;
  kept.add(file);

  const ast = ts.createSourceFile(
    file,
    fs.readFileSync(file, "utf8"),
    ts.ScriptTarget.Latest,
    true,
  );

  if (ast.parseDiagnostics.length) {
    throw new Error(
      `Limpeza cancelada: erro de sintaxe em ${path.relative(root, file)}.`,
    );
  }

  function add(specifier) {
    const result = ts.resolveModuleName(
      specifier,
      file,
      parsedConfig.options,
      ts.sys,
    ).resolvedModule;

    if (result) {
      const resolved = path.resolve(result.resolvedFileName);

      if (sourceSet.has(resolved)) visit(resolved);
      return;
    }

    const local =
      specifier.startsWith(".") ||
      specifier.startsWith("@/");

    if (!local) return;

    const base = specifier.startsWith("@/")
      ? path.join(root, specifier.slice(2))
      : path.resolve(path.dirname(file), specifier);

    // Estilos e assets são preservados; não são candidatos à remoção.
    if (fs.existsSync(base) && fs.statSync(base).isFile()) {
      return;
    }

    unresolved.push(
      `${path.relative(root, file)}: ${specifier}`,
    );
  }

  function scan(node) {
    if (
      (ts.isImportDeclaration(node) || ts.isExportDeclaration(node)) &&
      node.moduleSpecifier &&
      ts.isStringLiteral(node.moduleSpecifier)
    ) {
      add(node.moduleSpecifier.text);
    }

    if (
      ts.isImportEqualsDeclaration(node) &&
      ts.isExternalModuleReference(node.moduleReference) &&
      node.moduleReference.expression &&
      ts.isStringLiteral(node.moduleReference.expression)
    ) {
      add(node.moduleReference.expression.text);
    }

    if (
      ts.isImportTypeNode(node) &&
      ts.isLiteralTypeNode(node.argument) &&
      ts.isStringLiteral(node.argument.literal)
    ) {
      add(node.argument.literal.text);
    }
    if (
      ts.isCallExpression(node) &&
      (node.expression.kind === ts.SyntaxKind.ImportKeyword ||
        (ts.isIdentifier(node.expression) &&
          node.expression.text === "require"))
    ) {
      const argument = node.arguments[0];

      if (argument && ts.isStringLiteralLike(argument)) {
        add(argument.text);
      } else {
        throw new Error(
          `Limpeza cancelada: import dinâmico sem caminho literal em ${path.relative(root, file)}.`,
        );
      }
    }

    ts.forEachChild(node, scan);
  }

  scan(ast);
}

entries.forEach(visit);

if (unresolved.length) {
  throw new Error(
    `Limpeza cancelada: imports não resolvidos.\n${unresolved.join("\n")}`,
  );
}

if (!kept.has(catalog)) {
  throw new Error(
    "Limpeza cancelada: o catálogo da oficina ainda não está ligado às páginas da aplicação.",
  );
}

// A remoção fica limitada a src/components.
// Declarações de tipos globais são preservadas.
const unused = source.filter(
  (file) =>
    file.startsWith(components + path.sep) &&
    !/\.d\.[cm]?ts$/.test(file) &&
    !kept.has(file),
);

console.log(`Componentes sem uso: ${unused.length}.`);

for (const file of unused) {
  console.log(
    `${apply ? "Remover" : "Sem uso"}: ${path.relative(root, file)}`,
  );
}

if (apply) {
  for (const file of unused) {
    fs.unlinkSync(file);
  }

  console.log(
    "Limpeza concluída. Execute npx tsc --noEmit e npm run build.",
  );
} else {
  console.log(
    "Nenhum arquivo alterado. Confira a lista antes de executar com --aplicar.",
  );
}
```

#### 13.2. Conferir os arquivos sem uso

Verifique os tipos:

```powershell
npx tsc --noEmit
```

Execute a simulação na pasta `frontend`:

```powershell
node scripts/limpar-componentes.cjs
```

Esse comando apenas lista os arquivos que não são alcançados pelos imports da aplicação.

Confira a lista antes de aplicar a limpeza.

O script preserva dependências indiretas. Se `Select` importar `ComboBox`, o arquivo do ComboBox permanecerá, mesmo que o formulário utilize apenas o Select.

Se aparecer `Limpeza cancelada: imports não resolvidos`, nenhum componente será apagado, mesmo com `--aplicar`. Corrija os caminhos ou crie os auxiliares indicados e execute a simulação novamente.

- Os providers deste guia ficam em `src/providers`. No layout, use `@/providers/route-provider` e `@/providers/theme-provider`.
- O Select instalado pode importar ComboBox, que depende de `@/hooks/use-resize-observer`. O código completo desse hook está na etapa 3.3. Confira esse arquivo antes de repetir a simulação.

#### 13.3. Aplicar a limpeza

```powershell
node scripts/limpar-componentes.cjs --aplicar
```

Sem `--aplicar`, o script apenas simula: nenhum arquivo é removido. Com `--aplicar`, ele remove somente arquivos de código sem uso em `src/components`. As pastas vazias podem continuar aparecendo no explorador; isso não significa que a limpeza falhou.

Estilos, assets, configurações e arquivos fora dessa pasta são preservados.

Pacotes npm não são desinstalados pelo script.

A análise considera imports, reexports, referências de tipos e imports dinâmicos com caminhos literais. Se houver outro mecanismo de carregamento de componentes, inclua suas entradas na análise antes de aplicar a limpeza.

#### 13.4. Conferir e executar

```powershell
npx tsc --noEmit
npm run build
npm run dev
```

Abra:

```text
http://localhost:3000
```

Mantenha a API Flask ativa, seja no outro terminal ou no serviço Docker. O frontend continuará acessando a porta 5000.

Se o Turbopack falhar ao iniciar subprocessos neste Windows, use `next dev --webpack` e `next build --webpack` nos scripts de `package.json`. A aplicação entregue usa essa configuração.

#### Pacotes e arquivos de demonstração

O script não desinstala pacotes npm. Confira os imports do código, os plugins CSS e os arquivos de configuração antes de usar `npm uninstall`. Nesta versão completa do guia, `@tailwindcss/typography` processa a folha de tipografia e deve ser mantido. Providers, hooks, utilitários e configurações também ficam fora do alcance da remoção automática.

Se um starter antigo tiver trazido a biblioteca completa, remova o workflow de demonstração `sync-components.yml` para que ele não restaure os componentes descartados. Com npm, mantenha `package-lock.json` e remova apenas o lockfile de outro gerenciador que não esteja sendo utilizado.

<a id="etapa-14"></a>

### 14. Executar o fluxo da oficina

#### 14.1 Cadastrar uma área

Abra **Áreas acadêmicas → Cadastrar área**.

| Campo | Exemplo |
| --- | --- |
| Nome | Computação |
| Descrição | Sistemas e tecnologias da informação. |

#### 14.2 Cadastrar uma pesquisadora

Abra **Pesquisadoras → Cadastrar pesquisadora**.

| Campo | Exemplo |
| --- | --- |
| Nome | Ana Costa |
| Instituição | IFPB |
| Titularidade | Doutora |
| E-mail | ana@example.com |
| Biografia | Pesquisadora em divulgação científica. |
| Currículo Lattes | Deixe vazio |

#### 14.3 Cadastrar uma publicação pelo painel

Abra a aba **Publicações**.

| Campo | Exemplo |
| --- | --- |
| Título | Tecnologia e divulgação científica |
| Resumo | Estudo sobre o acesso à produção científica. |
| Situação | publicada |
| Data da publicação | 09/10/2026 |
| Link de acesso | https://example.com/publicacao |
| Pesquisadora | Ana Costa |
| Área acadêmica | Computação |

Clique em **Cadastrar publicação**.

O formulário será limpo após o sucesso. O link utilizado no exemplo é fictício.

#### 14.4 Consultar a publicação

Abra **Pesquisadoras → Visualizar**, na linha de Ana Costa.

Selecione **Publicações** dentro do perfil.

A publicação deverá aparecer com sua área acadêmica, resumo, situação, data e link.

#### 14.5 Cadastrar pelo perfil

No perfil de Ana Costa, clique em **Cadastrar publicação**.

A pesquisadora já estará selecionada. Preencha os demais campos e salve.

#### 14.6 Editar e excluir

Edite a biografia da pesquisadora e confira a alteração no perfil.

Para excluir, clique em **Excluir** na tabela e confirme.

A pesquisadora e as publicações vinculadas serão removidas. As áreas acadêmicas permanecerão cadastradas.

<a id="etapa-15"></a>

### 15. Relacionar a interface aos requisitos

| Requisito | Interface | Requisição |
| --- | --- | --- |
| RF01 | Modal cadastrar pesquisadora | POST `/pesquisadora` |
| RF02 | Tabela de pesquisadoras | GET `/pesquisadora` |
| RF03 | Perfil da pesquisadora | GET `/pesquisadora/{id}` |
| RF04 | Modal editar pesquisadora | PUT `/pesquisadora/{id}` |
| RF05 | Confirmação de exclusão | DELETE `/pesquisadora/{id}` |
| RF06 | Modal cadastrar área | POST `/area-academica` |
| RF07 | Tabela de áreas | GET `/area-academica` |
| RF08 | Painel Publicações ou cadastro pelo perfil | POST `/publicacao` |
| RF09 | Select de pesquisadora | `pesquisadora_id` no POST |
| RF10 | Select de área acadêmica | `area_academica_id` no POST |
| RF11 | Publicações no perfil | GET `/publicacao/pesquisadora/{id}` |

A interface exige as associações, e o backend valida os registros referenciados.

A remoção das publicações vinculadas à pesquisadora é executada pelo backend.


#### Ações disponíveis por aba

| Aba | Cadastrar | Visualizar | Editar | Excluir |
| --- | --- | --- | --- | --- |
| Pesquisadoras | Sim | Perfil e publicações | Sim | Sim |
| Publicações | Sim, pelo painel ou pelo perfil | Dentro do perfil | Não previsto | Não previsto |
| Áreas acadêmicas | Sim | Listagem | Não previsto | Não previsto |

Não adicione ações sem rotas no backend. As associações e a remoção dos registros vinculados são validadas e executadas pela API.

<a id="etapa-16"></a>

### 16. Fluxo da aplicação

```text
Pessoa usuária
      ↓
Aba / formulário / modal
      ↓
Validação da interface
      ↓
Cliente HTTP (src/lib/api.ts)
      ↓
Requisição /api no Next.js
      ↓
Rewrite configurado com API_URL
      ↓
API Flask
      ↓
Validação e regras do backend
      ↓
Banco de dados
```

Resposta:

```text
JSON ou resposta sem corpo da API
      ↓
Cliente HTTP
      ↓
Atualização dos estados React
      ↓
Tabela / perfil / mensagem na interface
```

O frontend apresenta e valida os campos da interface. A API mantém a validação definitiva, as regras de associação e a persistência. Erros são tratados pelo cliente HTTP e exibidos no componente responsável.

<a id="comandos-necessarios"></a>

## Comandos necessários

### Para um projeto criado do zero

Siga as etapas 1 a 7 para criar o projeto, instalar as dependências, adicionar componentes e preencher configurações, estilos, providers, tipos e cliente HTTP. Depois, implemente a Parte 2. Os comandos de instalação estão nas etapas 2 e 3.

### Para o frontend já entregue

Na pasta que contém o frontend:

```powershell
cd frontend
npm install
if (-not (Test-Path .env.local)) { Copy-Item .env.example .env.local }
npm run dev
```

Confira API_URL em .env.local e inicie o backend conforme a etapa 1. Reinicie o Next.js após mudar essa variável.

### No uso cotidiano, com o projeto configurado

Na raiz do frontend, mantenha a API ativa e execute:

```powershell
npm run dev
```

### Conferência e limpeza após implementar a interface

```powershell
npx tsc --noEmit
node scripts/limpar-componentes.cjs
```

Revise a lista apresentada pelo script. Para aplicar a limpeza descrita na etapa 13 e verificar o resultado:

```powershell
node scripts/limpar-componentes.cjs --aplicar
npx tsc --noEmit
npm run build
```

<a id="responsabilidades"></a>

## Resumo das responsabilidades

| Arquivo/Pasta | Responsabilidade |
| --- | --- |
| .env.local | Endereço da API usado pelo Next.js. |
| next.config.ts | Encaminhamento das chamadas /api por rewrites. |
| components.json | Aliases utilizados pela CLI de componentes. |
| postcss.config.mjs | Processamento do Tailwind CSS. |
| src/app/layout.tsx | Layout, providers e import dos estilos globais. |
| src/app/page.tsx | Exibição do catálogo da oficina. |
| src/styles/ | Tema, tipografia e personalizações visuais. |
| src/providers/ | Integração da navegação e do tema. |
| src/utils/ e src/hooks/ | Utilitários e auxiliares dos componentes. |
| src/lib/types.ts | Tipos das entidades e respostas. |
| src/lib/api.ts | Cliente HTTP e tratamento de respostas e erros. |
| src/lib/campos.ts | Definição dos campos, opções e validações dos formulários. |
| src/lib/dialog-portal.ts | Integração dos popovers com o portal dos modais. |
| src/components/base/, application/ e foundations/ | Componentes da biblioteca e seus auxiliares. |
| src/components/ui/ | Componentes de Dialog utilizados na oficina. |
| src/components/oficina/ | Formulários, modais, tabelas e catálogo específicos da aplicação. |
| scripts/limpar-componentes.cjs | Identificação e remoção opcional de componentes sem uso. |

## Problemas comuns

| Problema | O que conferir |
| --- | --- |
| Import de componente não encontrado | Instalação pela CLI, alias `@/*`, providers e auxiliares da etapa 3 |
| Componentes sem estilo | Tema, tipografia, plugins PostCSS e import de `globals.css` |
| Tema claro | Provider e classe `dark-mode` |
| Erro de hooks | `"use client"` nos componentes interativos |
| Falha de conexão | Flask ativo e `API_URL` |
| Erro 422 | Campos obrigatórios, limites, data e IDs |
| Select sem opções | Cadastro prévio de pesquisadoras e áreas |
| Popover do Select sem interação | Portal e integração com o Dialog |
| Props incompatíveis | Versão e tipos dos componentes instalados |
| Limpeza cancelada por import não resolvido | Corrigir caminhos e criar os auxiliares indicados antes de repetir |
| Componentes instalados em `src/y` | Mover para `src/components` e corrigir imports antes da limpeza |
| Erro “No such command db” após ImportError | Corrigir o carregamento do backend; no Windows bloqueado, executar o comando pelo Docker |
| Porta em uso | Manter pesquisadoras em 3000/5000/5432 e eventos em 3001/5001/5433 |

### Windows bloqueia o Python temporário do uv

Se `uv sync` falhar com “Uma política de Controle de Aplicativo bloqueou este arquivo” (`os error 4551`) ao executar o Python temporário em `uv/cache/builds-v0`, mas o Python da `.venv` executar normalmente, instale os requisitos de build no ambiente existente e compile sem o ambiente temporário isolado:

```powershell
uv pip install --python .venv/Scripts/python.exe hatchling editables
uv sync --no-build-isolation
```

O `hatchling` é o backend de build do projeto e `editables` é usado para instalar o pacote local em modo editável. O modo sem isolamento pressupõe que esses requisitos já estejam instalados. Esse procedimento não altera a política de segurança do Windows.

Depois de concluir, confira a sincronização normal:

```powershell
uv sync
```

Se o bloqueio ocorrer novamente ao reconstruir o pacote, repita os dois primeiros comandos. Se o próprio Python da `.venv` também estiver bloqueado, esse procedimento não se aplica; use um interpretador autorizado pelo Windows ou consulte quem administra a política do computador.

Esse procedimento resolve o bloqueio do ambiente temporário de build. Se a execução do Flask falhar depois com `DLL load failed` e a mesma mensagem de Controle de Aplicativo, o Windows também está bloqueando uma extensão nativa de uma dependência. Neste computador, isso foi observado no SQLAlchemy (`_util_cy`). A instalação concluída não significa que o backend já pode executar: essa biblioteca precisa ser autorizada pela administração da política, ou o backend deve ser configurado para rodar em um ambiente Linux, como um container Docker. O `compose.yaml` fornecido sobe somente o banco. O serviço adicional do backend está no `compose.override.yaml` preparado nesta implementação; siga a seção “Backend em Docker neste Windows” da etapa 1.

Referência: [build sem isolamento no uv](https://docs.astral.sh/uv/reference/cli/#uv-sync).

## Referências

- Ficha técnica de pesquisadoras fornecida para a oficina.
- Backend da branch `pesquisadoras-api`, com contrato conferido nas rotas e schemas.
- [Next.js](https://nextjs.org/docs/app/getting-started/installation).
- [Instalação manual do Untitled UI](https://www.untitledui.com/react/integrations/nextjs#manual-installation).
- [Código aberto do Untitled UI](https://github.com/untitleduico/react).
- [CLI do Untitled UI](https://www.untitledui.com/react/docs/cli).
- [Rewrites do Next.js](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites).
- [uv em Docker](https://docs.astral.sh/uv/guides/integration/docker/).
- Componentes vinculados na etapa 3.

### Licença dos estilos e utilitários de referência

Os estilos de base e os utilitários reproduzidos do repositório aberto do Untitled UI são distribuídos sob a licença MIT. As personalizações visuais da oficina ficam em `globals.css`.

```text
MIT License

Copyright (c) 2025 Untitled UI

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
