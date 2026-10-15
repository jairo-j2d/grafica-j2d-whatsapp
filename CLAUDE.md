# CLAUDE.md

This file provides comprehensive guidance to Claude AI when working with the Evolution API codebase.

## Project Overview

**Evolution API** is a powerful, production-ready REST API for WhatsApp communication that supports multiple WhatsApp providers:
- **Baileys** (WhatsApp Web) - Open-source WhatsApp Web client
- **Meta Business API** - Official WhatsApp Business API
- **Evolution API** - Custom WhatsApp integration

Built with **Node.js 20+**, **TypeScript 5+**, and **Express.js**, it provides extensive integrations with chatbots, CRM systems, and messaging platforms in a **multi-tenant architecture**.

## Common Development Commands

### Build and Run
```bash
# Development
npm run dev:server    # Run in development with hot reload (tsx watch)

# Production
npm run build        # TypeScript check + tsup build
npm run start:prod   # Run production build

# Direct execution
npm start           # Run with tsx
```

### Code Quality
```bash
npm run lint        # ESLint with auto-fix
npm run lint:check  # ESLint check only
npm run commit      # Interactive commit with commitizen
```

### Database Management
```bash
# Set database provider first
export DATABASE_PROVIDER=postgresql  # or mysql

# Generate Prisma client (automatically uses DATABASE_PROVIDER env)
npm run db:generate

# Deploy migrations (production)
npm run db:deploy      # Unix/Mac
npm run db:deploy:win  # Windows

# Development migrations (with sync to provider folder)
npm run db:migrate:dev      # Unix/Mac
npm run db:migrate:dev:win  # Windows

# Open Prisma Studio
npm run db:studio

# Development migrations
npm run db:migrate:dev      # Unix/Mac
npm run db:migrate:dev:win  # Windows
```

### Testing
```bash
npm test    # Run tests with watch mode
```

## Architecture Overview

### Core Structure
- **Multi-tenant SaaS**: Complete instance isolation with per-tenant authentication
- **Multi-provider database**: PostgreSQL and MySQL via Prisma ORM with provider-specific schemas and migrations
- **WhatsApp integrations**: Baileys, Meta Business API, and Evolution API with unified interface
- **Event-driven architecture**: EventEmitter2 for internal events + WebSocket, RabbitMQ, SQS, NATS, Pusher for external events
- **Microservices pattern**: Modular integrations for chatbots, storage, and external services

### Directory Layout
```
src/
├── api/
│   ├── controllers/     # HTTP route handlers (thin layer)
│   ├── services/        # Business logic (core functionality)
│   ├── repository/      # Data access layer (Prisma)
│   ├── dto/            # Data Transfer Objects (simple classes)
│   ├── guards/         # Authentication/authorization middleware
│   ├── integrations/   # External service integrations
│   │   ├── channel/    # WhatsApp providers (Baileys, Business API, Evolution)
│   │   ├── chatbot/    # AI/Bot integrations (OpenAI, Dify, Typebot, Chatwoot)
│   │   ├── event/      # Event systems (WebSocket, RabbitMQ, SQS, NATS, Pusher)
│   │   └── storage/    # File storage (S3, MinIO)
│   ├── routes/         # Express route definitions (RouterBroker pattern)
│   └── types/          # TypeScript type definitions
├── config/             # Environment and app configuration
├── cache/             # Redis and local cache implementations
├── exceptions/        # Custom HTTP exception classes
├── utils/            # Shared utilities and helpers
└── validate/         # JSONSchema7 validation schemas
```

### Key Integration Points

**Channel Integrations** (`src/api/integrations/channel/`):
- **Baileys**: WhatsApp Web client with QR code authentication
- **Business API**: Official Meta WhatsApp Business API
- **Evolution API**: Custom WhatsApp integration
- Connection lifecycle management per instance with automatic reconnection

