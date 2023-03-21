---
title: "Quick Beginner Guide Webscraping Python B4s"

date: 2023-03-20T20:18:29-03:00
url: /quick-beginner-guide-webscraping-python-b4s/
image: /images/2023-thumbs/python.webp
categories:
  - Networking
  - Python
tags:
  - Web Scraping
draft: false
---

## Introduction 
Due to the large amount of data (mostly unstructured) publicly available on the internet, web scraping is an essential tool for any developer working with data extraction.
<!--more-->

In this tutorial we briefly introduce the concept of web scraping and demonstrate how it can be applied with Python and the Beautiful Soup library.

By the end of this tutorial, you will have a basic understanding of web scraping and how to apply it with Python and Beautiful Soup.

## What is Web Scraping
Web Scraping is the process/technique to extract data from websites. This technique is widely used in analysis and data mining. Generally, web craping tools provide functionality to automate data collection, allowing easy analysis and navigation of website components.

## Prerequisites
* Basic command line knowledge of your operating system
* Basic knowledge of Python
* Familiarity structure of an HTML document

## Installing Dependencies
Before starting, we need to install the dependencies:
### Using pip
PIP is Python's default package manager. To install dependencies using pip use the commands below:
{{< highlight bash >}}
pip install requests beautifulsoup4
{{</highlight>}}
### Using the package manager (Ubuntu and ubuntu based distributions)
Although installing packages on the system as a whole is possible, prefer to install in virtual environments (using virtualenv, for example) to isolate dependencies for each project.

{{< highlight bash >}}
sudo apt install python3-bs4
sudo apt install python3-requests
{{</highlight>}}

## 1. Download target page
To download content from the web page, we can use the **requests** library as shown below.

{{< highlight python >}}
import requests

url = "https://danielsales.com.br/quick-beginner-guide-webscraping-python-b4s"

response = requests.get(url)

#Exit if the request is not successfull
if response.status_code != 200:
  return
{{</highlight>}}

## Parse page content with beautiful soup
With the HTML content of the page saved in a variable, we can instantiate Beuatiful Soup and start analyzing its data.
```Python
import BeautifulSoup

soup = BeautifulSoup(response.text, "html.parser")
# Display the title of the page
print(soup.title)
```

## Extract data
Beautiful Soup allows simple navigation between html elements. For example, here we extract all elements with the `<a>` tag and display their HREF link.
```Python
# Retrieve all elements with <a> tag
links = soup.find_all("a")
# Display all links to elements with <a> tag
for link in links:
  print(link.get("href"))
```

## Save the extracted data
With the extracted data, it is usually saved in CSV or in some database. In the example below we save the extracted links in a CSV file using the **csv** library built into python.
```Python
import csv

with open('links.csv', 'w') as csvfile:
  writer = scv.writer(csvfile)
  for link in links:
    writer.writerrow([link.get("href")])
```

## Conclusion
Here we briefly introduce the concept of web scraping and demonstrate how it can be done in Python with the Beautiful Soup library. With this knowledge you can start digging deeper and better exploring the library to prepare yourself to explore and analyze the enormous amount of data available on the internet.

### Credits
Post thumbnail: <a href="https://www.vecteezy.com/free-png/3d">3d PNGs by Vecteezy</a>
