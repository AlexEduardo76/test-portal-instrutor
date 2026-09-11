<div align="center">

#  Teste Portal do Instrutor

Sistema de gestão acadêmica para instrutores: turmas, unidades curriculares, alunos e controle de frequência em um único painel.

![Java](https://img.shields.io/badge/Java-21-orange?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1.0-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-Vanilla-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge)

</div>

---

## Sobre o projeto

O **Portal do Instrutor** é uma aplicação web para instrutores organizarem sua rotina em sala de aula: cadastrar turmas, dividir o conteúdo em Unidades Curriculares (UCs), gerenciar os alunos matriculados e registrar a frequência aula a aula, com relatórios de faltas automáticos.

O backend é uma API REST em **Spring Boot**, com autenticação baseada em sessão (cookie) e senhas protegidas com **BCrypt**. O frontend é **HTML, CSS e JavaScript puros** (sem framework), consumindo a API via `fetch`.

> Demonstração: adicione aqui um GIF do fluxo de login → dashboard → registro de frequência. Um GIF curto (5-10s) mostrando a tela funcionando vale mais que qualquer texto nesta seção.
>
> ```markdown
> ![demo](docs/demo.gif)
> ```

---

## Funcionalidades

- **Autenticação** — cadastro e login de instrutores, sessão via cookie, logout.
- **Turmas** — criação, edição, listagem e exclusão (lógica, por status) de turmas por instrutor.
- **Unidades Curriculares (UCs)** — cada turma pode ter várias UCs, cada uma com um número total de aulas planejadas.
- **Alunos** — matrícula de alunos em uma turma, com nome, matrícula e status.
- **Frequência** — registro de presença/falta por aula e por aluno, com contexto (turma + UC) e relatório consolidado de faltas.
- **Dashboard** — resumo com total de turmas, alunos, UCs, aulas registradas e alertas de frequência.

---

## Arquitetura

```
instrutor-portal/
├── backend/                    API REST (Spring Boot)
│   ├── src/main/java/br/com/portal/
│   │   ├── config/              Segurança e configuração web
│   │   ├── controller/          Endpoints REST
│   │   ├── dto/                 Objetos de requisição/resposta
│   │   ├── exception/           Exceções de negócio e handler global
│   │   ├── model/                Entidades JPA
│   │   ├── repository/          Repositórios Spring Data JPA
│   │   └── service/               Regras de negócio
│   ├── src/main/resources/
│   │   └── application.properties
│   └── Dockerfile
└── frontend/                   Cliente web (HTML + CSS + JS puro)
    ├── css/
    ├── js/
    ├── index.html               Dashboard
    ├── login.html
    ├── turmas.html
    ├── turma.html
    └── frequencia.html
```

O frontend não é servido pelo Spring Boot (não está em `resources/static`) — hoje ele roda como um site estático separado, consumindo a API em `/api/*`. Isso significa que, em desenvolvimento, você precisa abrir o `frontend/` com um servidor estático próprio (Live Server, `npx serve`, etc.), e configurar CORS ou usar um proxy se o domínio for diferente do backend.

### Modelo de dados

```
Usuario (instrutor)
   └── Turma
          ├── Aluno
          └── UnidadeCurricular
                 └── Aula
                        └── Frequencia (aluno + presença)
```

---

## Tecnologias

| Camada | Tecnologia |
|---|---|
| Linguagem | Java 21 |
| Framework | Spring Boot 4.1.0 (Web, Data JPA, Security, Validation) |
| Banco de dados | PostgreSQL |
| Build | Gradle (wrapper incluso) |
| Autenticação | Sessão HTTP (cookie) + BCrypt |
| Frontend | HTML5, CSS3, JavaScript (ES6+, sem frameworks) |
| Containerização | Docker (multi-stage build) |

---

## Pré-requisitos

- [Java 21](https://adoptium.net/) (JDK)
- [Docker](https://www.docker.com/) e Docker Compose (para o banco de dados)
- Um servidor estático simples para o frontend (ex.: extensão *Live Server* do VS Code, ou `npx serve frontend`)

Não é necessário instalar Gradle: o projeto já inclui o `gradlew`.

---

## Configuração

O backend lê as credenciais do banco por variáveis de ambiente, com valores padrão para desenvolvimento local:

| Variável | Padrão (dev) | Descrição |
|---|---|---|
| `DB_URL` | `jdbc:postgresql://localhost:5432/portal` | URL de conexão JDBC |
| `DB_USERNAME` | `portal` | Usuário do banco |
| `DB_PASSWORD` | `portal123` | Senha do banco |
| `PORT` | `10000` | Porta em que a API sobe |

Para produção, defina essas variáveis no ambiente (nunca deixe senha real hardcoded no `application.properties`).

> **Atenção antes de subir para o GitHub:** este repositório inclui uma pasta `.idea/` na raiz sem `.gitignore` correspondente. Adicione um `.gitignore` na raiz do projeto (não só dentro de `backend/`) cobrindo `.idea/`, `.vscode/`, `*.iml` e qualquer arquivo de configuração local do seu banco antes do primeiro push, para não vazar configuração de ambiente de desenvolvimento.

---

## Como rodar localmente

### 1. Subir o banco de dados

Se preferir usar o `compose.yaml.bak` como referência, renomeie para `compose.yaml` e ajuste para PostgreSQL (o arquivo atual está configurado para MySQL, mas o backend está com driver e dialect do PostgreSQL — ajuste um dos dois lados antes de subir). Alternativa rápida com Docker direto:

```bash
docker run --name portal-db -e POSTGRES_DB=portal -e POSTGRES_USER=portal -e POSTGRES_PASSWORD=portal123 -p 5432:5432 -d postgres:15
```

### 2. Rodar o backend

```bash
cd backend
./gradlew bootRun
```

A API sobe em `http://localhost:10000/api`.

### 3. Rodar o frontend

```bash
cd frontend
npx serve .
```

Abra `login.html` no navegador e crie uma conta em **Cadastrar**.

### 4. Rodar com Docker (só o backend)

```bash
cd backend
docker build -t instrutor-portal-backend .
docker run -p 8080:8080 -e DB_URL=jdbc:postgresql://host.docker.internal:5432/portal instrutor-portal-backend
```

---

## Endpoints da API

### Usuários e sessão

| Método | Rota | Descrição |
|---|---|---|
| `POST` | `/api/usuarios/cadastrar` | Cria um novo instrutor |
| `POST` | `/api/usuarios/login` | Autentica e abre sessão |
| `POST` | `/api/usuarios/logout` | Encerra a sessão |
| `GET` | `/api/usuarios/me` | Dados do instrutor logado |
| `PUT` | `/api/usuarios/me` | Atualiza o perfil |

### Turmas

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/api/turmas` | Lista as turmas do instrutor logado |
| `GET` | `/api/turmas/{id}` | Detalha uma turma |
| `POST` | `/api/turmas` | Cria uma turma |
| `PUT` | `/api/turmas/{id}` | Atualiza uma turma |
| `DELETE` | `/api/turmas/{id}` | Remove uma turma |

### Unidades Curriculares

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/api/ucs/turma/{turmaId}` | Lista UCs de uma turma |
| `POST` | `/api/ucs/turma/{turmaId}` | Cria uma UC |
| `PUT` | `/api/ucs/{id}` | Atualiza uma UC |
| `DELETE` | `/api/ucs/{id}` | Remove uma UC |

### Alunos

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/api/alunos/turma/{turmaId}` | Lista alunos de uma turma |
| `POST` | `/api/alunos/turma/{turmaId}` | Matricula um aluno |
| `PUT` | `/api/alunos/{id}` | Atualiza um aluno |
| `DELETE` | `/api/alunos/{id}` | Remove um aluno |

### Frequência

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/api/frequencias/contexto` | Contexto (turma + UC) para lançar frequência |
| `GET` | `/api/frequencias/aulas` | Lista aulas de uma UC |
| `POST` | `/api/frequencias/salvar-aula` | Cria uma aula e registra presenças |
| `GET` | `/api/frequencias/relatorio` | Relatório de faltas por turma/UC |

### Dashboard

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/api/dashboard` | Resumo com estatísticas do instrutor |

Todas as rotas, exceto cadastro e login, exigem sessão ativa (cookie enviado automaticamente pelo navegador). Sem sessão válida, a API responde com erro tratado por `SessaoExpiradaException`.

---

## Segurança

Pontos que valem atenção antes de considerar este projeto pronto para produção:

- O `SecurityConfig` atual libera **todas** as requisições (`anyRequest().permitAll()`) e desativa CSRF, form login e HTTP Basic. Isso significa que o controle de acesso hoje depende inteiramente da checagem manual de sessão dentro de cada `Service` — não existe uma camada de autorização centralizada do Spring Security de fato.
- Senhas são armazenadas com **BCrypt** (força 12), o que é uma boa prática.
- O frontend guarda uma cópia dos dados do usuário no `localStorage` além do cookie de sessão do backend — são duas fontes de verdade para o mesmo estado, que podem ficar dessincronizadas (ex.: sessão expira no servidor, mas o `localStorage` ainda mostra o usuário como logado até a próxima chamada à API falhar).
- Confira o aviso na seção **Configuração** sobre a pasta `.idea/` da raiz antes do primeiro push.

---

## Contribuindo

1. Faça um fork do projeto
2. Crie uma branch (`git checkout -b feature/minha-feature`)
3. Commit suas alterações (`git commit -m 'feat: adiciona minha feature'`)
4. Envie para o seu fork (`git push origin feature/minha-feature`)
5. Abra um Pull Request

---

## Licença

Distribuído sob a licença MIT. Veja `LICENSE` para mais detalhes.

---

<div align="center">

Desenvolvido por **PITs**

</div>
