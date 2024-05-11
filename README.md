<h1 align="center">Modelagem do banco de dados do Edellcation</h1>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O Edellcation é uma aplicação web desenvolvida para facilitar o acesso dos funcionários das linhas de montagem aos materiais técnicos e manuais de montagem de produtos da empresa, como computadores, servidores e notebooks. A plataforma permite que os funcionários estudem, revisem e acompanhem os processos de montagem de forma personalizada, mantendo-os atualizados sobre quaisquer alterações nos procedimentos ou inclusão de novos manuais.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Para construir o banco de dados, utilizamos a estrutura do SGBD PostgreSQL no software SQLDesigner. Em seguida, com o código gerado pelo SQLDesigner, criamos um banco de dados real no DBeaver, utilizando um banco PostgreSQL hospedado no render. A partir do DBeaver, extraiu-se o modelo físico do banco.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Após análises em três encontros com nossos parceiros, identificamos a necessidade de armazenar diversas informações, que foram agrupadas em: funcionários (employees), manuais (manuals), lista de tarefas (to-do), materiais dos manuais (materials), linhas de montagem (assembly_lines) e produtos (products). Essas são as entidades iniciais do nosso banco de dados, com a compreensão de que faremos ajustes e aprimoramentos conforme a aplicação evoluir. Abaixo está o modelo físico com detalhes sobre cardinalidade, nulabilidade e tipagem de dados:

<img src="assets/databaseDiagram.png" style="max-width:100%; height:auto;" alt="Diagrama da arquitetura MVC do Edellcation">

[Diagrama em PNG](assets/databaseDiagram.png)</br>
[Diagrama em PDF](assets/databaseDiagram.pdf)</br>
[Arquivo da Modelagem (XML)](assets/database.xml) 

## Relacionamentos
- **Funcionários (employees) (1,n) - (1,n) Linha de montagem (assembly_lines):** Um Funcionários pode trabalhar em várias linhas de montagem enquanto uma linha de montagem pode conter vários operadores trabalhando;
- **Funcionários (employees) (1,1) - (0,n) Lista de Tarefas (to-do):** Um funcionário pode ter vários afazeres enquanto estes afazeres podem estar relacionados a somente um funcionário;
- **Lista de Tarefas (to-do) (0,n) - (1,1) Manuais (manuals):** Um afazer tem que possuir no máximo um manual enquanto um manual pode ser relacionado a diversos afazeres;
- **Manuais (manuals) (1,n) - (0,1) Materiais (materials):** Um manual pode conter no máximo um material enquanto um material também pode ser atribuído em vários manuais;
- **Manuais (manuals) (1,1) - (1,n) Produtos (products):** Um manual pode ensinar a respeito de diversos produtos enquanto um produto pode ser ensinado em em somento um manual;
- **Linhas de montagens (assembly_lines) (1,n) - (1,1) Produtos (products):** Uma linha de montagem fabrica no máximo um produto enquanto um produto pode ser fabricado em mais de uma linha de montagem.

> **Observação:** Alguns dos relacionamentos mencionados anteriormente podem divergir do modelo conceitual. Isso ocorre porque o DBeaver só consegue gerar com precisão as relações "n", porque estas são mais tangíveis devido à existência das chaves estrangeiras nas tabelas.

Os relacionamentos mencionados foram fundamentais para a construção do nosso banco de dados. Eles foram extraídos do entendimento do negócio do nosso parceiro, seja por meio de encontros, discussões TAPI, orientações dos professores ou deduções. Abaixo, apresentamos o código funcional do banco, já testado.:

