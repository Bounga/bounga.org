---
layout: post
title: Errors handling using Exceptions
category: ruby
tags: ['ruby', 'exception']
---

<div class="post">
<h2 id="p97" class="post-title">Ruby: gérer les erreurs à l'aide des exceptions</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le dimanche,  3 décembre 2006, 19:47        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>Lorsqu'on développe à l'aide d'un langage objet, comme <a href="http://www.ruby-lang.org">Ruby</a>, il est courant d'avoir recourt aux exceptions pour traiter les erreurs. Malheureusement de nombreux développeurs ne sont pas à l'aise avec ce type de traitement et continuent à utiliser les tests conditionnels (if / case / etc) pour traiter les erreurs inattendues alors que ces tests sont prévus pour la logique du code plutôt que pour la gestion des erreurs.</p>
<p>Nous allons donc voir pourquoi et comment les exceptions peuvent simplifier votre code.</p></div>
    
<div class="post-content"><h2>Les exceptions ? Qu'est-ce que c'est ?</h2>
<p>Une <em>exception</em> est un type spécial d'objet, une instance de la classe <code>Exception</code> ou un descendant de cette classe. Lancer une exception (<code>raise</code>) sert à interrompre le chemin normal d'exécution du programme pour traiter le problème ou quitter le programme.</p>

<p>Le choix par l'interpréteur Ruby de quitter ou non le programme dépendra du fait que vous avez écrit ou non un bloc <code>rescue</code> pour le morceau de code qui a levé l'exception. Si vous n'avez pas fourni ce bloc, le programme quittera sinon l'interpréteur entrera dans votre bloc <code>rescue</code> pour prendre les mesures nécessaires.</p>

<p>Un exemple simple pourrait être : <code>1/0</code> qui lévera une exception du type <code>ZeroDivisionError</code> puisque vous avez tenté de diviser un nombre par zéro ce qui mathématiquement impossible. <code>ZeroDivisionError</code> est le nom d'une classe particulière d'exception qui hérite de la classe <code>Exception</code>.</p>

<p>Voici une liste des exceptions qu'on rencontre le plus souvent. Elles descendent toutes de la classe <code>Exception</code>.</p>

<table border="1" cellspacing="5" cellpadding="5">
<tr>
	<th>Nom de l'exception</th>
	<th>Raison</th>
	<th>Exemple</th>
</tr>
<tr>
	<td><code>RuntimeError</code></td>
	<td>Levé par défaut par <code>raise</code></td>
	<td><code>raise</code></td>
</tr>
<tr>
	<td><code>NoMethodError</code></td>
	<td>Un message inconnu a été envoyé à un objet</td>
	<td><code>Object.new.methode_inconnue</code></td>
</tr>
<tr>
	<td><code>NameError</code></td>
	<td>Un mot n'a pu être résolu en tant que variable ou nom de méthode</td>
	<td><code>variable_inconnue</code></td>
</tr>
<tr>
	<td><code>IOError</code></td>
	<td>Causé par la lecture d'un flux fermé, l'écriture sur un flux en lecture seule, etc</td>
	<td><code>STDIN.puts("Impossible d'écrire sur STDIN")</code></td>
</tr>
<tr>
	<td><code>TypeError</code></td>
	<td>Une méthode a reçue un argument qu'elle ne peut pas gérer</td>
	<td><code>3 + "un chaîne ..."</code></td>
</tr>
<tr>
	<td><code>ArgumentError</code></td>
	<td>Levé par l'utilisation d'un nombre incorrect d'arguments</td>
	<td><code>def m(x); end; m(1, 2, 3)</code></td>
</tr>
</table>

<h2>rescue est là pour vous aider !</h2>
<p>Comme je vous l'ai expliqué avant, lever une exception ne revient pas forcément à quitter le programme. Vous pouvez traiter les exceptions en solutionnant les problèmes ce qui vous permet de continuer l'exécution du programme. Pour cela vous devez utiliser le mot-clef <code>rescue</code>.</p>

<p>
Deux méthodes vous permettent de gérer vos exceptions : 
<ul>
	<li>Entourer le code à protéger des mots-clefs <code>begin</code>/<code>end</code></li>
	<li>Protéger une méthode compléte en ajoutant un bloc <code>rescue</code> à la fin de la définition de cette méthode</li>
</ul>
Voici un petit exemple qui vous protégera des divisions par zéro :
<p>
	<pre>
print "Veuillez entrer un chiffre : "
n = gets.to_i

begin
resultat = 100 / n
rescue
puts "Avez-vous tenté une division par zéro ?"
exit
end

puts "100/#{n} vaut #{resultat}."
	</pre>
