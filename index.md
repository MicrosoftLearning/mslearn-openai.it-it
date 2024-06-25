---
title: Esercizi di Azure OpenAI
permalink: index.html
layout: home
---

# Esercizi di Azure OpenAI

Gli esercizi seguenti sono progettati a supporto dei moduli su [Microsoft Learn](https://learn.microsoft.com/training/browse/?terms=OpenAI).


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
