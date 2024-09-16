# Projeto ETL com Pandas, PySpark e PostgreSQL

## Descrição do Projeto

Este projeto demonstra o processo de Extração, Transformação e Carga (ETL) de dados de um arquivo Excel para um banco de dados PostgreSQL utilizando Pandas e PySpark. O objetivo é carregar dados de um arquivo Excel, transformá-los em um formato apropriado, e inseri-los em uma tabela do PostgreSQL para uso posterior.

## Passos Realizados

1. **Carregamento do Arquivo Excel:**
   - Utilizamos a biblioteca **Pandas** para carregar o arquivo `data_faturamento.xlsx` em um DataFrame.
   - Este passo permite visualizar e manipular os dados antes de convertê-los para outro formato.

    ```bash
    # Código para carregar o arquivo Excel
    df = pd.read_excel(file_path)
    ```

2. **Conversão para CSV:**
   - Convertido o DataFrame do Pandas para um arquivo CSV (`data_faturamento.csv`).
   - A conversão para CSV é necessária para facilitar o processamento dos dados com PySpark.

    ```bash
    # Código para converter para CSV
    df.to_csv('data_faturamento.csv', index=False)
    ```

3. **Conexão com o PostgreSQL:**
   - Utilizamos a biblioteca **psycopg2** para conectar ao banco de dados PostgreSQL local.
   - Estabelecemos uma conexão com o servidor de banco de dados especificando o host, porta, nome do banco de dados, usuário e senha.

    ```bash
    # Código para conectar ao PostgreSQL
    conn = psycopg2.connect(
        host="localhost",
        port="5432",
        database="testdb",
        user="postgres",
        password="root"
    )
    ```

4. **Criação da Tabela no PostgreSQL:**
   - Executamos comandos SQL para criar a tabela `faturamento` no banco de dados PostgreSQL com a estrutura correta, incluindo colunas para data, número de pedido, faturamento, custo, nome do produto e tipo.
   - Utilizamos um comando `DROP TABLE IF EXISTS` para garantir que qualquer tabela existente com o mesmo nome seja removida antes de criar uma nova tabela.

    ```bash
    # Código SQL para criar a tabela
    cursor.execute("""
        DROP TABLE IF EXISTS faturamento;
        CREATE TABLE faturamento (
            data DATE,
            num_pedido INTEGER,
            faturamento NUMERIC(10, 2),
            custo NUMERIC(10, 2),
            nome_produto VARCHAR(255),
            tipo VARCHAR(50)
        );
    """)
    ```

5. **Configuração da SparkSession:**
   - Configuramos a **SparkSession** com o driver JDBC do PostgreSQL para permitir que o PySpark se conecte ao banco de dados.
   - O caminho para o driver JDBC do PostgreSQL (`postgresql-42.7.4.jar`) é especificado para que o Spark possa se comunicar com o banco de dados PostgreSQL.

    ```bash
    # Código para configurar a SparkSession
    spark = SparkSession.builder \
        .appName("Inserção de Dados no PostgreSQL") \
        .config("spark.jars", jdbc_driver_path) \
        .getOrCreate()
    ```

6. **Carregamento do CSV para PySpark:**
   - Utilizamos PySpark para carregar o arquivo CSV (`data_faturamento.csv`) em um DataFrame.
   - PySpark oferece suporte para processamento distribuído de dados, permitindo manipulação eficiente de grandes conjuntos de dados.

    ```bash
    # Código para carregar o CSV em PySpark
    spark_df = spark.read.csv('data_faturamento.csv', header=True, inferSchema=True)
    ```

7. **Conversão do Formato de Data:**
   - Usamos a função `to_date` do PySpark para converter a coluna "data" do formato string para o tipo de dado `DATE`.
   - Esta conversão é necessária para garantir que os dados sejam inseridos corretamente no PostgreSQL.

    ```bash
    # Código para converter a coluna de data
    spark_df = spark_df.withColumn("data", to_date(spark_df["data"], "dd/MM/yyyy"))
    ```

8. **Inserção dos Dados no PostgreSQL:**
   - Utilizamos a função `write` do PySpark para inserir os dados transformados no banco de dados PostgreSQL.
   - Especificamos o URL JDBC, a tabela de destino, o usuário, a senha e o driver JDBC para garantir que os dados sejam inseridos corretamente.

    ```bash
    # Código para inserir os dados no PostgreSQL
    spark_df.write \
        .format("jdbc") \
        .option("url", "jdbc:postgresql://localhost:5432/testdb") \
        .option("dbtable", "faturamento") \
        .option("user", "postgres") \
        .option("password", "root") \
        .option("driver", "org.postgresql.Driver") \
        .mode("append") \
        .save()
    ```

9. **Encerramento da SparkSession:**
   - Após a conclusão da operação de inserção, a **SparkSession** é encerrada para liberar os recursos.

    ```bash
    # Código para encerrar a SparkSession
    spark.stop()
    ```

## Pré-requisitos

- **Python** 3.x
- **Pandas** (`pip install pandas`)
- **PySpark** (`pip install pyspark`)
- **psycopg2** (`pip install psycopg2`)
- **PostgreSQL** instalado e configurado na máquina local
- Driver JDBC do PostgreSQL (`postgresql-42.7.4.jar`)

## Como Executar

1. **Certifique-se de que todos os pré-requisitos estejam instalados.**
2. **Coloque o arquivo Excel (`data_faturamento.xlsx`) no diretório correto.**
3. **Execute o script em uma célula de um Jupyter Notebook ou outra IDE compatível com Python.**
4. **Verifique o banco de dados PostgreSQL para garantir que a tabela `faturamento` foi criada e preenchida corretamente.**

## Problemas Conhecidos

- Certifique-se de que o caminho para o driver JDBC do PostgreSQL esteja correto.
- Verifique se o PostgreSQL está em execução e configurado corretamente para aceitar conexões do cliente.
- Caso encontre erros de tipo de dados, ajuste o formato dos dados no PySpark antes da inserção.
