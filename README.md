<h1 align="center">Modelagem do banco de dados do Edellcation</h1>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O Edellcation é uma Aplicação WEB criada para proporcionar aos funcionários das linhas de montagem um acesso fácil e eficiente a materiais técnicos e manuais de montagem de produtos da empresa, como computadores, servidores e notebooks. A plataforma permite que os funcionários estudem, revisem e acompanhem os processos de montagem de forma individualizada, ao mesmo tempo em que os mantêm atualizados sobre quaisquer alterações nos procedimentos ou inclusão de novos manuais.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Seu banco de dados foi desenvolvido de acordo com a estrutura do SGBD (Sistema de Gerenciamento de Banco de Dados) PostgreSQL no software SQLDesigner

<img src="assets/databaseDiagram.png" style="max-width:100%; height:auto;" alt="Diagrama da arquitetura MVC do Edellcation">

[Diagrama em PNG](assets/databaseDiagram.png)</br>
[Arquivo da Modelagem (XML)](assets/database.xml) 

Por que precisa dessas tabelas e bla bla bla

Código SQL:
```sql
CREATE TABLE MANUALS (
 id BIGSERIAL,
 id_MATERIALS INTEGER,
 PublicationDates DATE NOT NULL DEFAULT 'CURRENT_DATE',
 Descriptions VARCHAR(350),
 Titles VARCHAR(80) NOT NULL DEFAULT 'Sem Título',
 Versions DECIMAL DEFAULT 0.1
);


ALTER TABLE MANUALS ADD CONSTRAINT MANUALS_pkey PRIMARY KEY (id);
COMMENT ON TABLE "MANUALS" IS 'Manuais para o aprendizado da montagem de produtos';

CREATE TABLE EMPLOYEES (
 id BIGSERIAL,
 Names VARCHAR(200) NOT NULL DEFAULT 'NULL',
 IsEngineer BINARY(1) NOT NULL DEFAULT 'NULL'/* Campo binário para definição se é engenheiro ou não */,
 Emails VARCHAR(120) NOT NULL DEFAULT 'NULL',
 Passwords VARCHAR(50) NOT NULL DEFAULT 'NULL'
);


ALTER TABLE EMPLOYEES ADD CONSTRAINT EMPLOYEES_pkey PRIMARY KEY (id);
COMMENT ON TABLE "EMPLOYEES" IS 'Tabela que armazenará trodos os funcionários da empresa, seja engenheiro ou operador de linha de montagem.';
COMMENT ON COLUMN "EMPLOYEES"."IsEngineer" IS 'Campo binário para definição se é engenheiro ou não';

CREATE TABLE ASSEMBLY_LINES (
 id BIGSERIAL,
 Names VARCHAR(50) NOT NULL DEFAULT 'NULL',
 Descriptions VARCHAR(200) DEFAULT 'Sem descrição'
);


ALTER TABLE ASSEMBLY_LINES ADD CONSTRAINT ASSEMBLY_LINES_pkey PRIMARY KEY (id);
COMMENT ON TABLE "ASSEMBLY_LINES" IS 'Armazenará as linhas de montagens das quais os funcionários são pertencentes.';

CREATE TABLE TO-DOS (
 id BIGSERIAL,
 id_EMPLOYEES INTEGER,
 id_MANUALS INTEGER,
 Status BINARY NOT NULL DEFAULT 'NULL'/* Campo binário para definir se a tarefa foi concluída ou não */,
 AssignmentDates DATE NOT NULL DEFAULT 'NULL'
);


ALTER TABLE TO-DOS ADD CONSTRAINT TO-DOS_pkey PRIMARY KEY (id);
COMMENT ON TABLE "TO-DOS" IS 'Lista de manuais a serem lidos pelo funcionário';
COMMENT ON COLUMN "TO-DOS"."Status" IS 'Campo binário para definir se a tarefa foi concluída ou não';

CREATE TABLE PRODUCTS (
 id BIGSERIAL/* Número de série */,
 Names VARCHAR(200) NOT NULL DEFAULT 'NULL',
 Categorys VARCHAR(50) NOT NULL DEFAULT 'NULL'/* Exemplo: monitor, notebook, servidor, etc. */,
 Descriptions VARCHAR(300) DEFAULT 'Sem descrição'
);


ALTER TABLE PRODUCTS ADD CONSTRAINT PRODUCTS_pkey PRIMARY KEY (id);
COMMENT ON TABLE "PRODUCTS" IS 'Equipamentos montados pela Dell';
COMMENT ON COLUMN "PRODUCTS"."id" IS 'Número de série';
COMMENT ON COLUMN "PRODUCTS"."Categorys" IS 'Exemplo: monitor, notebook, servidor, etc.';

CREATE TABLE ASSEMBLY_LINES__EMPLOYEES (
 id_EMPLOYEES INTEGER,
 id_ASSEMBLY_LINES INTEGER
);


ALTER TABLE ASSEMBLY_LINES__EMPLOYEES ADD CONSTRAINT ASSEMBLY_LINES__EMPLOYEES_pkey PRIMARY KEY ();
COMMENT ON TABLE "ASSEMBLY_LINES__EMPLOYEES" IS 'Intermédio entre as tabelas de linha de montagem e a tabela de funcionários por conta da relação n pra n';

CREATE TABLE MATERIALS (
 id BIGSERIAL,
 Archives VARCHAR(150) NOT NULL DEFAULT 'NULL'/* Caminho para o material */,
 Types CHAR(3) NOT NULL DEFAULT 'NULL'/* Campo para informar a extensão do arquivo do manual (PDF, vídeo, modelo 3D, etc) */,
 Titles VARCHAR(80) NOT NULL DEFAULT 'Sem Título',
 Descriptions VARCHAR(350) DEFAULT 'Sem descrição'
);


ALTER TABLE MATERIALS ADD CONSTRAINT MATERIALS_pkey PRIMARY KEY (id);
COMMENT ON TABLE "MATERIALS" IS 'Arquivos que serão utilizados nos manuais';
COMMENT ON COLUMN "MATERIALS"."Archives" IS 'Caminho para o material';
COMMENT ON COLUMN "MATERIALS"."Types" IS 'Campo para informar a extensão do arquivo do manual (PDF, vídeo, modelo 3D, etc)';

CREATE TABLE MANUALS_PRODUCTS (
 id_PRODUCTS INTEGER/* Número de série */,
 id_ASSEMBLY_LINES INTEGER
);


ALTER TABLE MANUALS_PRODUCTS ADD CONSTRAINT MANUALS_PRODUCTS_pkey PRIMARY KEY ();
COMMENT ON TABLE "MANUALS_PRODUCTS" IS 'Intermédio entre as tabelas de manuais e a tabela de produtos por conta da relação n pra n';
COMMENT ON COLUMN "MANUALS_PRODUCTS"."id_PRODUCTS" IS 'Número de série';

CREATE TABLE ASSEMBLY_LINES__PRODUCTS (
 id_PRODUCTS INTEGER/* Número de série */,
 id_MANUALS INTEGER
);


ALTER TABLE ASSEMBLY_LINES__PRODUCTS ADD CONSTRAINT ASSEMBLY_LINES__PRODUCTS_pkey PRIMARY KEY ();
COMMENT ON TABLE "ASSEMBLY_LINES__PRODUCTS" IS 'Intermédio entre as tabelas de manuais e a tabela de produtos por conta da relação n pra n';
COMMENT ON COLUMN "ASSEMBLY_LINES__PRODUCTS"."id_PRODUCTS" IS 'Número de série';

ALTER TABLE MANUALS ADD CONSTRAINT MANUALS_id_MATERIALS_fkey FOREIGN KEY (id_MATERIALS) REFERENCES MATERIALS(id);
ALTER TABLE TO-DOS ADD CONSTRAINT TO-DOS_id_EMPLOYEES_fkey FOREIGN KEY (id_EMPLOYEES) REFERENCES EMPLOYEES(id);
ALTER TABLE TO-DOS ADD CONSTRAINT TO-DOS_id_MANUALS_fkey FOREIGN KEY (id_MANUALS) REFERENCES MANUALS(id);
ALTER TABLE ASSEMBLY_LINES__EMPLOYEES ADD CONSTRAINT ASSEMBLY_LINES__EMPLOYEES_id_EMPLOYEES_fkey FOREIGN KEY (id_EMPLOYEES) REFERENCES EMPLOYEES(id);
ALTER TABLE ASSEMBLY_LINES__EMPLOYEES ADD CONSTRAINT ASSEMBLY_LINES__EMPLOYEES_id_ASSEMBLY_LINES_fkey FOREIGN KEY (id_ASSEMBLY_LINES) REFERENCES ASSEMBLY_LINES(id);
ALTER TABLE MANUALS_PRODUCTS ADD CONSTRAINT MANUALS_PRODUCTS_id_PRODUCTS_fkey FOREIGN KEY (id_PRODUCTS) REFERENCES PRODUCTS(id);
ALTER TABLE MANUALS_PRODUCTS ADD CONSTRAINT MANUALS_PRODUCTS_id_ASSEMBLY_LINES_fkey FOREIGN KEY (id_ASSEMBLY_LINES) REFERENCES ASSEMBLY_LINES(id);
ALTER TABLE ASSEMBLY_LINES__PRODUCTS ADD CONSTRAINT ASSEMBLY_LINES__PRODUCTS_id_PRODUCTS_fkey FOREIGN KEY (id_PRODUCTS) REFERENCES PRODUCTS(id);
ALTER TABLE ASSEMBLY_LINES__PRODUCTS ADD CONSTRAINT ASSEMBLY_LINES__PRODUCTS_id_MANUALS_fkey FOREIGN KEY (id_MANUALS) REFERENCES MANUALS(id);
```