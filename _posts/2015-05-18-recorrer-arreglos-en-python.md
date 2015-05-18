---
layout: post
title: "Recorrer arreglos en python"
description: ""
category: 
tags: [python, loops, listas]
---

Python es un lenguaje que me encata tiene una forma muy elegante en su sintaxis, algo que me agrada mucho es la forma en la que se
puede recorrer las listas.

Por ejemplo si se quiere recorrer una lista e ir obteniendo los valores contenidos se puede hacer lo siguiente:

{% highlight python %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-
lista = ['1','2','3','4','5','6','7']
for v in lista:
	print "valor %s" %v
{% endhighlight %}

y obtendremos:

{% highlight bash %}
valor 1
valor 2
valor 3
valor 4
valor 5
valor 6
valor 7
{% endhighlight %}

Ahora si lo que queremos es recorrer la lista y al mismo tiempo obtener el indice podemos usar la función enumerate:

{% highlight python %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-
lista = ['1','2','3','4','5','6','7']
for idx,v in enumerate(lista):
	print "indice %s valor %s" %(idx,v)
{% endhighlight %}

y obtendremos:

{% highlight bash %}
indice 0 valor 1
indice 1 valor 2
indice 2 valor 3
indice 3 valor 4
indice 4 valor 5
indice 5 valor 6
indice 6 valor 7
{% endhighlight %}

Como pueden ver es muy sencillo de recorres los arrays, ahora si tuvieramos 2 listas y los queremos recorrer al mismo tiempo, podemos
usar la función zip:

{% highlight python %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-
lista1 = ['1','2','3','4','5','6','7']
lista2 = ['uno','dos','tres','cuatro','cinco','seis','siete']
for v1,v2 in zip(lista1,lista2):
	print "v1 %s v2 %s" %(v1,v2)
{% endhighlight %}

y obtendremos:

{% highlight bash %}
v1 1 v2 uno
v1 2 v2 dos
v1 3 v2 tres
v1 4 v2 cuatro
v1 5 v2 cinco
v1 6 v2 seis
v1 7 v2 siete
{% endhighlight %}

Por el momento no he econtrado una forma mas sencilla de recorrer 2 listas e ir obteniendo el indice pero igual podemos hacer uso 
de una combinación de las funciones enumerate y zip:

{% highlight python %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-
lista1 = ['1','2','3','4','5','6','7']
lista2 = ['uno','dos','tres','cuatro','cinco','seis','siete']
for idx,v in enumerate(zip(lista1,lista2)):
	print "idx %s v1 %s v2 %s" %(idx, v[0], v[1])
{% endhighlight %}

y obtendremos:

{% highlight bash %}
idx 0 v1 1 v2 uno
idx 1 v1 2 v2 dos
idx 2 v1 3 v2 tres
idx 3 v1 4 v2 cuatro
idx 4 v1 5 v2 cinco
idx 5 v1 6 v2 seis
idx 6 v1 7 v2 siete
{% endhighlight %}


Bueno por ultimo quiero comentar que estos casos se aplican perfectamente a las tuplas.
{% include JB/setup %}
