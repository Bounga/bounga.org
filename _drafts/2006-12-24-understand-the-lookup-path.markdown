---
layout: post
title: Understand the lookup path
category: ruby
tags: ['ruby', 'tips', 'meta-programming']
---

<div class="post">
<h2 id="p102" class="post-title">Ruby : comprendre le chemin de consultation des méthodes</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le dimanche, 24 décembre 2006, 15:41        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt">Lorsqu'un objet reçoit un message, l'interpréteur recherche une méthode du même nom définie dans la classe de l'objet, dans l'une de ses super-classes ou dans un module qui a été mixé dans l'une de ces classes (mixin).
<p>Mais savez-vous comment tout cela fonctionne ? Comment sont gérés les cas ambigus tels qu'une méthode définie à la fois dans la classe et le mixin. Quelle méthode sera exécutée par l'objet ?</p>
<p>Comme toujours, comprendre tous les détails du fonctionnement de <a href="http://www.ruby-lang.org">Ruby</a> vous permettra d'écrire du code plus propre, plus concis mais surtout plus puissant. Nous allons donc voir dans cet article comment fonctionne le chemin de consultation des méthodes.</p></div>
    
<div class="post-content"><p>Fort heureusement, ce processus de recherche est plutôt simple à comprendre.</p>
<p>Le plus simple pour expliquer la démarche suivie par l'interpréteur Ruby pour faire ses choix est tout simplement d'écrire du code simple avec quelques méthodes qui ne feront qu'afficher un message pour qu'on sache où l'on se trouve dans le code lors de son exécution.</p>
<p>Nous allons donc écrire un morceau de code qui regroupe tous les cas possibles :</p>
<pre>module M
def affichage
puts "méthode d'affichage dans le module M"
end
end
</pre><pre>class C
include M
end
</pre><pre>class D &lt; C
end
</pre><pre>objet = D.new
objet.affichage
</pre>
<p>On crée donc un objet de la classe D puis on appel sa méthode <code>affichage</code>. L'interpréteur Ruby voit que la méthode est inexistante dans la classe D, il regarde donc si cette classe mixe des modules, ce qui n'est pas le cas. L'interpréteur va donc regarder si la superclasse de D (C), implémente une telle méthode ce qui n'est toujours pas le cas. Il regarde donc si la classe C mixe un module, c'est le cas (M). L'interpréteur vérifie donc si ce module définit une méthode <code>affichage</code>, c'est le cas donc l'interpréteur l'utilise.</p>
<p>Nous venons de voir un exemple typique de cheminement de l'interpréteur dans le lookup path pour trouver une méthode qui correspond au message envoyé.</p>
<h2>Jusqu'où peut aller la recherche ?</h2>
<p>Eh bien, sachant que tout objet en Ruby est une instance ou un descendant de la classe <code>Object</code>, l'interpréteur pourra toujours remonter jusqu'à ce niveau, et même plus loin. En effet, la classe <code>Object</code> mixe le module <code>Kernel</code> qui est toujours vérifié en dernier. C'est d'ailleurs ce module qui nous donne accès à des méthodes comme <code>respond_to?</code> ou <code>send</code> depuis tous nos objets.</p>
<p>Si vous arrivez jusqu'au module <code>Kernel</code> et que votre méthode n'a toujours pas été trouvé, l'interpréteur abandonnera la recherche (puisqu'il n'a plus aucun endroit où chercher) et vous renverra une exception <code>NoMethodError</code>.</p>
<h2>Schéma du chemin de recherche des méthodes</h2>
<center><img src="../../../../../public/lookup_path.png" alt="" style="margin-top: 0; margin-right: auto; margin-bottom: 0; margin-left: auto; display: block; " title="Ruby Lookup Path, sep 2008" /><br /></center>
<p>Ce schéma vous donne le chemin le plus complet qui peut être parcouru. Si une méthode portant le même nom est définie plusieurs fois à différents niveaux, alors la définition rencontrée en premier sera utilisée.</p>
<h2>Oui mais si je mixe plusieurs modules qui ont les mêmes méthodes ?</h2>
<p>C'est toujours aussi simple, le principe de moindre surprise est là pour nous. Imaginons que l'on fasse :</p>
<pre>module M
def affichage
puts "Nous sommes dans M"
end
end
</pre><pre>module N
def affichage
puts "Nous sommes dans N"
end
end
</pre><pre>Class C
include M
include N
end
</pre><pre>obj = C.new
obj.affichage
# => "Nous sommes dans N"
</pre>
<p>C'est tout simplement la dernière définition de méthode qui prend le dessus et donc celle définie par le dernier module inclus, dans notre cas le module N.</p>
<h2>Et si je veux forcer la remontée dans le lookup path ?</h2>
<p>C'est possible ! Il suffit de passer par l'utilisation du mot-clef <code>super</code> qui indique à l'interpréteur de poursuivre sa recherche même s'il a trouvé une méthode correspondante. Cette astuce est particulièrement pratique lorsque vous avez une méthode de la classe parent qui fait quasiment tout le nécessaire mais ne traite pas quelques détails. Vous pouvez donc utiliser cette astuce pour faire les choses spécifiques puis appeler la méthode parent pour achever le travail comme dans l'exemple suivant :</p>
<pre>class Velo
attr_reader :roues, :places
</pre><pre>  def initialize()</pre><pre>    @roues = 2</pre><pre>    @places = 1
end
end
</pre><pre>class Tricycle
def initialize()
super
@roues = 3
end
end
</pre>
<p><code>super</code> nous permet donc d'initialiser notre tricycle comme un vélo (2 roues et 1 place) puis de faire ensuite les modifications nécessaires pour qu'il ait les caractéristiques d'un tricycle (3 roues). Cette technique nous évite des répétitions inutiles.</p>
<h2>Conclusion</h2>
<p>Il est donc important de bien comprendre comment l'interpréteur recherche une méthode donnée. Comprendre ces principes vous permet de mettre en place des structures de code plus puissantes et avec moins de redondances. Une fois ces principes intégrés, vous aurez la possibilité d'utiliser l'héritage et les modules de façon optimale puisque vous aurez toutes les clefs en main pour comprendre comment et pourquoi une méthode est sélectionnée plutôt qu'une autre.</p>
<p>Et maintenant, il ne me reste plus qu'une chose à faire, c'est vous souhaiter une merveilleux Noël !</p></div>

  </div>

