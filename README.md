# Atualização Automática de Pedidos no Google Sheets para Dashboards no Looker Studio

Este projeto fornece um script em Google Apps Script para automatizar a atualização de dados de pedidos em uma planilha do Google Sheets a partir de consultas realizadas no Metabase. Os dados atualizados podem ser usados para criar dashboards no Looker Studio.

## Pré-requisitos

Antes de começar, certifique-se de que você tenha:

1. **Conta no Metabase**: Com as credenciais de acesso.
2. **Google Sheets**: Acesso a uma planilha onde os dados serão armazenados.
3. **Permissões de Google Apps Script**: Permissões necessárias para criar e executar scripts no Google Sheets.

## Configuração

### 1. Clonar o Repositório

Clone ou baixe este repositório para obter o código do script.

### 2. Configurar o Google Sheets

- Crie ou abra a planilha do Google Sheets onde deseja armazenar os dados.
- Anote o **ID da planilha** (`spreadsheetId`). Este é o código que aparece na URL da planilha, entre `/d/` e `/edit`.

### 3. Configurar o Metabase

- Acesse o Metabase e identifique a consulta que deseja automatizar.
- Anote o **ID da consulta** (`queryId`). Você pode encontrar esse ID na URL quando visualizar a consulta no Metabase.

### 4. Configurar o Google Apps Script

1. **Abra o Google Sheets**:
   - No menu superior, vá em `Extensões` > `Apps Script`.

2. **Criar um Novo Projeto**:
   - Cole o código abaixo no editor de Apps Script:

   ```javascript
   function AtualizarPedidos1() {
     var metabaseUrl = 'https://metabase.seudominio.com';
     var queryId = 'your-query-id';
     var metabaseUsername = 'your-username';
     var metabasePassword = 'your-password';

     var spreadsheetId = 'google-sheets-ID';
     var sheetName = 'PGN01';
     var metabaseLoginEndpoint = metabaseUrl + '/api/session';
     var payload = {
       username: metabaseUsername,
       password: metabasePassword
     };
     var headersLoginMetabase = {
       'Content-Type': 'application/json'
     };

     var responseLoginMetabase = UrlFetchApp.fetch(metabaseLoginEndpoint, {
       method: 'post',
       headers: headersLoginMetabase,
       payload: JSON.stringify(payload)
     });

     var sessionToken = JSON.parse(responseLoginMetabase.getContentText()).id;

     var metabaseQueryEndpoint = metabaseUrl + '/api/card/' + queryId + '/query/json';
     var headersMetabase = {
       'Content-Type':'application/json',
       'X-Metabase-Session': sessionToken
     };

     var responseMetabase = UrlFetchApp.fetch(metabaseQueryEndpoint, {
       method: 'post',
       headers: headersMetabase
     });

     var sheet = SpreadsheetApp.openById(spreadsheetId).getSheetByName(sheetName);

     sheet.clear();

     var responseBody = JSON.parse(responseMetabase.getContentText());

     if (responseBody.length > 0) {
       var headerRow = Object.keys(responseBody[0]);
       var values = [headerRow].concat(responseBody.map(obj => headerRow.map(key => obj[key])));
       
       sheet.getRange(1, 1, values.length, values[0].length).setValues(values);
       Logger.log('Resultados da query do Metabase adicionados à planilha.');
     } else {
       Logger.log('A query do Metabase não retornou resultados.');
     }
   }
   ```

3. **Modificar o Código**:
   - Substitua as seguintes variáveis pelos valores reais:
     - `metabaseUrl`: A URL do seu Metabase.
     - `queryId`: O ID da consulta no Metabase.
     - `metabaseUsername`: Seu nome de usuário no Metabase.
     - `metabasePassword`: Sua senha no Metabase.
     - `spreadsheetId`: O ID da planilha do Google Sheets.
     - `sheetName`: O nome da aba onde os dados serão armazenados (ex: `PGN01`).

4. **Autorizar o Script**:
   - Clique no ícone de "Salvar" e em seguida em "Executar" o script pela primeira vez. Será necessário autorizar o script para acessar seu Google Sheets e fazer requisições externas.

5. **Configurar o Trigger Automático**:
   - Para atualizar os dados automaticamente, configure um acionador para o script:
     - Vá para `Relógio (Triggers)` > `Adicionar Trigger`.
     - Selecione a função `AtualizarPedidos1` e configure para ser executada em intervalos de tempo desejados (por exemplo, diariamente).

## Criação do Dashboard no Looker Studio

1. **Importar os Dados**:
   - No Looker Studio, importe a planilha do Google Sheets com os dados atualizados.

2. **Criar Visualizações**:
   - Use as ferramentas de visualização do Looker Studio para criar gráficos, tabelas e outras visualizações com os dados.

3. **Personalizar e Compartilhar**:
   - Personalize o design do seu dashboard e compartilhe com sua equipe ou partes interessadas.

## Contribuições

Sinta-se à vontade para enviar pull requests ou abrir issues para melhorias e sugestões.
