---
layout: post
title: Yield and code blocks
category: ruby
tags: ['ruby', 'tips', 'yield']
---

<div class="post">
<h2 id="p101" class="post-title">Ruby : yield et les blocs de code</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le dimanche, 17 décembre 2006, 23:31        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>Comme vous le savez sûrement, quand vous appelez une méthode vous pouvez lui fournir un bloc de code qui permet de définir son comportement. Ce bloc de code contient du code <a href="http://www.ruby-lang.org">Ruby</a> traditionnel. Vous l'avez très certainement déjà utilisé sous l'une des deux formes suivantes :
</p>

<p>
<pre>
objet.methode { #bloc de code }	

objet.methode do
# bloc de code
end
</pre>
</p>

<p>Un exemple d'utilisation courante dans la vie d'un programmeur Ruby peut-être le suivant :</p>

<p>
<pre>
(1..10).find_all { |i| i % 3 == 0 } # => [3, 6, 9]
</pre>
</p>

<p>Ici, à l'aide de <code>find_all</code>, on parcourt les entiers de 1 à 10 pour trouver ceux qui sont des multiples de 3. Le comportement de <code>find_all</code> est définit par notre bloc de code placé entre les accolades.</p>

<p>Cette construction permet des utilisations très puissantes et cela grâce à <code>yield</code>. Nous allons donc voir dans cet article à quoi sert <code>yield</code> et comment l'utiliser pour rendre vos méthodes plus dynamiques.</p></div>
    
<div class="post-content"><h2>Alors ça fait quoi yield ?</h2>

<p>Si vous fournissez un bloc de code quand vous appelez une méthode, alors au sein de cette méthode il sera possible de prendre le contrôle de ce bloc de code grâce à <code>yield</code>. Votre méthode sera exécutée normalement, une fois le mot-clef <code>yield</code> rencontré, le bloc de code fourni sera exécuté et finalement les instructions de la méthode placées après le <code>yield</code> seront à leur tour exécutées.</p>

<p>Pour vous faire plus facilement une représentation de ce mécanisme, vous pouvez penser <code>yield</code> comme une sorte de callback. Lorsque que le mot-clef <code>yield</code> est rencontré, l'interpréteur sait qu'il doit stopper temporairement l'exécution du corps de la méthode pour aller exécuter le bloc de code qui a été passé juste derrière l'appel de méthode.</p>

<p>Prenons un exemple pour mieux comprendre :</p>

<p>
<pre>
def test_de_yield
puts "On entre dans le corps de notre méthode"
puts "Nous allons maintenant appeler yield"
yield
puts "Nous sommes revenus dans le corps de la méthode"
end

test_de_yield { puts "Un petit coucou depuis le bloc de code !" }
</pre>
</p>

<p>L'exécution de cet exemple nous donne la chose suivante en résultat :</p>
<p>
<pre>
On entre dans le corps de notre méthode
Nous allons maintenant appeler yield
Un petit coucou depuis le bloc de code !
Nous sommes revenus dans le corps de la méthode
</pre>
</p>

<p>On voit donc que lorsqu'on appelle la méthode <code>test_de_yield</code>, l'interpréteur fait un saut dans cette même méthode, affiche nos deux premières lignes puis rencontre le mot-clef <code>yield</code>. L'interpréteur va donc comprendre qu'il doit maintenant exécuter le bloc de code qui a été passé à la méthode. Une fois ce bloc terminé, l'interpréteur retourne dans le corps de la méthode, juste après le <code>yield</code>.</p>

<h2>On peut passer des arguments à un bloc de code ?</h2>

<p>Oui !</p>

<p>Les blocs de code ont de nombreux points communs avec les méthodes. Ce sont tout deux des suites d'instructions qui peuvent être exécutés et qui peuvent attendre des arguments.</p>

<p>Pour envoyer des arguments à un bloc de code, il faut les passer à <code>yield</code>. <code>yield(1,2)</code> vous montre comment passer deux arguments à votre bloc de code.</p>

<p>Le bloc qui est exécuté par notre <code>yield</code> peut donc utiliser ces arguments. Il existe pour cela une syntaxe particulière qui utilise les pipes (||) comme dans l'exemple ci-dessous :</p>

<p>
<pre>
methode_yield do |a, b|
# le code ici
end 
</pre>
ou
<pre>
methode_yield { |a,b| # le code ici }
</pre>
</p>

<h3>Comment utiliser yield et les arguments ?</h3>

<p>Prenons un exemple qui ne sert pas à grand chose mais qui a l'avantage de présenter un morceau de code utilisant <code>yield</code> et les arguments :</p>

<p>
<pre>
def yield_aleatoire
yield(rand(10))
end

yield_aleatoire { |x| puts "Le nombre généré est #{x}"}
</pre>
</p>

<p>Notre méthode tire un nombre aléatoire (entre 0 et 9) puis la passe à <code>yield</code>. Ensuite, dans notre appel de méthode, nous affectons ce nombre à la variable <code>x</code> grâce aux pipes puis nous l'affichons.</p>

<h2>Mais comment faire si je veux renvoyer une valeur à ma méthode depuis mon bloc de code ?</h2>

<p>Il est vrai que les blocs auraient un usage limité s'il n'était pas possible de leurs faire renvoyer une valeur. Fort heureusement, un bloc de code est capable de renvoyer une valeur, cette valeur est celle qui est évaluée en dernier dans le bloc. Ainsi dans la méthode, la valeur de retour du bloc est en fait la valeur de retour de <code>yield</code>. Voyons un exemple :</p>

<p>
<pre>
def retour_yield
resultat = yield(3)
puts "Le résultat obtenu est : #{resultat}."
end

retour_yield { |x| x + 2 }  # => Le résultat obtenu est : 5
</pre>
</p>

<p>Notre bloc reçoit un argument, l'assigne à la variable <code>x</code> et retourne le résultat de <code>x + 2</code> (rappelez vous, en Ruby, la valeur de retour est toujours le résultat de la dernière expression évaluée). Ce résultat est ensuite récupéré dans notre méthode en tant que valeur de retour de <code>yield</code>, ce qui nous permet ensuite de l'afficher.</p>

<h2>Mais un seul yield c'est nul ! Je peux itérer ?</h2>

<p>Et là encore, je vous répond oui !</p>

<p>Toutes les méthodes qui utilisent <code>yield</code> et les blocs sont appelées itérateurs et ce n'est pas pour rien :-). En toute logique, un itérateur est créé pour faire de la répétition, travailler sur une liste ou une collection d'objets. En pratique certaines méthodes qui utilisent <code>yield</code> ne font pas de répétition mais la grande majorité de ces méthodes en font. On peut donc utiliser ces méthodes pour traiter une liste de valeurs et retourner un résultat après traitement de ces valeurs.</p>

<p>Voyons un exemple :</p>

<p>
<pre>
def places_bus(objets)
objets.each do |o|
resultat = yield(o)
if resultat ==  true
  puts "Le bus est pas disponible pour #{o} personnes"
else
  puts "Le bus n'est pas disponible pour #{o} personnes"
end
end
end

USAGERS = [10, 4, 34, 8, 890, 54]
places_bus(USAGERS) { |u| u < 40 }
</pre>
</p>

<p>Que faisons-nous dans cet exemple ? Nous avons écris une méthode <code>places_bus</code> qui prend un argument. L'argument attendu est un tableau d'objets. Chaque objet est parcouru puis envoyé à notre bloc grâce à <code>yield</code>. Nous stockons la valeur de retour de yield (qui doit être <code>true</code> ou <code>false</code> pour faire un test et afficher un message différent en fonction du résultat de ce test).</p>
<p>Nous créons ensuite un tableau de nombres puis nous utilisons notre nouvelle méthode pour faire un test simple, si le nombre est supérieur à 40 nous renvoyons <code>false</code>, sinon <code>true</code>. On a donc maintenant un itérateur simple et flexible qui nous permet de vérifier si un bus est assez spacieux pour nos voyageurs ou non. Nous pouvons mettre le test que nous voulons dans notre bloc de code du moment qu'il renvoit soit <code>true</code>, soit <code>false</code>.</p>

<p>Nous avons donc partager le travail entre notre méthode et notre bloc de code, le résultat est obtenu en sautant de l'un à l'autre. Cela peut vous paraître inutile mais le partage du travail entre la méthode et le bloc de code nous permet de gagner en flexibilité puisque nous pourrons faire des vérifications semblables avec une seule méthode en changeant simplement le bloc lors de l'appel de notre méthode.</p>

<p>Si demain vos bus ne prennent la route qu'à partir de 15 personnes, il vous suffira de changer le bloc pour faire vos vérifications et vous n'aurez pas à modifier la méthode :</p>

<p>
<pre>
USAGERS = [10, 4, 34, 8, 890, 54]
places_bus(USAGERS) { |u| u < 40 && u >= 15 }	
</pre>
</p>

<h2>Conclusion</h2>

Vous aurez vu ici que l'utilisation de yield et des blocs de code vous permet de mettre en place des méthodes très dynamiques et multi-usages. Pensez donc à <code>yield</code> lorsque vous souhaitez écrire une méthode assez généraliste, notamment pour une librairie.</div>

  </div>



            <div id="comments">
    <h3>Commentaires</h3>
  <dl>
      <dt id="c93" class=" odd first"><a
  href="#c93" class="comment-number">1.</a>
  Le mardi, 19 décembre 2006, 11:33      par <a href="http://www.typouype.org" rel="nofollow">pouype</a></dt>
  
  <dd class=" odd first">

        
  <p>Merci Bounga !!! <br />
C'est nickel comme explication, certain point m'échapais encore <img src="../../../../../themes/default/smilies/wink.png" alt=";-)" class="smiley" /></p>
        </dd>
              <dt id="c94" class="  "><a
  href="#c94" class="comment-number">2.</a>
  Le vendredi, 29 décembre 2006, 21:14      par Noé</dt>
  
  <dd class="  ">

        
  <p>et l'alternative :<br />
<br />
def plop(&amp;mon_super_block)<br />
mon_super_block.call(mes_arguments)<br />
end<br />
</p>
        </dd>
      </dl>
  </div>