\# Smart Excel Stream \& Validator (ODC)



\*\*Validates, cleans, and parses massive Excel files at runtime using rule-driven JSON configuration. Designed for high performance via C# streaming to eliminate timeouts and payload errors in OutSystems Developer Cloud.\*\*



\---



\## 🎯 Porquê escolher este componente?



Processar ficheiros Excel massivos (com mais de 50.000 ou 100.000 linhas) diretamente nos fluxos visuais do OutSystems gera tradicionalmente três grandes problemas: \*\*Timeouts do servidor\*\*, \*\*alto consumo de memória RAM\*\* e \*\*erros de tamanho de payload (`OS-FLRT-60009` ao passar do limite de 5.5MB)\*\*.



O \*\*Smart Excel Stream \& Validator\*\* resolve isto de forma nativa:

\* \*\*Performance Extrema:\*\* Processa 100.000 linhas em menos de 4 segundos usando \*C# Streaming\* (.NET Core).

\* \*\*Consumo de Memória Inteligente:\*\* Lê o ficheiro linha a linha, sem carregar o binário inteiro na memória RAM do container.

\* \*\*Arquitetura à Prova de Falhas (Chunking):\*\* Implementa paginação nativa na própria lógica C# para fatiar o retorno do JSON, contornando o limite de 5.5MB da plataforma ODC.

\* \*\*Validação Dinâmica (Rule-Driven):\*\* Não precisa de desenhar dezenas de `Ifs` visuais. Defina as regras de validação num esquema JSON simples e o motor trata do resto.



\---



\## 🏗️ Como Funciona a Assinatura (API Reference)



O componente expõe uma única Server Action centralizada: \*\*`ValidateExcelFile`\*\*.



\### Parâmetros de Entrada (Inputs)

\* \*\*`FileBinary`\*\* (\*Binary Data\*, Obrigatório): O conteúdo binário do ficheiro Excel (`.xlsx`) carregado pelo utilizador.

\* \*\*`ValidationRulesJSON`\*\* (\*Text\*, Obrigatório): Array JSON que define as regras e restrições de cada coluna.

\* \*\*`PageNumber`\*\* (\*Integer\*, Obrigatório): O número da página/lote que pretende ler (Baseado em 1, ex: `1` para o primeiro lote).

\* \*\*`PageSize`\*\* (\*Integer\*, Obrigatório): O tamanho do lote de dados que deseja receber (Recomendado: `5000` linhas por lote para manter o payload leve).



\### Parâmetros de Saída (Outputs)

\* \*\*`CleanedDataJSON`\*\* (\*Text\*): Array JSON contendo os dados das linhas limpas e purificadas pertencentes apenas à página solicitada. Pronto para ser desserializado no OutSystems.

\* \*\*`ErrorListJSON`\*\* (\*Text\*): Lista estruturada em JSON contendo todos os erros de validação encontrados nas linhas processadas (ex: linha, coluna, valor e mensagem de erro).

\* \*\*`IsValid`\*\* (\*Boolean\*): Retorna `True` se o lote processado cumprir todas as regras estipulated; retorna `False` se existirem erros ou avisos estruturais.



\---



\## ⚙️ Configuração do JSON de Regras (Schema)



O motor aceita um array direto de objetos JSON. Cada objeto representa uma coluna que deseja validar no Excel.



\### Formato Suportado

```json

\[

&#x20; {

&#x20;   "column": "Name",

&#x20;   "required": true

&#x20; },

&#x20; {

&#x20;   "column": "Age",

&#x20;   "min": 18

&#x20; }

]

```



\### Propriedades Disponíveis:

\* \*\*`column`\*\*: Nome exato do cabeçalho da coluna na primeira linha do Excel (\*Case-Sensitive\*).

\* \*\*`required`\*\*: Se definido como `true`, rejeita a linha caso a célula esteja vazia.

\* \*\*`min`\*\*: Define o valor mínimo aceite para dados numéricos.



\---



\## 🚀 Guia de Implementação (Passo a Passo)



Para implementar este componente no seu ecrã reativo, siga estes 4 passos simples:



\### Passo 1: Captura do Ficheiro

1\. Arraste o widget \*\*Upload\*\* para o seu ecrã.

2\. Associe as propriedades \*File Content\* e \*File Name\* a duas variáveis locais (`UploadedContent` e `UploadedName`).

3\. Mude a propriedade \*Accept\* do Upload para `.xlsx`.



\### Passo 2: Criar as Estruturas para Desserializar

Crie as seguintes \*\*Structures\*\* no seu módulo OutSystems para receber os dados limpos do JSON:

\* \*\*`ExcelRowResult`\*\*: Crie os atributos que coincidem com as colunas do seu Excel (ex: `Name` do tipo \*Text\* e `Age` do tipo \*Integer\*).

\* \*\*`ExcelValidationError`\*\*: Atributos para capturar erros: `Row` (Integer), `Column` (Text), `Value` (Text), `Error` (Text).



\### Passo 3: Configurar o Fluxo Lógico (Screen Action)

No clique do botão de processamento, monte o seguinte fluxo:

1\. Chame a Server Action \*\*`ValidateExcelFile`\*\*. Passando o arquivo, o JSON de regras, a página atual (ex: `1`) e o tamanho do lote (ex: `5000`).

2\. Adicione o nó \*\*JSON Deserialize\*\* apontando para o output `CleanedDataJSON` e use a estrutura `ExcelRowResult List` como Data Type.

3\. Atribua o resultado do Deserialize a uma variável local de lista (`CleanedRowsList`) para alimentar o seu ecrã.



\### Passo 4: Paginação da Interface

Para ficheiros gigantes, crie um loop ou utilize a paginação padrão do widget \*\*Table\*\* do OutSystems. Sempre que o utilizador mudar de página na tabela, re-execute a Screen Action incrementando o parâmetro \*\*`PageNumber`\*\* (ex: Página 1, Página 2, Página 3) até que o `CleanedDataJSON` retorne um array vazio, indicando o fim do documento.



\---



\## 📈 Melhores Práticas \& Limitações

\* \*\*Aba Padrão:\*\* O motor faz a leitura automática da \*\*primeira Worksheet (Aba)\*\* ativa dentro do ficheiro Excel.

\* \*\*Cabeçalhos:\*\* A primeira linha do ficheiro Excel é estritamente tratada como a linha de cabeçalhos. Certifique-se de que não existem linhas em branco no topo do documento.

\* \*\*Tamanho Máximo do Lote:\*\* Embora o motor C# consiga processar fatias maiores, evite usar um `PageSize` superior a `10000` linhas para garantir que os dados trafegam de forma instantânea e sem latência na rede.