**Chatbot Integrations** (`src/api/integrations/chatbot/`):
- **EvolutionBot**: Native chatbot with trigger system
- **Chatwoot**: Customer service platform integration
- **Typebot**: Visual chatbot flow builder
- **OpenAI**: AI capabilities including GPT and Whisper (audio transcription)
- **Dify**: AI agent workflow platform
- **Flowise**: LangChain visual builder
- **N8N**: Workflow automation platform
- **EvoAI**: Custom AI integration

**Event Integrations** (`src/api/integrations/event/`):
- **WebSocket**: Real-time Socket.io connections
- **RabbitMQ**: Message queue for async processing
- **Amazon SQS**: Cloud-based message queuing
- **NATS**: High-performance messaging system
- **Pusher**: Real-time push notifications

**Storage Integrations** (`src/api/integrations/storage/`):
- **AWS S3**: Cloud object storage
- **MinIO**: Self-hosted S3-compatible storage
- Media file management and URL generation

### Database Schema Management
- Separate schema files: `postgresql-schema.prisma` and `mysql-schema.prisma`
- Environment variable `DATABASE_PROVIDER` determines active database
- Migration folders are provider-specific and auto-selected during deployment

### Authentication & Security
- **API key-based authentication** via `apikey` header (global or per-instance)
- **Instance-specific tokens** for WhatsApp connection authentication
- **Guards system** for route protection and authorization
- **Input validation** using JSONSchema7 with RouterBroker `dataValidate`
- **Rate limiting** and security middleware
- **Webhook signature validation** for external integrations

## Important Implementation Details

### WhatsApp Instance Management
- Each WhatsApp connection is an "instance" with unique name
- Instance data stored in database with connection state
- Session persistence in database or file system (configurable)
- Automatic reconnection handling with exponential backoff

### Message Queue Architecture
- Supports RabbitMQ, Amazon SQS, and WebSocket for events
- Event types: message.received, message.sent, connection.update, etc.
- Configurable per instance which events to send

### Media Handling
- Local storage or S3/Minio for media files
- Automatic media download from WhatsApp
- Media URL generation for external access
- Support for audio transcription via OpenAI

### Multi-tenancy Support
- Instance isolation at database level
- Separate webhook configurations per instance
- Independent integration settings per instance

## Environment Configuration

Key environment variables are defined in `.env.example`. The system uses a strongly-typed configuration system via `src/config/env.config.ts`.

Critical configurations:
- `DATABASE_PROVIDER`: postgresql or mysql
- `DATABASE_CONNECTION_URI`: Database connection string
- `AUTHENTICATION_API_KEY`: Global API authentication
- `REDIS_ENABLED`: Enable Redis cache
- `RABBITMQ_ENABLED`/`SQS_ENABLED`: Message queue options

## Development Guidelines

The project follows comprehensive development standards defined in `.cursor/rules/`:

### Core Principles
- **Always respond in Portuguese (PT-BR)** for user communication
- **Follow established architecture patterns** (Service Layer, RouterBroker, etc.)
- **Robust error handling** with retry logic and graceful degradation
- **Multi-database compatibility** (PostgreSQL and MySQL)
- **Security-first approach** with input validation and rate limiting
- **Performance optimizations** with Redis caching and connection pooling

### Code Standards
- **TypeScript strict mode** with full type coverage
- **JSONSchema7** for input validation (not class-validator)
- **Conventional Commits** enforced by commitlint
- **ESLint + Prettier** for code formatting
- **Service Object pattern** for business logic
- **RouterBroker pattern** for route handling with `dataValidate`

### Architecture Patterns
- **Multi-tenant isolation** at database and instance level
- **Event-driven communication** with EventEmitter2
- **Microservices integration** pattern for external services
- **Connection pooling** and lifecycle management
- **Caching strategy** with Redis primary and Node-cache fallback

## Testing Approach

Currently, the project has minimal formal testing infrastructure:
- **Manual testing** is the primary approach
- **Integration testing** in development environment
- **No unit test suite** currently implemented
- Test files can be placed in `test/` directory as `*.test.ts`
- Run `npm test` for watch mode development testing

