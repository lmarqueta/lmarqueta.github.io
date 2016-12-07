---
layout: post
title:  Migración a Jekyll
excerpt: Migración a Jekyll
date:   2016-12-06
categories: blog
tags: [blog, jekyll, so-simple-theme, git]
---
Hace un tiempo migré la vieja web semiabandonada [marqueta.org] a Jekyll alojado en [GitHub]. Mola, porque es como aquéllas primeras webs escritas en vi(m) con plantillas y expresiones regulares, pero a lo bestia :)

Ahora aprovecho para hecer un poco de limpieza y cambiar el tema por uno basado en [so-simple-theme]. Poco a poco intentaré ir migrando posts antiguos. O no...

## Jekyll en Debian Jessie

En Debian la instalación es sencilla, pero hay que tener en cuenta que el tema en cuestión requiere Jekyll 3, que no es el que empaqueta Debian 8, así que hay que instalarlo como _gema_:

```shell
sudo apt-get install ruby-full build-essential
sudo gem install jekyll
```

Hay abundante documentación de [Jekyll docs][jekyll] para el que esté interesado, aunque con lo justito vale para crear una página estática básica. Yo no necesito más.

## So-simple-theme: ejemplos

Esto es lo básico para formatear con Jekyll. En las fuentes hay mucho más:

# Heading 1

## Heading 2

### Heading 3

#### Heading 4

##### Heading 5

###### Heading 6

### Body text

Lorem ipsum dolor sit amet, test link adipiscing elit. **This is strong**. Nullam dignissim convallis est. Quisque aliquam.

![Smithsonian Image]({{ site.url }}/images/sampleimg1.jpg)
{: .pull-right}

*This is emphasized*. Donec faucibus. Nunc iaculis suscipit dui. 53 = 125. Water is H<sub>2</sub>O. Nam sit amet sem. Aliquam libero nisi, imperdiet at, tincidunt nec, gravida vehicula, nisl. The New York Times <cite>(That’s a citation)</cite>. <u>Underline</u>. Maecenas ornare tortor. Donec sed tellus eget sapien fringilla nonummy. Mauris a ante. Suspendisse quam sem, consequat at, commodo vitae, feugiat in, nunc. Morbi imperdiet augue quis tellus.

HTML and <abbr title="cascading stylesheets">CSS<abbr> are our tools. Mauris a ante. Suspendisse quam sem, consequat at, commodo vitae, feugiat in, nunc. Morbi imperdiet augue quis tellus. Praesent mattis, massa quis luctus fermentum, turpis mi volutpat justo, eu volutpat enim diam eget metus.

### Blockquotes

> Lorem ipsum dolor sit amet, test link adipiscing elit. Nullam dignissim convallis est. Quisque aliquam.

## List Types

### Ordered Lists

1. Item one
   1. sub item one
   2. sub item two
   3. sub item three
2. Item two

### Unordered Lists

* Item one
* Item two
* Item three

## Tables

| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|----
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|=====
| Foot1   | Foot2   | Foot3   |
{: .table}

## Code Snippets

Syntax highlighting via Rouge

```css
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
```

Non Rouge code example

    <div id="awesome">
        <p>This is great isn't it?</p>
    </div>

## Buttons

Make any link standout more when applying the `.btn` class.

<div markdown="0"><a href="#" class="btn">This is a button</a></div>

[so-simple-theme]: https://mmistakes.github.io/so-simple-theme/
[marqueta.org]: http://marqueta.org
[GitHub]: https://github.com
[jekyll]: http://jekyllrb.com
