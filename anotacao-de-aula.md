# 04/09
- Variaveis declaradas no banco
```
DECLARE @VAR INT, @NOME VARCHAR(100);
SET @VAR = 10;
SELECT @NOME = F.nome
FROM Funcionario AS F
WHERE F.id = 1234
CAST (@VAR*10 AS VARCHAR 100) AS Resultado;
```
- Consulta de Funcionário com CAST e converção com CONVERT
```
USE EMPRESA
GO

DECLARE @SALARIO DECIMAL(10, 2),
        @NOME VARCHAR(100),
        @DATA DATE;

SET @NOME = 'Jennifer';

SELECT 
    @SALARIO = F.Salario,
    @DATA = F.Datanasc
FROM FUNCIONARIO AS F
WHERE F.Pnome = @NOME;

PRINT 'O Funcionario(a) ' + @NOME +
      ' tem um salario de R$' + CAST(@SALARIO AS VARCHAR(100)) +
      '. E nasceu em ' + CONVERT(VARCHAR(10), @DATA, 103);

GO
```
- Estrutura IF/ELSE em um banco de dados
````
DECLARE @IDADE INT;

SET @IDADE = 20;

IF @IDADE >= 18
BEGIN
    PRINT 'Maior de idade';
END
ELSE
BEGIN
    PRINT 'Menor de idade';
END

````
- Exemplo pratico
````
USE EMPRESA
GO

DECLARE @SALARIO DECIMAL (10, 2), @NOME VARCHAR(100), @DATA DATE, @SALARIO_MEDIO DECIMAL(10,2);

SET @NOME = 'Jennifer';

SELECT @SALARIO = F.Salario, @DATA = F.Datanasc
FROM FUNCIONARIO AS F
WHERE F.Pnome = @NOME

SELECT @SALARIO_MEDIO = AVG(F.Salario)
FROM FUNCIONARIO AS F

PRINT 'O Funcionario(a) '+@NOME+' tem um salario de R$'+ CAST (@SALARIO AS VARCHAR(100)) + '. E nasceu em '+ CONVERT(VARCHAR(10), @DATA, 103);

IF(@SALARIO>@SALARIO_MEDIO)
	PRINT 'O funcionario '+@NOME+' ganha acima da média. Media = '+CAST (@SALARIO_MEDIO AS VARCHAR(100))
ELSE
	PRINT 'O funcionario '+@NOME+' ganha acima da média. Media = '+CAST (@SALARIO_MEDIO AS VARCHAR(100))

GO
````

-Função IIF(CONDIÇÃO);__(VERDADE);__(FALSIDADE)) utilizada dentro do SELECT
```
USE EMPRESA


SELECT 
    F.Pnome,
    F.Unome,
    F.Salario,
    IIF(F.Salario < 20000, 'Baixo', 'Alto') AS CATEGORIA
FROM FUNCIONARIO AS F;
```


# 21/08
- **JOINS**
- INNER JOIN sempre retorna o comum entre as tabelas
- ALTER JOIN retorna todas as tabelas em integra
- LEFT JOIN Retorna apenas a tabela 1 ou "a tablea a esquerda"
- RIGTH JOIN retorna apenas a tabela 2 ou "a tabela a direita"
- SELF JOIN compara a tabela com ela mesma
- SQL union/intersect/except, tabelas devem possuir a mesma quantidade de colunas
- union "junta as tabelas"

# 31/07
- Modelo Entidade Relacionamento Conceitual (MER)
- Convenções para SQL (snake_case, camel case, etc...)
- Utilizamos o MER para evitar duplicidade de dados em grandes volumes de informações
- Atributos Multivalorados (possivel no MER, impossivel no SQL)
- BRModelo
- N sempre "puxa" a FK
