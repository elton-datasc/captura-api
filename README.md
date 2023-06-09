# Projeto Az 1.0 - Captura de dados de API
### Consumo de dados de API utilizando tecnologias Azure - API disponível em https://economia.awesomeapi.com.br/last
### Parte de infra pré-configurada para o projeto:
- Azure Data Factory
- SQL Server e Azure SQL Database
- Linked Services
- Integration Runtime

## Consumindo API de cotação de criptomoedas e inserindo dados em tabela SQL Server
### Objetivo
A pipeline tem como objetivo obter dados de uma API externa que fornece cotações de criptomoedas em tempo real e armazená-los em uma tabela SQL Server para posterior análise.

<div align="center">
  
![Arquitetura](img/arq.JPG)
  
 </div>


### Atividades da pipeline - Definições
O pipeline é composta por três atividades :

Atividade "Web"
A atividade "Web" consome a API externa que fornece dados de cotação de criptomoedas.
Ela recebe como entrada uma URL que representa a chamada à API e retorna os dados da cotação em um formato JSON.

![Atividade Web](img/atvweb.JPG)

Na pipeline em questão, a URL usada é https://economia.awesomeapi.com.br/last/BTC-BRL, que retorna os dados de cotação da criptomoeda Bitcoin (BTC) em relação ao real brasileiro (BRL).

Exemplo de resultado da API:

~~~JSON
{"BTCBRL":{"code":"BTC","codein":"BRL","name":"Bitcoin/Real Brasileiro","high":"145901","low":"143500","varBid":"-296","pctChange":"-0.2","bid":"144461","ask":"144508","timestamp":"1680460621","create_date":"2023-04-02 15:37:01"}}
~~~

Atividade "Set Variable"

A atividade "Set Variable" extrai os valores relevantes dos dados de cotação obtidos pela atividade "Web" e os armazena em variáveis para uso posterior na pipeline.

Nesta pipeline, a atividade "Set Variable" armazena os seguintes valores em variáveis:

ativo: a sigla da criptomoeda (BTC).

![Atividade set var ativo](img/atvsetvaratv.JPG)

cotacao: o valor da cotação da criptomoeda em relação ao real brasileiro.

![Atividade set var cotacao](img/atvsetvarcot.JPG)

data: a data e hora da cotação.

![Atividade set var data](img/atvsetvardta.JPG)


Atividade "Script"

A atividade "Script" insere os dados de cotação da criptomoeda em uma tabela SQL Server por meio de uma consulta SQL personalizada.

![Atividade Script](img/atvscript.JPG)

## Pipeline de consumo de API de cotação de criptomoedas

### Atividade Web

A atividade Web utiliza o método HTTP GET para acessar a API de cotação de criptomoedas no seguinte endereço: https://economia.awesomeapi.com.br/last/BTC-BRL. Ela obtém os dados de cotação atualizados da criptomoeda Bitcoin em relação ao Real brasileiro.

### Atividade Set Variable

Na atividade Set Variable foram definidas três variáveis: "ativo", "cotacao" e "data", com base nos dados obtidos pela atividade Web. Ela utiliza o seguinte código no campo "value" do setting:

`@{activity('get_data_api').output.BTCBRL.code}`
`@{activity('get_data_api').output.BTCBRL.ask}`
`@{formatDateTime(activity('get_data_api').output.BTCBRL.create_date, 'yyyy-MM-dd HH:mm:ss')}`

Colunas esperadas:

Id   | ativo | cotacao | data 
---- | ----- |---------|-----

### Atividade Script

A atividade Script executa uma consulta SQL para inserir os dados de cotação obtidos pela atividade Set Variable em uma tabela do banco de dados. Ela utiliza o seguinte código no campo "query" do setting:

~~~SQL
INSERT INTO dbo.cotacoes (ativo, cotacao, data) VALUES ('@{variables('ativo')}', '@{variables('cotacao')}', '@{variables('data')}')
~~~

Essa consulta insere os valores das variáveis "ativo", "cotacao" e "data" na tabela "cotacoes" do banco de dados.

Ela insere um novo registro na tabela cotacoes do banco de dados, com os valores armazenados nas variáveis ativo, cotacao e data. A sintaxe @{variables('...')} é usada para acessar os valores das variáveis armazenadas pela atividade "Set Variable" e usá-los como parâmetros na consulta SQL.

Exemplo de banco de dados populado:

![Exemplo Banco de dados](img/bdetl.JPG)


### Fluxo de Execução
A atividade "Web" é executada e consome a API externa que fornece dados de cotação de criptomoedas.
Os valores relevantes dos dados de cotação são extraídos pela atividade "Set Variable" e armazenados nas variáveis ativo, cotacao e data.
A atividade "Script" é executada e insere os valores armazenados nas variáveis na tabela cotacoes do banco de dados.
Conclusão
A pipeline consome uma API externa que fornece dados de cotação de criptomoedas, extrai os valores relevantes dos dados e os armazena em variáveis, e insere esses valores em uma tabela SQL Server por meio de uma consulta SQL personalizada. 

A imagem abaixo ilustra o fluxo do pipeline do datafactory:

![Fluxo do Pipeline](img/adfpipelineetlbtc.JPG)

Exemplo dos outoputs esperados:

![Atividades Output](img/adfoutetlbtc.JPG)

Este projeto é inicial.Posteriormente a ideia é enriquecer o projeto, trabalhando outros ativos, com tratamento, armazenamento e com tecnologias diferentes (outras clouds e ferramentas open source) incluindo também ferramentas de vizualização e análise.
