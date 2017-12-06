---
layout: post
title:  "Rails: Como acessar o contador em um renderização de coleção."
date:   2017-12-01 21:00:00
categories: rails
---

Olá, hoje vou deixar aqui uma pequena dica, mas acho que de grande ajuda.

Como acessar o “contador” em um renderização de coleção.

As vezes me deparo com essa situação, pois tenho que adicionar os “rows” das colunas em um renderização html utilizando o bootstrap.

Então vamos lá, suponha que você tenha uma coleção com 8 artigos e que deseja que sejam exibidos 4 artigos por linha e que no loop do 5 artigo (ou início de cada linha) se adicione uma div “.row” para que o layout não “quebre”, então vamos ao exemplo código sem o contador e com o contador.

***View “index.html.erb”***

Na view chamamos a coleção dos objetos, podemos fazer de duas formas.

A primeira informando o partial e passando a coleção.

{% highlight ruby %}
<div class="row"><%= render partial: 'article', collection: @articles %></div>
{% endhighlight %}

Ou na segunda, simplesmente passando a coleção, assumindo que “@articles” é um coleção de “Article”, então se poderia definir assim.

{% highlight ruby %}
<div class="row"><% render @articles %></div>
{% endhighlight %}

***Partial “_article.html.erb”***

Exemplo antes do uso do “counter”.

{% highlight ruby %}
<div class="col-md-3">
<h3><%= article.title %></h3>
<%= truncate(article.text) { link_to "Ver mais", article }
</div>
{% endhighlight %}

Da forma como está acima ele poderia ficar alinhado 4 colunas e na quinta ele cair, mas imagine que um título ou texto tenha um tamanho diferente, poderia acabar “quebrando” a visualização, então nesse caso podemos utilizar o contador do laço criado na coleção. 
O Rails fornece uma variável chamada “#{partial_name}_counter” para se utilizar no partial, em nosso exemplo essa varíavel seria “article_counter”, lembrando que inicia que o valor começa com 0.

Então vamos corrigir nosso partial ***_article.html.erb***.

{% highlight ruby %}
<%= '</div><div class=".row">'.html_safe if article_counter % 4 == 0 %>

<div class="col-md-3">
<h3><%= article.title %></h3>
<%= truncate(article.text) { link_to "Ver mais", article }
</div>
{% endhighlight %}

Chamando a variável “article_counter” podemos calcular o módulo por 4 e imprimir o fechamento e abertuda da div “.row”, sempre que se iniciar uma nova linha.

É isso, eu irei continuar publicando essas pequenas dicas sobre o que vou vendo no dia a dia com o Rails.

Valeu até a próxima.
