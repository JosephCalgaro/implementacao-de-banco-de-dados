# 18/09
- Procedures
```
CREATE PROCEDURE SP_NOME(@PARAM INT)
AS
BEGIN
	BLOCO;
END

EXEC SP_NOME
```
- Funçoes
```
CREATE FUNCTION FN_NOME(@PARAM INT, @PARAM INT)
RETURNS TIPO
AS
BEGIN
	BLOCO;
END
```

## Condicionais e laços

### IIF
Função condicional embutida, utilizada dentro de consultas `SELECT`.

```sql
SELECT IIF(Salario > 5000, 'Alto', 'Padrão') AS CategoriaSalario
FROM FUNCIONARIO;
```

- Se a condição for verdadeira, retorna o segundo valor.
- Se a condição for falsa, retorna o terceiro valor.

### CASE
Estrutura condicional de seleção. Permite testar várias condições.

```sql
SELECT 
    Pnome,
    CASE 
        WHEN Salario > 5000 THEN 'Sênior'
        WHEN Salario BETWEEN 3000 AND 5000 THEN 'Pleno'
        ELSE 'Júnior'
    END AS Nivel
FROM FUNCIONARIO;
```

### WHILE
Laço de repetição que executa um bloco enquanto uma condição for verdadeira.

```sql
WHILE (Condição)
BEGIN
    -- Executar bloco repetidamente enquanto a condição for verdadeira
END;
```

---

# User-Defined Functions (UDFs)

Funções criadas pelo usuário para reutilizar uma determinada lógica no banco de dados.

## 1. Função Escalar

Retorna um único valor.

### Função que calcula o dobro

```sql
CREATE OR ALTER FUNCTION fn_dobro(@Numero DECIMAL(10, 2))
RETURNS DECIMAL(10,2)
AS
BEGIN
    RETURN @Numero * 2;
END;
GO
```

Exemplos de uso:

```sql
SELECT dbo.fn_dobro(5);

SELECT F.Pnome, F.Unome, F.Salario, dbo.fn_dobro(F.Salario) AS 'Dobro'
FROM FUNCIONARIO AS F
WHERE F.Pnome = 'Maria';

SELECT F.Pnome, F.Unome, F.Salario, dbo.fn_dobro(F.Salario) AS 'Dobro'
FROM FUNCIONARIO AS F
WHERE F.Pnome = 'Carlos';
```

Buscar funcionários que ganham mais que o dobro do menor salário:

```sql
DECLARE @menor_salario DECIMAL(10,2);

SELECT @menor_salario = MIN(Salario)
FROM FUNCIONARIO;

SELECT Pnome, Unome, F.Salario
FROM FUNCIONARIO AS F
WHERE F.Salario > dbo.fn_dobro(@menor_salario);
```

### Função para calcular idade

```sql
CREATE OR ALTER FUNCTION fn_calcular_idade(@data_nasc DATE)
RETURNS INT
AS
BEGIN
    DECLARE @idade INT;

    SET @idade = DATEDIFF(YEAR, @data_nasc, GETDATE());

    IF (MONTH(@data_nasc) > MONTH(GETDATE())
        OR (MONTH(@data_nasc) = MONTH(GETDATE())
        AND DAY(@data_nasc) > DAY(GETDATE())))
        SET @idade = @idade - 1;

    RETURN @idade;
END;
GO
```

Calcular idade dos dependentes:

```sql
SELECT 
    D.Nome_dependente,
    D.Datanasc,
    dbo.fn_calcular_idade(D.Datanasc) AS 'Idade'
FROM DEPENDENTE AS D;
```

Calcular idade dos funcionários e formatar a data:

```sql
SELECT 
    F.Pnome,
    F.Unome,
    CONVERT(VARCHAR, F.Datanasc, 103) AS 'Data de Nascimento',
    dbo.fn_calcular_idade(F.Datanasc) AS 'Idade'
FROM FUNCIONARIO AS F;
```

---

## 2. Função In-Line (Tabela In-Line)

Retorna uma tabela diretamente a partir de uma consulta.

Exemplo: retornar os funcionários de um determinado departamento.

```sql
CREATE OR ALTER FUNCTION fn_func_dpt(@nome_dpt VARCHAR(50))
RETURNS TABLE
AS
RETURN (
    SELECT F.Pnome, F.Unome
    FROM FUNCIONARIO AS F
    JOIN DEPARTAMENTO AS D ON F.Dnr = D.Dnumero
    WHERE D.Dnome = @nome_dpt
);
GO
```

Executando:

```sql
SELECT *
FROM dbo.fn_func_dpt('Pesquisa');
```

---

## 3. Função Multi-Statement

Retorna uma tabela que é preenchida dentro de um bloco `BEGIN/END`.

Exemplo: retornar nome completo, salário mensal e salário anual estimado.

```sql
CREATE OR ALTER FUNCTION fn_salariAnual()
RETURNS @SalAno TABLE (
    nome_comp VARCHAR(100),
    salario DECIMAL(10,2),
    salario_anual DECIMAL(10,2)
)
AS
BEGIN
    INSERT INTO @SalAno
    SELECT
        CONCAT(F.Pnome, ' ', F.Minicial, ' ', F.Unome),
        F.Salario, 
        F.Salario * 13 + (F.Salario * 0.3)
    FROM FUNCIONARIO AS F;

    RETURN;
END;
GO
```

