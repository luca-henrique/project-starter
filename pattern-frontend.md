# Padrões e Arquitetura

Este documento define os padrões obrigatórios de arquitetura, organização de código, boas práticas, acessibilidade governamental e auditoria.

---

> **Diretriz Geral:** Este projeto adota a arquitetura **Feature-Sliced Design (FSD)** adaptada para o ecossistema **Next.js 16 (App Router)** com **Server-Side Rendering (SSR)** e **Atomic Design**, utilizando a stack **React 19, TypeScript, Turbopack, TailwindCSS, TanStack Query, React Hook Form, Zod e Sonner**. Todos os desenvolvimentos e refatorações devem seguir rigorosamente as diretrizes abaixo.

---

## 📑 Sumário de Tópicos

<details open>
<summary><b>Clique para expandir / recolher o índice</b></summary>

### 🏛️ Arquitetura, Next.js & Estrutura

- [**0. Feature-Sliced Design (FSD) & Atomic Design**](#0-feature-sliced-design-fsd-no-nextjs-app-router--atomic-design) — Hierarquia de camadas (`app`, `widgets`, `features`, `entities`, `shared`) e átomos/moléculas.
- [**1. Diretrizes de Next.js, SSR & Server Components**](#1-diretrizes-de-nextjs-server-side-rendering-ssr--server-components) — Regra de ouro (Server-first), Server Components por padrão, isolamento de `"use client"`, hidratação segura e `next/image`.
- [**2. Modais Isolados e Acessíveis**](#2-modais-isolados-e-acessíveis-wcag--wai-aria) — Requisitos de overlay, tecla ESC, focus trap, lock de scroll e 90vh.
- [**3. Nomenclatura e Convenções de Arquivos**](#3-nomenclatura-e-convenções-de-arquivos) — Padrão `kebab-case`, prefixo `use-`, schemas `*.schema.ts` e tipagem estrita.

### ⚙️ Formulários, Estado & Comunicação

- [**4. Formulários, Schemas e Custom Hooks**](#4-formulários-schemas-e-custom-hooks-react-hook-form--zod) — React Hook Form + Zod, separação de responsabilidades e feedback com `role="alert"`.
- [**5. Gerenciamento de Estado de Servidor (TanStack Query)**](#5-gerenciamento-de-estado-de-servidor-tanstack-query) — Cache, invalidação de queries e proibição de `window.location.reload()`.
- [**6. Camada de Serviços, API & Padrão de Debounce (`src/shared/api/`)**](#6-camada-de-serviços-comunicação-com-api-srcsharedapi--padrão-de-debounce-em-buscas) — Instância tipada de `httpClient`, contratos e padrão obrigatório de **debounce (300ms–500ms)** com `useDebounce` em inputs de busca.
- [**7. Feedback Visual, Estados Assíncronos & Prevenção**](#7-feedback-visual-estados-assíncronos--prevenção-de-erros) — Skeletons, Empty States, toasts Sonner e prevenção de duplo envio.
- [**8. Centralização de Formatações e Máscaras**](#8-centralização-de-formatações-e-máscaras-srcsharedlibformat) — Utilitários obrigatórios para CPF, telefone, CEP, datas pt-BR e BRL.

### 🎨 Design System, Acessibilidade & Mobile

- [**9. Design System, Tokens e Padronização Tipográfica**](#9-design-system-tokens-e-padronização-tipográfica) — Átomo `Typography`, `page-title`, tokens semânticos e proibição de cores arbitrárias.
- [**10. Acessibilidade Governamental (e-MAG / WCAG 2.1 AA)**](#10-acessibilidade-governamental-e-mag--wcag-21-aa) — Alto contraste, ampliação de fonte, dislexia, modo escuro e leitores de tela.
- [**11. Privacidade e Proteção de Dados (LGPD)**](#11-privacidade-e-proteção-de-dados-lgpd---lei-137092018) — Mascaramento de CPF, consentimento para GPS/foto e limites de upload.
- [**12. Priorização Mobile-First e Usabilidade Tátil**](#12-priorização-mobile-first-e-usabilidade-tátil) — Touch targets >= 44x44px, thumb-zone e prevenção de overflow horizontal.

### 🛡️ Qualidade, Testes & Segurança

- [**13. Princípios SOLID, Clean Code e DRY**](#13-princípios-solid-clean-code-e-dry) — Responsabilidade única, desacoplamento e eliminação de duplicidades.
- [**14. Rastreabilidade de Bugs & Auditoria Visual**](#14-rastreabilidade-de-bugs--auditoria-visual-docsbug-traceability) — Inspeção nos 4 breakpoints e catalogação padronizada.
- [**15. Estratégia e Diretrizes de Testes Obrigatórios**](#15-estratégia-e-diretrizes-de-testes-obrigatórios) — Pirâmide de testes, jornadas críticas E2E, metas de Core Web Vitals e DoD.
- [**16. Segurança no Frontend e Defesa em Profundidade**](#16-segurança-no-frontend-e-defesa-em-profundidade) — Validação dupla, MFA, CSRF, Clickjacking, SRI e auditoria de pacotes.

### 🌐 Internacionalização & Localização (i18n / l10n)

- [**17. Internacionalização (i18n) e Localização (l10n)**](#17-internacionalização-i18n-e-localização-l10n) — Suporte multilíngue (`pt-BR`, `en`, `es`), dicionários tipados no FSD, formatação `Intl`, tags semânticas e acessibilidade fonética para leitores de tela.

### 🧩 Componentização, Design Patterns & Storybook

- [**18. Padrão de Componentização (Até 120 Linhas), Storybook & Design Patterns no React**](#18-padrão-de-componentização-até-120-linhas-storybook--design-patterns-no-react) — Limite estrito de 120 linhas por arquivo de componente, documentação com Storybook e os 6 padrões de projeto React fundamentais.

</details>

---

### 0. Feature-Sliced Design (FSD) no Next.js App Router & Atomic Design

A estrutura de diretórios em `src/` segue a hierarquia rigorosa do FSD, onde camadas superiores podem consumir camadas inferiores, mas nunca o inverso:

```text
src/
├── app/          # Roteamento Next.js, Layouts, Metadata, SSR e Providers
├── widgets/      # Blocos autônomos de interface (Header, Footer, BottomNav, ServicesGrid)
├── features/     # Casos de uso do cidadão/servidor com valor de negócio (auth, service-request, track-request, accessibility)
├── entities/     # Modelos de domínio, dados e componentes de entidade (service, request)
└── shared/       # Primitivas agnósticas (shared/ui, shared/lib/format, shared/api, shared/hooks)
```

- **Atomic Design em `src/shared/ui/` & Features**:
  - **Atoms**: Elementos fundamentais indivisíveis (`Typography`, `Button`, `Input`, `Badge`, `Skeleton`).
  - **Molecules**: Combinação de átomos com função direta (`FormField`, `PasswordField`, `EmptyState`, cards específicos).
  - **Organisms**: Blocos maiores e autônomos de interface (seções de formulário, modais, cabeçalhos de tela).
- **Regra de Ouro da Componentização (Teto de 120 Linhas)**:
  - Nenhum arquivo de componente React deve ultrapassar **120 linhas de código**. Se um componente atingir esse limite, ele **deve ser obrigatoriamente fatiado** em subcomponentes atômicos (átomos/moléculas) e ter sua lógica extraída para um custom hook.
- **Documentação com Storybook**:
  - Componentes reutilizáveis em `src/shared/ui/` e componentes visuais de features devem possuir arquivos de estórias `*.stories.tsx` para validação isolada de estados e acessibilidade visual.

---

### 1. Diretrizes de Next.js, Server-Side Rendering (SSR) & Server Components

#### 1.1 A Regra de Ouro do Framework (Server-First)

> **A regra de ouro é simples:**  
> **Server-first.**  
> **Client apenas quando necessário.**

Quer extrair o melhor do framework? Aqui vão alguns pontos que fazem a diferença:

- **[→] Prefira fetch nativo**  
  O `fetch()` do Next é otimizado pra rodar no server, com cache e awareness automático.

- **[→] Use formulários como se estivesse no backend**  
  Form actions + validação com schema no server (`zod`, `yup`, `joi`) resolve 90% dos casos.

- **[→] Não precisa instalar SWR ou React Query por reflexo**  
  O cache do Next já faz um bom trabalho, além de ter integração ao sistema de revalidação nativa.

- **[→] Escreva server components por padrão**  
  Quanto mais você delega ao server, menos JS vai pro cliente. Isso impacta no bundle, TTI, e performance real. É aí que o Next brilha.

- **[→] Esqueça spinner com estado**  
  Prefira usar o `loading.tsx` na rota com suspense. É limpo, performático e já integrado ao fluxo do framework.

- **[→] Evite styled-components**  
  Além de estar defasado no ecossistema, exige soluções extras no SSR. Tailwind resolve isso de forma elegante e já vem pronto pra uso.

- **[→] Proxy é outro escondido**  
  Autenticação? Redirecionamento? Geolocalização?  
  O `proxy.ts` roda antes da renderização, no edge e sem custo de JS no client.

- **[→] E se tudo o que você quer são rotas... use Vite**  
  Se a app for toda client-side, não tem por que usar Next. React + Vite = build mais rápido, bundle menor, zero overhead de SSR.

**Resumo da ópera:**
1. Next vai além das rotas, ele toca arquitetura pensada pro server.
2. Se você não entende bem render cycle, RSC, e streaming, vai acabar escrevendo uma SPA mais complexa do que precisava ser.

#### 1.2 Regras de Arquitetura e Implementação SSR / RSC

- **Server Components (RSC) por Padrão**:
  - Todas as páginas (`page.tsx`), layouts (`layout.tsx`) e componentes estáticos (ex: `Footer`) são **Server Components** por padrão, maximizando performance de carregamento, SEO e reduzindo o bundle JavaScript enviado ao cliente.
- **Isolamento de `"use client"`**:
  - A diretiva `"use client"` deve residir exclusivamente nas folhas da árvore de componentes (componentes com listeners de eventos, hooks de estado `useState`/`useEffect`, formulários ou modais interativos).
  - Nunca transforme uma página inteira do App Router em `"use client"` se ela puder ser uma casca de Server Component compondo widgets interativos.
- **Prevenção de Hydration Mismatches (SSR Safe)**:
  - É proibido acessar APIs do navegador (`window`, `localStorage`, `document.documentElement`) diretamente no corpo da renderização ou no valor inicial de um `useState` (ex: `useState(localStorage.getItem(...))`).
  - Para sincronizar estados do cliente (como dark mode ou acessibilidade), utilize **`React.useSyncExternalStore`** com `MutationObserver` ou hooks que garantam renderização segura após a montagem do cliente.
- **Geração Estática (SSG) & Static Params**:
  - Rotas dinâmicas de catálogo (ex: `/servico/[serviceId]`) devem exportar `generateStaticParams()` para pré-renderização estática no build das 15 rotas municipais oficiais.
- **Metadados Dinâmicos e SEO**:
  - Toda rota pública deve fornecer metadados descritivos via `metadata` estático ou função `generateMetadata({ params })` dinâmica, incluindo título padronizado, descrição oficial da prefeitura e OpenGraph para compartilhamento no WhatsApp e redes sociais.
- **Otimização de Imagens (`next/image`)**:
  - Imagens com a propriedade `fill` **devem obrigatoriamente** incluir a prop `sizes` correspondente ao viewport (ex: `sizes="(max-width: 640px) 40px, 52px"`).
  - Imagens com dimensões fixas devem declarar `width` e `height` explícitos.
  - Imagens acima da dobra (above the fold) devem utilizar a propriedade `priority`.

---

### 2. Modais Isolados e Acessíveis (WCAG & WAI-ARIA)

- Todo modal deve residir em arquivo dedicado dentro da sua respectiva feature ou widget (ex: `src/features/service-request/ui/confirmation-modal.tsx`, `src/features/accessibility/ui/accessibility-sheet.tsx`).
- **Requisitos Obrigatórios de Acessibilidade**:
  - Fechamento ao clicar no backdrop (`overlay`).
  - Suporte nativo a cancelamento via tecla `ESC` (`keydown listener`).
  - Foco inicial adequado no primeiro elemento interativo ao abrir.
  - Bloqueio de scroll do body (`overflow: hidden`) enquanto o modal estiver aberto, restaurando o estilo original ao fechar.
  - Limite vertical respeitando a viewport com rolagem interna suave (`max-h-[90vh] overflow-y-auto`).
  - Marcação semântica com `role="dialog"`, `aria-modal="true"` e `aria-labelledby`.

---

### 3. Nomenclatura e Convenções de Arquivos

- **Pastas e Arquivos de Componentes**: Padrão `kebab-case`:
  - Ex: `src/shared/ui/password-field.tsx`, `src/features/auth/ui/auth-form.tsx`, `src/widgets/bottom-nav/ui/bottom-nav.tsx`.
- **Custom Hooks**: Padrão `kebab-case` com prefixo `use-`:
  - Ex: `src/features/auth/model/use-login-form.ts`, `src/shared/hooks/use-mobile.ts`.
- **Schemas de Validação Zod**:
  - Ex: `src/features/auth/model/login.schema.ts`, `src/features/service-request/model/service-form.schema.ts`.
- **Tipagem TypeScript Estrita**:
  - Interfaces exportadas explicitamente nos arquivos de modelo (`types.ts`, `model/*.ts`).
  - É **estritamente proibido** o uso de `any` ou type assertions inseguras (`as unknown as Type`).

---

### 4. Formulários, Schemas e Custom Hooks (React Hook Form + Zod)

- Formulários **devem** utilizar **React Hook Form** integrado com **Zod** via `@hookform/resolvers/zod`.
- **Separação Obrigatória de Responsabilidades**:
  1. **Schema Zod (`*.schema.ts`)**: Validações estritas de tamanho, formato de e-mail, telefone brasileiro, CPF/CNPJ alfanumérico com DV, campos obrigatórios e sanitização de strings (`.trim()`).
  2. **Custom Hook (`use-*.ts`)**: Isole a lógica de estado do formulário, etapas (stepper), upload de fotos, geolocalização e envio assíncrono.
  3. **Componente de UI (`*.tsx`)**: Focado exclusivamente na apresentação visual e experiência do usuário.
- **Regras Estritas de Validação de Entradas**:
  - **Sanitização de Strings**: Campos de texto livre (ex: descrição de ocorrência) devem aplicar `.trim()` e eliminar marcadores HTML/scripts potenciais no schema Zod antes da submissão.
  - **Validação Rigorosa de Uploads**: Formulários com anexos devem validar no cliente o tamanho do arquivo (máx. 5MB), o tipo MIME (`image/jpeg`, `image/png`, `image/webp`) e a extensão de arquivo aceita.
  - **Debounce em Buscas de API**: Inputs de consulta dinâmica em tempo real (ex: autocompletar endereço, busca de protocolos ou filtro de serviços) devem obrigatoriamente utilizar `useDebounce` (`src/shared/hooks/use-debounce.ts`, 300ms–500ms) conforme padronizado no [Item 6.1](#61-padrão-obrigatório-de-debounce-em-inputs-conectados-a-apis).
- **Feedback de Validação & Banner de Erro Global**:
  - Erros de campos específicos devem ser conectados através da prop `errorText` nos componentes `FormField` e `PasswordField`.
  - Falhas gerais de autenticação ou de API devem registrar erro via `form.setError("root", { message })` e ser renderizadas com banner acessível (`role="alert"` e `aria-live="polite"`).

---

### 5. Gerenciamento de Estado de Servidor (TanStack Query)

- Utilizar **TanStack Query** (`@tanstack/react-query`) para chamadas assíncronas que dependam de cache, revalidação em segundo plano e sincronização de dados.
- **Invalidação de Cache Obrigatória**:
  - Toda mutação (criação de protocolo, cancelamento ou alteração de status) deve invalidar a query correspondente no callback `onSuccess`:
    ```ts
    queryClient.invalidateQueries({ queryKey: ["requests"] });
    ```
  - **É estritamente proibido** utilizar recarregamento forçado de página (`window.location.reload()`) para sincronização de estado.

---

### 6. Camada de Serviços, Comunicação com API (`src/shared/api/`) & Padrão de Debounce em Buscas

- As requisições HTTP devem ser centralizadas utilizando a instância tipada de `httpClient` (`src/shared/api/http-client.ts`).
- Contratos de tipos estritos para todos os endpoints, incluindo tipagem de resposta, códigos de status e tratamento uniforme de erros (`HttpError`).
- Headers padrão de comunicação em JSON (`Content-Type: application/json`, `Accept: application/json`).

#### 6.1 Padrão Obrigatório de Debounce em Inputs Conectados a APIs

Todo e qualquer campo de entrada da interface (inputs de texto, busca de protocolos, autocompletar de endereços/CEP, filtros dinâmicos de catálogo) que execute consultas assíncronas contra APIs ou banco de dados **deve obrigatoriamente aplicar a técnica de Debounce**:

1. **Uso Canônico do Hook `useDebounce` (`src/shared/hooks/use-debounce.ts`)**:
   - É **estritamente proibido** criar implementações dispersas de `setTimeout`/`clearTimeout` ou debounces manuais ad-hoc em componentes JSX.
   - Deve-se consumir o hook oficial compartilhado `useDebounce<T>(value, delay)`.
   - Exemplo padrão de uso integrado ao TanStack Query:

     ```tsx
     import { useState } from "react";
     import { useQuery } from "@tanstack/react-query";
     import { useDebounce } from "@/shared/hooks";

     export function ServiceSearchInput() {
       const [searchTerm, setSearchTerm] = useState("");
       const debouncedSearch = useDebounce(searchTerm, 400); // 400ms recomendado

       // TanStack Query dispara apenas quando o valor com debounce for atualizado
       const { data, isFetching } = useQuery({
         queryKey: ["search-services", debouncedSearch],
         queryFn: ({ signal }) =>
           serviceApi.search(debouncedSearch, { signal }),
         enabled: debouncedSearch.trim().length >= 3,
       });

       return (
         <div className="relative">
           <input
             value={searchTerm}
             onChange={(e) => setSearchTerm(e.target.value)}
             placeholder="Buscar serviços municipais..."
             aria-busy={isFetching}
           />
           {isFetching && <Spinner className="absolute right-3" />}
         </div>
       );
     }
     ```

2. **Janelas Temporais Recomendadas**:
   - **Buscas Rápidas / Catálogos Locais / Filtros em Memória:** `300ms` a `400ms`.
   - **Consultas Remotas a APIs Municipais / Autocompletar / Geocodificação:** `400ms` a `500ms` (equilíbrio ideal entre responsividade percebida e economia de requisições).
   - **Buscas Textuais Extensas / Bases Históricas Pesadas:** `500ms` a `800ms`.

3. **Gatilho por Quantidade Mínima de Caracteres (`minChars`)**:
   - Para evitar chamadas vazias ou excessivamente amplas à API, a requisição só deve ser habilitada a partir de um limite mínimo de caracteres:
     - Nomes de serviços, ruas e termos livres: mínimo de **3 caracteres** (`debouncedSearch.trim().length >= 3`).
     - Códigos de protocolo: mínimo de **4 a 6 caracteres**.
     - CEP: busca disparada estritamente quando atingir **8 dígitos limpos**.

4. **Cancelamento Automático de Requisições em Trânsito (`AbortSignal`)**:
   - Toda consulta disparada após o debounce deve repassar o `signal` do TanStack Query (`queryFn: ({ signal }) => ...`) para a chamada HTTP.
   - Caso o cidadão continue digitando e um novo valor debounced seja emitido, a requisição anterior ainda em andamento é cancelada automaticamente pelo navegador, prevenindo _race conditions_ (respostas lentas antigas sobrescrevendo resultados mais recentes).

5. **Acessibilidade e Feedback Visual (e-MAG / WCAG)**:
   - Exibir um indicador discreto de carregamento (spinner animado ou skeleton) enquanto `isFetching` estiver ativo após o debounce.
   - O input **nunca deve perder o foco** ou travar a digitação do cidadão enquanto a busca acontece em segundo plano.
   - Declarar `aria-busy={isFetching}` no input e expor o total de resultados encontrados através de uma região viva `aria-live="polite"` para tecnologias assistivas.

---

### 7. Feedback Visual, Estados Assíncronos & Prevenção de Erros

- **Skeletons**: Utilizar `src/shared/ui/skeleton.tsx` para todas as telas, cards de serviços e históricos de protocolo enquanto `isLoading` for verdadeiro.
- **Empty States**: Listagens e buscas sem resultados devem renderizar a molécula `EmptyState` (`src/shared/ui/empty-state.tsx`) contendo ícone temático, mensagem clara e botão de ação primária (CTA).
- **Notificações (Toasts)**: Notificar ações do usuário exclusivamente com a biblioteca **Sonner** (`toast.success`, `toast.error`, `toast.info`).
- **Prevenção de Duplo Envio**: Botões de submissão de chamados e login devem receber `disabled={isSubmitting}` e exibir indicador visual de carregamento (spinner) durante a mutação.

---

### 8. Centralização de Formatações e Máscaras (`src/shared/lib/format/`)

Funções de máscara, limpeza e conversão devem obrigatoriamente residir em utilitários centralizados em `src/shared/lib/format/`:

- **CPF (`cpf.ts`)**:
  - `cleanCPF(val)`: Remove caracteres de pontuação.
  - `formatCPF(val)`: Aplica a máscara `999.999.999-99`.
  - `validateCPF(val)`: Validação matemática real dos dois dígitos verificadores (rejeita CPFs fictícios ou sequências repetidas).
- **CNPJ Alfanumérico (`cnpj.ts`) — [Conformidade COANA/RFB 2026]**:
  - `cleanCNPJ(val)`: Remove pontuações mantendo caracteres alfanuméricos e converte para maiúsculas.
  - `formatCNPJ(val)`: Aplica a máscara `XX.XXX.XXX/XXXX-XX` aceitando o novo padrão de letras e números emitido pela Receita Federal a partir de 2026.
  - `validateCNPJ(val)`: Validação algorítmica compatível com o cálculo de DV para CNPJs numéricos tradicionais e alfanuméricos.
- **Telefone (`phone.ts`)**:
  - `formatPhone(val)`: Suporte a números fixos de 10 dígitos `(87) 3821-0000` e celulares de 11 dígitos `(87) 99999-9999`.
- **CEP (`cep.ts`)**:
  - `formatCEP(val)`: Aplica a máscara `99999-999` com valor padrão do município de Arcoverde (`56500-000`).
- **Datas e Horários (`date.ts`)**:
  - `formatDateTime(val)`: Formato brasileiro oficial `DD/MM/YYYY HH:mm`.
  - `formatDate(val)`: Formato `DD/MM/YYYY`.
  - `formatRelativeTime(val)`: Tempo relativo humanizado (`"Há 5 min"`, `"Ontem"`).
- **Moeda BRL (`currency.ts`)**:
  - `formatCurrency(val)`: Conversão oficial via `Intl.NumberFormat("pt-BR", { style: "currency", currency: "BRL" })`.
- **Proibição**: É proibida a replicação de lógicas de formatação inline, uso de regexes legadas que limpem apenas dígitos (`/\D/g` em CNPJ) ou expressões regulares soltas em componentes JSX.

---

### 9. Design System, Tokens e Padronização Tipográfica

- **Padronização Oficial de Títulos e Textos (`src/shared/ui/typography.tsx`)**:
  - Todas as páginas e blocos de conteúdo devem utilizar o componente centralizado `Typography`:
    - **Page Title (`variant="page-title"` ou `variant="title"`):** `text-2xl sm:text-3xl font-bold text-foreground mb-1 tracking-tight leading-tight` (renderizado como `h1` semântico).
    - **Page Subtitle (`variant="page-subtitle"` ou `variant="subtitle"`):** `text-muted-foreground text-xs sm:text-sm leading-relaxed` (renderizado como `p`).
    - **Seções (`variant="h2"` ou `variant="h3"`):** Cabeçalhos semânticos para subdivisão de conteúdo.
- **Consistência de Cores e Tokens Semânticos**:
  - Utilizar sempre as variáveis CSS semânticas do tema (`text-foreground`, `bg-background`, `text-primary`, `bg-card`, `border-border`, `text-muted-foreground`, `text-destructive`).
  - **Proibido** o uso de cores hexadecimais arbitrárias soltas inline (ex: `text-[#F25C05]`, `bg-[#ffffff]`) quando houver token semântico correspondente.

---

### 10. Acessibilidade Governamental (e-MAG / WCAG 2.1 AA)

Por se tratar de um serviço público digital de atendimento ao cidadão, a aplicação deve obedecer às diretrizes do **e-MAG (Modelo de Acessibilidade em Governo Eletrônico)**:

- **Recursos de Acessibilidade Integrados (`AccessibilitySheet`)**:
  - **Alto Contraste (`.high-contrast`)**: Inversão equilibrada de fundos e realce de bordas para cidadãos com baixa visão.
  - **Texto Ampliado (`.font-large`)**: Aumento proporcional da escala tipográfica para facilitar leitura sem quebra de layout.
  - **Fonte para Dislexia (`.font-dyslexic`)**: Tipografia otimizada com maior distinção entre glifos.
  - **Modo Escuro (`.dark`)**: Redução de ofuscamento visual e economia energética em dispositivos móveis.
- **Semântica e Leitores de Tela**:
  - Estrutura clara de marcos semânticos (`<header>`, `<main id="conteudo-principal">`, `<footer>`, `<nav>`).
  - Links e botões icônicos devem conter atributos descritivos `aria-label` ou texto auxiliar visível apenas para leitores de tela (`.sr-only`).
  - Mensagens de status dinâmico e erros com `role="alert"` e `aria-live="polite"`.

---

### 11. Privacidade e Proteção de Dados (LGPD - Lei 13.709/2018)

- **Ofuscação de Dados Pessoais do Cidadão**:
  - Em telas de consulta pública de protocolos, dados sensíveis do solicitante (especialmente CPF e e-mail) não devem ser exibidos integralmente para terceiros. O CPF deve ser exibido mascarado (ex: `***.456.789-**`).
- **Consentimento em Recursos do Dispositivo**:
  - A captura de localização via GPS e o envio de fotografias do local da ocorrência devem solicitar permissão explícita ao cidadão, informando que os dados serão utilizados unicamente para triagem da equipe técnica municipal.
- **Sanitização de Uploads**:
  - Upload de fotos restrito a tipos MIME de imagem (`image/jpeg`, `image/png`, `image/webp`) com limite máximo rigoroso de **5MB**.

---

### 12. Priorização Mobile-First e Usabilidade Tátil

- **Filosofia Mobile-First no Tailwind**:
  - As classes base sempre definem o comportamento para smartphones (`w-full`, `p-4`, `flex-col`), enquanto modificadores progressivos (`sm:`, `md:`, `lg:`, `xl:`) expandem para telas maiores.
- **Área de Toque Mínima (Touch Targets >= 44x44px)**:
  - Todos os botões, ícones clicáveis e itens da `BottomNav` devem respeitar a dimensão mínima recomendada de **44x44px** (WCAG 2.1 / Apple HIG) ou possuir padding tátil suficiente para evitar cliques acidentais em smartphones.
- **Prevenção de Rolagem Horizontal (`overflow-x: hidden`)**:
  - Nenhuma tela deve gerar rolagem horizontal involuntária. Elementos de rolagem horizontal (como chips de categorias de serviços) devem possuir container isolado com `overflow-x-auto` e classe de controle de toque `touch-pan-x`.

---

### 13. Princípios SOLID, Clean Code e DRY

- **Single Responsibility (SRP)**: Cada componente, hook e utilitário deve ter uma única responsabilidade clara.
- **Desacoplamento**: Lógica de validação de formulários reside em schemas Zod, lógica de manipulação e API reside em custom hooks, e os componentes JSX focam exclusivamente na renderização visual.
- **Princípio DRY**: Quando um padrão visual ou bloco (card de serviço, item de histórico, linha de contato) se repetir 2 ou mais vezes, deve ser extraído para um componente em `src/shared/ui/` ou `src/entities/`.
- **Coleções Mapeadas**: Telas com múltiplos indicadores ou itens de navegação devem declarar estruturas tipadas e iterar via `.map()`, centralizando manutenções futuras.

---

### 14. Rastreabilidade de Bugs & Auditoria Visual (`docs/bug-traceability/`)

Antes de considerar uma tela ou funcionalidade como concluída, ela deve ser rigorosamente auditada nos 4 breakpoints fundamentais:

- **Mobile Web (360px a 390px)**: Sem scroll horizontal, touch targets >= 44px, preenchimento tátil confortável.
- **Tablet (768px a 820px)**: Grids em 2 colunas equilibradas, navegação adaptada.
- **Desktop Padrão (1024px a 1366px)**: Alinhamento harmônico com o container centralizado (`max-w-7xl`).
- **Desktop Ultrawide (1440px+)**: Limitação de largura máxima sem deformação assimétrica de elementos.

Bugs detectados em homologação devem ser catalogados na documentação de rastreabilidade no formato padrão:

```markdown
### [BUG-T-XX-001] Título Resumido da Falha

- **Resolução / Dispositivo:** ...
- **Severidade:** Baixa | Média | Alta | Crítica
- **Status:** 🔴 Aberto | 🟡 Em Correção | 🟢 Resolvido
- **Comportamento Anterior:** ...
- **Comportamento Corrigido:** ...
```

---

### 15. Estratégia e Diretrizes de Testes Obrigatórios

Para assegurar estabilidade contínua, conformidade legal com o **e-MAG** e prevenção de falhas no registro de solicitações da população, todo novo desenvolvimento ou refatoração deve obedecer à **Pirâmide de Testes do Alerta Cidadão**:

```text
       /\
      /  \     E2E (Playwright) - Jornadas Críticas (Abertura, Rastreamento, Login)
     /----\    Acessibilidade (axe-core) - e-MAG / WCAG 2.1 AA (44px touch, teclado, contraste)
    /------\   Integração (Testing Library) - Features (Stepper, RHF, Zod, Modais)
   /--------\  Unitários (Vitest) - Formatters (CPF, Fone, CEP, Datas) e Schemas Zod
```

#### 15.1 Matriz de Cobertura por Camada FSD

| Camada FSD                | Tipo de Teste  | Escopo / Requisito Obrigatório                                                                                                 | Ferramenta             |
| :------------------------ | :------------- | :----------------------------------------------------------------------------------------------------------------------------- | :--------------------- |
| **`shared/lib/format/`**  | Unitário       | Testar exaustivamente algoritmos de dígitos de CPF (`validateCPF`), máscaras de telefone (10 e 11 dígitos), CEP e datas pt-BR. | Vitest                 |
| **`features/*/model/`**   | Unitário       | Schemas Zod: validar mensagens amigáveis de erro, limites de caracteres e campos obrigatórios.                                 | Vitest                 |
| **`features/*/ui/`**      | Integração     | Validação de formulários com RHF, bloqueio de avanço de etapa no Stepper com dados inválidos, prevenção de duplo clique.       | Testing Library        |
| **`shared/ui/` & Modais** | Acessibilidade | Inspeção de foco inicial, suporte a fechamento via tecla `ESC`, backdrop click e touch targets mínimos de **44x44px**.         | axe-core / Playwright  |
| **`app/` & Rotas**        | E2E            | Fluxo completo do cidadão: abertura de chamado, geração de protocolo e consulta em `/acompanhar`.                              | Playwright             |
| **Layout Global**         | Responsividade | Verificação de ausência de scroll horizontal em viewports móveis de 360px a 390px.                                             | Playwright Screenshots |

#### 15.2 Jornadas Críticas End-to-End (E2E)

Todo ciclo de release deve validar com sucesso os seguintes cenários automatizados:

1. **Jornada do Cidadão - Abertura de Solicitação:**
   - Navegação: Início -> Seleção de Serviço (ex: "Tapa Buraco").
   - Passo 1 (Identificação): Preenchimento com validação de máscara de CPF e telefone -> Avançar.
   - Passo 2 (Localização): Preenchimento de endereço e bairro (ou acionamento de GPS) -> Avançar.
   - Passo 3 (Ocorrência): Descrição detalhada (>10 caracteres) e upload simulado de foto -> Submissão.
   - Pós-envio: Exibição obrigatória do `ConfirmationModal` com número de protocolo formatado (`ARC-2026-XXXXXX`) e botão de compartilhamento no WhatsApp funcional.
2. **Jornada de Rastreabilidade - Consulta de Protocolo:**
   - Acesso a `/acompanhar`.
   - Busca por protocolo válido -> Exibição do card com linha do tempo, endereço e badge de status ("Aguardando Análise").
   - Busca por identificador inexistente -> Renderização da molécula `EmptyState` com mensagem orientativa.
3. **Jornada de Acesso Funcional - Autenticação de Servidores:**
   - Acesso a `/login`.
   - Tentativa com campos vazios -> Renderização de `errorText` nos inputs correspondentes.
   - Tentativa com credenciais inválidas -> Renderização de banner de erro global (`role="alert"`).
   - Credenciais válidas -> Redirecionamento e notificação de toast Sonner.

#### 15.3 Testes Automatizados de Acessibilidade (e-MAG / WCAG 2.1 AA)

- **Zero Violações Críticas no Axe:** Executar auditoria de acessibilidade automatizada em todas as rotas estáticas geradas.
- **Navegação Exclusiva por Teclado:**
  - Garantir que todos os elementos interativos sejam navegáveis via `Tab` com anel visual de foco (`focus-visible:ring-2 focus-visible:ring-primary`).
  - Modais abertos devem reter o foco (focus trap) e liberar o foco de volta ao elemento de origem ao fechar.
- **Alternadores de Acessibilidade:**
  - Garantir que a ativação de `.high-contrast`, `.font-large`, `.font-dyslexic` e `.dark` no `AccessibilitySheet` não cause truncamento ilegível de textos nem quebra estrutural de grids.

#### 15.4 Metas de Performance & Core Web Vitals

Os testes de integração contínua (CI) devem assegurar os seguintes limites nos relatórios do Lighthouse:

- **LCP (Largest Contentful Paint)**: < 2.5 segundos em rede 4G simulada.
- **INP (Interaction to Next Paint)**: < 200 milissegundos para eventos de toque e clique.
- **CLS (Cumulative Layout Shift)**: < 0.1 (prevenindo saltos visuais durante o carregamento de imagens e ícones).

#### 15.5 Critérios de Aceitação para Pull Requests (DoD - Definition of Done)

Nenhum código deve ser mesclado à branch principal sem atender à seguinte checagem:

- [ ] `npx tsc --noEmit` executa com **zero erros de TypeScript**.
- [ ] `npm run lint` executa sem warnings ou erros de ESLint.
- [ ] `npm run build` compila perfeitamente todas as rotas estáticas (`generateStaticParams`) e Server Components.
- [ ] Testes unitários de formatadores e schemas executam com 100% de aprovação.
- [ ] Nenhuma rolagem horizontal involuntária em viewport mobile de 360px.

---

### 16. Segurança no Frontend e Defesa em Profundidade

Para resguardar os dados dos cidadãos e a integridade dos serviços municipais de Arcoverde, a camada cliente deve atuar como a primeira linha ativa de defesa, sem nunca substituir a validação final do servidor:

#### 16.1 Interface e Validações

- **Validação em Camada Dupla (Front-end + Back-end):**
  - Aplicar validações no Front-end (via Zod + React Hook Form) prioritariamente para usabilidade (UX), feedback visual imediato e economia de banda.
  - O Front-end assume como premissa inegociável que qualquer requisição pode ser interceptada ou forjada; logo, o Back-end **deve reproduzir rigorosamente as mesmas regras e validações** como barreira final intransponível.
- **Validação Estrita de Inputs na UI:**
  - Verificação no cliente de formatos de e-mail, limites de caracteres (`maxLength`), máscaras de entrada e campos obrigatórios antes do envio, prevenindo payloads malformados ou ataques de negação de serviço por payloads excessivamente volumosos.
- **Prevenção de CSS Injection na UI:**
  - Restrição ao uso de `style` inline arbitrário com dados vindos de usuários ou APIs não confiáveis.
  - Controle rígido de seletores visuais para impedir que injeções de estilo alterem a opacidade de elementos legítimos, criem sobreposições falsas (_click overlays_) ou exfiltrem dados confidenciais por meio de carregamento condicional de imagens em seletores CSS.

#### 16.2 Autenticação, Sessão e Proteção de Dados

- **MFA (Autenticação Multifator) para Servidores:**
  - Para rotas de gestão municipal e painel do servidor (`/login`), prever fluxo de telas para coleta e verificação de códigos temporários (TOTP via Google Authenticator, WebAuthn/FIDO2 ou SMS/WhatsApp) imediatamente após o login inicial de usuário e senha.
- **Armazenamento Seguro de Tokens de Autenticação:**
  - **Preferência por Cookies `HttpOnly` com flags `Secure` e `SameSite=Lax/Strict`:** Credenciais de longa duração e refresh tokens nunca devem ser armazenados em `localStorage` ou `sessionStorage`, pois são vulneráveis a ataques de Cross-Site Scripting (XSS).
  - Caso tokens de acesso JWT sejam manipulados no cliente, devem ser mantidos estritamente em **memória volátil** da aplicação durante o ciclo de vida da aba.
- **Prevenção de Cache Sensível em Equipamentos Públicos:**
  - Garantir que telas contendo dados sensíveis do cidadão (como extrato de solicitações ou dados pessoais) declarem cabeçalhos e diretivas de revalidação no Server Component / Route Handler para impedir que navegadores em computadores compartilhados mantenham cópias em cache local após o logout (`Cache-Control: no-store`).

#### 16.3 Defesas de Camada Web e Rede

- **Anti-CSRF (Injeção Automática de Tokens):**
  - Configuração da camada centralizada de serviços (`src/shared/api/http-client.ts`) para injetar automaticamente o token anti-CSRF nos cabeçalhos de requisições mutativas (`POST`, `PUT`, `DELETE`) via header `X-CSRF-Token` ou campo protegido de formulário.
- **Proteção contra Clickjacking (Frame Busting / CSP):**
  - Defesa na camada cliente e cabeçalhos de segurança (`X-Frame-Options: DENY` ou `Content-Security-Policy: frame-ancestors 'none'`) para impedir que qualquer tela do portal seja renderizada dentro de iframes invisíveis de terceiros com intuito de roubo de cliques ou protocolos.
- **Subresource Integrity (SRI):**
  - Utilização obrigatória de hashes criptográficos (`integrity="sha384-..."` e `crossorigin="anonymous"`) em tags `<script>` e `<link>` carregadas a partir de CDNs externas (como fontes do Google Fonts ou bibliotecas externas), garantindo que arquivos adulterados sejam imediatamente bloqueados pelo navegador.

#### 16.4 Auditoria Contínua de Dependências (Supply Chain Security)

- **Gestão Ativa de Vulnerabilidades:**
  - Execução periódica de `npm audit` no pipeline de CI/CD para detectar e corrigir proativamente dependências com vulnerabilidades conhecidas (CVEs).
  - Bloqueio de pacotes com vulnerabilidades de severidade Alta ou Crítica antes da publicação em produção.

---

### 17. Internacionalização (i18n) e Localização (l10n)

Para garantir a universalidade do acesso aos serviços municipais, o acolhimento de visitantes/turistas e a plena conformidade com padrões de governo digital, a aplicação adota arquitetura de internacionalização (i18n) e localização (l10n) integrada nativamente ao **Next.js 16 App Router**:

#### 17.1 Idiomas Suportados & Idioma Padrão

- **Idioma Oficial (Padrão):** `pt-BR` (Português do Brasil).
- **Idiomas Suplementares (Acolhimento Cidadão/Turista):**
  - `en` (Inglês) — para comunicação institucional, navegação geral e atendimento a turistas e investidores.
  - `es` (Espanhol) — para integração com visitantes do Mercosul e comunidades hispanofalantes.
- **Detecção e Persistência do Locale:**
  - O idioma preferido deve ser detectado prioritariamente via cookie persistente (`NEXT_LOCALE`), seguido pelo cabeçalho `Accept-Language` da requisição HTTP do navegador, com fallback definitivo para `pt-BR`.
  - A troca de idioma pelo usuário atualiza o cookie de forma assíncrona sem recarregar a página bruscamente (`router.refresh()` ou re-renderização de contexto).

#### 17.2 Estrutura de Dicionários no Feature-Sliced Design (FSD)

- **Localização dos Dicionários:**
  - Os dicionários de tradução residem centralizados na camada `src/shared/config/i18n/` (ou `src/shared/i18n/`):
    ```text
    src/shared/config/i18n/
    ├── dictionaries/
    │   ├── pt-br.json       # Fonte única da verdade (Single Source of Truth)
    │   ├── en.json          # Tradução espelho com as mesmas chaves
    │   └── es.json          # Tradução espelho com as mesmas chaves
    ├── types.ts             # Tipagem estrita inferida do dicionário pt-BR
    ├── get-dictionary.ts    # Carregador assíncrono para Server Components
    └── index.ts             # Exports públicos e constantes de locales
    ```
- **Type-Safety Estrita (Zero Chaves Órfãs ou Ausentes):**
  - O TypeScript infere as chaves do dicionário base (`pt-br.json`), garantindo em tempo de compilação que nenhum componente requisite uma chave inexistente ou com erro de digitação (`keyof Translation`):
    ```ts
    // src/shared/config/i18n/types.ts
    import type ptBR from "./dictionaries/pt-br.json";
    export type Translation = typeof ptBR;
    export type Locale = "pt-BR" | "en" | "es";
    ```
- **Proibição de Textos Hardcoded:**
  - É proibido manter strings soltas em componentes JSX para rótulos de botões, títulos de seções, mensagens de erro ou orientações ao cidadão. Todos os textos exibidos devem ser consumidos via dicionário tipado.

#### 17.3 Formatação de Dados Culturalmente Sensíveis (l10n)

Todas as formatações numéricas, temporais e monetárias devem delegar para a API nativa `Intl` localizada em `src/shared/lib/format/`, recebendo o `locale` ativo:

- **Moedas (`currency.ts`):** `Intl.NumberFormat(locale, { style: "currency", currency: "BRL" })` (mantendo BRL como padrão legal monetário da prefeitura, adaptando apenas o padrão de pontuação decimal/milhar conforme o locale).
- **Datas e Horas (`date.ts`):** `Intl.DateTimeFormat(locale, { dateStyle: "long" })` para garantir formatações naturais (ex: "18 de outubro de 2026" em `pt-BR` vs "October 18, 2026" em `en`).
- **Números e Quantidades:** Utilização de `Intl.PluralRules` para tratar regras de pluralização complexas entre diferentes idiomas sem concatenações manuais (`"1 item"` vs `"2 itens"`).

#### 17.4 Acessibilidade Fonética e SEO Multilíngue (e-MAG / WCAG)

- **Atributo `lang` no HTML Root:**
  - A tag `<html lang="...">` no layout raiz deve refletir dinamicamente o `locale` atual da sessão, permitindo que softwares de leitura de tela (NVDA, JAWS, VoiceOver, TalkBack) acionem o sintetizador de voz com o sotaque e fonética corretos.
- **Trechos de Idioma Divergente (`lang` inline):**
  - Caso um texto contenha termos estrangeiros (ex: nomes de programas ou marcas internacionais), o elemento deve ser envolvido com `<span lang="en">...</span>` para não prejudicar a síntese de voz do leitor de tela.
- **Seletor de Idiomas na Interface:**
  - O seletor de idiomas deve estar localizado em local de destaque previsível (Header e Sheet de Acessibilidade), com `aria-label="Selecionar idioma"`, teclado 100% navegável e touch target mínimo de **44x44px**.
  - As opções devem exibir o nome nativo de cada idioma (ex: _Português_, _English_, _Español_) para que o usuário estrangeiro reconheça seu próprio idioma mesmo antes da tradução.

#### 17.5 Resiliência de Layout à Variação Textual

- A interface deve ser projetada considerando a expansão ou retração de texto (_Text Expansion_ de 20% a 40% comum entre idiomas):
  - Proibido o uso de larguras ou alturas estáticas em pixels (`w-[120px]`, `h-[40px]`) que possam causar quebra de linha indesejada ou truncamento de texto (_text-overflow ellipsis_ inadvertido).
  - Componentes de botões, abas e cartões devem possuir `min-w`, padding flexível e layout flexbox/grid resiliente para evitar sobreposição ou quebra de grid.

---

### 18. Padrão de Componentização (Até 120 Linhas), Storybook & Design Patterns no React

Para assegurar legibilidade, testabilidade, manutenção a longo prazo e máxima coesão de código, o projeto estabelece regras mandatórias de tamanho de arquivo, documentação com Storybook e adoção sistemática dos padrões de projeto clássicos do ecossistema React.

#### 18.1 Regra de Limite de Tamanho (Até 120 Linhas por Componente)

- **Teto Estrito de 120 Linhas**:
  - Nenhum arquivo de componente (`*.tsx`) deve exceder **120 linhas de código** (incluindo imports, JSX e tipagens locais).
- **Diretriz de Fatiamento (Componentizar)**:
  - Se um componente atingir ou se aproximar de 120 linhas, ele **deve ser fatiado obrigatoriamente** utilizando os níveis do **Atomic Design**:
    - **Átomos (`ui/atoms/`)**: Elementos visuais pontuais e indivisíveis (badges, botões estilizados, steppers, contadores, ícones encapsulados).
    - **Moléculas (`ui/molecules/`)**: Combinações funcionais de átomos (cards de resumo, grupos de botões de ação, itens de timeline, campos com validação).
    - **Organismos (`ui/organisms/`)**: Seções compostas que combinam moléculas e átomos em blocos maiores de interface.
  - **Separação de Preocupações**:
    - Tipos complexos e interfaces devem residir em `model/types.ts`.
    - Lógica com estado, handlers e side-effects devem ser extraídos para custom hooks dedicados (`model/use-[feature].ts`).
    - Dicionários estáticos ou arrays de configuração devem residir em arquivos de dados (`model/[nome].data.ts`).

#### 18.2 Documentação Visual e Catálogo de Componentes com Storybook

- **Catálogo de Componentes (`*.stories.tsx`)**:
  - Todo componente fundamental de `src/shared/ui/` e componentes reutilizáveis de features devem possuir sua respectiva estória Storybook (ex.: `button.stories.tsx`, `intro-stepper.stories.tsx`).
- **Validação de Estados e Variantes**:
  - As estórias devem cobrir todas as variantes visuais e funcionais:
    - Estado padrão (_Default_ / _Primary_ / _Secondary_ / _Outline_);
    - Estados interativos (_Hover_, _Active_, _Focus-Visible_);
    - Estados assíncronos (_Loading_, _Disabled_, _Empty_, _Error_);
    - Temas claro e escuro (_Light Mode_ e _Dark Mode_).
- **Auditoria de Acessibilidade no Storybook**:
  - Utilização do addon `@storybook/addon-a11y` para inspecionar contraste de cores, labels ARIA e navegabilidade por teclado diretamente no catálogo antes da integração nas páginas.

#### 18.3 Design Patterns Fundamentais no React

A construção e refatoração de componentes no projeto devem adotar conscientemente os seguintes padrões de projeto consagrados:

##### 1. Compound Components (Componentes Compostos)

- **O que é**: Um grupo de componentes que trabalham juntos para formar uma interface completa, compartilhando um estado implícito (geralmente gerenciado via React Context API).
- **Exemplo clássico**: `<Select>` com `<Select.Option>`, `<Accordion>` com `<Accordion.Item>` e `<Accordion.Trigger>`, ou `<Tabs>` com `<Tabs.List>` e `<Tabs.Content>`, onde o componente pai gerencia a seleção/estado e os filhos apenas se encaixam declarativamente na estrutura.
- **Vantagem**: API intuitiva, flexível e altamente customizável para quem consome o componente, evitando prop drilling e excesso de props de configuração no componente raiz.

##### 2. Container / Presentational Components (Contêiner vs. Apresentação)

- **O que é**: Separa rigorosamente a lógica de negócios e estado da aparência visual:
  - **Container**: Busca dados na API, gerencia estados complexos, trata erros e lida com efeitos colaterais. Não possui estilos visuais complexos ou HTML detalhado.
  - **Presentational (Apresentação)**: Apenas recebe dados e callbacks via props e renderiza o HTML/CSS (componentes puros ou sem lógica acoplada).
- **Vantagem**: Facilita testes unitários rápidos, possibilita mock visual imediato no Storybook e maximiza a reutilização da interface visual em diferentes fluxos da aplicação.

##### 3. Provider Pattern (Provedor)

- **O que é**: Utiliza a React Context API para disponibilizar dados globais (como tema, usuário autenticado na sessão, opções de acessibilidade, idioma ativo ou estado de formulário multipart) para qualquer componente na árvore hierárquica sem a necessidade de repassar propriedades manualmente nível a nível (_prop drilling_).
- **Vantagem**: Elimina repetições desnecessárias de props na árvore de componentes e centraliza o ciclo de vida do estado global de maneira previsível.

##### 4. Custom Hooks (Ganchos Personalizados)

- **O que é**: Extração de lógica com estado (`useState`, `useReducer`), efeitos colaterais (`useEffect`) ou subscrições para funções reutilizáveis que começam obrigatoriamente com o prefixo `use` (ex.: `useLoginForm`, `useDebounce`, `useIntroFlow`, `useAccessibility`).
- **Vantagem**: Permite compartilhar lógica stateful complexa entre diferentes componentes visuais mantendo a árvore de renderização limpa e desacoplada da implementação de negócio.

##### 5. Higher-Order Components (HOC)

- **O que é**: Uma função pura que recebe um componente React como argumento e retorna um novo componente enriquecido com funcionalidades ou dados adicionais (ex.: `withAuth(MeuComponente)`, `withAccessibility(MeuComponente)`).
- **Vantagem**: Reutilização de lógica transversal (como verificação de autenticação/permissões, injeção de telemetria ou logging), servindo como alternativa estrutural complementar aos Custom Hooks.

##### 6. Render Props

- **O que é**: Um padrão onde um componente recebe uma função como propriedade (prop ou como `children`) que retorna um elemento React, delegando ao consumidor a responsabilidade exata do que e como renderizar enquanto provê internamente o estado ou controle.
- **Vantagem**: Excelente para compartilhamento dinâmico de comportamento, renderização customizável de listas virtuais e componentes de controle interativo com UI flexível.
