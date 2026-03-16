# LabBD_LinkedInApp: Sistema de Gerenciamento de Vagas e Currículos

Este repositório contém o código-fonte de um aplicativo web para gerenciamento de vagas de emprego e currículos, desenvolvido como projeto para a disciplina de Laboratório de Banco de Dados. A aplicação é construída em Python usando a biblioteca Streamlit.

## 🌐 Deploy do projeto
O deploy foi feito por meio da Streamlit Community Cloud. Este projeto está hospedado na URL https://fakelinkedinlabbd.streamlit.app/.

# <br>✨ Funcionalidades do Sistema

### 🔐 Autenticação e Perfis (RBAC)
* **Sistema de login seguro:** autenticação com hash de senhas (`bcrypt`) e persistência de sessão.
* **Múltiplos perfis:** controle de acesso baseado em papéis (*Role-Based Access Control*):
    * 🎓 **Candidato:** cusca vagas e gerencia seu próprio currículo.
    * 🏢 **Empregador:** cadastra vagas e busca talentos, com visão exclusiva das suas oportunidades.
    * 🔧 **Administrador:** acesso irrestrito para gestão e manutenção do sistema.

### 💼 Gestão inteligente de vagas
* **Dashboard interativo:** painel visual com métricas (KPIs) de média salarial e volume de vagas.
* **Filtros avançados:** busca por palavras-chave, slider de faixa salarial e tipo de contratação.
* **Visão personalizada:** empregadores acessam o painel "Minhas Vagas", enquanto candidatos visualizam o "Mural de Oportunidades".

### 👥 Banco de talentos
* **Cadastro estruturado:** formulário completo para candidatos (experiência, skills, idiomas) com bloqueio de múltiplos cadastros (1 currículo por usuário).
* **Visualização de perfil:** o candidato pode visualizar seu currículo formatado como um cartão profissional.
* **Busca de Candidatos:** Recrutadores podem filtrar profissionais por competências técnicas e formação.

### 🧠 Infraestrutura de IA (Vector Search)
* **Busca semântica:** integração com **MongoDB Atlas Vector Search** e **Google Gemini** para gerar *embeddings* (vetores) dos perfis e vagas.
* **Matching inteligente:** função oferecida para **administradores** que possui capacidade de encontrar vagas ou candidatos baseados no *sentido* do texto, e não apenas em palavras exatas (funcionalidade suportada pelo *backend*).
* **Resiliência:** sistema de *fallback* que mantém o funcionamento normal mesmo se a cota da API de IA for excedida.

### ☁️ Arquitetura moderna
* **Banco de dados NoSQL:** substituição de arquivos CSV por **MongoDB Atlas**, garantindo performance e escalabilidade na nuvem.
* **Interface reativa:** desenvolvido em **Streamlit**, com *feedbacks* visuais instantâneos (toasts, balloons, barras de progresso).

# <br>🔐 Documentação de perfis e permissões
O sistema utiliza controle de acesso baseado em papéis (RBAC - Role-Based Access Control), definido pelo campo tipo_usuario na coleção usuarios do MongoDB.

## 👥 Perfis de Usuário
**Existem três perfis distintos no sistema:**

### 1. 🎓 Candidato (```tipo_usuario: "candidato"```)
Usuário final que busca oportunidades de emprego.

- Objetivo: cadastrar seu perfil profissional e encontrar vagas compatíveis.
- Restrições:
  - Não pode visualizar currículos de outros candidatos;
  - Não pode cadastrar vagas.

**Lógica de dados:**
possui um campo ```id_curriculo``` no banco de dados.
- Estado inicial: ```id_curriculo: null``` (permite acessar o formulário de cadastro).
- Estado pós-cadastro (exemplo): ```id_curriculo: 105``` (o formulário é bloqueado e substituído pela visualização **"Meu Currículo"**).

### 2. 🏢 Empregador (```tipo_usuario: "empregador"```)
Representante de uma empresa que busca talentos.
- Objetivo: divulgar vagas e encontrar candidatos qualificados.
- Restrições:
  - Não pode cadastrar um currículo pessoal;
  - Não pode criar vagas associadas a outras empresas.

