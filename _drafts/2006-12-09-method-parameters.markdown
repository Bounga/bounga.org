---
layout: post
title: Method parameters
category: ruby
tags: ['ruby', 'tips']
---

<div class="post">
<h2 id="p99" class="post-title">Ruby : les paramètres de méthode</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le samedi,  9 décembre 2006, 21:29        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt">Savoir manipuler correctement les paramètres de méthode permet de facilement s'approprier une API existante mais aussi d'être en mesure de créer une API élégante, cohérente, compréhensible, puissante et facile à prendre en main. Pour arriver à ce résultat, il faut être conscient des différentes possibilités qui existent et de comment les utiliser.

Ce billet va tenter de vous expliquer tous les détails de la création d'une liste d'arguments.</div>
    
<div class="post-content"><h2>Les arguments obligatoires</h2>

<p>Les méthodes que vous écrivez en Ruby peuvent attendre un argument, plusieurs arguments ou aucun. Vos méthodes peuvent également attendre un nombre variable d'arguments</p>

<p>Lorsque vous appelez une méthode Ruby, vous devez absolument lui passer le bon nombre d'arguments sous peine de lever une exception. Si vous faîtes par exemple :</p>

<p>
<pre>
def ma_methode(x)
end

ma_methode(1,2,3,4)
</pre>
</p>

<p>alors l'interpréteur Ruby n'hésitera pas à vous lancer une exception <code>ArgumentError: wrong number of arguments (4 for 1)</code> pour vous rappeler que vous n'avez pas passé le bon nombre d'arguments à la méthode.</p>

<h2>Les arguments optionnels</h2>

<p>Il est possible d'écrire des méthodes auxquelles vous pourrez passer un nombre variable d'arguments. Pour faire cela, il faut mettre une étoile (*) devant le nom du dernier argument comme ici :</p>

<p>
<pre>
def methode_multi_arguments(*x)
end
</pre>
</p>

<p>Cette notation permet de dire à l'interpréteur que cette méthode peut être appelée avec un nombre quelconque d'arguments, y compris zéro. En utilisant cette technique, vous obtenez un tableau dans votre méthode. Ici <code>x</code> est donc un tableau que vous pouvez manipuler comme tel. Chaque élément du tableau contient l'un des objets que vous avez passé en paramètre lors de l'appel de méthode.</p>

<p>Vous pouvez également préciser de façon plus fine le nombre d'arguments qu'attend votre méthode en mixant arguments obligatoires et arguments optionnels, comme ici :</p>

<p>
<pre>
def deux_arguments_ou_plus(a, b, *x)
end
</pre>
</p>

<p>Dans cet exemple, les arguments <code>a</code> et <code>b</code> sont requis. Le dernier argument (<code>x</code>) quant à lui est optionnel. Vous pouvez ne rien lui passer, lui passer une valeur, ou plusieurs. Les arguments restants seront capturés et stockés dans un tableau nommé <code>x</code>.</p>

<h2>Les arguments avec valeur par défaut</h2>

<p>Vous pouvez rendre un argument optionnel en lui donnant une valeur par défaut. Si l'argument n'est pas renseigné lors de l'appel de méthode, la valeur par défaut sera utilisée. Voici un exemple :</p>

<p>
<pre>
def argument_par_defaut(a, b, c = 3)
puts "Les arguments passés sont : ", a, b, c
end

argument_par_defaut(1, 2)
</pre>
</p>

<p>qui nous donnera :</p>

<p>
<pre>
Les arguments passés sont :
1
2
3
</pre>
</p>

<p>On voit donc ici qu'aucune valeur n'a été donnée à l'argument <code>c</code> lors de l'appel de méthode et c'est donc la valeur par défaut qui a été utilisée à savoir 3. Si vous aviez passé une troisième valeur, celle-ci aurait été utilisée à la place de la valeur par défaut. Voyons un exemple :</p>

<p>
<pre>
argument_par_defaut(1, 2, 7)

Les arguments passés sont :
1
2
7
</pre>
</p>

<h2>Les arguments nommés</h2>

<p>Ce type d'argument est de plus en plus utilisé par les Rubyistes et a été très largement mis en avant par l'équipe de développement de <a href="http://www.rubyonrails.org">Ruby on Rails</a> puisque l'API de ce framework repose quasi-uniquement sur l'utilisation d'arguments nommés.</p>

<p>Les arguments nommés ont l'avantage de bien documenter l'utilisation d'une méthode et en facilite l'utilisation. Avec ces arguments, plutôt que de passer les arguments dans un ordre bien précis, on va les passer dans un ordre quelconque en précisant pour chaque argument, son nom. Ainsi, l'interpréteur sera capable d'associer les données à nos variables de façon cohérente.</p>

<p>Pour utiliser les arguments nommés, il faut avoir un minimum d'acquis à propos des tableaux associatif (<code>Hash</code>) et des symboles.</p>

<p>Le but de cette technique étant de permettre des appels de méthode comme suit :</p>

<p>
<pre>
afficher_news(:titre => "Mon titre", :auteur => "Nico", :contenu => "Ici le corps du texte ...")
</pre>
</p>