</p>
</p>

<p>
Si vous tentez d'exécuter ce morceau de code en entrant, le programme va lever une exception de type <code>ZeroDivisionError</code>, notre code étant dans un bloc <code>begin</code>/<code>rescue</code>, la bloc <code>rescue</code> sera appelé et notre message d'erreur affiché. Si vous entrez une valeur correcte le code sera exécuté normalement et le bloc <code>rescue</code> ne sera pas interprété. 
</p>

<p>Vous pouvez également utiliser une clause <code>rescue</code> qui ne gérera qu'un type précis d'exception. Dans notre cas, nous aurions pu utiliser <code>rescue ZeroDivisionError</code>. Dans ce cas les autres exceptions ne seront pas attrapées par notre bloc <code>rescue</code>.</p>

<h2>Et si je veux lever une exception dans mon code ?</h2>
<p>Utiliser le système d'exception vous permettra d'avoir un code plus flexible et compact. Il est donc tout naturel que vous puissiez lever des exceptions dans votre propre code et même créer de nouvelles exceptions.</p>

<p>Pour lever une exception, il vous suffit d'utiliser le mot-clef <code>raise</code>. Vous pouvez éventuellement lui passer le nom de l'exception que vous voulez lever et une chaîne qui sera utilisée comme message d'erreur.</p>

<p>Voici un exemple d'utilisation :</p>
<p>
<pre>
def methode_explosive(x)
raise ArgumentError, "Le chiffre est supérieur à 10 !" unless x < 10
end

methode_explosive(20)
</pre>
</p>

<p>On obtient donc : <em>ArgumentError: Le chiffre est supérieur à 10 !</em></p>

<p>Vous pouvez bien évidemment gérer cette erreur :</p>
<p>
<pre>
begin
methode_explosive(20)
rescue ArgumentError
puts "Ce chiffre est trop grand !"
end
</pre>
</p>

<p>On obtient maintenant : <em>Ce chiffre est trop grand !</em></p>

<p>Ici on ne traite que <code>ArgumentError</code> pour être sûr de ne faire notre traitement d'erreur que dans le cas précis d'une erreur d'argument. Les autres types d'erreurs ne seront pas traités et ne seront donc pas masqués par notre code de traitement des erreurs. Vous aurez compris qu'il faut éviter les <code>rescue</code> généralistes sous peine de vous retrouver avec des bugs introuvables puisque masqués.</p>

<p>Notez qu'il vous est possible de relancer une exception que vous avez traité. De cette façon, vous pourrez transmettre l'exception aux autres blocs <code>rescue</code> pour plus de traitement.</p>

<p>Dans notre exemple, on pourrait faire :</p>
<p>
<pre>
begin
methode_explosive(20)
rescue ArgumentError
puts "Ce chiffre est trop grand !"
raise
end
</pre>
</p>

<p>On obtient maintenant : <br/>
<em>
Ce chiffre est trop grand !<br/>
ArgumentError: Le chiffre est supérieur à 10 !</p>
</em>

<h2>D'accord, mais comment je crée mes propres exceptions ?</h2>
<p>Il peut vous arriver d'avoir à créer vos propres exceptions pour plus de clarté (c'est ce qui est fait très massivement dans <a href="http://www.rubyonrails.org/">Rails</a> par exemple). Il vous faudra donc créer une nouvelle classe d'exception héritant d'<code>Exception</code> ou de l'une de ses sous-classes.</p>

<p>En pratique, voilà comment on s'y prend :</p>
<p>
<pre>
class MonException < Exception
end

raise MonException, "Eh ! C'est ma nouvelle exception !"
</pre>
</p>

<p>L'un des avantages de créer vos propres classes d'exceptions, en plus de la clarté, est de pouvoir appeler vos exceptions par leur nom :</p>
<p>
<pre>
begin
puts "On va lever notre nouvelle exception"
raise MonException
rescue MonException => e
puts "Une exception a été levée : #{e}"
end
</pre>
</p>
inattendues 

<p>
Ce qui nous donnera :<br/>
<em>
On va lever notre nouvelle exception<br/>
Une exception a été levée : MonException
</em>
</p>

<h2>Conclusion</h2>
<p>Vous aurez donc compris qu'utiliser les exceptions vous permettra d'avoir un code plus flexible, qui sera capable de gérer des erreurs parfois difficilement récupérables autrement. Créer vos propres exceptions vous permettra également de mieux documenter votre code et d'afficher des messages plus précis aux utilisateurs. Alors n'hésitez plus et mettez vous aux exceptions <img src="../../../../../themes/default/smilies/wink.png" alt=";-)" class="smiley" /></p></div>

  </div>