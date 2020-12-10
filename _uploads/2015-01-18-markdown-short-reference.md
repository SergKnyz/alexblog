---
title: Markdown
date: 2015-01-18 00:00:00 Z
permalink: markdown-short-reference
categories:
- Руководства
tags:
- markdown
color: "#8c5353"
layout: post
---

Краткое руководство по синтаксису Markdown.

---

**[Markdown](http://ru.wikipedia.org/wiki/Markdown "Markdown"){: rel="nofollow"}** — это облегченный язык разметки, призванный облегчить подготовку текстов для публикации в Интернете. Был создан для удобства чтения и написания размеченных текстов. Движок markdown генерирует валидный XHTML. Авторы - John Gruber и Aaron Swartz.
Собственно, Markdown — это простой текст.

**Jekyll** использует *markdown* нативно, вообще данный язык разметки очень любят на *[GitHub](https://www.github.com/ "GitHub"){: rel="nofollow"}*, используют его везде, где только
можно (и в комментариях, и в отчетах, и в readme файлах). Данный блог использует *jekyll*, потому этот пост - это маленькая шпаргалка по Markdown.

## Синтаксис

Оригинальное описание синтаксиса находится здесь (англ.): [http://daringfireball.net/projects/markdown/syntax](http://daringfireball.net/projects/markdown/syntax){: rel="nofollow"}

Ниже следует краткое описание синтаксиса.

* Абзацы разделяются пустой строкой

* Два или более пробела на конце строки задают разрыв строки

* Шрифты: **жирный**: `**жирный**`, *курсив*:`_курсив_`, ***жирный и курсив***: `***жирный и курсив***`, `моноширинный`:``моноширинный``

* Заголовки:

  * Atx-style: `#первый уровень#`, `##второй уровень##` и т.д.

  * Setext-style: подчеркивание знаками `=` задает первый уровень, дефисами `-` — второй

* Цитаты: `> текст цитаты`

* Списки:

  * неупорядоченные: `* элемент списка` (также могут использоваться символы `-` или `+`).

  * упорядоченные: `1. элемент списка`

* Блок кода — каждая строка начинается с 4 или более пробелов

* Горизонтальная черта: три или более дефиса или звездочки

* Ссылки:

  * встроенные `[label](url "url title")`

  * автоматические `<url>`

  * в виде сносок

* Изображения:

  * встроенные `![alt text](url "image title")`

  * в виде сносок

* Экранирование символов — чтобы вставить спецсимвол, используемый в разметке, как обычный символ, его нужно предварить
  символом обратной косой черты. Экранироваться должны следующие символы: `* _ { } [ ] ( ) # + - . !`

Таблицы:

        | First Header  | Second Header |
        | ------------- | ------------- |
        | Row1 Cell1    | Row1 Cell2    |
        | Row2 Cell1    | Row2 Cell2    |  

Результат:

| First Header  | Second Header |
| ------------- | ------------- |
| Row1 Cell1    | Row1 Cell2    |
| Row2 Cell1    | Row2 Cell2    |

Наглядная визуальная шпаргалка:
![](https://manage.siteleaf.com/api/v2/sites/5e0e2c38ffbe5049e70c1206/source/assets/2015-01-18-markdown-short-reference/Markdown_cheatsheet.png?download "Изображение кликабельно")

<p>Copyright &copy; 2015,
   Статья <a href='http://alexprivalov.org/'>Алексея Привалова.</a>
    </p>