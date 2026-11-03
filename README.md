## Controladoria Pública — Análise de Contratações 2025

**Projeto Final | SENAI — Análise de Dados com Python e PostgreSQL**

Uma esteira analítica de ponta a ponta para transformar dados de contratações públicas do PNCP em informações confiáveis, indicadores de desempenho e análises para apoio à tomada de decisão da Controladoria Pública.

## 1. Sobre o projeto
O projeto foi desenvolvido como atividade avaliativa do curso de **Análise de Dados com Python — SENAI**, com o objetivo de investigar contratações públicas federais realizadas em 2025, identificando oportunidades de economia, concentração de fornecedores, eficiência dos processos de compras e distribuição dos gastos públicos.

O desafio consiste em transformar dados brutos, com inconsistências de formatos, identificadores, valores monetários e datas, em uma base analítica estruturada, confiável e adequada para consultas e visualização.

A solução utiliza Python para auditoria, limpeza e transformação dos dados, PostgreSQL para armazenamento e modelagem dimensional, SQL para responder às perguntas de negócio e um dashboard executivo para apresentar os resultados.

### Objetivos
- Auditar e tratar inconsistências nos dados originais.
- Integrar informações de compras, itens e resultados.
- Construir um modelo dimensional (Star Schema).
- Implementar integridade referencial no PostgreSQL.
- Desenvolver consultas SQL para responder às perguntas da Controladoria.
- Apresentar indicadores e resultados em um dashboard.
- Documentar e versionar todo o desenvolvimento utilizando Git e GitHub.

## 2. Fonte e escopo dos dados
**Fonte oficial:** Portal Nacional de Contratações Públicas (PNCP).
**Conjunto de dados:** Contratações Públicas / Licitações e Itens Contratados.
**Recorte temporal:** 02/01/2025 a 31/12/2025.

Foi utilizada a amostra de contratações federais disponibilizada pelo SENAI, abrangendo os 12 meses de 2025. Portanto, os resultados representam o conjunto analisado e não a totalidade das contratações públicas brasileiras.

### Arquivos originais

| Arquivo | Conteúdo |
|---|---|
| `compras.csv` | Cabeçalho das contratações, órgãos, datas, modalidades e localidades. |
| `itens.csv` | Itens licitados, descrições, quantidades e valores estimados. |
| `resultados.csv` | Fornecedores classificados e valores homologados por item. |

O relacionamento entre os arquivos utiliza os identificadores `numero_controle_pncp` e `numero_item`, respeitando o grão de análise definido para a tabela fato: um registro por item contratado.

## 3. Tecnologias e ferramentas

| Tecnologia | Aplicação |
|---|---|
| Python 3.12 | Processamento e transformação dos dados. |
| Pandas | Manipulação, limpeza e validação. |
| PostgreSQL 16 | Banco de dados relacional. |
| SQL | Modelagem, consultas e análises. |
| psycopg2 | Conexão entre Python e PostgreSQL. |
| Pytest | Testes automatizados. |
| Git e GitHub | Versionamento e documentação. |
| Looker Studio | Dashboard e visualização dos indicadores. |
| Trello | Planejamento e acompanhamento das Sprints. |

## 4. Arquitetura e pipeline de dados

A solução foi organizada em camadas, permitindo separar o tratamento, a modelagem, o armazenamento e a análise.

```text
PNCP — Dados brutos
        |
        v
Python + Pandas
Auditoria, limpeza e transformação
        |
        v
Modelagem dimensional
1 tabela fato + 4 dimensões
        |
        v
PostgreSQL
Armazenamento e integridade referencial
        |
        v
SQL
6 perguntas de negócio
        |
        v
Dashboard executivo
Indicadores e visualizações
```

### Etapas do processamento
1. **Extração e leitura:** importação dos três arquivos CSV disponibilizados pelo SENAI.
2. **Auditoria e limpeza:** identificação e tratamento das inconsistências com Python e Pandas.
3. **Transformação:** integração das informações e aplicação das regras de negócio.
4. **Modelagem:** construção das dimensões e da tabela fato.
5. **Carga:** armazenamento dos dados tratados no PostgreSQL.
6. **Análise:** execução das consultas SQL para responder às perguntas da Controladoria.
7. **Visualização:** organização dos indicadores em um dashboard executivo.

## 5. Tratamento e qualidade dos dados
Durante a auditoria e preparação da base, foram consideradas as seguintes categorias de inconsistências:

