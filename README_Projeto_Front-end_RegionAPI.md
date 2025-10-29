
# Projeto Front-end RegionAPI

## Descrição

Este é um projeto de **frontend** desenvolvido com **Vue.js 3**, que consome a **API do backend** para gerenciar **regiões** de um sistema. O objetivo deste projeto é implementar um **CRUD de regiões**, permitindo a criação, edição, ativação e inativação de regiões.

A **API** backend já está pronta e deve ser consumida pela aplicação Vue.js. Este projeto serve para praticar a integração entre o **frontend** e **backend** usando **Vue.js 3**.

---

## Funcionalidades

- **Cadastro de Região:** Permite adicionar uma nova região com a sigla do estado (UF) e nome.
- **Edição de Região:** Edita as informações de uma região já cadastrada.
- **Ativação/Inativação de Região:** Permite ativar ou inativar uma região.
- **Listagem das Regiões:** Exibe as regiões cadastradas em uma tabela.

---

## Tecnologias Utilizadas

- **Frontend:** Vue.js 3
- **Backend:** API em C# .NET
- **Banco de Dados:** PostgreSQL (no backend)
- **Gerenciamento de Estado:** Pinia (Vue 3)
- **Roteamento:** Vue Router

---

## Como Rodar

### Requisitos

- **Node.js** (versão 16 ou superior)
- **NPM** ou **Yarn**
- **PostgreSQL** (para rodar o backend)

### Passos para Executar o Projeto

1. **Clonar o repositório**

   ```bash
   git clone https://github.com/ArthurDp78/Projeto_Front-end_RegionAPI.git
   cd Projeto_Front-end_RegionAPI
   ```

2. **Instalar Dependências do Frontend**

   No diretório `frontend/`, instale as dependências do Vue.js:

   ```bash
   cd frontend
   npm install
   ```

3. **Rodar o Frontend**

   Após a instalação, execute o comando para rodar o servidor de desenvolvimento:

   ```bash
   npm run dev
   ```

   O frontend estará disponível em `http://localhost:5173`.

4. **Rodar o Backend**

   O **backend** já deve estar configurado e rodando. Se você estiver utilizando o Visual Studio, basta abrir o projeto `RegionAPI` e rodar o servidor. Certifique-se de que o banco de dados **PostgreSQL** esteja corretamente configurado.

---

## Licença

Projeto desenvolvido por Arthur Soares Marques.
