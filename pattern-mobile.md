# Padrões e Arquitetura (Adaptação para React Native / Expo)

Este documento define os padrões obrigatórios de arquitetura, organização de código, boas práticas, acessibilidade nativa e auditoria para aplicações móveis.

---

> **Diretriz Geral:** Este projeto adota a arquitetura **Feature-Sliced Design (FSD)** adaptada para o ecossistema **React Native (com Expo Router)** e **Atomic Design**, utilizando a stack **React Native 0.74+, TypeScript, NativeWind (ou StyleSheet Restyle), TanStack Query, React Hook Form, Zod e bibliotecas nativas de UI**. Todos os desenvolvimentos e refatorações devem seguir rigorosamente as diretrizes abaixo.

---

## 📑 Sumário de Tópicos

<details open>
<summary><b>Clique para expandir / recolher o índice</b></summary>

### 🏛️ Arquitetura, Navegação & Estrutura

- [**0. Feature-Sliced Design (FSD) no React Native & Atomic Design**](#0-feature-sliced-design-fsd-no-react-native--atomic-design) — Hierarquia de camadas e limite de 120 linhas.
- [**1. Diretrizes de Roteamento, Navegação & Gestão de Imagens**](#1-diretrizes-de-roteamento-navegação--gestão-de-imagens) — Expo Router, Deep Linking e otimização.
- [**2. Modais Isolados e Acessíveis (Bottom Sheets)**](#2-modais-isolados-e-acessíveis-bottom-sheets--safearea) — Uso de `@gorhom/bottom-sheet` e SafeArea.
- [**3. Nomenclatura e Convenções de Arquivos**](#3-nomenclatura-e-convenções-de-arquivos) — Padrão `kebab-case`, prefixo `use-`, schemas `*.schema.ts` e tipagem estrita.

### ⚙️ Formulários, Estado & Comunicação

- [**4. Formulários, Schemas e Custom Hooks**](#4-formulários-schemas-e-custom-hooks-react-hook-form--zod) — React Hook Form + Zod e manipulação do teclado virtual.
- [**5. Gestão de Estado de Servidor e Modo Offline (TanStack Query)**](#5-gestão-de-estado-de-servidor-e-modo-offline-tanstack-query) — Cache offline-first e invalidação.
- [**6. Camada de Serviços, API & Padrão de Debounce**](#6-camada-de-serviços-api--padrão-de-debounce) — Instância `httpClient` e `useDebounce` (300ms-500ms).
- [**7. Feedback Visual, Estados Assíncronos & Prevenção**](#7-feedback-visual-estados-assíncronos--prevenção) — ActivityIndicators, Empty States e toasts nativos.
- [**8. Centralização de Formatações e Máscaras**](#8-centralização-de-formatações-e-máscaras-srcsharedlibformat) — Utilitários obrigatórios para CPF, telefone, CEP e moedas.

### 🎨 Design System, Acessibilidade & Mobile-First

- [**9. Design System, Tokens e Padronização Tipográfica**](#9-design-system-tokens-e-padronização-tipográfica) — Uso do NativeWind e proibição de cores hexadecimais soltas.
- [**10. Acessibilidade Nativa (VoiceOver / TalkBack)**](#10-acessibilidade-nativa-voiceover--talkback) — APIs de acessibilidade do RN e suporte a Dynamic Type.
- [**11. Privacidade, Permissões e LGPD**](#11-privacidade-permissões-e-lgpd) — Fluxos de consentimento para GPS/Câmera e ofuscação de dados.
- [**12. Priorização Mobile-First e Usabilidade Tátil**](#12-priorização-mobile-first-e-usabilidade-tátil) — Touch targets >= 44x44px, Thumb-Zone e uso de hitSlop.

### 🛡️ Qualidade, Testes & Segurança

- [**13. Princípios SOLID, Clean Code e DRY**](#13-princípios-solid-clean-code-e-dry) — Desacoplamento e extração de lógicas.
- [**14. Rastreabilidade de Bugs & Auditoria Visual**](#14-rastreabilidade-de-bugs--auditoria-visual) — Breakpoints mobile e catalogação de bugs.
- [**15. Estratégia de Testes Mobile**](#15-estratégia-de-testes-mobile) — Detox/Maestro (E2E), RNTL (Integração) e métricas de Framerate.
- [**16. Segurança no Frontend e Defesa em Profundidade**](#16-segurança-no-frontend-e-defesa-em-profundidade) — Armazenamento seguro via Keychain/Keystore e SSL Pinning.

### 🌐 Internacionalização & Componentização

- [**17. Internacionalização (i18n) e Localização (l10n)**](#17-internacionalização-i18n-e-localização-l10n) — Suporte multilíngue detectado pelo SO.
- [**18. Padrão de Componentização e Design Patterns**](#18-padrão-de-componentização-e-design-patterns-fundamentais) — Regra de 120 linhas, Storybook e padrões React.

</details>

---

### 0. Feature-Sliced Design (FSD) no React Native & Atomic Design

A estrutura de diretórios em `src/` segue a hierarquia rigorosa do FSD, onde camadas superiores podem consumir camadas inferiores, mas nunca o inverso:

- `app/` (no caso do Expo Router) ou `navigation/`: Roteamento, Stack Navigators, Tab Navigators e Providers globais.
- `widgets/`: Blocos autônomos de interface (BottomTabs, Headers customizados, Grelhas de Serviços).
- `features/`: Casos de uso com valor de negócio (autenticação, pedido-servico).
- `entities/`: Modelos de domínio e dados.
- `shared/`: Primitivas agnósticas (`shared/ui`, `shared/lib/format`, `shared/api`).

**Regra de Ouro da Componentização**: Nenhum arquivo de componente deve ultrapassar **120 linhas de código**. Caso atinja esse limite, deve ser obrigatoriamente fatiado em subcomponentes atômicos.

---

### 1. Diretrizes de Roteamento, Navegação & Gestão de Imagens

- **Navegação e Deep Linking**: Utilizar Expo Router (baseado em arquivos) ou React Navigation para a gestão de telas. Configurar corretamente os esquemas de _Deep Linking_ para permitir o compartilhamento de rotas e abertura de notificações.
- **Prevenção de Renderizações Bloqueantes**: Evitar cálculos pesados na thread principal de UI. O uso do `InteractionManager` ou `Reanimated` deve ser preferido para não bloquear as transições de navegação.
- **Otimização de Imagens**: É estritamente recomendado o uso da biblioteca `expo-image` ou `react-native-fast-image` (em vez do `<Image>` padrão) para tirar proveito de cache em disco e memória, transições suaves e carregamento assíncrono.

---

### 2. Modais Isolados e Acessíveis (Bottom Sheets & SafeArea)

- Todo modal ou _Bottom Sheet_ deve residir em um arquivo dedicado dentro de sua feature.
- **Requisitos Mobile**:
  - Utilizar bibliotecas robustas e otimizadas, como `@gorhom/bottom-sheet`, que rodam nativamente na thread de animações a 60fps.
  - Respeitar a `SafeAreaView` e o `KeyboardAvoidingView` para evitar que o teclado nativo ou os recortes da tela (notch/dynamic island) sobreponham o conteúdo útil.
  - Suporte a fechamento por gestos (swipe down para fechar) e bloqueio de toques fora do modal (_backdrop press_).

---

### 3. Nomenclatura e Convenções de Arquivos

- Manter o padrão `kebab-case` para pastas e componentes (ex: `src/shared/ui/password-field.tsx`).
- Hooks personalizados devem utilizar o prefixo `use-` e os schemas de validação o sufixo `*.schema.ts`.
- É **estritamente proibido** o uso de `any` ou asserções de tipo inseguras (`as unknown as Type`).

---

### 4. Formulários, Schemas e Custom Hooks (React Hook Form + Zod)

- Formulários devem utilizar **React Hook Form** integrado com **Zod**.
- A lógica deve ser dividida em 3 camadas: Schema Zod (validação), Custom Hook (gestão do estado/submissão) e Componente UI (apresentação).
- **Peculiaridades Mobile**: É fundamental tratar o fechamento do teclado através de `TouchableWithoutFeedback` e `Keyboard.dismiss()`, além de focar automaticamente o próximo input utilizando a tecla "Next/Avançar" do teclado virtual (configurando a prop `returnKeyType`).

---

### 5. Gestão de Estado de Servidor e Modo Offline (TanStack Query)

- Utilizar **TanStack Query** para chamadas assíncronas.
- Aplicações móveis sofrem com conectividade intermitente. Recomenda-se integrar o React Query com o `@react-native-async-storage/async-storage` ou `react-native-mmkv` para estabelecer um _cache_ persistente (Estratégia Offline-First).
- Toda mutação deve invalidar a respectiva _query_ no callback `onSuccess`.

---

### 6. Camada de Serviços, API & Padrão de Debounce

- Manter a instância de comunicação (`httpClient`) centralizada em `src/shared/api/`.
- O padrão de **debounce (300ms–500ms)** continua obrigatório para pesquisas (ex: autocompletar endereços ou serviços) e deve utilizar o hook `useDebounce`, repassando também o `signal` de cancelamento nativo (`AbortController`).

---

### 7. Feedback Visual, Estados Assíncronos & Prevenção

- Exibir Skeletons ou o componente nativo `ActivityIndicator` durante carregamentos. Listas vazias devem exibir um componente de `EmptyState` com um Call-to-Action.
- Prevenir duplo envio desativando o botão de submissão (`disabled={isSubmitting}`) e exibindo feedback de "loading".
- Alertas e notificações de sucesso/erro devem ser gerenciados por bibliotecas otimizadas para mobile, como `react-native-toast-message` ou `burnt` (para invocar o alerta nativo do iOS/Android).

---

### 8. Centralização de Formatações e Máscaras (`src/shared/lib/format/`)

Aplica-se a mesma rigorosidade do ambiente web:

- Funções isoladas para tratar **CPF** (`formatCPF`, `validateCPF`), **CNPJ** (em conformidade com novo padrão alfanumérico).
- Formatações de **Telefone**, **CEP** e **Moeda BRL** via API global `Intl` do JavaScript ou utilitários customizados sem dependências pesadas.

---

### 9. Design System, Tokens e Padronização Tipográfica

- Utilizar componentes atômicos consistentes como `Typography` em toda a aplicação.
- Se o projeto utilizar **NativeWind**, as classes do Tailwind são transpiladas para estilos nativos, aplicando os mesmos _tokens_ semânticos (ex: `text-primary`, `bg-background`).
- As cores hexadecimais arbitrárias soltas inline continuam expressamente proibidas.

---

### 10. Acessibilidade Nativa (VoiceOver / TalkBack)

A acessibilidade móvel difere do e-MAG / WCAG HTML, baseando-se nas APIs dos sistemas operacionais móveis:

- Substituir as tags `aria-*` por propriedades equivalentes do React Native: `accessibilityRole`, `accessibilityLabel`, `accessibilityHint` e `accessibilityState`.
- Garantir a compatibilidade com o **Dynamic Type** (iOS) e opções de ampliação de texto do Android, impedindo que a UI se quebre ao aumentar a escala das fontes nas configurações do dispositivo.
- Testar o suporte aos leitores de tela nativos (VoiceOver no iOS e TalkBack no Android), assegurando a ordem lógica de leitura.

---

### 11. Privacidade, Permissões e LGPD

- **Consentimento Nativo**: A captura de GPS e Câmera utiliza permissões estritas do SO (`expo-location`, `expo-camera`). Deve ser exibido um _rationale_ (justificativa clara via modal/tela) ao usuário **antes** de invocar o pedido de permissão nativo do sistema, cumprindo com a LGPD.
- Ocultação de dados confidenciais nas telas de consulta pública (ex: mascaramento de CPF `***.456.789-**`).

---

### 12. Priorização Mobile-First e Usabilidade Tátil

- **Área de Toque Mínima (Touch Targets >= 44x44px)**: É uma diretriz crítica em telas de smartphones.
- Assegurar que botões fundamentais de conversão residem na "Thumb-Zone" (zona de alcance fácil e natural do polegar, geralmente na metade inferior da tela).
- Garantir uma experiência de clique agradável através da propriedade nativa `hitSlop` para expandir a área interativa de botões visualmente muito pequenos (como ícones de fechar "X").

---

### 13. Princípios SOLID, Clean Code e DRY

- Componentes JSX devem ser dedicados unicamente à apresentação visual. Extração rigorosa de regras de negócio complexas para _custom hooks_.
- Criação de componentes bem isolados e modulares para prevenir duplicação de layouts.

---

### 14. Rastreabilidade de Bugs & Auditoria Visual

As resoluções de _breakpoints_ de Web adaptam-se para grupos físicos de dispositivos:

- **Pequenos (SE/Mini)**: Largura ~320 a 375px. Foco na prevenção de sobreposições de textos ou quebras de botões em telas espremidas.
- **Padrão**: Larguras ~390px a 414px (ex: iPhone 14/15, linhas Galaxy S recentes).
- **Max/Ultra**: Telas extra-longas e densas.
- **Tablets (iPad/Android Foldables)**: Garantir o uso inteligente do espaço, possivelmente convertendo _Stacks_ de navegação lineares em painéis laterais interativos (_Split Views_).

---

### 15. Estratégia de Testes Mobile

A pirâmide de testes ajusta suas ferramentas para a realidade nativa:

- **E2E (Jornadas Críticas)**: Utilizar **Detox** ou **Maestro** (em vez de Playwright) para rodar e testar os fluxos em simuladores reais de abertura de pedidos e rastreio de protocolos.
- **Integração**: Utilizar `@testing-library/react-native` (RNTL) para validação da estrutura de UI e interações integradas com o `React Hook Form`.
- **Unitários**: Vitest ou Jest para atestar a lógica matemática de máscaras e os schemas do Zod.
- **Métricas de Performance**: O rastreio muda de Core Web Vitals (LCP, CLS) para **App Launch Time**, **JS Framerate** e **UI Framerate** (com o objetivo vitalício de reter os frames perto dos 60fps).

---

### 16. Segurança no Frontend e Defesa em Profundidade

O ambiente móvel não sofre das mesmas ameaças da Web (como CSRF baseado em cookies ou Clickjacking), mas possui ameaças físicas e de payload próprias:

- **Armazenamento Seguro Mandatório**: É **estritamente proibido** salvar tokens de autenticação (JWT) ou senhas no `AsyncStorage`, pois os mesmos residem em texto plano no disco. Deve-se utilizar o **`expo-secure-store`** (que criptografa e interage com a Keychain do iOS e Keystore do Android).
- **SSL Pinning / Certificados**: Recomenda-se configurar a fixação de certificados HTTPS nas instâncias críticas do Axios/fetch para evitar ataques _Man-in-the-Middle_ (MITM) em redes públicas abertas.
- Ativar ferramentas de ofuscação de código no build nativo.

---

### 17. Internacionalização (i18n) e Localização (l10n)

- Utilizar bibliotecas robustas nativas como `i18next` em conjunto com `expo-localization` para detectar o idioma do sistema operacional, sem a necessidade de cookies.
- O idioma padrão se mantém `pt-BR`, com suporte planejado a `en` e `es`.
- A API global `Intl` continua sendo a responsável pelo trato de dados culturais sensíveis, e as chaves de tradução devem permanecer fortemente tipadas via TypeScript.

---

### 18. Padrão de Componentização e Design Patterns Fundamentais

- Manter a regra máxima de **120 linhas** por arquivo.
- O **Storybook** possui uma versão robusta para o ambiente React Native (`@storybook/react-native`) que deve ser adotada para a construção e documentação do catálogo de componentes e átomos visuais.
- Os padrões fundamentais do React (como _Custom Hooks_, separação de componentes em _Container vs. Apresentação_ e _Provider Pattern_ para injeção de dependência e estado global) mantêm-se como arquitetura obrigatória no projeto móvel.
