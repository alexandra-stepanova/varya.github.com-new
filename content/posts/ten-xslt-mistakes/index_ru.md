---
title: 10 ошибок XSLT-программистов

categories: ru issues
old: true
date: 2010-08-12

layout: post
---

Перевод [статьи "The ten most common XSLT programming
mistakes"](http://saxonica.blogharbor.com/blog/_archives/2010/6/11/4550606.html),
ссылка на которую недавно опубликована в [клубе XSLT на
Я.ру](http://clubs.ya.ru/xslt/).

Недавно я сказал в ответ пользователю, что он попадает в наиболее
распространённые ловушки для программистов на XSLT. Вместо того, чтобы быть
раздраженным, что я почти ожидал, он поблагодарил меня и спросил, не мог бы я
рассказать ему о двугих ловушках.<excerpt/> Некоторые из нас помогают людям избежать этих
ловушек в течение многих лет, но, несмотря на это, я не припомню, чтобы видел
список таких ловушек. Так что я решил потратить полчаса, чтобы составить такой
список.

- Обрабатывать элементы в дефолтном пространстве имён (namespace). Если
  исходный XML-документ содержит декларацию дефолтного пространства имён
  `xmlns="something"`, то каждый раз, когда вы ссылаетесь на имя элемента в
  XPath-выражении или значении атрибута match, вы должны ясно давать понять, что
  вы имеете в виду элементы из этого пространства имён. В XSLT 1.0 вы должны
  связать префикс с этим пространством имён (например `xmlns:p="something"` в
  элементе `xsl:stylesheet`) и затем везде использовать этот префикс, напимер,
  `match="p:chapter/p:section"`. В XSLT 2.0 есть альтернатива - задекларировать в
  элементе xslt:stylesheet `default-xpath-namespace="something"`.
- Использование относительных путей. xsl:apply и xsl:for-each принимают текущий
  узел; в рамках "цикла" пути должны быть написаны относительно текущего узла.
  Например,

&lt;xsl:for-each select="chapter">&lt;xsl:value-of select="title"/>&lt;/xsl:for-each>

<li>Распространённая ошибка - использование абсолютных путей внутри цикла (например <em>select="//title"</em>) или повторение имени контекстного узла в относительном пути (<em>select="chapter/title"</em>).</li>
  <li>Переменные содержат значения, а не фрагменты синтаксических выражений. Некоторые люди думают, что ссылка на переменную $x подобна макросу, распространяющемуся и на синтаксис xPath-выражений путйм буквального замещения - как в языках типа shell. Это не так: вы можете использовать переменные только там, где вы можете использовать значение. Например, если $N содержит строку 'para', то выражение <em>chapter/$N</em> не означает того же, что и <em>chapter/para</em>. Вместо этого вам нужно <em>chapter/*[name()=$N]</em>. Если переменная содержит что-то более сложное, чем просто имя (например, запись xPath-пути), вам понадобится расширение, подобное saxon:evaluate(), чтобы вычислить это.</li>
  <li>Шаблонные правила <em>xsl:apply-templates</em> - это не расширенные возможности языка для подвинутых пользователей. Это самые основные, фундаментальные конструкции в XSLT. Не откладывайте тот день, когда вы начнёте их использовать. Если вы не используете их, вы делаете свою жизнь излишне сложной.</li>
  <li>XSLT принимает дерево на вход и отдаёт дерево на выходе. Непонимание этого является причиной многих разочарований, которые возникают у новичков в XSLT. XSLT не может обработать вещи, которые не представлены в дереве, предоставленном XML-парсером (области CDATA, ссылки на сущности (entity), XML-декларация), и не может сгенерировать эти вещи на выходе. Если вы думаете, что вам это необходимо, спросите "почему?". Возможно что-то не так с вашими требованиями или замыслом.</li>
  <li>Пространства имён (namespace) - это трудно. Нет лёгкого способа опровергнуть это. Возможно, это требует отдельной статьи. Разгадка в понимании модели пространств имён. Пространства имён проявляются в двух ипостасях:
<ol>  <li>Каждый элемент или атрибут имеет имя, состоящее из префикса, собственно имени и URI.</li>
  <li>Элементы обладают узлами пространств имён, представляющими все префикс-uri соответствия в границах этого элемента.</li>
</ol>
Когда вы поймёте это, вы сможете понять особенности различных правил и их влияние на пространства имён в результирующем дереве. Чаще всего, всё, что вам нужно делать, это гарантировать, что создаваемые вами элементы находятся в верном пространстве имён, всё остальное произойдёт само собой.
</li>
  <li>Не используйте <em>disable-output-escaping</em>. Некоторые люди используют эту магическую приправу, но не понимают, что она делает. Они надеются, что это может заставить код работать лучше. Этот атрибут только для профессионалов, и профессионалы используют это только как последнее средство спасения. В 95% случаях, если вы встретили в преобразовании <em>disable-output-escaping</em>, это говорит о том, что автор был новичком, не понимающим, что он делает.</li>
  <li>Инструкция <em>&lt;xsl:copy-of></em> создаёт точную копию исходного дерева, пространств имён и всего остального. (Есть исключение - в XSLT 2.0 вы можете сказать <em>copy-namespaces="no"</em>). Если вы хотите скопировать дерево с изменениями, вы не можете использовать <em>xsl:copy-of</em>. Вместо этого используйте шаблон идентичного преобразования: шаблон, который использует создание поверхностной копии элемента и применяет <em>applies-templates</em> ко всем его потомкам, дополненный шаблонами, переопределяющими это поведение для отдельных элементов.</li>
  <li>Не используйте
[cc lang="xml"]
&lt;xsl:variable name="x">&lt;xsl:value-of select="y"/>&lt;/xsl:variable>
[/cc]
Вместо этого используйте
[cc lang="xml"]
&lt;xsl:variable name="x" select="y"/>
[/cc]
Последняя запись короче, более действенна при исполнении, и в любом случае она корректна.</li>
  <li>Когда вам нужно найти информацию, используйте ключи. Также как и в случае с шаблонами, не откладывайте изучение использования ключей и не выбрасыйте их из головы как "продвинутую" возможность. Это важнейший инструмент разработки. Поиск информации без использования ключей сродни забиванию гвоздей отвёрткой.</li>