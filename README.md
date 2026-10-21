<p align="center">
  <img src="docs/assets/logo-wordmark.png" alt="AI Nexus" width="320" />
</p>

# AI Nexus

Plataforma SaaS B2B em que empresas configuram e conversam com agentes de IA especializados. Este repositório é a porta de entrada do projeto: reúne a visão geral e aponta para os repositórios de cada aplicação.

## Repositórios

| Repositório | O que é                              | Estado                                              |
|-------------|--------------------------------------|-----------------------------------------------------|
| Backend     | API em Java 21 + Spring Boot         | **Autenticação** (refresh token e recuperação de senha por e-mail), **agentes** (listar, ver e editar) e **conversas** (listar, abrir, carregar mensagens antigas e criar) prontos. O envio de mensagens (roteador de agentes + Gemini) existe no serviço, mas ainda não tem rota. |
| Frontend    | React + TypeScript + Vite            | Interface completa. **Autenticação, recuperação de senha, agentes e conversas (listar, abrir, criar)** integrados ao backend; o resto roda sobre dados mockados. |

> Cada repositório tem um README próprio com instruções de execução, configuração e detalhes técnicos. Este documento cobre só o que atravessa os dois.

## Integração entre as duas aplicações

O frontend usa o backend de verdade para `register`, `login`, `me`, `refresh`, `logout`, a recuperação de senha, `/api/agents` e as rotas de leitura e criação de `/api/conversations`.

Continuam mockados no frontend: o envio de mensagens (chat em streaming), renomear, arquivar e excluir conversas, base de conhecimento, dashboard e organização. Ferramentas, fontes de conhecimento e permissões dos agentes ainda não existem no backend.

| Funcionalidade                                         | Backend | Frontend |
|--------------------------------------------------------|---------|----------|
| Cadastro, login, `me`, refresh e logout                | Pronto  | Integrado |
| Recuperação de senha (código por e-mail)               | Pronto  | Integrado |
| Agentes (listar, ver e editar)                         | Pronto  | Integrado |
| Conversas (listar, abrir, mensagens antigas, criar)    | Pronto  | Integrado |
| Envio de mensagem e resposta do agente                 | Serviço pronto, sem rota | Mock |
| Renomear, arquivar e excluir conversas                 | Serviço pronto, sem rota | Mock |
| Base de conhecimento                                   | Não existe | Mock |
| Dashboard                                              | Não existe | Mock |

## Como as peças se encaixam

```
Navegador (React)
   │  /api/*  (proxy do Vite em desenvolvimento; encaminhamento de /api em produção)
   ▼
Backend (Spring Boot, porta 8080)
   ├── PostgreSQL        usuários, tokens, agentes, conversas e mensagens
   ├── Google Gemini     roteador de agentes e geração de respostas
   └── SMTP              código de recuperação de senha
```

- **Autenticação:** o login devolve um access token JWT de curta duração, que o frontend manda em `Authorization: Bearer`. O refresh token vai num cookie `HttpOnly` e é trocado por um novo par quando o access token expira. O backend não usa sessão.
- **Dono dos dados:** o `userId` sai sempre do JWT. O frontend nunca informa o usuário, e uma conversa de outro usuário responde como inexistente.
- **Chat:** a cada mensagem, um agente roteador (Gemini) escolhe qual agente responde, e a resposta é salva com tokens de entrada, tokens de saída e latência.

## Stack

| Camada   | Tecnologias                                                                                           |
|----------|-------------------------------------------------------------------------------------------------------|
| Backend  | Java 21, Spring Boot 4.1.1 (Web MVC, Data JPA, Validation, OAuth2 Resource Server), Spring AI (Google GenAI), Nimbus JOSE, PostgreSQL, Spring Mail, springdoc-openapi, H2 (testes) |
| Frontend | React 19, TypeScript, Vite, Tailwind CSS 4, Radix UI (estilo shadcn/ui), TanStack Query, React Router 7, Zod, Recharts, react-markdown |

## Rodando o projeto completo

1. Suba o backend seguindo o README do repositório do backend. Ele precisa de PostgreSQL, de uma conta de e-mail para a recuperação de senha e de uma chave do Gemini. O Swagger UI fica em `http://localhost:8080/swagger-ui.html`.
2. Suba o frontend seguindo o README do repositório do frontend. Em desenvolvimento ele faz proxy de `/api` para `http://localhost:8080`, então não é preciso configurar CORS.

## Próximos passos

- Rota de envio de mensagem (`POST /api/conversations/{id}/messages`), com resposta em streaming, e a troca do chat mockado do frontend por ela.
- Rotas de renomear, arquivar, desarquivar e excluir conversas.
- `updatedAt` e prévia da última mensagem na listagem de conversas, para o frontend ordenar e mostrar a lista como no mock.
- Handler de erros das conversas no backend (`404` e `502`).
- Criação e exclusão de agentes.
- Ferramentas, fontes de conhecimento e permissões dos agentes.
- Base de conhecimento, dashboard e organização no backend.