<p>On voit tout de suite le confort de lecture et d'utilisation que peuvent apporter les arguments nommés. Voyons donc comment les mettre en place :</p>

<p>
<pre>
def afficher_news(specs)
puts "Titre : #{specs[:titre]}" if specs[:titre]
puts "Auteur : #{specs[:auteur]}" if specs[:auteur]
puts "Contenu : #{specs[:contenu]}" if specs[:contenu]
end
</pre>
</p>

<p>Lorsque l'on utilise cette forme, Ruby comprend que l'on veut utiliser un <code>Hash</code>, il associe chaque valeur passée à notre variable en créant un <code>Hash</code>, puis en associant la valeur à la clef que l'on a spécifié. Dans notre méthode, nous n'avons plus qu'à parcourir ces clefs pour récupérer les valeurs et les traiter.</p>

<p>Notez toutefois que si votre <code>Hash</code> ne se trouve pas en dernière position dans la liste d'argument, vous devrez entourer vos symboles d'accolades pour spécifier à l'interpréteur que c'est bien un <code>Hash</code> que vous voulez utiliser.</p>

<p>Si vous utilisez cette façon de faire, quelques méthodes vous seront utiles, notamment <code>specs.has_key?(:la_clef)</code> qui permet de savoir si le <code>Hash</code> contient la clef passée en paramètre, <code>specs.empty?</code> qui permet de savoir si le <code>Hash</code> contient des clefs ou non et <code>specs.each_key { |k| puts k}</code> qui permet de parcourir chaque clef du <code>Hash</code>.</p>

<h2>L'ordre des arguments</h2>

<p>Si vous utilisez des arguments optionnels ou avec valeur par défaut, l'ordre dans lequel vous les passez est très important et il y a des règles à respecter.</p>

<p>Si vous incluez un argument optionnel (<code>*x</code>), il doit absolument venir après tous les autres arguments. Si l'argument optionnel était placé en plein milieu de la liste d'argument, il serait impossible pour l'interpréteur de savoir comment faire les affectations.</p>

<p>Pour clarifier les choses, je vous propose un tableau récapitulatif des signatures de méthode à utiliser pour mettre en place des arguments obligatoires, optionnels et avec valeur par défaut, tout ceci mixé de différentes façons.</p>

<p>
<table border="1" cellspacing="5" cellpadding="5">
	<tr>
		<th>Types d'arguments</th>
		<th>Signature</th>
		<th>Exemple d'appel</th>
		<th>Affectations</th>
	</tr>
	<tr>
		<td>Requis</td>
		<td><code>def m(a, b, c)</code></td>
		<td><code>m(1, 2, 3)</code></td>
		<td>a = 1, b = 2, c = 3</td>
	</tr>
	<tr>
		<td>Optionnels</td>
		<td><code>def m(*a)</code></td>
		<td><code>m(1, 2, 3)</code></td>
		<td>a = [1,2,3]</td>
	</tr>
	<tr>
		<td>Valeur par défaut</td>
		<td><code>def m(a=1)</code></td>
		<td><code>m(1, 2, 3)</code><hr/><code>m(2)</code></td>
		<td>a = 1<hr/>a = 2</td>
	</tr>
	<tr>
		<td>Requis / Optionnel</td>
		<td><code>def m(a, *b)</code></td>
		<td><code>m(1)</code><hr/><code>m(1, 2, 3)</code></td>
		<td>a = 1, b = []<hr/>a = 1, b = [2,3]</td>
	</tr>
	<tr>
		<td>Requis / Défaut</td>
		<td><code>def m(a, b=1)</code></td>
		<td><code>m(2)</code><hr/><code>m(2, 3)</code></td>
		<td>a = 2, b = 1<hr/>a = 2, b = 3</td>
	</tr>
	<tr>
		<td>Défaut / Optionnel</td>
		<td><code>def m(a=1, *b)</code></td>
		<td><code>m()</code><hr/><code>m(2)</code><hr/><code>m(2, 3, 4)</code></td>
		<td>a = 1, b = []<hr/>a = 2, b = []<hr/>a = 2, b = [3,4]</td>
	</tr>
	<tr>
		<td>Requis / Défaut / Optionnel</td>
		<td><code>def m(a, b=2, *c)</code></td>
		<td><code>m(1)</code><hr/><code>m(1, 3)</code><hr/><code>m(1, 3, 4, 5)</code></td>
		<td>a = 1, b = 2, c = []<hr/>a = 1, b = 3, c = []<hr/>a = 1, b = 3, c = [4, 5]</td>
	</tr>
</table>
</p>

<h2>Conclusion</h2>

<p>En maîtrisant bien les différents types d'arguments que vous pouvez passer à une méthode, vous aurez toutes les clefs en main pour créer des méthodes plus puissantes, plus dynamiques et plus génériques. Les arguments nommés vous permettront de créer une API facile d'accès pour vos utilisateurs. Les arguments optionnels vous donneront la possibilité de passer et traiter des listes d'arguments, séparés par des virgules ou sous forme de tableau. Un bon usage de ces techniques vous procurera beaucoup de souplesse, il est donc important de bien comprendre ces concepts.</p></div>

  </div>