Código SQL:
```sql
DROP TABLE IF EXISTS MANUALS_PRODUCTS CASCADE;
DROP TABLE IF EXISTS ASSEMBLY_LINES_PRODUCTS CASCADE;
DROP TABLE IF EXISTS ASSEMBLY_LINES_EMPLOYEES CASCADE;
DROP TABLE IF EXISTS TO_DOS CASCADE;
DROP TABLE IF EXISTS MANUALS CASCADE;
DROP TABLE IF EXISTS EMPLOYEES CASCADE;
DROP TABLE IF EXISTS ASSEMBLY_LINES CASCADE;
DROP TABLE IF EXISTS PRODUCTS CASCADE;
DROP TABLE IF EXISTS MATERIALS CASCADE;

CREATE TABLE MATERIALS (
    id BIGSERIAL PRIMARY KEY,
    Archives VARCHAR(150) NOT NULL DEFAULT 'NULL', -- Caminho para o material
    Types CHAR(50) NOT NULL DEFAULT 'NULL', -- Campo para informar o tipe do arquivo do manual (PDF, vídeo, modelo 3D, etc)
    Titles VARCHAR(80) NOT NULL DEFAULT 'Sem Título',
    Descriptions VARCHAR(350) DEFAULT 'Sem descrição'
);

COMMENT ON TABLE MATERIALS IS 'Arquivos que serão utilizados nos manuais';
COMMENT ON COLUMN MATERIALS.Archives IS 'Caminho para o material';
COMMENT ON COLUMN MATERIALS.Types IS 'Campo para informar a extensão do arquivo do manual (PDF, vídeo, modelo 3D, etc)';

CREATE TABLE EMPLOYEES (
    id BIGSERIAL PRIMARY KEY,
    Names VARCHAR(200) NOT NULL DEFAULT 'NULL',
    IsEngineer BOOLEAN NOT NULL DEFAULT 'f', -- Campo booleano para definir se é engenheiro ou não
    Emails VARCHAR(120) NOT NULL DEFAULT 'NULL',
    Passwords VARCHAR(50) NOT NULL DEFAULT 'NULL'
);

COMMENT ON TABLE EMPLOYEES IS 'Tabela que armazenará todos os funcionários da empresa, seja engenheiro ou operador de linha de montagem.';
COMMENT ON COLUMN EMPLOYEES.IsEngineer IS 'Campo booleano para definir se é engenheiro ou não';

CREATE TABLE ASSEMBLY_LINES (
    id BIGSERIAL PRIMARY KEY,
    Names VARCHAR(50) NOT NULL DEFAULT 'NULL',
    Descriptions VARCHAR(200) DEFAULT 'Sem descrição'
);

COMMENT ON TABLE ASSEMBLY_LINES IS 'Armazenará as linhas de montagens das quais os funcionários são pertencentes.';

CREATE TABLE PRODUCTS (
    id BIGSERIAL PRIMARY KEY, -- Número de série
    Names VARCHAR(200) NOT NULL DEFAULT 'NULL',
    Categorys VARCHAR(50) NOT NULL DEFAULT 'NULL', -- Exemplo: monitor, notebook, servidor, etc.
    Descriptions VARCHAR(300) DEFAULT 'Sem descrição',
    Subcategory VARCHAR
);

COMMENT ON TABLE PRODUCTS IS 'Equipamentos montados pela Dell';
COMMENT ON COLUMN PRODUCTS.id IS 'Número de série';
COMMENT ON COLUMN PRODUCTS.Categorys IS 'Exemplo: monitor, notebook, servidor, etc.';

CREATE TABLE MANUALS (
    id BIGSERIAL PRIMARY KEY,
    id_MATERIALS INTEGER,
    PublicationDates DATE NOT NULL DEFAULT CURRENT_DATE,
    Descriptions VARCHAR(350),
    Titles VARCHAR(80) NOT NULL DEFAULT 'Sem Título',
    Versions DECIMAL DEFAULT 0.1,
    CONSTRAINT fk_manuals_materials FOREIGN KEY (id_MATERIALS) REFERENCES MATERIALS(id)
);

COMMENT ON TABLE MANUALS IS 'Manuais para o aprendizado da montagem de produtos';

CREATE TABLE TO_DOS (
    id BIGSERIAL PRIMARY KEY,
    id_EMPLOYEES INTEGER,
    id_MANUALS INTEGER,
    Status BOOLEAN NOT NULL DEFAULT 'f', -- Campo booleano para definir se a tarefa foi concluída ou não
    AssignmentDates DATE NOT NULL DEFAULT CURRENT_DATE,
    CONSTRAINT fk_to_dos_employees FOREIGN KEY (id_EMPLOYEES) REFERENCES EMPLOYEES(id),
    CONSTRAINT fk_to_dos_manuals FOREIGN KEY (id_MANUALS) REFERENCES MANUALS(id)
);

COMMENT ON TABLE TO_DOS IS 'Lista de manuais a serem lidos pelo funcionário';
COMMENT ON COLUMN TO_DOS.Status IS 'Campo booleano para definir se a tarefa foi concluída ou não';

CREATE TABLE ASSEMBLY_LINES_EMPLOYEES (
    id_EMPLOYEES INTEGER,
    id_ASSEMBLY_LINES INTEGER,
    PRIMARY KEY (id_EMPLOYEES, id_ASSEMBLY_LINES),
    CONSTRAINT fk_assembly_lines_employees FOREIGN KEY (id_EMPLOYEES) REFERENCES EMPLOYEES(id),
    CONSTRAINT fk_assembly_lines FOREIGN KEY (id_ASSEMBLY_LINES) REFERENCES ASSEMBLY_LINES(id)
);

COMMENT ON TABLE ASSEMBLY_LINES_EMPLOYEES IS 'Intermédio entre as tabelas de linha de montagem e a tabela de funcionários por conta da relação n para n';

CREATE TABLE ASSEMBLY_LINES_PRODUCTS (
    id_PRODUCTS INTEGER,
    id_ASSEMBLY_LINES INTEGER,
    PRIMARY KEY (id_PRODUCTS, id_ASSEMBLY_LINES),
    CONSTRAINT fk_manuals_products FOREIGN KEY (id_PRODUCTS) REFERENCES PRODUCTS(id),
    CONSTRAINT fk_manuals_assembly_lines FOREIGN KEY (id_ASSEMBLY_LINES) REFERENCES ASSEMBLY_LINES(id)
);

COMMENT ON TABLE ASSEMBLY_LINES_PRODUCTS IS 'Intermédio entre as tabelas de linhas de montagem e a tabela de produtos por conta da relação n para n';

CREATE TABLE MANUALS_PRODUCTS (
    id_PRODUCTS INTEGER,
    id_MANUALS INTEGER,
    PRIMARY KEY (id_PRODUCTS, id_MANUALS),
    CONSTRAINT fk_assembly_lines_products FOREIGN KEY (id_PRODUCTS) REFERENCES PRODUCTS(id),
    CONSTRAINT fk_assembly_lines_manuals FOREIGN KEY (id_MANUALS) REFERENCES MANUALS(id)
);

COMMENT ON TABLE MANUALS_PRODUCTS IS 'Intermédio entre as tabelas de manuais e a tabela de produtos por conta da relação n para n';
```