| Categoria | Tratamento realizado |
|---|---|
| Identificadores e documentos | Padronização de CNPJ/CPF, máscaras e zeros à esquerda. |
| Valores monetários | Normalização dos separadores decimais, milhares e símbolos monetários. |
| Datas | Padronização dos formatos e validação da coerência cronológica. |
| Valores nulos | Identificação de representações textuais de nulos e tratamento conforme o tipo de campo. |
| Textos e codificação | Sanitização de caracteres e padronização de nomenclaturas. |
| Relacionamentos | Validação dos identificadores e integridade entre as tabelas. |

### Decisões de negócio
- A análise financeira considera os valores dos itens, respeitando o grão da tabela fato.
- A economia é calculada somente para itens classificados como comparáveis (`comparavel = 1`).
- O percentual de economia é calculado pela razão entre a soma da economia e a soma dos valores estimados comparáveis, evitando médias simples de percentuais.
- O ciclo de compras considera a diferença entre a publicação e a homologação.
- A região geográfica é associada a partir da UF, pois o arquivo original não fornece diretamente essa informação.
- O perfil dos gastos é classificado entre bens, serviços e obras e engenharia.

## 6. Modelagem dimensional — Star Schema

O banco foi estruturado com uma tabela fato central e quatro dimensões, permitindo análises integradas por órgão, fornecedor, modalidade e localidade.

### Tabela fato
**`fato_item_contratado`**
Armazena os registros dos itens contratados, incluindo:

- Identificação da contratação e do item.
- Chaves estrangeiras das dimensões.
- Datas de publicação e homologação.
- Descrição e perfil do gasto.
- Quantidades e valores estimados e homologados.
- Indicador de comparabilidade.
- Economia calculada.
- Dias do ciclo de contratação.

A chave primária é composta por `numero_controle_pncp` e `numero_item`.

### Tabelas dimensionais

| Dimensão | Descrição |
|---|---|
| `dim_orgao` | Identificação, CNPJ, nome e poder do órgão público. |
| `dim_fornecedor` | Identificação, nome, tipo de pessoa e porte do fornecedor. |
| `dim_modalidade` | Identificação e descrição da modalidade de contratação. |
| `dim_localidade` | UF, nome do estado e região brasileira. |

Os relacionamentos são estabelecidos por chaves primárias e estrangeiras, com cardinalidade 1:N entre cada dimensão e a tabela fato.

### Diagrama do modelo dimensional
>Esquema Estrela — Projeto SENAI (docs/imagens/esquema_estrela.png)

## 7. Perguntas de negócio da Controladoria
Foram desenvolvidas seis análises principais, implementadas em `sql/02_perguntas.sql`.

**1. Economia global do poder público**
Qual foi a economia obtida entre os valores estimados e homologados, considerando apenas itens comparáveis?

**2. Concentração e risco de fornecedores**
Quais são os dez fornecedores com maior volume financeiro homologado e qual é a participação deles no total das contratações?

**3. Eficácia por modalidade**
Quais modalidades apresentam maior economia e maior concentração de recursos financeiros?

**4. Tempo do ciclo de compras por órgão**
Qual é o tempo médio, em dias, entre a publicação do edital e sua homologação, considerando os órgãos com pelo menos cinco itens com ciclo calculado?

**5. Distribuição regional**
Quais estados e regiões concentram os maiores volumes homologados e quais apresentam os melhores indicadores de economia?

**6. Perfil dos gastos**
Como os recursos públicos estão distribuídos entre bens, serviços e obras e engenharia?

## 8. Resultados da análise e validação
Após o processamento e a validação dos dados, foram obtidos os seguintes indicadores:

| Indicador | Resultado |
|---|---:|
| Itens na tabela fato | 6.126 |
| Contratações distintas | 1.962 |
| Fornecedores | 2.652 |
| Órgãos públicos | 363 |
| Itens comparáveis | 5.457 |
| Economia percentual agregada | 14,46% |
| Ciclo médio de contratação | 48,1 dias |

A economia percentual foi calculada utilizando a soma das diferenças entre os valores estimados e homologados dos itens comparáveis, dividida pela soma dos valores estimados desses mesmos itens.

Os indicadores foram utilizados como referência para as consultas SQL e a apresentação executiva.

