# 🛒 Mercado da Vila — Banco de Dados

> Projeto acadêmico desenvolvido para a disciplina de **Modelagem de Banco de Dados & SQL** — UniFECAF  
> Sistema de gestão para supermercado de grande porte com 4 filiais e 43 funcionários.

---

## 📋 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Funcionalidades](#funcionalidades)
- [Modelagem](#modelagem)
  - [Modelo Conceitual (ERD)](#modelo-conceitual-erd)
  - [Entidades e Atributos](#entidades-e-atributos)
  - [Relacionamentos](#relacionamentos)
  - [Modelo Lógico](#modelo-lógico)
- [Modelo Físico (MySQL)](#modelo-físico-mysql)
- [Como Executar](#como-executar)
- [Consultas SQL](#consultas-sql)
- [Tecnologias](#tecnologias)

---

## Sobre o Projeto

O **Mercado da Vila** é um supermercado de grande porte com 4 filiais. Este projeto modela o banco de dados do sistema de gestão da empresa, cobrindo desde o cadastro de clientes e produtos até o controle de vendas, estoque e fornecedores.

O trabalho cobre as quatro etapas clássicas de modelagem:

1. **Levantamento de Requisitos** — entrevista simulada com o cliente
2. **Modelagem Conceitual** — diagrama ER com entidades, atributos e relacionamentos
3. **Modelo Lógico** — definição de chaves primárias, estrangeiras e cardinalidades
4. **Modelo Físico** — scripts DDL e DML em MySQL

---

## Funcionalidades

- ✅ Cadastro de clientes com múltiplos endereços e e-mail único
- ✅ Catálogo de produtos organizado por categorias
- ✅ Controle de estoque com alerta de estoque mínimo
- ✅ Registro de vendas com múltiplas formas de pagamento (dinheiro, débito, crédito, PIX)
- ✅ Identificação do colaborador e caixa responsável por cada venda
- ✅ Vínculo de produtos a múltiplos fornecedores
- ✅ Histórico de compras por cliente para ações de fidelização
- ✅ Suporte a descontos por item e por venda

---

## Modelagem

### Modelo Conceitual (ERD)

O diagrama ER contempla as seguintes entidades principais e seus relacionamentos:

<img width="516" height="349" alt="image" src="https://github.com/user-attachments/assets/2a17a0fd-dafb-4b6d-9436-0ccf12231888" />


### Entidades e Atributos

| Tabela | Atributos principais |
|---|---|
| `tbl_clientes` | id, nome, cpf (UNIQUE), data_nascimento, data_cadastro |
| `tbl_email` | id, email (UNIQUE), id_cliente |
| `tbl_endereco` | id, logradouro, bairro, cep, cidade, estado, pais |
| `tbl_colaboradores` | id, nome, cpf (UNIQUE), cargo, salario, data_admissao, ativo |
| `tbl_caixas` | id, numero, descricao, ativo, data_cadastro |
| `tbl_categorias` | id, nome, descricao, data_cadastro |
| `tbl_produtos` | id, nome, codigo_barras (UNIQUE), preco, estoque, estoque_min, id_categoria |
| `tbl_fornecedores` | id, nome, cnpj (UNIQUE), telefone, email, data_cadastro |
| `tbl_formas_pagamento` | id, nome |
| `tbl_vendas` | id, data_venda, total, desconto, id_cliente, id_colaborador, id_caixa |
| `tbl_itens_venda` | id, id_venda, id_produto, quantidade, preco_unitario, desconto_item |
| **Associativas** | |
| `tbl_clientes_enderecos` | PK composta (id_cliente, id_endereco) |
| `tbl_vendas_formas_pagamento` | PK composta (id_venda, id_forma_pagamento), valor_pago |
| `tbl_produtos_fornecedores` | PK composta (id_produto, id_fornecedor) |

### Relacionamentos

| Relacionamento | Cardinalidade | Descrição |
|---|---|---|
| Cliente → Email | 1:1 | Cada cliente tem um único e-mail |
| Cliente → Endereço | N:M | Um cliente pode ter vários endereços |
| Cliente → Venda | 1:N | Um cliente realiza várias vendas |
| Colaborador → Venda | 1:N | Um colaborador regista várias vendas |
| Caixa → Venda | 1:N | Um caixa processa várias vendas |
| Venda → Forma de Pagamento | N:M | Uma venda pode usar múltiplas formas de pagamento |
| Venda → Item de Venda | 1:N | Uma venda tem vários itens |
| Produto → Item de Venda | 1:N | Um produto aparece em vários itens de venda |
| Categoria → Produto | 1:N | Uma categoria agrupa vários produtos |
| Produto → Fornecedor | N:M | Um produto pode ter múltiplos fornecedores |

### Modelo Lógico

Principais decisões de modelagem lógica:

- **tbl_email**: `id_cliente` com `UNIQUE` para garantir relação 1:1
- **tbl_clientes_enderecos**: tabela associativa com PK composta para relação N:M
- **tbl_vendas_formas_pagamento**: tabela associativa com `valor_pago` para rastrear quanto foi pago em cada modalidade
- **tbl_produtos_fornecedores**: tabela associativa para suportar múltiplos fornecedores por produto

---

## Modelo Físico (MySQL)

### Pré-requisitos

- MySQL 8.0+
- MySQL Workbench, TablePlus ou cliente equivalente

### Estrutura de ficheiros

```
mercado-da-vila/
├── README.md
└── mercado_da_vila.sql   ← DDL + DML completo
```

### Script completo

```sql
-- =====================================================
-- MERCADO DA VILA — Script completo (DDL + DML)
-- =====================================================

CREATE DATABASE IF NOT EXISTS db_mercado_da_vila;
USE db_mercado_da_vila;

-- Tabela de categorias
CREATE TABLE tbl_categorias (
  id           INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nome         VARCHAR(50) NOT NULL,
  descricao    TEXT,
  data_cadastro DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de fornecedores
CREATE TABLE tbl_fornecedores (
  id           INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nome         VARCHAR(100) NOT NULL,
  cnpj         VARCHAR(18) UNIQUE NOT NULL,
  telefone     VARCHAR(20),
  email        VARCHAR(100),
  data_cadastro DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de clientes
CREATE TABLE tbl_clientes (
  id              INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nome            VARCHAR(100) NOT NULL,
  cpf             VARCHAR(14) UNIQUE NOT NULL,
  data_nascimento DATE,
  data_cadastro   DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de colaboradores
CREATE TABLE tbl_colaboradores (
  id            INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nome          VARCHAR(100) NOT NULL,
  cpf           VARCHAR(14) UNIQUE NOT NULL,
  cargo         VARCHAR(50) NOT NULL,
  salario       DECIMAL(10,2) NOT NULL,
  data_admissao DATE NOT NULL,
  ativo         BOOLEAN DEFAULT TRUE
);

-- Tabela de caixas
CREATE TABLE tbl_caixas (
  id           INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  numero       INT NOT NULL,
  descricao    VARCHAR(50),
  ativo        BOOLEAN DEFAULT TRUE,
  data_cadastro DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Tabela de formas de pagamento
CREATE TABLE tbl_formas_pagamento (
  id   INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(50) NOT NULL
);

-- Tabela de e-mail (1:1 com clientes)
CREATE TABLE tbl_email (
  id         INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  email      VARCHAR(45) UNIQUE NOT NULL,
  id_cliente INT NOT NULL UNIQUE,
  CONSTRAINT FK_Cliente_Email
    FOREIGN KEY (id_cliente) REFERENCES tbl_clientes(id)
);

-- Tabela de endereços
CREATE TABLE tbl_endereco (
  id         INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  logradouro VARCHAR(45) NOT NULL,
  bairro     VARCHAR(45) NOT NULL,
  cep        VARCHAR(45) NOT NULL,
  cidade     VARCHAR(45) NOT NULL,
  estado     VARCHAR(45) NOT NULL,
  pais       VARCHAR(45) NOT NULL
);

-- Tabela associativa: clientes ↔ endereços (N:M)
CREATE TABLE tbl_clientes_enderecos (
  id_cliente  INT NOT NULL,
  id_endereco INT NOT NULL,
  PRIMARY KEY (id_cliente, id_endereco),
  CONSTRAINT FK_Clientes_Enderecos
    FOREIGN KEY (id_cliente) REFERENCES tbl_clientes(id),
  CONSTRAINT FK_Enderecos_Clientes
    FOREIGN KEY (id_endereco) REFERENCES tbl_endereco(id)
);

-- Tabela de produtos
CREATE TABLE tbl_produtos (
  id            INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nome          VARCHAR(100) NOT NULL,
  codigo_barras VARCHAR(50) UNIQUE,
  descricao     TEXT,
  preco         DECIMAL(10,2) NOT NULL,
  estoque       INT NOT NULL,
  estoque_min   INT DEFAULT 10,
  id_categoria  INT,
  data_cadastro DATETIME DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT FK_Produto_Categoria
    FOREIGN KEY (id_categoria) REFERENCES tbl_categorias(id)
);

-- Tabela de vendas
CREATE TABLE tbl_vendas (
  id             INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  data_venda     DATETIME DEFAULT CURRENT_TIMESTAMP,
  total          DECIMAL(10,2) NOT NULL,
  desconto       DECIMAL(10,2) DEFAULT 0.00,
  id_cliente     INT,
  id_colaborador INT,
  id_caixa       INT,
  CONSTRAINT FK_Venda_Cliente
    FOREIGN KEY (id_cliente) REFERENCES tbl_clientes(id),
  CONSTRAINT FK_Venda_Colaborador
    FOREIGN KEY (id_colaborador) REFERENCES tbl_colaboradores(id),
  CONSTRAINT FK_Venda_Caixa
    FOREIGN KEY (id_caixa) REFERENCES tbl_caixas(id)
);

-- Tabela associativa: vendas ↔ formas de pagamento (N:M)
CREATE TABLE tbl_vendas_formas_pagamento (
  id_venda          INT NOT NULL,
  id_forma_pagamento INT NOT NULL,
  valor_pago        DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (id_venda, id_forma_pagamento),
  CONSTRAINT FK_Vendas_Formas_Pagamento
    FOREIGN KEY (id_venda) REFERENCES tbl_vendas(id),
  CONSTRAINT FK_Formas_Pagamento_Vendas
    FOREIGN KEY (id_forma_pagamento) REFERENCES tbl_formas_pagamento(id)
);

-- Tabela de itens de venda
CREATE TABLE tbl_itens_venda (
  id             INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  id_venda       INT NOT NULL,
  id_produto     INT NOT NULL,
  quantidade     INT NOT NULL,
  preco_unitario DECIMAL(10,2) NOT NULL,
  desconto_item  DECIMAL(10,2) DEFAULT 0.00,
  CONSTRAINT FK_Itens_Venda_Vendas
    FOREIGN KEY (id_venda) REFERENCES tbl_vendas(id),
  CONSTRAINT FK_Itens_Venda_Produtos
    FOREIGN KEY (id_produto) REFERENCES tbl_produtos(id)
);

-- Tabela associativa: produtos ↔ fornecedores (N:M)
CREATE TABLE tbl_produtos_fornecedores (
  id_produto    INT NOT NULL,
  id_fornecedor INT NOT NULL,
  PRIMARY KEY (id_produto, id_fornecedor),
  CONSTRAINT FK_Produtos_Fornecedores
    FOREIGN KEY (id_produto) REFERENCES tbl_produtos(id),
  CONSTRAINT FK_Fornecedores_Produtos
    FOREIGN KEY (id_fornecedor) REFERENCES tbl_fornecedores(id)
);

-- =====================================================
-- DADOS DE EXEMPLO (DML)
-- =====================================================

INSERT INTO tbl_categorias (nome, descricao) VALUES
  ('Mercearia',  'Arroz, feijão, açúcar e similares'),
  ('Laticínios', 'Leite, queijo, iogurte'),
  ('Bebidas',    'Refrigerantes, sumos, água'),
  ('Limpeza',    'Produtos de limpeza doméstica'),
  ('Hortifrúti', 'Frutas, verduras e legumes');

INSERT INTO tbl_fornecedores (nome, cnpj, telefone, email) VALUES
  ('Distribuidora Norte Lda', '12.345.678/0001-90', '219000001', 'norte@dist.com'),
  ('Laticínios Sul S.A.',     '98.765.432/0001-10', '219000002', 'sul@latic.com');

INSERT INTO tbl_clientes (nome, cpf, data_nascimento) VALUES
  ('Ana Lima',    '111.222.333-44', '1995-06-10'),
  ('Bruno Costa', '222.333.444-55', '1990-03-22'),
  ('Carla Nunes', '333.444.555-66', '1998-11-05');

INSERT INTO tbl_email (email, id_cliente) VALUES
  ('ana@email.com',   1),
  ('bruno@email.com', 2),
  ('carla@email.com', 3);

INSERT INTO tbl_endereco (logradouro, bairro, cep, cidade, estado, pais) VALUES
  ('Rua das Flores, 10', 'Centro',     '01310-100', 'São Paulo', 'SP', 'Brasil'),
  ('Av. Paulista, 500',  'Bela Vista', '01311-000', 'São Paulo', 'SP', 'Brasil'),
  ('Av. Brasil, 250',    'Jardim Sul', '04020-040', 'São Paulo', 'SP', 'Brasil'),
  ('Rua do Comércio, 5', 'Vila Norte', '02030-010', 'São Paulo', 'SP', 'Brasil');

INSERT INTO tbl_clientes_enderecos (id_cliente, id_endereco) VALUES
  (1, 1), (1, 2), (2, 3), (3, 4);

INSERT INTO tbl_colaboradores (nome, cpf, cargo, salario, data_admissao) VALUES
  ('João Silva',   '444.555.666-77', 'Caixa',   1200.00, '2022-01-10'),
  ('Maria Santos', '555.666.777-88', 'Gerente', 2800.00, '2020-05-15');

INSERT INTO tbl_caixas (numero, descricao) VALUES
  (1, 'Caixa 1 — Entrada principal'),
  (2, 'Caixa 2 — Corredor central');

INSERT INTO tbl_formas_pagamento (nome) VALUES
  ('Dinheiro'), ('Cartão de Débito'), ('Cartão de Crédito'), ('PIX');

INSERT INTO tbl_produtos (nome, codigo_barras, descricao, preco, estoque, estoque_min, id_categoria) VALUES
  ('Arroz 1kg',   '7891234560001', 'Arroz branco tipo 1', 1.50, 100, 20, 1),
  ('Feijão 1kg',  '7891234560002', 'Feijão preto',        2.20,  80, 15, 1),
  ('Leite 1L',    '7891234560003', 'Leite meio gordo',    0.95, 120, 30, 2),
  ('Açúcar 1kg',  '7891234560004', 'Açúcar refinado',    1.10,  90, 20, 1),
  ('Óleo 1L',     '7891234560005', 'Óleo vegetal',        2.80,  60, 10, 2);

INSERT INTO tbl_produtos_fornecedores (id_produto, id_fornecedor) VALUES
  (1, 1), (2, 1), (3, 2), (4, 1), (5, 2);

INSERT INTO tbl_vendas (data_venda, total, desconto, id_cliente, id_colaborador, id_caixa) VALUES
  ('2026-04-21 10:00:00', 3.95, 0.00, 1, 1, 1),
  ('2026-04-21 11:30:00', 8.30, 0.00, 2, 2, 1);

INSERT INTO tbl_vendas_formas_pagamento (id_venda, id_forma_pagamento, valor_pago) VALUES
  (1, 1, 1.95),
  (1, 4, 2.00),
  (2, 2, 8.30);

INSERT INTO tbl_itens_venda (id_venda, id_produto, quantidade, preco_unitario) VALUES
  (1, 1, 2, 1.50),
  (1, 3, 1, 0.95),
  (2, 2, 2, 2.20),
  (2, 4, 1, 1.10),
  (2, 5, 1, 2.80);
```

---

## Como Executar

### Via terminal (MySQL CLI)

```bash
mysql -u root -p < mercado_da_vila.sql
```

### Via MySQL Workbench / TablePlus

1. Abrir o cliente e conectar ao servidor MySQL
2. Abrir o ficheiro `mercado_da_vila.sql`
3. Executar o script completo (⌘ + Enter / Ctrl + Enter)

---

## Consultas SQL

### 📊 Perfil completo do cliente

Retorna para cada cliente: número de visitas, total gasto, ticket médio, data da última compra, formas de pagamento utilizadas e produtos comprados.

```sql
SELECT
  c.nome                                                                          AS cliente,
  c.cpf,
  e.email,
  GROUP_CONCAT(DISTINCT en.cidade ORDER BY en.cidade SEPARATOR ', ')              AS cidades,
  COUNT(DISTINCT v.id)                                                            AS total_visitas,
  COALESCE(SUM(DISTINCT v.total), 0)                                              AS total_gasto,
  COALESCE(ROUND(AVG(DISTINCT v.total), 2), 0)                                   AS ticket_medio,
  MAX(v.data_venda)                                                               AS ultima_compra,
  GROUP_CONCAT(DISTINCT fp.nome ORDER BY fp.nome SEPARATOR ', ')                  AS formas_pagamento_usadas,
  GROUP_CONCAT(DISTINCT p.nome ORDER BY p.nome SEPARATOR ', ')                    AS produtos_comprados
FROM tbl_clientes c
LEFT JOIN tbl_email                   e   ON e.id_cliente        = c.id
LEFT JOIN tbl_clientes_enderecos      ce  ON ce.id_cliente        = c.id
LEFT JOIN tbl_endereco                en  ON en.id               = ce.id_endereco
LEFT JOIN tbl_vendas                  v   ON v.id_cliente         = c.id
LEFT JOIN tbl_vendas_formas_pagamento vfp ON vfp.id_venda         = v.id
LEFT JOIN tbl_formas_pagamento        fp  ON fp.id               = vfp.id_forma_pagamento
LEFT JOIN tbl_itens_venda             iv  ON iv.id_venda          = v.id
LEFT JOIN tbl_produtos                p   ON p.id                = iv.id_produto
GROUP BY c.id, c.nome, c.cpf, e.email
ORDER BY total_gasto DESC;
```

### 📦 Produtos abaixo do estoque mínimo

```sql
SELECT nome, estoque, estoque_min
FROM tbl_produtos
WHERE estoque <= estoque_min
ORDER BY estoque ASC;
```

### 💰 Vendas por forma de pagamento

```sql
SELECT fp.nome AS forma_pagamento, SUM(vfp.valor_pago) AS total_arrecadado
FROM tbl_vendas_formas_pagamento vfp
JOIN tbl_formas_pagamento fp ON fp.id = vfp.id_forma_pagamento
GROUP BY fp.nome
ORDER BY total_arrecadado DESC;
```

---

## Tecnologias

| Tecnologia | Uso |
|---|---|
| **MySQL 8.0** | SGBD relacional |
| **MySQL Workbench** | Modelagem e execução de queries |
| **TablePlus** | Cliente alternativo para macOS |
| **draw.io** | Diagramas ER (conceitual e lógico) |

---

## Contexto Académico

| | |
|---|---|
| **Instituição** | UniFECAF |
| **Disciplina** | Modelagem de Banco de Dados & SQL |
| **Projeto** | Mercado da Vila |
| **Etapas** | Levantamento de Requisitos → Modelo Conceitual → Modelo Lógico → Modelo Físico |
