# Documentação da Arquitetura do Backend - Hope Benk

Este documento descreve a arquitetura do backend da **API do Hope Benk**, projetada para escalabilidade, manutenibilidade e alta reutilização em futuros projetos Node.js.

## 🚀 Stack Tecnológica

- **Core**: [Node.js](https://nodejs.org/) com [Express 5](https://expressjs.com/en/5x/api.html)
- **Linguagem**: [TypeScript](https://www.typescriptlang.org/) para tipagem estática segura
- **Banco de Dados**: [PostgreSQL](https://www.postgresql.org/) gerenciado via [TypeORM](https://typeorm.io/)
- **Cache e Controle de Taxa**: [Redis](https://redis.io/)
- **Filas em Segundo Plano**: [BullMQ](https://bullmq.io/) para processamento de filas e workers assíncronos
- **Validação**: [Zod](https://zod.dev/) para esquemas de requisição estritamente tipados
- **Autenticação**: [JWT](https://jwt.io/) (JSON Web Tokens) armazenado via Cookies seguros
- **Logs**: [Winston](https://github.com/winstonjs/winston)
- **Documentação**: [Swagger](https://swagger.io/) + [Scalar](https://scalar.com/)
- **Linters/Formatadores**: [Biome](https://biomejs.dev/)

---

## 🏗 Padrão Arquitetural: Modular e em Camadas (Clean Architecture Lite)

O projeto segue uma **Arquitetura Modular em Camadas**. Cada funcionalidade (Domínio) é encapsulada em um módulo, garantindo que as regras de negócio fiquem separadas da infraestrutura e dos mecanismos de entrega (HTTP).

### Camadas Principais dentro de `src/modules/{nome_do_modulo}`:

1.  **Entidades (`/entities`)**: Objetos e interfaces centrais de negócio. Define _o que_ os dados representam.
2.  **Casos de Uso (`/usecases`)**: Regras de negócio puras. Orquestra o fluxo de dados entre entidades e repositórios. Não possui conhecimento sobre Express ou banco de dados.
3.  **Repositórios (`/repositories`)**: Interfaces e implementações para persistência de dados. É onde reside a lógica específica do TypeORM.
4.  **Controladores (`/controllers`)**: Camada de entrega. Manipula requisições HTTP, extrai parâmetros, chama os Casos de Uso e retorna as respostas HTTP.
5.  **Validações (`/validations`)**: Esquemas do Zod utilizados para validar corpos, parâmetros e queries das requisições recebidas.
6.  **Templates (`/templates`)**: Templates opcionais de HTML ou estilização (como layouts de e-mail) específicos do módulo.
7.  **Requisitos (`/requirements`)**: Documentações de especificações funcionais e não-funcionais do domínio.
8.  **Rotas (`routes.ts`)**: Mapeamento dos verbos e caminhos HTTP para controladores específicos.

---

## 📂 Mapa da Estrutura de Diretórios

```text
api-hope-benk/
├── src/
│   ├── modules/              # Módulos específicos de domínio (Auth, Onboarding, etc.)
│   │   └── {module}/
│   │       ├── controllers/
│   │       ├── entities/
│   │       ├── repositories/
│   │       ├── usecases/
│   │       ├── validations/
│   │       ├── templates/    # Templates do domínio (ex: layouts de e-mail)
│   │       ├── requirements/ # Arquivos markdown de especificação
│   │       └── routes.ts
│   ├── infra/                # Configurações globais de infraestrutura
│   │   ├── database.ts       # Conexão e configuração do TypeORM
│   │   ├── migrations/       # Controle de versão do esquema do BD
│   │   └── jobs/             # Filas e Workers do BullMQ (ex: Email, Recibo)
│   ├── middlewares/          # Middlewares compartilhados do Express (Auth, RateLimit)
│   ├── config/               # Configuração global de recursos (Upload, Mail, etc.)
│   ├── shared/               # Lógica e serviços compartilhados
│   │   ├── services/         # Serviços globais compartilhados (ex: MailService)
│   │   └── types/            # Tipos e enums TypeScript globais
│   ├── redis.ts              # Conexão com o cliente Redis
│   ├── index.ts              # Ponto de entrada da aplicação e inicialização do servidor
│   └── docsViewer.ts         # Renderizador da documentação da API
├── logs/                     # Logs de execução da aplicação (combined.log, error.log)
├── uploads/                  # Armazenamento de arquivos estáticos (Local)
│   └── logger.ts             # Configuração global do logger Winston
├── biome.json                # Configuração rápida de linting e formatação
├── jest.config.ts            # Configuração de testes unitários e de integração
└── package.json              # Dependências e scripts da aplicação
```

---

## 🛠 Detalhes Chave da Implementação

### 1. Fluxo de Autenticação Seguro

- **Criptografia**: Senhas codificadas com `bcrypt`.
- **Sessão**: Utiliza cookies HTTP-Only, Secure e SameSite para mitigar ataques de XSS e CSRF.
- **Middleware**: Um `authMiddleware` centralizado valida o JWT contido nos cookies antes de permitir o acesso às rotas privadas.

### 2. Tratamento Global de Erros

Um middleware de tratamento de erros (`errorHandler`) centralizado captura todas as exceções (Síncronas e Assíncronas), garantindo que o servidor nunca pare de rodar e sempre retorne um formato JSON padronizado:

```json
{
  "message": "Descrição do erro",
  "status": "error"
}
```

### 3. Processamento de Tarefas Assíncronas (Filas)

Para tarefas computacionalmente intensivas ou que dependem de integrações externas, o backend delega a execução a Workers do **BullMQ** conectados ao **Redis**:

- **Geração de Recibos**: Emissão assíncrona de comprovantes de pagamento em formato PDF.
- **Envio de E-mails**: Processado em segundo plano via **Resend** com regras de reenvio automático (3 tentativas com recuo exponencial) e limite de taxa (máximo de 10 e-mails por segundo).
- **Rastreamento de Ciclo de Vida**: Logs via Winston registram cada etapa da execução dos jobs (Criado, Iniciado, Sucesso, Falha) para auditoria e monitoramento.

---

## 🔄 Como Reutilizar esta Estrutura em um Novo Projeto

1.  **Clonar o boilerplate**: Copie a estrutura de pastas e os arquivos de configuração (`package.json`, `tsconfig.json`, `biome.json`).
2.  **Configurar o Ambiente**: Copie o arquivo `.env.example` para `.env` e ajuste as credenciais do Banco de Dados, Redis e Resend.
3.  **Criar um Módulo**:
    - Defina primeiro a **Entidade** (Decoradores do TypeORM).
    - Crie a interface do **Repositório** e sua implementação concreta.
    - Implemente o **Caso de Uso** contendo as regras de negócio.
    - Envolva o caso de uso em um **Controlador** para gerenciar Entrada/Saída HTTP.
    - Registre as rotas em `routes.ts` e inclua-as no arquivo principal `src/index.ts`.
4.  **Executar Migrações**: Utilize `npm run migration:generate -- Nome` para versionar alterações do banco.
