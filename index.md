## Hadoop vs Spark na GCP: Uma comparação Shakesperiana

Apesar de serem tecnologias diferentes, a comparação dos dois frameworks mantidos pela Apache é inevitável. Para tirar essa dúvida de inciante, iremos nesse projeto comparar como as duas tecnologias lidam com uma tarefa estilo "Hello World", um contador de frequências de palavras de um 
dataset com todas as obras de William Shakespeare, um [.txt com mais de 5 MB](https://www.kaggle.com/kewagbln/shakespeare-word-count-with-spark-python?select=t8.shakespeare.txt). Para isso, usaremos a infraestrutura oferecida pela Google Cloud Platform (GCP), que permite facilmente subir clusters hadoop e spark, de forma gratuita no caso de pequenos projetos como esse.

Os seguintes passos precisam ser realizados:

* Criar um cluster DataProc na GCP;
* Acessar o cluster através de SSH;
* Criar um notebook Jupyter para escrever o script PySpark;
* Criar os scripts mapper e reducer para o script Hadoop;
* Executar as tasks no Spark e Hadoop;
* Comparar os resultados.

### Criando o cluster DataProc na Google Cloud Platform

Depois de criada uma conta google padrão, cada usuário tem direito a $300 para testar os serviços de cloud oferecidos pela Google. Tendo uma conta em mãos e feito o login, é necessário acessar o [console](https://cloud.google.com/cloud-console) do Google Cloud. Feito isso, o primeiro projeto é criado automaticamente e temos acesso aos serviços da GCP.

Para criar o cluster, vamos na aba "Dataproc", acessível pelo menu lateral que aparece na esquerda do site, conforme a imagem abaixo.
<a href="https://imgur.com/xPzYHha">
![](https://imgur.com/xPzYHha)

Feito isso, precisamos apenas configurar o cluster. Como o objetivo desse artigo não é ensinar [como configurar um cluster na GCP](https://www.youtube.com/watch?v=6DD-vBdJJxk&t=602s&ab_channel=LearningJournal), colocarei abaixo apenas as configurações que utilizei em forma de comando da gcloud shell.

```
gcloud dataproc clusters create cluster-2508 --region us-central1 --subnet default --zone us-central1-b --master-machine-type n1-standard-4 --master-boot-disk-size 25 --num-workers 2 --worker-machine-type n1-standard-1 --worker-boot-disk-size 25 --image-version 1.3-debian10 --optional-components ANACONDA,JUPYTER --project reflected-post-291510

```

Resumindo, foi criado um cluster com master de 4 núcleos (n1-standard-4), com 25GB de espaço e 2 slaves de 2 núcleos (n1-standard-4) de 25GB, com Jupyter e Anaconda instalados.
O legal do GCP é que tudo que precisamos para começar a trabalhar será criado com apenas um clique e em minutos.

### Acessando o cluster através de SSH

Antes de começar a escrever os scripts, é necessário transferir o .txt para a máquina master, subir esse arquivo pro HDFS e criar os diretórios do projeto. Para isso, a GCP oferece excelentes ferramentas de SSH que nos permitem acessar facilmente a máquina master, literalmente no apertar de um botão.

![](https://imgur.com/zjVz84B")
Agora que temos acesso a nossa master, basta criar dois diretórios no /home, o /home/files e /home/scripts. Dentro de /home/files, executamos o seguinte comando para baixar o .txt contendo as obras de Shakespeare:

```
sudo wget https://raw.githubusercontent.com/andretadeu/spark-wordcount/master/src/main/resources/shakespeare.txt

```
Feito isso, subimos o arquivo no HDFS usando o seguinte comando:

```
hdfs dfs -mkdir /files
hdfs dfs -put /home/files/shakespeare.txt /files
```

Agora temos o necessário para poder começar o experimento.

### Criando job PySpark usando Notebooks Jupyter

Como selecionamos na criação do cluster o pacote adicional do Anaconda e do Jupyter, podemos acessar com um botão esses ambientes na nossa máquina master. 

![](https://imgur.com/5UDnkZx)

O script corresponde em iniciar a SparkSession, carregar o arquivo .txt, pré-processar o arquivo e iniciar as operações de map e reduce. Para uma visão mais detalhada, pode-se acessar o [html do notebook Jupyter](https://github.com/matheusferreira195/hadoop-spark-word-counter/blob/main/resources/SparkJob.html).

O tempo de execução dessa tarefa, para o .txt de 5MB, foi de 6.8835 segundos.

## Criando job Hadoop usando Hadoop Streaming + Python

O Hadoop Streaming permite que sejam utilizados scripts customizados para as etapas de Map e de Reduce. Tomaremos proveito dessa feature para usar scripts python.

Primeiro, o script de map:

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
 
# Obtendo texto do stdin
for line in sys.stdin:
    # Dividindo texto em palavras
    line = line.strip()
    words = line.split(' ')

    # Escrevendo palavras no stdout
    for word in words: 
        print '%s\t%s' % (word, "1")

```
Esse script simplesmente coleta o que estiver no stdin, divide em palavras e escreve no stdout.

Para o reduce, temos:

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys
 
# Armazena a relacao de palavras e sua frequencia
word2count = {}
 
# Coleta input do stdin
for line in sys.stdin:
    # Remove espacos
    line = line.strip()

    # Parseia o que foi escrito pelo mapper no stdin
    word, count = line.split('\t', 1)
    
    # Converte contagem em inteiro
    try:
        count = int(count)
    except ValueError:
        continue

    try:
        word2count[word] = word2count[word]+count
    except:
        word2count[word] = count
 
# Escreve tuplas no stdout
for word in word2count.keys():
    print '%s\t%s'% ( word, word2count[word] )
```
O script de reduce também é muito simples.

Agora podemos executar o job utilizando o seguinte comando:

```
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-streaming.jar -file /home/scripts/mapper.py -mapper mapper.py -file /home/scripts/reducer.py -reducer reducer.py -input /files/shakespeare.txt -output /files/output
```
Esse job foi completados em impressionantes 61 segundos, conforme podemos conferir no log do Hadoop abaixo.

![](https://imgur.com/Or4XRIx)


## Comparação: Hadoop vs Spark

Os resultados foram bem claros, o Spark conseguiu finalizar a task aproximadamente 10x mais rápido. Isso se deve ao fato de o Spark realizar suas operações em memória, o que traz a desvantagem de demandar máquinas mais poderosas para executar tasks mais exigentes.