## 9. Dashboard executivo
O dashboard apresenta os principais indicadores e resultados da análise das contratações públicas de 2025, organizados para facilitar a interpretação dos dados e apoiar a tomada de decisão.

Entre as visualizações estão:
- Economia e valores homologados.
- Concentração financeira por fornecedor.
- Desempenho por modalidade de contratação.
- Tempo médio do ciclo de compras.
- Distribuição dos gastos por estado e região.
- Perfil dos gastos por categoria.

**Relatório completo:**
Visualizar Dashboard — Controladoria Pública: Análise de Contratações 2025
(docs/Controladoria_Pública_—_Análise_de_Contratações_2025.pdf)
> Link Looker Studio: https://datastudio.google.com/reporting/5db5f971-6072-44e0-8bfd-e475e9c9328d

## 10. Planejamento e desenvolvimento — Trello

O desenvolvimento foi organizado em Sprints, utilizando o Trello para acompanhar as atividades, etapas técnicas e evolução do projeto.

### Registros do planejamento
Trello — Sprint 1 (docs/imagens/Trello_1.png)
Trello — Sprint 2 (docs/imagens/Trello_2.png)
Trello — Sprint 3 (docs/imagens/Trello_3.png)
Trello — Sprint 4 (docs/imagens/Trello_4.png)

## 11. Estrutura do repositório

```
Projeto 4/
│
├── data/
│   ├── raw/
│   │   ├── compras.csv
│   │   ├── itens.csv
│   │   └── resultados.csv
│   │
│   └── processed/
│
├── docs/
│   ├── imagens/
│   │   ├── Trello_1.png
│   │   ├── Trello_2.png
│   │   ├── Trello_3.png
│   │   ├── Trello_4.png
│   │   └── esquema_estrela.png
│   │
│   └── Controladoria_Pública_—_Análise_de_Contratações_2025.pdf
│
├── sql/
│   ├── 01_criar_banco.sql
│   └── 02_perguntas.sql
│
├── src/
│   ├── limpeza.py
│   ├── montar_tabelas.py
│   ├── banco.py
│   ├── carregar.py
│   └── config.py
│
├── requirements.txt
├── .env
├── .gitignore
└── README.md
```

## 12. Como executar o projeto

### Pré-requisitos
- Python 3.10 ou superior.
- PostgreSQL.
- Git.
- Arquivos CSV originais na pasta `data/raw/`.

### 1. Clonar o repositório
git clone URL_DO_REPOSITORIO
cd "Projeto 4"


### 2. Criar e ativar o ambiente virtual
python3 -m venv .venv
source .venv/bin/activate

### 3. Instalar as dependências
pip install -r requirements.txt

### 4. Configurar as variáveis de ambiente
Criar o arquivo `.env` conforme as variáveis utilizadas em `src/config.py`.
Não compartilhar senhas ou credenciais no repositório.

### 5. Executar a limpeza
python -m src.limpeza

### 6. Construir as tabelas dimensionais e fato
python -m src.montar_tabelas

### 7. Criar o banco e as tabelas
Criar o banco de dados no PostgreSQL e executar o script:
psql -U postgres -d NOME_DO_BANCO -f sql/01_criar_banco.sql


### 8. Carregar os dados
python -m src.carregar

### 9. Executar as consultas SQL
Executar as consultas de:
sql/02_perguntas.sql

As consultas produzem as respostas das seis perguntas de negócio da Controladoria.

## 13. Melhorias futuras
- Ampliar a documentação das regras de tratamento e validação.
- Desenvolver testes automatizados adicionais para as etapas do pipeline.
- Otimizar consultas e operações de carga.
- Explorar análises temporais e comparações entre períodos.
- Evoluir a visualização executiva e ampliar a interatividade dos indicadores.

## 14. Apresentação do projeto
**Vídeo — Review das Sprints e defesa técnica:**

> LINK DO VÍDEO NO GOOGLE DRIVE:" https://drive.google.com/file/d/1-1WhxsCq5RnWkunTO4I_ayCx0p5WDeNS/view?usp=sharing "

## 15. Autoria
**Camyla K. Fernandes**

Projeto desenvolvido como atividade avaliativa do curso **Análise de Dados com Python — SENAI SCTEC**.

**Área de interesse:** Análise de Dados, tecnologia e inteligência aplicada à gestão pública.

*Projeto acadêmico desenvolvido com dados do Portal Nacional de Contratações Públicas (PNCP), disponibilizados para fins de estudo e análise.*
