# Company Data Transformation | MySQL + Power BI

Projeto do desafio **Integrando Dados com MySQL e Transformando com Power BI**, da formação Power BI Analyst da DIO.

O objetivo foi conectar o Power BI a um banco MySQL, identificar problemas na base **Company** e tratar os dados no Power Query.

## Tecnologias

- MySQL Server 8.0 e MySQL Workbench
- Power BI Desktop (Power Query)
- Connector/NET

## Ambiente

O desafio sugere criar o MySQL na Azure. Neste projeto o banco rodou **localmente** (`localhost:3306`), pois não tive acesso a uma assinatura Azure. A integração com o Power BI é a mesma nos dois casos.

1. Instalei o MySQL Server e o Workbench.
2. Rodei `script_bd_company.sql` (estrutura) e `insercao_de_dados_e_queries_sql.sql` (dados) no Workbench, criando o banco `azure_company` com 6 tabelas: `departament`, `dependent`, `dept_locations`, `employee`, `project` e `works_on`.
3. No Power BI Desktop: **Obter dados > Banco de dados MySQL**, servidor `localhost`, banco `azure_company`, autenticação **Básico** (usuário e senha do MySQL).

### Ajustes necessários nos scripts

- O Workbench interrompe o script no primeiro erro. Comandos `alter table ... drop` de exemplo falhavam por tentarem apagar restrições inexistentes. Corrigi o `drop foreign key` em `dept_locations` e criei a tabela `dependent` separadamente.
- Na inserção dos dados, usei `SET FOREIGN_KEY_CHECKS = 0;` antes e `= 1;` depois, porque `employee` referencia `departament` e ela mesma.
- Troquei `use company_constraints` por `use azure_company`.
- O nome `departament` (com essa grafia) vem do script original e foi mantido.

## Transformações realizadas (Power Query)

| # | Diretriz | O que foi feito |
|---|----------|-----------------|
| 1 | Cabeçalhos e tipos | Conferidos em todas as tabelas. Removi as colunas de navegação (tipo *Table*) criadas pelo conector |
| 2 | Valores monetários | `Salary` convertida para **Número Decimal Fixo** |
| 3 | Nulos | Apenas `Super_ssn` em `employee` tem 1 nulo (12%). Demais colunas sem nulos |
| 4 | Colaboradores sem gerente | Apenas **James Borg**, gerente de topo. Mantido, pois é um dado correto e não um erro |
| 5 e 6 | Departamentos sem gerente | Nenhum: `Mgr_ssn` tem 100% de válidos em `departament`. Não foi preciso preencher lacunas |
| 7 | Horas dos projetos | `Hours` em `works_on` tem 100% de válidos (16 registros), valores entre 0 e 40. Um registro tem **0 horas** (James Borg no projeto 20). Mantido, pois é um dado possível (gerente associado ao projeto sem horas lançadas) |
| 8 | Colunas complexas | `Address` (ex.: `975-Fire-Oak-Humble-TX`) dividida em `Numero`, `Rua`, `Cidade` e `Estado`. Como há endereços com hífen na rua, dividi em 3 passos: primeiro hífen da esquerda (número), depois último hífen da direita duas vezes (estado e cidade) |
| 9 e 10 | Mescla employee + departament | Mesclar Consultas com `Dno` = `Dnumber`, junção **Externa Esquerda**, expandindo só `Dname` |
| 11 | Nome do gerente | Mescla de `employee` com ela mesma (`Super_ssn` = `Ssn`), junção Externa Esquerda, coluna `Gerente` |
| 12 | Nome + Sobrenome | `Fname` e `Lname` mescladas em `Nome_Completo` |
| 13 | Departamento + localização | Em `dept_locations`, mesclei com `departament` (`Dnumber`) para trazer o `Dname` e criei `Departamento_Local` (ex.: `Research - Houston`), com 100% de válidos |
| 15 | Colaboradores por gerente | Consulta de referência `colaboradores_por_gerente`, com Agrupar Por `Gerente` e contagem de linhas: Franklin Wong 3, James Borg 2, Jennifer Wallace 2 e sem gerente (James Borg) 1 |
| 16 | Colunas desnecessárias | _preencher: quais colunas foram removidas de cada tabela_ |

### Por que a junção Externa Esquerda?

A mescla tem como base `employee`. Com a junção externa esquerda, todos os colaboradores permanecem na tabela, mesmo os que não têm departamento ou gerente associado. É o caso do James Borg, que não tem gerente. Uma junção interna eliminaria essas linhas e causaria perda de dados.

### Por que usar "mesclar" e não "atribuir" (departamento + localização)?

Mesclar colunas junta os valores de duas colunas em um único texto, e os dois valores já estão na **mesma linha**. Não há relação entre tabelas a resolver, apenas a criação de uma chave única (departamento-local), que servirá de apoio para o modelo estrela em um módulo futuro.

## Estrutura do repositório

```
├── README.md
├── script_bd_company_corrigido.sql
├── company.pbix
└── prints/
```

## Autor

**Gustavo Lima de Sena**
