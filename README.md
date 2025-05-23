
# Roteiro Prático – Hadoop MapReduce no *Play With Docker* 

> **Objetivo**  
> Exercitar, em um ambiente 100 % on‑line, os conceitos de **Docker & Containers**, **Hadoop** e o paradigma **Map Reduce** usando textos públicos do *Projeto Gutenberg*.

---

## Visão Geral

| Tópico | O que você fará |
|--------|-----------------|
| 1 | Criar (ou baixar) uma **imagem customizada** derivada de `apache/hadoop:3.3.6` que mantém os serviços Hadoop em execução. |
| 2 | Executar o contêiner no **Play With Docker** (PWD) e expor as interfaces Web. |
| 3 | Baixar livros do *Projeto Gutenberg*, carregá‑los no HDFS e rodar o job *wordcount*. |
| 4 | Registrar **pontos de verificação** e gerar **evidência final** do resultado obtido. |

> **Tempo estimado:** 45 minutos  
> **Pré‑requisitos dos alunos:** Conta no Docker Hub e conhecimento teórico prévio sobre Docker e Hadoop.

---

## 1  Preparar o ambiente no Play With Docker

1. Acesse <https://labs.play-with-docker.com> e clique em **Login** (use suas credenciais do Docker Hub).  
2. Clique em **Start** para iniciar uma sessão de 4 horas. Um nó com Alpine Linux será criado.

### ✅ Checkpoint 1  
Execute `docker version`. A saída deve mostrar *Client* e *Server*.

---

## 2  Obter a imagem customizada


```bash
docker pull infrabigdata/hadoop-infra:3.3.6
```

## 3  Executar o contêiner Hadoop

```bash
docker run -d --name hadoop -p 9870:9870 -p 8088:8088 infrabigdata/hadoop-infra:3.3.6
```

### ✅ Checkpoint 3  
`docker ps` deve mostrar o contêiner **Up**.  
Acesse:

- **NameNode UI:** `http://IP_DO_PWD:9870`
- **ResourceManager UI:** `http://IP_DO_PWD:8088`

---

## 4  Baixar o dataset do Projeto Gutenberg

1. Entre no contêiner:

    ```bash
    docker exec -it hadoop bash
    ```

2. Crie uma pasta e faça download de dois livros:

    ```bash
    mkdir -p /data/gutenberg && cd /data/gutenberg
    wget https://www.gutenberg.org/cache/epub/1342/pg1342.txt  -O pride_prejudice.txt
    wget https://www.gutenberg.org/cache/epub/2600/pg2600.txt  -O war_peace.txt
    ```

### ✅ Checkpoint 4  
`ls -lh /data/gutenberg` deve mostrar os dois arquivos.

---

## 5  Carregar arquivos no HDFS

```bash
hdfs dfs -mkdir -p /user/root/gutenberg
hdfs dfs -put /data/gutenberg/*.txt /user/root/gutenberg/
hdfs dfs -ls /user/root/gutenberg
```

### ✅ Checkpoint 5  
Os arquivos aparecem listados no HDFS.

---

## 6  Executar o job *wordcount* (Map Reduce)

```bash
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar   wordcount /user/root/gutenberg /user/root/gutenberg_out
```

*Monitore o job na UI do ResourceManager (porta 8088).*

### ✅ Checkpoint 6  
O job finaliza com **SUCCEEDED**.

---

## 7  Verificar resultados e gerar evidência

1. Mostre as primeiras 20 palavras do resultado do processamento

    ```bash
    hdfs dfs -cat /user/root/gutenberg_out/part-r-00000 | head -20
    ```

2. Salve a saída em um arquivo local (opcional):

    ```bash
    hdfs dfs -cat /user/root/gutenberg_out/part-r-00000  | head -20 > /data/wordcount_result.txt
    ```

### 🎯 Evidência final  
Tire um **print** ou **copie** o conteúdo das 20 linhas 

---

## 8  Encerrar

```bash
docker stop hadoop && docker rm hadoop
```

Ou simplesmente encerre a sessão do PWD; tudo será descartado.

---

### 🎉 Parabéns!

Se você completou todos os checkpoints e apresentou a evidência final, concluiu com sucesso a atividade prática de Docker + Hadoop MapReduce no Play With Docker.
