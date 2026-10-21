<p align="center">
  <img src="docs/assets/logo-wordmark.png" alt="AI Nexus" width="320" />
</p>

# AI Nexus

Plataforma SaaS B2B em que empresas configuram e conversam com agentes de IA especializados. Este repositório é a porta de entrada do projeto: reúne a visão geral e aponta para os repositórios de cada aplicação.

**Aplicação no ar:** https://witty-coast-0d3519b10.5.azurestaticapps.net

## Repositórios

| Repositório | O que é | Estado |
|-------------|---------|--------|
| [Backend](https://github.com/joaopLima07/IA-Agent-functioApp) | API em Java 21 + Spring Boot | **Autenticação** (refresh token e recuperação de senha por e-mail), **agentes** (listar, ver e editar), **conversas** (listar, abrir, carregar mensagens antigas, criar, renomear, arquivar, desarquivar e excluir), **envio de mensagens com resposta em streaming** (roteador de agentes + Gemini) e **painel de métricas** prontos. |
| [Frontend](https://github.com/joaopLima07/IA-AgentWebApp) | React + TypeScript + Vite | Interface completa e **integrada ao backend** em todas as telas: autenticação, recuperação de senha, agentes, conversas, chat em streaming e dashboard. Não há mais dados mockados. |

> Cada repositório tem um README próprio com instruções de execução, configuração e detalhes técnicos. Este documento cobre só o que atravessa os dois.

## Integração entre as duas aplicações

O frontend usa o backend de verdade em tudo o que existe nele: `/api/auth`, `/api/agents`, `/api/conversations` (incluindo o envio de mensagens por SSE) e `/api/panel`.

Ferramentas, fontes de conhecimento e permissões dos agentes, base de conhecimento e organização ainda não existem no backend. A interface já tem os cartões de chamada de ferramenta, mas nada os alimenta.

| Funcionalidade | Backend | Frontend |
|----------------|---------|----------|
| Cadastro, login, `me`, refresh e logout | Pronto | Integrado |
| Recuperação de senha (código por e-mail) | Pronto | Integrado |
| Agentes (listar, ver e editar) | Pronto | Integrado |
| Conversas (listar, abrir, mensagens antigas, criar) | Pronto | Integrado |
| Renomear, arquivar, desarquivar e excluir conversas | Pronto | Integrado |
| Envio de mensagem e resposta do agente (SSE) | Pronto | Integrado |
| Dashboard (`/api/panel`) | Pronto | Integrado |
| Ferramentas dos agentes (tool calls) | Não existe | Interface pronta, sem dados |
| Base de conhecimento | Não existe | Não existe |

## Como as peças se encaixam

```
Navegador (React) ── Azure Static Web Apps
   │  /api/*  (proxy do Vite em desenvolvimento; em produção, chamadas diretas ao backend com CORS)
   ▼
Backend (Spring Boot, porta 8080) ── Azure Function App (custom handler)
   ├── PostgreSQL        usuários, tokens, agentes, conversas e mensagens
   ├── Google Gemini     roteador de agentes e geração de respostas
   └── SMTP              código de recuperação de senha
```

- **Autenticação:** o login devolve um access token JWT de curta duração, que o frontend manda em `Authorization: Bearer`. O refresh token vai num cookie `HttpOnly` e é trocado por um novo par quando o access token expira. O backend não usa sessão.
- **Dono dos dados:** o `userId` sai sempre do JWT. O frontend nunca informa o usuário, e uma conversa de outro usuário responde como inexistente.
- **Chat:** a cada mensagem, um agente roteador (Gemini) escolhe qual agente responde. A resposta chega ao frontend em streaming (SSE) e é salva com tokens de entrada, tokens de saída e latência.
- **Papéis:** o JWT traz o papel do usuário (`OWNER`, `ADMIN`, `MEMBER`). Hoje só o frontend o usa, para esconder o dashboard e bloquear a edição de agentes; o backend ainda não verifica o papel.

## Stack

| Camada   | Tecnologias |
|----------|-------------|
| Backend  | Java 21, Spring Boot 4.1.1 (Web MVC, Data JPA, Validation, OAuth2 Resource Server), Spring AI (Google GenAI), Nimbus JOSE, PostgreSQL, Spring Mail, springdoc-openapi, H2 (testes) |
| Frontend | React 19, TypeScript, Vite, Tailwind CSS 4, Radix UI (estilo shadcn/ui), TanStack Query, React Router 7, Zod, Recharts, react-markdown |

## Rodando o projeto completo

1. Suba o backend seguindo o README do [repositório do backend](https://github.com/joaopLima07/IA-Agent-functioApp). Ele precisa de PostgreSQL, de uma conta de e-mail para a recuperação de senha e de uma chave do Gemini, e localmente deve rodar com `SPRING_PROFILES_ACTIVE=dev` e `REFRESH_TOKEN_COOKIE_SAME_SITE=Lax`. O Swagger UI fica em `http://localhost:8080/swagger-ui.html`.
2. Suba o frontend seguindo o README do [repositório do frontend](https://github.com/joaopLima07/IA-AgentWebApp). Em desenvolvimento ele faz proxy de `/api` para `http://localhost:8080`, então não é preciso configurar CORS.

O chat só responde se o banco tiver o agente roteador (`id = 10`) e agentes com visibilidade `PUBLIC`; os detalhes estão no README do backend.

## Deploy

Cada aplicação tem seu próprio workflow do GitHub Actions, disparado a cada push em `main`:

- **Frontend:** build do Vite publicado no Azure Static Web Apps. O workflow define `VITE_API_BASE_URL` com a origem do backend.
- **Backend:** o jar do Spring Boot é empacotado com um JRE embutido e publicado como *custom handler* num Azure Function App. As variáveis de ambiente ficam nas *Application settings*.

Como frontend e API ficam em sites diferentes, o CORS é configurado no Function App (com credenciais habilitadas) e o cookie de refresh usa `Secure` + `SameSite=None`.

## Próximos passos

- Criação e exclusão de agentes.
- Ferramentas, fontes de conhecimento e permissões dos agentes, com o frontend já pronto para exibir as chamadas de ferramenta.
- Base de conhecimento e organização no backend.
- Verificação do papel (`OWNER`/`ADMIN`) no backend para o painel e a edição de agentes.
- `updatedAt` e prévia da última mensagem na listagem de conversas, para o frontend ordenar e mostrar a lista por atividade.
- Handler de erros das conversas no backend para o caso de agente inexistente (`404`).
- Perfil de produção explícito no backend e migrações versionadas do banco.
