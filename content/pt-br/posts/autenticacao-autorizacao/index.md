---
date: '2025-10-24T11:23:09-03:00'
draft: false
title: 'Básico sobre Autenticação e Autorização'
translationKey: "auth-vs-authz"
slug: 'autenticacao-autorizacao'
tags: ["seguranca"]

cover:
  image: "images/security-metal-detector.webp"  
  alt: "Ilustração em preto e branco de pessoas passando por detectores de metal, simbolizando autenticação"
  relative: true
---



## **Qual é a diferença entre autenticação e autorização?**

Uma das dúvidas mais comuns no desenvolvimento de sistemas seguros é compreender a diferença entre autenticação e autorização. Em resumo:

- **Autenticação** verifica **quem você é**.
- **Autorização** define **o que você pode fazer**.

Enquanto a autenticação confirma a identidade de um usuário, a autorização determina os níveis de acesso e permissões que esse usuário possui sobre determinados recursos.

Para garantir a segurança, é essencial criar APIs que implementem mecanismos de autenticação e autorização. Entre os métodos mais comuns estão:

- **API Key**
- **OAuth 2.0**
- **JSON Web Tokens (JWT)**
- **Autenticação básica (Basic Authentication)**
- **OpenID Connect**

Essas técnicas variam desde as mais simples até as mais robustas, com diferentes níveis de segurança e complexidade.

Uma API de autenticação ajuda a prevenir acessos não autorizados a recursos e dados. A escolha do método ideal depende de diversos fatores, como:

- Requisitos de segurança
- Tipo de clientes (web, mobile, IoT)
- Facilidade de implementação
- Escalabilidade
- Maturidade da equipe de desenvolvimento

Uma boa estratégia de autenticação e autorização é crucial para a **manutenibilidade**, **segurança** e **controle de uso** em aplicações modernas.

## Gerenciamento de Autenticação: Sessões, Tokens e seus Tipos

### Sessão Vs Token

Existem dois modelos amplamente utilizados:  sessões geralmente são armazenadas no servidor e tokens armazenados no cliente. Ambos têm o mesmo objetivo, mas funcionam de maneiras diferentes, com implicações diretas em escalabilidade, segurança e arquitetura da aplicação.

**Sessão**

- Armazenada **no servidor**.
- Quando o usuário faz login, o servidor cria uma sessão e armazena informações como ID do usuário, permissões, tempo de expiração, etc., geralmente em memória ou banco de dados
- Um **ID de sessão** é enviado ao cliente, normalmente via cookie.
- Cada vez que o cliente faz uma requisição, ele envia esse ID e o servidor consulta os dados da sessão.
- Escalabilidade limitada: sessões exigem que o servidor mantenha estado, o que pode exigir sincronização entre múltiplas instâncias em sistemas distribuídos.
- Mais comum em **aplicações tradicionais (web server full-stack)**.

**Token**

- Armazenado **no cliente** (geralmente no `localStorage`, `sessionStorage` ou cookie).
- Após o login, o servidor gera um **token** que carrega informações codificadas (ou apenas uma referência).
- O cliente envia esse token em cada requisição (por header, como `Authorization: Bearer <token>`).
- O servidor **valida o token**, mas não precisa guardar estado.
- **Mais escalável**: o servidor é **stateless** (não precisa manter sessão), o que é ideal para **APIs RESTful, SPAs (Single Page Applications)** e **aplicações móveis**.

### Token Opaco vs Token Auto-Contido (JWT)

Dentro do modelo de autenticação via tokens, há uma distinção importante entre dois tipos principais: **token opaco** e **token auto-contido**, sendo este último representado principalmente pelos **JWTs (JSON Web Tokens)**.

**Token Opaco**
- O token **não carrega informações úteis** para quem o recebe; é apenas uma referência (ex: um ID aleatório).
	- Exemplo: `8f6d1c3a0e1f...`
- O servidor precisa **consultar um banco de dados** ou cache para saber quem é o usuário ou quais permissões ele tem.
- **Mais seguro** por não expor dados sensíveis.
- Fácil de invalidar (só apagar do banco).
- Comum em sistemas como **OAuth2 com introspecção de token**, ou **Auth0 com tokens opacos**.

**Token Auto-contido (ex: JWT - JSON Web Token)**

- O token **contém os dados do usuário** (como ID, permissões, tempo de expiração etc.) codificados em formato JSON.
	Exemplo: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`
- É **autônomo**: o servidor não precisa consultar banco para validar, basta verificar a assinatura.
- **Mais rápido**, bom para sistemas **sem estado**.
- Porém, **mais difícil de invalidar** antes do vencimento.
- Riscos se mal utilizados (ex: expiração longa, dados sensíveis no payload).

Resumo comparativo entre modelos de autenticação:

| Tipo        | Armazenamento | Contém dados? | Validação          | Escalável? | Segurança                 |
| ----------- | ------------- | ------------- | ------------------ | ---------- | ------------------------- |
| Sessão      | Servidor      | Sim           | Pelo ID            | Média      | Alta (dados no server)    |
| Token Opaco | Cliente       | Não           | Via consulta       | Alta       | Alta (sem dados visíveis) |
| Token JWT   | Cliente       | Sim           | Local (assinatura) | Alta       | Boa (com cuidados)        |

# Conclusão

Compreender bem os conceitos de autenticação e autorização é essencial para construir sistemas **seguros, escaláveis e de fácil manutenção**.
A segurança não deve ser um complemento, mas sim **parte da arquitetura desde o início**.

# Referências

- https://roadmap.sh/guides/token-authentication
- https://blog.bughunt.com.br/triade-cia/
- https://zitadel.com/blog/jwt-vs-opaque-tokens
- https://blog.logto.io/opaque-token-vs-jwt
