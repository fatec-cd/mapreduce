
# Roteiro Prático: Aprendendo **MapReduce** com Containers

> **Tempo estimado:** 2 – 3 h de atividade prática  

---

## 1. Visão geral

Neste roteiro você aprenderá a:

1. **Criar** um cluster Hadoop de nó único dentro de um contêiner Docker **sem instalar nada localmente**.  
2. **Executar** um job MapReduce de _WordCount_ sobre um dataset público (logs HTTP da NASA – agosto/1995).  
3. **Interpretar** resultados e experimentar variações.

Toda a prática ocorre no navegador utilizando o serviço gratuito **Play with Docker** (PWD).

---

## 2. Pré‑requisitos

| Recurso | Detalhes |
|---------|----------|
| Navegador | Chrome, Edge ou Firefox recente |
| Conta GitHub **ou** Docker Hub | Para autenticar‑se no PWD |
| Conhecimentos básicos | Terminal Linux e comandos Docker; noções de Big Data |

---

## 3. Ambiente de trabalho (Play with Docker)

1. Acesse <https://labs.play-with-docker.com> e clique em **_Start_**.  
2. Faça login com sua conta GitHub/Docker Hub.  
3. Dentro do painel, clique em **_Add New Instance_** para obter um terminal Alpine Linux com Docker & Docker‑Compose já instalados (a sessão dura 4 h).  

> 💡 **Dica:** use o botão **_Share_** para salvar a URL da sessão caso precise reconectar.

---

## 4. Passo a passo

### 4.1 Preparar o diretório de trabalho

```bash
mkdir ~/hadoop && cd ~/hadoop
```

### 4.2 Clonar o repositório com o _docker‑compose_

```bash
git clone https://github.com/big-data-europe/docker-hadoop.git
cd docker-hadoop
```

### 4.3 Subir o cluster Hadoop (nó único)

```bash
docker compose up -d        # ~1 min
```

*Contêineres criados*  
- **namenode** (HDFS + YARN ResourceManager)  
- **datanode**  
- **resourcemanager & nodemanager**  
- **historyserver**

Verifique:

```bash
docker ps --format "table {{.Names}}	{{.Status}}"
```

### 4.4 Entrar no contêiner **namenode**

```bash
docker exec -it namenode bash
```

O prompt mudará para `root@namenode:/#`.

### 4.5 Baixar e carregar o dataset público

```bash
# ainda dentro do contêiner
cd /tmp
curl -O https://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz
gunzip NASA_access_log_Aug95.gz      # ~21 MB → 167 MB
hdfs dfs -mkdir -p /datasets/nasa
hdfs dfs -put NASA_access_log_Aug95 /datasets/nasa/
```

Confirme:

```bash
hdfs dfs -ls /datasets/nasa
```

### 4.6 Executar o job MapReduce (WordCount)

```bash
export JAR=$(hadoop classpath | tr ':' ' ' | grep hadoop-mapreduce-examples | head -n1)
hadoop jar "$JAR" wordcount       /datasets/nasa /output/nasa_wc
```

O YARN exibirá progresso de **Map → Shuffle → Reduce**. Ao final:

```
INFO mapreduce.Job:  map 100% reduce 100%
INFO mapreduce.Job: Job job_... completed successfully
```

### 4.7 Visualizar resultados

```bash
hdfs dfs -cat /output/nasa_wc/part-r-00000 | head -n 20
```

Saída esperada (valores aproximados):

```
GET                 1,563,000
HTTP/1.0            1,560,000
200                 1,290,000
/index.html         245,000
/images/ksc.gif      97,000
...
```

> 🔍 **Interpretação:** a contagem evidencia que a maior parte das linhas começa com a operação `GET`. URLs e códigos de status aparecem como tokens frequentes.

---

## 5. Resultados esperados

| Etapa | Indicador de sucesso |
|-------|----------------------|
| 4.3   | `docker ps` mostra 5 contêineres `Up` |
| 4.5   | `hdfs dfs -ls /datasets/nasa` lista o arquivo de ~167 MB |
| 4.6   | Mensagem `completed successfully` no final do job |
| 4.7   | Arquivo `part-r-00000` com pares **palavra ↔ frequência** |

---



## 6. Encerramento & limpeza

```bash
# no host PWD
docker compose down
exit    # sai do contêiner
```

Ao finalizar, feche a aba ou espere a sessão expirar.

---

## 7. Referências

* Play with Docker – laboratório online (Docker Inc.)  
* Repositório **docker-hadoop** – Big Data Europe  
* Dataset **NASA HTTP logs 1995** – Internet Traffic Archive  
