---
title: "Guia Rápido para Web Scraping com Python e Beautiful Soup"

date: 2023-03-20T20:18:06-03:00
url: /quick-beginner-guide-webscraping-python-b4s/
image: /images/2023-thumbs/python.webp
categories:
  - Networking
  - Python
tags:
  - Web Scraping
draft: false
---

## Introdução
Em decorrência da grande quantidade de dados (em sua grande maioria desestruturado) disponível publicamente na internet, web scraping é uma ferramenta necessária para qualquer desenvolvedor que trabalha com extração de dados.
<!--more-->
Neste tutorial introduzimos brevemente o conceito de web scraping, e demonstraremos como pode ser aplicado com Python e a biblioteca Beautiful Soup.

Ao fim deste tutorial você terá um conhecimento básico sobre web scraping e como aplicá-lo com python e beautiful soup.

## O que é Web Scraping
Web Scraping é o processo/técnica para extrair dados de websites. Esta técnica é amplamente utilizada em análise e mineração de dados. Geralmente, ferramentas de webs craping proveem funcionalidade para automatizar a coleta de dados, permitindo fácil análise e navegação dos componentes do website.

## Pré-requisitos
* Conhecimento Básico da linha de comando do seu sistema operacional
* Conhecimento Básico de Python
* Familiaridade estrutura de um documento HTML

## Instalando Dependências
Antes de começar precisamos instalar as dependências:
### Utilizando pip
PIP é o gerenciador de pacotes padrão de Python. Para instalar as dependências utilizando o pip utilize os comandos abaixo:
{{< highlight bash >}}
pip install requests beautifulsoup4
{{</ highlight >}}
### Utilizando o gerenciador de pacotes (Ubuntu e distribuições baseadas em ubuntu)
Apesar de instalar pacotes no sistema como um todo seja possível, prefira instalar em ambientes virtuais (utilizando virtualenv, por exemplo) para isolar dependências de cada projeto.

{{< highlight bash >}}
sudo apt install python3-bs4
sudo apt install python3-requests
{{</ highlight >}}

## 1. Faça o download da página alvo
Para fazer o download do conteúdo da página web, podemos utilizar a biblioteca **requests** como feito abaixo.

{{< highlight python >}}
import requests

url = "https://danielsales.com.br/quick-beginner-guide-webscraping-python-b4s"

resposta = requests.get(url)

#Exit if the request is not sucessfull
if resposta.status_code != 200:
  return
{{</ highlight >}}

## Analise o conteúdo da página com o beautiful soup
Com o conteúdo HTML da página salvo em uma variável, podemos instanciar o Beuatiful Soup e começar a analisar os dados dela.
```Python
import BeautifulSoup

soup = BeautifulSoup(resposta.text, "html.parser")
# Exibe o título da página
print(soup.title)
```

## Extrair dados
Beautiful Soup permite a navegação simples entre os elementos html. Por exemplo, aqui extraimos todos os elementos com tag <a> e exibimos o link HREF dele.
```Python
# Recupera todos os elementos com tag <a>
links = soup.find_all("a")
# Exibe todos os links dos elementos com tag <a>
for link in links:
  print(link.get("href"))
```

## Salvar os dados extraídos
Com os dados extraídos geralmente se salva ele em CSV ou em alguma base de dados. No exemplo abaixo salvamos os links extraidos em um arquivo CSV usando a biblioteca **csv** imbutida no python.
```Python
import csv

with open('links.csv', 'w') as csvfile:
  writer = scv.writer(csvfile)
  for link in links:
    writer.writerow([link.get("href")])
```

## Conclusão
Aqui brevemente introduzimos o conceito de web scraping e demonstramos como é possível ser feito em Python com a biblioteca Beautiful Soup. Com este conhecimento você pode começar a se aprofundar e explorar melhor a biblioteca para se preparar para explorar e analisar a quantidade enorme de dados disponíveis na internet.

### Credits
Post thumbnail: <a href="https://www.vecteezy.com/free-png/3d">3d PNGs by Vecteezy</a>