### Recommended Testing Strategy
- Focus on **critical business logic** in services
- **Mock external dependencies** (WhatsApp APIs, databases)
- **Integration tests** for API endpoints
- **Manual testing** for WhatsApp connection flows

## Deployment Considerations

- Docker support with `Dockerfile` and `docker-compose.yaml`
- Graceful shutdown handling for connections
- Health check endpoints for monitoring
- Sentry integration for error tracking
- Telemetry for usage analytics (non-sensitive data only)
## Publicações de redes sociais da Gráfica J2D (Instagram / stories)

Preferências do Jairo para posts e stories. Seguir sempre que ele pedir uma publicação.

- **Negócio:** a Gráfica J2D faz **impressão digital e corte a laser**. Não fazemos offset. Nunca citar offset nos textos.
- **Especialidade do Jairo:** sistemas de informação e inteligência artificial.
- **Tema das curiosidades:** preferir boas práticas e informações úteis de **IA e informática aplicadas ao ambiente gráfico**
  (resolução e dpi, RGB x CMYK, vetor x bitmap, formatos de arquivo, uso correto de IA para gerar artes, backup, segurança de arquivos).
  Evitar curiosidades soltas sem utilidade prática. O conteúdo deve trazer benefício real ao cliente.
- **Precisão:** conferir contas e fatos antes de publicar (ex.: 1024 px ÷ 300 dpi × 2,54 = 8,7 cm). Não afirmar serviços ou materiais que a J2D não confirmou.
- **Formato:** stories em **1080×1920** (vertical), com margem de segurança de ~190 px no topo e ~240 px na base. Post de feed só se o Jairo pedir (1080×1350).
- **Identidade visual:** fundo azul-escuro (#0d1b2a) com manchas nas cores CMYK, fonte Inter, logo oficial em `grafica-j2d-landing-page/src/assets/logo-j2d-27bae.png`,
  rodapé com "Gráfica J2D · Impressão digital e corte a laser" e um botão de chamada para ação.
- **Fluxo:** montar a arte em HTML, renderizar em PNG com Playwright, enviar ao Jairo para aprovar e entregar também a legenda/texto sugerido em português (PT-BR).
- **Modelo preferido (aprovado e elogiado pelo Jairo):** stories de **dicas de boas práticas** com selo "Boa prática",
  título curto em forma de pergunta e resposta, comparação visual lado a lado (errado ✕ magenta / certo ✓ ciano),
  passo a passo numerado ("Como fazer") e um alerta curto antes do rodapé. Priorizar esse formato.
- **Já publicados:** "O K do CMYK vem de Key", "Imagem de IA de 1024 px imprime só 8,7 cm" e
  "Arte pelo WhatsApp? Envie como documento" e "Você sabia? As cores deste escudo não são pintadas" (vídeo exploded do escudo).
  Não repetir esses temas.
- **Stories em vídeo:** vídeos de produto (ex.: efeito "Exploded View Loop" da Higgsfield, ~36 créditos por geração) recebem
  camada de texto por cima com ffmpeg, deixando o produto livre no centro. Sempre terminam com a **vinheta do logo da J2D com EFEITO LASER** (padrão do Jairo):
  um feixe de laser grava o logo de cima para baixo soltando faíscas, depois contorna a borda do círculo; em seguida entram
  "Gráfica J2D", "Impressão digital e corte a laser" e "Chama no direct". A vinheta tem **som de laser** sincronizado
  (zumbido do cabeçote, chiado, estalos das faíscas, "zap" no contorno), e o **áudio original do vídeo deve ser mantido**. Antes da vinheta, deixar a frase final do vídeo
  aparecer inteira (congelar ~0,9 s) e fazer transição suave. Fundo da vinheta = último quadro do vídeo desfocado; detalhes em dourado.
  Referência: as vinhetas originais com laser estão entre os vídeos enviados à Higgsfield (7 a 10/out/2026); pedir um exemplo ao Jairo
  se precisar copiar o estilo exato, pois a rede da sessão bloqueia o download desses arquivos.
