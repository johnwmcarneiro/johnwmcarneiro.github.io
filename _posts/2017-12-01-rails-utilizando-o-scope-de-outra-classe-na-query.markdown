---
layout: post
title:  "Rails: Utilizando o scope de outra classe na query."
date:   2017-12-01 21:00:00
categories: rails
---

Hoje tive uma situação onde eu tinha a model “A” e fazia um join na model “B” e precisava utilizar um scope de B (para não ter quer duplicar código), então cheguei no método “Merge” do ActiveRecord, abaixo vou exemplificar como utilizar.

Imagine que tenhamos duas models, a primeira Offer (Oferta) e a segunda Lot (lote).

Onde:
* Cada oferta pode ter muitos lotes.
* O período de disponibilidade é definido em Lote.

Teríamos mais ou menos o seguinte cenário.

{% highlight ruby %}
# Fields
# id
# name (string)
# status (integer)
class Offer < ApplicationRecord
  enum status: [:active, :inactive]  
  has_many :lots
end

# fields
# id (int)
# offer_id (int)
# start_date (datetime)
# end_date (datetime) 
class Lot < ApplicationRecord
  belongs_to :offer

  scope :available, -> { where('? BETWEEN lots.start_date AND lots.end_date', Time.current) }
end
{% endhighlight %}

Agora imagine que deseja buscar todas as ofertas ativas que possuam lotes, sendo assim facilmente conseguiríamos com o código abaixo:

{% highlight ruby %}
@offers = Offer.active.joins(:lots).group(:id)
{% endhighlight %}

Mas agora imagine que desejamos buscar todas as ofertas ativas, que tenha ligação com os lotes e que estes lotes estejam disponíveis. 
Como faremos? 
E se tentarmos chamar o scope “available” de “Lot” na query como no exemplo abaixo, o que aconteceria?

{% highlight ruby %}
@offers = Offer.active.joins(:lots).group(:id).available
{% endhighlight %}

Provavelmente irá gerar um erro como esse “undefined method `only_available’ for <#Offer…”, não é mesmo? 
Obviamente o método não está disponível para a model “Offer”, sendo assim como podemos fazer?

Então nesse caso podemos utilizar o método “merge”, ela permitir que façamos essa interseção. Então vamos para o código...

{% highlight ruby %}
@offers = Offer.active.joins(:lots).merge(Lot.available).group(:id)
{% endhighlight %}

Assim eles irá gerar uma SQL mais ou menos assim.

{% highlight sql %}
SELECT ... 
FROM offers 
INNER JOIN lots ... 
WHERE 'DATA_ATUAL' BETWEEN lots.start_date AND lots.end_date
AND offers.status = 0
GROUP BY offers.id
{% endhighlight %}

Agora percebe-se que ele gerou corretamente, pois ele lista as ofertas com lotes ligados, onde essa oferta deve ser ativa e a data atual deve estar dentro do período do lote.

Vale lembrar que utilizei o “GROUP BY” para evitar registros duplicados das ofertas, dado que ele faz um JOIN com os lotes e poderá ter mais de uma lote com período válido para cada oferta, gerando assim a duplicação de registros de oferta na listagem.

Você pode conferir o funcionamento do método “merge” através do link abaixo.

[http://api.rubyonrails.org/classes/ActiveRecord/SpawnMethods.html#method-i-merge](http://api.rubyonrails.org/classes/ActiveRecord/SpawnMethods.html#method-i-merge)

Até a próximo!