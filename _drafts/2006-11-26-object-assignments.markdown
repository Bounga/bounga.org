---
layout: post
title: Assignments explained
category: ruby
tags: ['ruby', 'tips']
---

<div class="post">
<h2 id="p96" class="post-title">Ruby : les affections d'objets aux variables</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le dimanche, 26 novembre 2006, 20:43        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>Cela fait maintenant quelques temps que je n'avais pas posté de billet mais le temps m'a manqué avec mon embauche chez <a href="http://www.webpulser.com/">webpulser</a>. J'ai décidé de faire une petite série d'articles sur <a href="http://www.ruby-lang.org/">Ruby</a> (<a href="http://www.rubyonrails.org">on Rails</a>) dans lesquels j'essayerai de vous donner des petites astuces et de vous faire comprendre des techniques avancées utilisées en Ruby et Rails.</p>

<p>Si le temps me le permet, je posterai un billet par semaine en abordant à chaque fois un sujet très précis qui vous permettra de mieux maîtriser votre code et donc d'être plus productif et efficace.</p>

<p>Les thèmes abordés seront très largement inspirés de mes différentes lectures, notamment <a href="http://rubyhacker.com/coralbook/">The Ruby Way</a>, <a href="http://www.manning.com/black/">Ruby For Rails</a> et <a href="http://www.pragmaticprogrammer.com/titles/fr_rr/">Rails Recipes</a>.</p>

<p>Cette semaine, nous commenceront en douceur puisque le billet sera dédié aux manipulations d'objets et plus particulièrement aux affections de ces objets à des variables.</p></div>
    
<div class="post-content"><p>La manipulation d'objets et leur affectations sont des opérations tellement courantes qu'on n'y prête même plus attention, pourtant si vous travaillez avec une quantité importante d'objets, il est nécessaire de maîtriser tous les détails des affectations pour optimiser vos programmes. Nous utiliserons un objet de type chaîne dans nos exemples car il est particulièrement adapté aux démonstrations.</p>
</p>

<h2>Un objet, plusieurs références</h2>

<p>Si nous écrivons le code suivant :</p>

<p>
<code>
str = "Bonjour"<br/>
abc = str<br/>
str.replace("Au revoir")<br/>
puts str => "Au revoir"<br/>
puts abc => "Au revoir"<br/>
</code>
</p>

<p>On affecte une chaîne à <code>str</code> puis on fait pointer <code>abc</code> sur <code>str</code>. On utilise ensuite la méthode <code>replace</code> sur notre objet <code>str</code> pour remplacer son contenu. Finalement on affiche la valeur de nos deux variable et on voit que les deux ont le même contenu. Pourquoi ? C'est tout simplement que nous disons à <code>abc</code> de faire référence à <code>str</code> et non pas de copier son contenu. Nos deux variables pointent donc sur le même objet et si l'on modifie le contenu de cet objet en passant par l'une des variables, l'autre variable nous retournera la version modifiée puisqu'elle fait référence au même objet.
</p>

<p>Vérifions :</p>

<p>
<code>
str.object_id => 271492<br/>
abc.object_id => 271492<br/>
</code>
</p>

<p>C'est donc confirmé les deux variables pointent sur un seul et même objet (identifiant identique). Les variables <code>abc</code> et <code>str</code> faisant référence au même objet, la consultation de leur contenu nous retourne la même valeur.</p>

<h2>Ré-affection de variables</h2>
<p>Considérons maintenant le code suivant :</p>
<p>
<code>
	str = "Bonjour"<br/>
	abc = str<br/>
	str = "Au revoir"<br/>
	puts str => "Au revoir"<br/>
	puts abc => "Bonjour"
</code>
</p>
<p>On voit ici que les deux variables ne nous affichent pas les mêmes valeurs. Pourquoi ? Simplement parce qu'on ne modifie pas un objet existant mais qu'on en crée un autre ! Lorsque l'on fait <code>str = "Au revoir"</code>, on crée un nouvel objet de type chaîne et on demande (via le = ) à <code>str</code> d'y faire référence. Les variables <code>str</code> et <code>abc</code> ne pointe donc plus sur le même objet à partir de cette ligne.</p>

<p>Vérifions :</p>

<p>
<code>
	str.object_id => 238212<br/>
	abc.object_id => 271492<br/>
</code>
</p>

<p>L'utilisation du "=" sur notre variable <code>str</code> a entraîné la création d'un nouvel objet et le pointage de <code>str</code> sur ce nouvel objet. La variable <code>abc</code> quant à elle pointe toujours sur l'ancien objet. On travaille donc maintenant sur deux objets différents.</p>

<p>Il est donc important de savoir que <code>str = "Autre chaîne"</code> est différent de <code>str.replace("Autre chaîne")</code>. On retombe dans le même cas avec <code>str + "Autre chaîne"</code> qui est bien différent de <code>str << "Autre chaîne"</code>. La première forme crée un nouvelle chaîne qui nous est retournée alors que la deuxième forme modifie l'objet sur lequel on travail.</p>

<h2>Conclusion</h2>
Vous aurez donc compris que chaque appel à "=" entraîne le pointage de la variable concernée vers un nouvel objet. L'ancien objet n'est donc plus pointé par cette variable (et s'il n'est plus pointé par aucune variable le Garbage Collector s'occupera de libérer l'espace mémoire qu'il occupe) et il ne peut donc plus être modifié via cette variable. Il faut donc savoir que si vous avez plusieurs variables dans votre code qui référencent le même objet, vous devrez passer par une méthode de cet objet pour modifier son contenu. Si vous passez par une ré-affection, alors la variable concernée ne pointera plus le même objet alors que toute les autres variables continueront à pointer vers l'ancien objet. Si vous n'y prêtez pas attention, vous pourrez alors facilement vous retrouver avec deux variables qui selon vous pointent le même objet mais qui en réalité ne vous retourneront pas les mêmes informations. Vous aurez donc le droit à une session chasse aux bugs <img src="../../../../../themes/default/smilies/wink.png" alt=";-)" class="smiley" /></div>

  </div>