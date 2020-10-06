## Hadoop vs Spark na GCP: Uma comparação Shakesperiana

Apesar de serem tecnologias diferentes, a comparação dos dois frameworks mantidos pela Apache é inevitável. Para tirar essa dúvida de inciante, iremos nesse projeto comparar como as duas tecnologias lidam com uma tarefa estilo "Hello World", um contador de frequências de palavras de um 
dataset com todas as obras de William Shakespeare, um [.txt com mais de 5 MB](https://www.kaggle.com/kewagbln/shakespeare-word-count-with-spark-python?select=t8.shakespeare.txt). Para isso, usaremos a infraestrutura oferecida pela Google Cloud Platform (GCP), que permite facilmente subir clusters hadoop e spark, de forma gratuita no caso de pequenos projetos como esse.

Então faremos o seguinte:

* Criar um cluster Data Proc na GCP;
* Acessar o cluster através de SSH;
* Criar um notebook Jupyter para escrever o script PySpark;
* Criar os scripts mapper e reducer para o script Hadoop;
* Executar as tasks no Spark e Hadoop;
* Comparar os resultados.

You can use the [editor on GitHub](https://github.com/matheusferreira195/hadoop-spark-word-counter/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/matheusferreira195/hadoop-spark-word-counter/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
