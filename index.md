---
title: Latihan Azure AI Bahasa
permalink: index.html
layout: home
---

# Latihan Azure AI Bahasa

Latihan berikut ini dirancang untuk memberi Anda pengalaman pembelajaran langsung di mana Anda akan menjelajahi tugas-tugas umum yang dilakukan pengembang saat membangun solusi bahasa alami di Azure. 

> {**Catatan**: Untuk menyelesaikan latihan, Anda memerlukan langganan Azure. Jika Anda belum memilikinya, Anda bisa mendaftar untuk mendapatkan [akun Azure](https://azure.microsoft.com/free). Ada opsi uji coba gratis untuk pengguna baru yang menyertakan kredit selama 30 hari pertama.

## Latihan

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

<hr>

> **Catatan**: Meskipun Anda dapat menyelesaikan latihan ini sendiri, latihan ini dirancang untuk melengkapi modul di [Microsoft Learn](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/); di mana Anda akan menemukan penyelaman yang lebih dalam ke dalam beberapa konsep dasar yang menjadi konsep dasar latihan ini. 
