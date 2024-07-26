
# Mineração das datas

## Disclaimer

- Os passos a seguir assumem que você já possui um arquivo de texto com as strings para serem extraídas.

## Passos para minerar

1. Para minerar as datas é necessário primeiramente clonar o repositório de onde se deseja extrair as datas. Vá para a página do repositório, e clique no botão verde escrito **"< > Code"** e copie o o link http do repositório (se tiver familiaridade com o uso de chaves SSH, pode ir por ela também).

2. Abra o Git Bash no diretório onde você deseja clonar o repositório.

3. Execute o comando **git clone <link_do_repositório>** no bash.

4. Com o repositório clonado, crie um arquivo bash na raiz do repositório clonado (ex: **datas.sh**), e cole o seguinte script nesse arquivo (**Note que na quarta linha do código é necessário alterar parte do nome do arquivo para o nome do repositório a ser minerado**):

```
#!/bin/bash

# Read the strings from the file into an array
mapfile -t strings < ./data--NOME_DO_REPOSITÓRIO.txt

# Loop over each string
for str in "${strings[@]}"; do
    # Trim leading and trailing whitespace from the string
    str=$(echo "$str" | xargs) # to skip the xargs single quote error, add -0 after xargs
    
    #echo "Processing: '$str'"

    # Git command to search for the string in the repository and get the date of the first occurrence
    resultado=$(git log -S "$str" --pretty=format:"%ad" --name-only -- '*.c' '*.h' '*.cpp' '*.java' | tac | awk 'NR==2{print;exit}')
    
    # Check if the string was found
    if [ -n "$resultado" ]; then
        echo "$resultado"
    else
        echo "Data not found for: $str"
    fi
done
```

5. Cole o arquivo de texto com as strings também na raiz do repositório.

6. Abra novamente o Git Bash dentro da raiz do repositório clonado, e execute o comando **./nome_do_script.sh** (se os passos anteriores até aqui foram executados corretamente, em poucos segundos você começará a ver os outputs da mineração em andamento sendo printados no Git Bash).

## Depois de minerar

1. Após o fim da mineração, selecione todos os outputs printados no Git Bash com o cursor do mouse, copie e cole em uma planilha vazia do Google Sheets.

2. Na barra de opções da planilha, clique em **Extensões**, e abra em outra guia a opção **Apps Script**.

3. Copie o script a seguir e cole no editor de texto do Apps Script, e salve o arquivo:

```
function deleteRowsContainingSpecificString() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var lastRow = sheet.getLastRow();
  var lastColumn = sheet.getLastColumn();
  var searchString = 'xargs:';
  
  // Start from the last row and move upwards
  for (var i = lastRow; i >= 1; i--) {
    var range = sheet.getRange(i, 1, 1, lastColumn);
    var values = range.getValues();
    var containsString = false;
    
    // Check if any cell in the row contains the specific string
    for (var j = 0; j < values[0].length; j++) {
      if (values[0][j].toString().indexOf(searchString) !== -1) {
        containsString = true;
        break;
      }
    }
    
    // Log the row being checked and whether it contains the string
    Logger.log("Row " + i + " contains the string: " + containsString);
    
    // Delete the row if it contains the string
    if (containsString) {
      sheet.deleteRow(i);
    }
  }
}
```

4. Certifique-se de que dentre as abas do Google Sheets do seu navegador, a única aberta é a com o output da mineração, e em seguida, clique em **Executar** para rodar o script.

5. Quando terminar de rodar o script, o seu output já estará tratado.