**Lógica de Dados:**
possui o campo empresa fixo no cadastro (ex: "Microsoft Brasil").
Ao listar vagas, o sistema aplica um filtro automático para exibir apenas registros onde ```empresa == Usuário.empresa```.

### 3. 🔧 Administrador (```tipo_usuario: "admin"```)
Superusuário responsável pela gestão e manutenção do sistema.

- Objetivo: moderação, cadastro manual e manutenção técnica.
- Privilégios exclusivos:
  - Acesso a ferramentas de sistema (ex: Gerador de Embeddings/Backfill);
  - Visão global de todas as vagas e currículos sem filtros;
  - Pode cadastrar múltiplos currículos (para fins de inserção manual de dados).

### 🚦 Matriz de permissões
| Funcionalidade                    |Candidato| Empregador           | Admin             |
|-----------------------------------|-|----------------------|-------------------|
| Login / Logout                    |✅| ✅                    | ✅                 |
| Ver **Mural de vagas (todas)**    |✅| ❌(Vê apenas as suas) | ✅                 |
| Cadastrar vaga                    |⛔| ✅ (empresa fixa)     | ✅ (empresa livre) |
| Ver **Banco de talentos (Todos)** |⛔|✅|✅|
| Cadastrar currículo               |✅ (apenas 1 vez)|⛔|✅ (ilimitado)|
| Visualizar próprio currículo      |✅|N/A|N/A|
| Admin: Gerar Embeddings|⛔|⛔|✅|

# <br>🧠 Funcionalidades de Inteligência Artificial (RAG)
O sistema oferece aos **ADMINISTRADORES** uma funcionalidade que utiliza MongoDB Atlas Vector Search e Google Gemini para realizar buscas semânticas (baseadas no sentido do texto, não apenas palavras-chave).

# <br>🛠️ Detalhes técnicos do Banco de Dados

## Coleção ```usuarios```
Estrutura básica dos documentos de login:
```
{
  "_id": ObjectId("..."),
  "email": "usuario@exemplo.com",
  "password_hash": Binary(...), // Hash seguro (bcrypt)
  "tipo_usuario": "candidato",  // ou "empregador", "admin"
  "data_cadastro": ISODate("..."),
  
  // Se Empregador:
  "empresa": "Nome da Empresa",
  
  // Se Candidato:
  "id_curriculo": 105 // ou null se ainda não cadastrou
}
```

## Segurança
- **Hash de senha:** as senhas nunca são salvas em texto puro. Utilizamos ```bcrypt``` com salt automático.
- **Proteção de rotas:** todas as páginas internas verificam ```st.session_state['logged_in']``` e ```st.session_state['tipo_usuario']``` antes de renderizar qualquer conteúdo.

# <br>🚀 Como executar localmente
Para testar a aplicação em sua máquina local, siga os passos abaixo.

**Pré-requisitos:** Python 3.8+, Git.

**Passos:**
1. Clone o repositório e navegue até a pasta do projeto:
    ```
    git clone https://github.com/Julia-Amadio/LabBD_App.git
    cd LabBD_App 
    ```
2. Crie um ambiente virtual:
    ```
    python -m venv .venv
    ```
3. Ative o ambiente virtual:
    - No **Windows**:
        - CMD (Prompt de Comando):
        ```
        .\.venv\Scripts\activate
        ```
        - No PowerShell:
        ```
        .\.venv\Scripts\Activate.ps1
        ```
    - No **macOS/Linux**:
    ```
    source .venv/bin/activate
    ```
4. Instale as dependências:
    ```
    pip install -r requirements.txt
    ```
5. Execute a aplicação Streamlit. O Streamlit irá executar o arquivo app.py (sua página de login) como ponto de entrada.
    ```
    streamlit run Login.py
    ```
6. Acesse o app abrindo o endereço http://localhost:8501 no seu navegador.

# <br>👩‍💻 Autores
- Julia Amadio
- João Bastasini
