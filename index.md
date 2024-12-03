---
title: 使用 Azure DevOps 练习，通过管道实现安全性
permalink: index.html
layout: home
---

# 使用 Azure DevOps 练习，通过管道实现安全性

以下练习旨在为[使用 DevOps 通过管道实现安全性](https://learn.microsoft.com/training/paths/implement-security-through-pipeline-using-devops/)模块提供支持。

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