Executando:

```sql
SELECT *
FROM dbo.fn_salariAnual();
```

---

# Stored Procedures (Procedimentos Armazenados)

Procedures são blocos de comandos SQL armazenados no banco e executados por meio de `EXEC`.

Estrutura básica:

```sql
CREATE PROCEDURE SP_NOME(@PARAM INT)
AS
BEGIN
    BLOCO;
END;
GO

EXEC SP_NOME;
```

## Procedure simples

Exibe uma mensagem:

```sql
CREATE OR ALTER PROCEDURE sp_exibe_meu_nome
AS 
BEGIN
    PRINT 'Rafael Maruyama Dias';
END;
GO

EXEC sp_exibe_meu_nome;
GO
```

## Procedure com parâmetro

Aplica aumento percentual a todos os funcionários:

```sql
CREATE OR ALTER PROCEDURE sp_aumento(@porcentagem DECIMAL(3,1))
AS
BEGIN
    UPDATE FUNCIONARIO
    SET Salario = Salario * (1 + (@porcentagem / 100));
END;
GO

EXEC dbo.sp_aumento @porcentagem = 5;

SELECT *
FROM FUNCIONARIO;
```

## Procedure com múltiplos parâmetros

Aplica aumento somente ao funcionário informado pelo CPF:

```sql
CREATE OR ALTER PROCEDURE sp_aumento(
    @porcentagem DECIMAL(3,1),
    @cpf CHAR(11)
)
AS
BEGIN
    UPDATE FUNCIONARIO
    SET Salario = Salario * (1 + (@porcentagem / 100))
    WHERE Cpf = @cpf;
END;
GO

EXEC dbo.sp_aumento
    @porcentagem = 50,
    @cpf = '98765432300';

SELECT *
FROM FUNCIONARIO;
```

> Observação: os dois exemplos usam o mesmo nome `sp_aumento`. Ao executar no mesmo banco, o segundo `CREATE OR ALTER` altera a procedure existente para a versão com dois parâmetros.

## Procedure com ENCRYPTION

Impede a visualização direta do código-fonte da procedure:

```sql
CREATE OR ALTER PROCEDURE sp_funcionarios
WITH ENCRYPTION
AS
    SELECT *
    FROM FUNCIONARIO;
GO

EXEC sp_help sp_funcionarios;
```

## Procedure com validação

Insere um departamento e sua localização somente se o departamento informado ainda não existir.

```sql
CREATE OR ALTER PROCEDURE sp_novo_dplc(
    @departamento VARCHAR(50),
    @localidade VARCHAR(50),
    @numero INT
)
AS
BEGIN
    IF EXISTS(
        SELECT 1
        FROM DEPARTAMENTO
        WHERE Dnome = @departamento
          AND Dnumero = @numero
    )
    BEGIN
        PRINT 'Esse departamento já existe';
        RETURN;
    END
    ELSE
    BEGIN
        INSERT INTO DEPARTAMENTO(Dnome, Dnumero)
        VALUES (@departamento, @numero);

        INSERT INTO LOCALIZACAO_DEP(Dlocal, Dnumero)
        VALUES (@localidade, @numero);

        PRINT @departamento + ' inserido com sucesso';
        PRINT @localidade + ' inserido com sucesso';
    END;
END;
GO
```

Executando:

```sql
EXEC dbo.sp_novo_dplc
    @departamento = 'TI_Dev',
    @localidade = 'São Paulo',
    @numero = 110;
```

Consulta para verificar departamentos e suas localizações:

```sql
SELECT *
FROM DEPARTAMENTO AS D
JOIN LOCALIZACAO_DEP AS L
    ON D.Dnumero = L.Dnumero;
```

## Resumo rápido — 18/09

- **UDF**: função criada pelo usuário para reutilizar uma lógica.
- **Função escalar**: retorna um único valor.
- **Função In-Line**: retorna uma tabela baseada em uma única consulta.
- **Função Multi-Statement**: monta e retorna uma tabela usando vários comandos.
- **Procedure**: bloco de comandos armazenado no banco e executado com `EXEC`.
- **IIF**: condição simples dentro de um `SELECT`.
- **CASE**: permite trabalhar com várias condições.
- **WHILE**: repete comandos enquanto a condição for verdadeira.
- **BREAK**: interrompe o `WHILE`.
- **CONTINUE**: pula a execução restante da iteração atual e volta para a condição do laço.
- **GO**: separa lotes de comandos no SQL Server.

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
- Loop while
```
DECLARE @CONTADOR INT = 0;

WHILE @CONTADOR < 10
BEGIN 
    IF @CONTADOR = 5
        BREAK;
    SET @CONTADOR = @CONTADOR+1
    PRINT 'CONTADOR: '+CAST(@CONTADOR AS VARCHAR(100))
END
```
- Exemplo com CONTINUE
```
DECLARE @CONTADOR INT = 0;

WHILE @CONTADOR < 10
BEGIN 
    SET @CONTADOR = @CONTADOR+1
    IF @CONTADOR % 2 = 0
        CONTINUE
    PRINT 'CONTADOR: '+CAST(@CONTADOR AS VARCHAR(100))
END
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
