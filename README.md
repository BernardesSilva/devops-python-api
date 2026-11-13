# 🚀 API DevOps de Tarefas — Pipeline CI/CD com Docker, GitHub Actions e AWS

![Python](https://img.shields.io/badge/Python-3.13-blue?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?logo=fastapi&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED?logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-2088FF?logo=github-actions&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-ECR_%26_EC2-FF9900?logo=amazon-aws&logoColor=white)

Projeto desenvolvido para a disciplina de **Integração DevOps**, demonstrando a implementação completa de uma esteira automatizada de **Integração Contínua (CI)** e **Entrega Contínua (CD)** para uma API REST desenvolvida em Python com FastAPI.

* **Aluno:** Leandro Bernardes da Silva (RA: `22503400`)
* **Repositório:** [github.com/BernardesSilva/devops-python-api](https://github.com/BernardesSilva/devops-python-api)
* **Aplicação em Produção (AWS EC2):** [http://54.221.157.241/](http://54.221.157.241/)
* **Documentação Interativa (Swagger UI):** [http://54.221.157.241/docs](http://54.221.157.241/docs)

---

## 📋 Índice

1. [Arquitetura da Solução](#-arquitetura-da-solução)
2. [Estrutura do Projeto](#-estrutura-do-projeto)
3. [Endpoints da API](#-endpoints-da-api)
4. [Execução Local](#-execução-local)
5. [Esteiras de CI/CD (GitHub Actions)](#-esteiras-de-cicd-github-actions)
6. [Segredos Configurados (GitHub Secrets)](#-segredos-configurados-github-secrets)

---

## 🏗️ Arquitetura da Solução

O fluxo DevOps deste repositório automatiza todo o ciclo de vida da aplicação, desde a validação do código até a publicação em nuvem:

```text
[Desenvolvedor] 
      │ (git push / Pull Request)
      ▼
[GitHub Actions - CI] ──► Instala dependências e executa testes com Pytest
      │ (Merge na branch main)
      ▼
[GitHub Actions - CD] ──► Constrói imagem Docker e envia para o Amazon ECR
      │ (Deploy via SSH)
      ▼
[Instância AWS EC2]   ──► Faz pull da nova imagem e sobe o contentor na porta 80
```

---

## 📂 Estrutura do Projeto

```text
devops-python-api/
├── .github/
│   └── workflows/
│       ├── ci.yml             # Workflow de Integração Contínua (Testes no PR)
│       └── deploy.yml         # Workflow de Entrega Contínua (Build ECR + Deploy EC2)
├── app/
│   └── main.py                # Código-fonte da API FastAPI
├── tests/
│   └── test_api.py            # Testes automatizados com Pytest e TestClient
├── Dockerfile                 # Definição da imagem Docker da aplicação
├── requirements.txt           # Dependências de produção
└── README.md                  # Documentação do repositório
```

---

## 🔌 Endpoints da API

A API disponibiliza rotas para verificação de saúde e gestão simples de tarefas:

| Método | Rota | Descrição |
| :--- | :--- | :--- |
| `GET` | `/` | Mensagem inicial de boas-vindas da API. |
| `GET` | `/health` | Verifica o estado de saúde e a versão ativa do serviço (`{"status": "ok"}`). |
| `GET` | `/tasks` | Lista todas as tarefas registadas. |
| `POST` | `/tasks` | Cria uma nova tarefa. |
| `PATCH` | `/tasks/{task_id}/done` | Marca uma tarefa específica como concluída. |

---

## 💻 Execução Local

### Pré-requisitos
* Python 3.13+ ou Docker instalado.

### Executando com Python
1. Clone o repositório:
   ```bash
   git clone https://github.com/BernardesSilva/devops-python-api.git
   cd devops-python-api
   ```
2. Crie e ative um ambiente virtual, depois instale as dependências:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # No Windows: .venv\Scripts\activate
   pip install -r requirements.txt pytest httpx
   ```
3. Execute os testes automatizados:
   ```bash
   pytest
   ```
4. Inicie o servidor local:
   ```bash
   uvicorn app.main:app --reload --port 8000
   ```
   Aceda a `http://localhost:8000/docs`.

### Executando com Docker
```bash
docker build -t devops-python-api .
docker run -d --name api -p 8000:8000 devops-python-api
```

---

## ⚙️ Esteiras de CI/CD (GitHub Actions)

### 1. Integração Contínua (`.github/workflows/ci.yml`)
* **Gatilho:** Abertura ou atualização de *Pull Requests* direcionados à branch `main`.
* **Ações:** Configura o ambiente Python 3.13, instala as dependências da aplicação e de testes (`pytest`, `httpx`) e valida a suíte de testes antes de permitir o *merge*.

### 2. Entrega Contínua (`.github/workflows/deploy.yml`)
* **Gatilho:** *Push* ou *merge* efetuado na branch `main`.
* **Ações:**
  1. Autentica-se na AWS e realiza o login no **Amazon ECR**.
  2. Constrói a imagem Docker (`docker build`) e envia a tag `latest` para o repositório `devops-python-api` no ECR.
  3. Conecta-se via **SSH** à instância **Amazon EC2** (`54.221.157.241`), descarrega a imagem atualizada, encerra o contentor antigo e inicia o novo mapeando a porta `80` do host para a porta `8000` do contentor.

---

## 🔐 Segredos Configurados (GitHub Secrets)

Para garantir a segurança das credenciais de infraestrutura, os seguintes segredos foram configurados em *Settings > Secrets and variables > Actions*:

* `AWS_ACCESS_KEY_ID` — Chave de acesso IAM da AWS.
* `AWS_SECRET_ACCESS_KEY` — Chave secreta IAM da AWS.
* `AWS_SESSION_TOKEN` — Token de sessão temporário (AWS Academy).
* `AWS_REGION` — Região AWS utilizada (`us-east-1`).
* `EC2_HOST` — IP público da instância EC2 (`54.221.157.241`).
* `EC2_USER` — Utilizador SSH da instância Linux.
* `EC2_SSH_KEY` — Chave privada `.pem` para autenticação SSH.