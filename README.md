# 📡 IntraHub — Dashboard de Comunicação Interna

> Protótipo funcional de um dashboard corporativo que centraliza postagens, comentários e perfis de usuários, consumindo dados em tempo real via arquitetura **Cliente-Servidor RESTful**.

---

## 📋 Sobre o Projeto

O **IntraHub** foi desenvolvido como pré-projeto (pitch) para validar a viabilidade técnica de um Dashboard de Comunicação Interna sem a necessidade de infraestrutura própria no MVP. O sistema consome a API pública [JSONPlaceholder](https://jsonplaceholder.typicode.com) como backend simulado, demonstrando o padrão arquitetural **MVC** aplicado no front-end.

### 🎯 Problema que resolve

| Antes                                              | Depois                                           |
| -------------------------------------------------- | ------------------------------------------------ |
| Comunicados dispersos em e-mails, Slack e WhatsApp | Canal único e centralizado                      |
| Perfis de colaboradores inacessíveis              | Diretório de usuários com postagens associadas |
| Sem rastreamento de discussões                    | Comentários por postagem organizados            |

---

## 🚀 Funcionalidades

- **Dashboard geral** com métricas de usuários, postagens e comentários
- **Listagem de postagens** com títulos e conteúdo em português
- **Filtro por autor** — filtra postagens por colaborador
- **Busca em tempo real** por título de postagem
- **Perfil de usuário** com todas as postagens associadas
- **Modal de comentários** por postagem
- **Indicador de status da API** em tempo real
- **Tratamento visual de erros** — mensagem amigável quando a API falha
- **Dados localizados** — nomes, e-mails e empresas brasileiras

---

## 🏗️ Arquitetura

O projeto segue o padrão **Cliente-Servidor** com organização **MVC** no lado do cliente:

```
┌──────────────────────────────────────────────────────┐
│                    CLIENTE (Browser)                 │
│                                                      │
│  ┌──────────┐    ┌──────────┐    ┌───────────────┐  │
│  │  MODEL   │    │   VIEW   │    │  CONTROLLER   │  │
│  │          │◄──►│          │◄──►│               │  │
│  │ fetch()  │    │ HTML/CSS │    │ loadAll()     │  │
│  │ timeout  │    │ DOM      │    │ renderHome()  │  │
│  │ try/catch│    │ eventos  │    │ showPage()    │  │
│  └────┬─────┘    └──────────┘    └───────────────┘  │
│       │                                              │
└───────┼──────────────────────────────────────────────┘
        │ HTTP GET
        ▼
┌──────────────────────────────────────────────────────┐
│              SERVIDOR — JSONPlaceholder API          │
│                                                      │
│   GET /users  ·  GET /posts  ·  GET /comments        │
└──────────────────────────────────────────────────────┘
```

### Camadas MVC

| Camada               | Responsabilidade                  | Principais funções                                |
| -------------------- | --------------------------------- | --------------------------------------------------- |
| **Model**      | Acesso à API com resiliência    | `fetchWithTimeout()`                              |
| **View**       | Renderização da interface       | `postCardHTML()`, `renderUsersGrid()`           |
| **Controller** | Lógica de negócio e navegação | `loadAll()`, `showPage()`, `openPostDetail()` |

---

## 🛡️ Resiliência e Tratamento de Falhas

| Cenário            | Estratégia                                                                    |
| ------------------- | ------------------------------------------------------------------------------ |
| ⏱ API sem resposta | `AbortController` cancela a requisição após **8 segundos**          |
| ❌ HTTP 404 / 500   | `response.ok` verifica o status — erro lançado com código HTTP            |
| 📡 Sem conexão     | `fetch()` lança `TypeError` capturado no `catch` global                 |
| 🔄 Mudança no JSON | Camada Model isolada — apenas `fetchWithTimeout()` precisa de atualização |

```javascript
async function fetchWithTimeout(url) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), 8000);
  try {
    const res = await fetch(url, { signal: controller.signal });
    if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);
    return await res.json();
  } catch (err) {
    if (err.name === 'AbortError') throw new Error('Timeout: API demorou mais de 8s.');
    throw err;
  } finally {
    clearTimeout(timer);
  }
}
```

---

## 📐 Diagrama de Classes (UML)

```
┌─────────────────────┐        1       *  ┌─────────────────────┐
│       Usuario       │──────────────────►│      Postagem       │
├─────────────────────┤                   ├─────────────────────┤
│ + id: int           │                   │ + id: int           │
│ + nome: String      │                   │ + titulo: String    │
│ + email: String     │                   │ + corpo: String     │
│ + username: String  │                   │ + userId: int       │
│ + website: String   │                   ├─────────────────────┤
│ + empresa: Empresa  │                   │ + getComentarios()  │
├─────────────────────┤                   │ + exibir(): void    │
│ + getPostagens()    │                   └──────────┬──────────┘
│ + exibirPerfil()    │                              │ 1       *
└─────────────────────┘                              ▼
                                           ┌─────────────────────┐
                                           │     Comentario      │
                                           ├─────────────────────┤
                                           │ + id: int           │
                                           │ + postId: int       │
                                           │ + nome: String      │
                                           │ + email: String     │
                                           │ + corpo: String     │
                                           ├─────────────────────┤
                                           │ + exibir(): void    │
                                           └─────────────────────┘
```

---

## 📖 Histórias de Usuário

| ID             | Como...      | Desejo...                                        | Para...                                      |
| -------------- | ------------ | ------------------------------------------------ | -------------------------------------------- |
| **US01** | Funcionário | Ver uma lista de postagens recentes              | Me manter atualizado sobre a empresa         |
| **US02** | Funcionário | Visualizar os comentários de uma postagem       | Acompanhar discussões da equipe             |
| **US03** | Gestor       | Consultar o perfil de cada colaborador           | Conhecer contatos e postagens associadas     |
| **US04** | Funcionário | Filtrar postagens por autor                      | Encontrar conteúdo de um colega específico |
| **US05** | Usuário     | Ser informado quando a API estiver indisponível | Sem que o dashboard trave ou perca dados     |

---

## 🔌 Endpoints Consumidos

| Método | Endpoint      | Descrição                             | Registros     |
| ------- | ------------- | --------------------------------------- | ------------- |
| `GET` | `/users`    | Colaboradores com nome, e-mail, empresa | 10            |
| `GET` | `/posts`    | Postagens com título, corpo e autor    | 15 (limitado) |
| `GET` | `/comments` | Comentários vinculados às postagens   | ~75           |

> **Nota:** Os dados retornados pela API são fictícios.

---

## 🗂️ Estrutura do Projeto

```
intrahub/
│
├── dashboard-comunicacao.html   # Aplicação completa (single-file)
├── README.md                    # Este arquivo
└── intrahub-pitch.pptx          # Apresentação do pré-projeto (pitch)
```

> O projeto é **single-file** por design — facilita a portabilidade e demonstração do protótipo sem necessidade de servidor local.

---


## 🛠️ Tecnologias Utilizadas

| Tecnologia                | Uso                                                |
| ------------------------- | -------------------------------------------------- |
| **HTML5**           | Estrutura da aplicação                           |
| **CSS3**            | Estilização, animações e layout (Grid/Flexbox) |
| **JavaScript ES6+** | Lógica da aplicação, fetch API, async/await     |
| **JSONPlaceholder** | API REST pública para prototipagem                |
| **Google Fonts**    | Tipografia (Syne, DM Sans, IBM Plex Mono)          |

---

## 📊 Entregáveis do Projeto

- [X] **Código-fonte** do protótipo integrado ao JSONPlaceholder (`dashboard-comunicacao.html`)
- [X] **Apresentação de slides** do Pitch (`intrahub-pitch.pptx`)
- [X] **README** com documentação técnica completa

---

## 👥 Equipe

Desenvolvido por Matheus dos Santos Tenório, Matheus Vaconcelos, Pedro Augusto e José Gabriel Dâmaso.
