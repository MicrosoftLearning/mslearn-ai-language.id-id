---
title: Latihan Azure AI Bahasa
permalink: index.html
layout: home
---

# Latihan Azure AI Bahasa

Latihan berikut dirancang untuk mendukung modul di Microsoft Learn untuk [Mengembangkan solusi bahasa alami](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/).


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
