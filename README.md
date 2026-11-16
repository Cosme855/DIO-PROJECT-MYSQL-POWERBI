# Integração MySQL e Transformação de Dados no Power BI
Projeto desenvolvido durante a **Formação Power BI Analyst** da DIO. 

O objetivo aqui foi sair da base relacional bruta no MySQL , conectar o banco ao Power BI Desktop e aplicar todo o tratamento de ETL no Power Query para preparar os dados para modelagem analítica.

---

## 🛠️ Ferramentas Utilizadas
* **MySQL Server 8.0 & MySQL Workbench:** Criação do schema, estruturação das tabelas e carga inicial dos dados.
* **Power BI Desktop (Power Query):** Extração, limpeza, auditoria de integridade e modelagem das consultas.

---

## 🗄️ Estrutura da Base de Dados
O banco `azure_company` conta com 6 tabelas relacionais que representam o dia a dia operacional da empresa:
* `employee` — Cadastro de colaboradores, salários, endereços e hierarquia de supervisão.
* `departament` — Departamentos da empresa, seus gerentes e datas de criação.
* `dept_locations` — Localizações físicas de cada departamento.
* `project` — Projetos ativos, seus códigos e departamentos responsáveis.
* `works_on` — Alocação de horas dos colaboradores em cada projeto.
* `dependent` — Dependentes vinculados aos colaboradores.

---

## 🔍 Auditoria e Decisões de Tratamento (ETL)

### 1. Limpeza Inicial e Tipagem
* **Remoção de colunas auxiliares:** Ao importar do MySQL, o Power Query traz colunas de navegação de chaves estrangeiras (`Value` / `Table`). Removi essas colunas nas 6 consultas para evitar ruído e erros de carregamento.
* **Ajuste monetário:** A coluna `Salary` (em `employee`) foi convertida para **Número Decimal Fixo (`Currency.Type`)** para garantir precisão nos cálculos financeiros.

### 2. Análise de Nulos e Integridade
* **Colaborador sem gerente (`Super_ssn` nulo):** Identifiquei apenas um registro nulo na coluna `Super_ssn` — o colaborador **James E. Borg** (`Ssn: 888665555`). **Decisão:** O registro foi mantido, pois ele atua como gerente-geral da empresa (responsável pelo departamento *Headquarters*), não possuindo um superior direto acima dele.
* **Gerentes por departamento:** Todos os departamentos (`Headquarters`, `Administration` e `Research`) possuem `Mgr_ssn` devidamente preenchido, sem lacunas para corrigir.
* **Horas de projeto (`works_on`):** Todos os registros de `Hours` estão consistentes (entre `0` e `40` horas), sem valores nulos ou inconsistentes.

### 3. Transformações Estruturais no Power Query
* **Decomposição de Endereço (`Address`):** A coluna trazia número, rua, cidade e estado unidos por hífen (ex: `731-Fondren-Houston-TX`). Como havia nomes de rua compostos com hífen (caso de `975-Fire-Oak-Humble-TX`), realizei a divisão usando o delimitador `-` primeiro **da direita para a esquerda** (extraindo `State` e `City`) e depois **da esquerda para a direita** (separando `Address_Number` e `Street`). Isso evitou que o nome da rua quebrasse em colunas erradas.
* **Departamento por Colaborador:** Mesclei `employee` com `departament` (`Externa Esquerda` via `Dno` = `Dnumber`) para trazer o nome do departamento diretamente para a tabela de funcionários, removendo em seguida o código numérico `Dno`.
* **Nome do Gerente (`Manager_Name`):** Realizei uma automescla na própria tabela `employee` ligando `Super_ssn` ao `Ssn`, expandindo nome e sobrenome do supervisor e concatenando ambos na coluna `Manager_Name`.
* **Nome Completo do Colaborador (`Employee_Name`):** Uni as colunas de nome e sobrenome em uma única coluna identificadora para facilitar a leitura nos relatórios.
* **Chave Departamento-Local (`Dept_Location`):** Para não duplicar registros na tabela principal de departamentos (o que quebraria a cardinalidade 1-para-muitos no modelo), criei uma nova consulta mesclada (`dept_locations_merged`) unindo `departament` e `dept_locations`, gerando combinações únicas no formato `Departamento - Local` (ex: `Research - Bellaire`, `Headquarters - Houston`).

---

## 📁 Arquivos do Repositório
* `PB_MYSQL.pbix` — Arquivo do Power BI Desktop com todas as etapas aplicadas no Power Query.
* `README.md` — Documentação técnica das decisões de tratamento e modelagem.

---

## 🚀 Como Reproduzir
1. Suba o banco `azure_company` em uma instância local ou na nuvem do **MySQL**.
2. Abra o arquivo `PB_MYSQL.pbix` no **Power BI Desktop**.
3. Vá em **Transformar Dados > Configurações da Fonte de Dados** e atualize as credenciais/servidor para apontar para o seu banco MySQL.
4. Clique em **Atualizar Visualização** e depois em **Fechar e Aplicar**.
