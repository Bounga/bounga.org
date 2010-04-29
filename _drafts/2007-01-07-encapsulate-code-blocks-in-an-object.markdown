---
layout: post
title: Encapsulate a code block in an object
category: ruby
tags: ['ruby', 'tips']
---

<div class="post">
<h2 id="p104" class="post-title">Ruby : encapsuler un bloc de code dans un objet</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le dimanche,  7 janvier 2007, 20:29        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>En <a href="http://www.ruby-lang.org">Ruby</a>, il est possible d'encapsuler un morceau de code dans un objet. Grâce à cela, il vous est possible d'utiliser ce même morceau de code à plusieurs endroits ou encore d'exécuter un morceau de code donné en fonction du contexte courant. En somme, toute la puissance des blocs embarqués dans un objet.</p>

<p>Cette possibilité est offerte par les objets de type <code>Proc</code>. Nous essayerons de voir dans ce billet comment mettre à profit l'utilisation des <code>Proc</code> pour simplifier votre code et le rendre un peu plus dynamique.</p></div>
    
<div class="post-content"><h2>Comment créer un Proc ?</h2>

<p>Rien de plus simple :</p>

<p>
<pre>
proc = Proc.new { |x| puts "Le paramètre est #{x}" }
proc.call('test') # => Le paramètre est test
</pre>
</p>

<p>On a donc ici créé une nouvelle instance de la classe <code>Proc</code> en utilisant la méthode <code>new</code> à laquelle on a passé un bloc de code qui sera utilisé lors de l'appel à la méthode <code>call</code>. Nous avons, au début de notre bloc, définie une variable qui pourra être affectée lors de l'appel et utilisée par notre bloc.</p>

<p>Il est également possible de créer des objets <code>Proc</code> de façon automatique dans les appels de méthode :</p>

<p>
<pre>
def methode_avec_bloc(x, &amp;block)
puts block.class
x.times { |i| block[i, i*i] }
end

methode_avec_bloc(4) { |n, c| puts "Le carré de #{n} est #{c}" }

# => Proc
# => Le carré de 0 est 0
# => Le carré de 1 est 1
# => Le carré de 2 est 4
# => Le carré de 3 est 9
</pre>
</p>

<p>On a donc définie une méthode qui prend deux paramètres, un entier et un bloc a exécuter. Notre méthode affiche la classe de notre bloc pour s'assurer que c'est bien un <code>Proc</code> qu'on manipule puis renvoit un entier et son carré au bloc de code <code>x</code> fois. Lorsque nous appelons notre méthode, nous lui passons en premier paramètre le nombre d'itérations à faire puis un bloc (manipulant deux paramètres) qui se charge de l'affichage.</p>

<h2>Oui mais si je stocke déjà mon code dans un objet Proc ?</h2>

<p>Il est également possible de passer directement un objet <code>Proc</code> à une méthode attendant un bloc en précédant son nom de variable par un &amp; :</p>

<p>
<pre>
proc = Proc.new { |n| puts n }
3.times(&amp;proc)

# => 0
# => 1
# => 2
</pre>
</p>

<h2>Oui mais où sont les exemples utiles ?</h2>

<p>Vous êtes exigeants en plus ! Bon d'accord &hellip; Disons que vous écrivez une application en <a href="http://www.rubyonrails.org">Rails</a> et qu'il vous faut pouvoir trier une liste d'articles par titre, auteur, identifiant décroissant ou par statut (publié ou non). L'utilisation des <code>Proc</code> pourrait bien vous simplifier la vie.</p>

<p>Vous pourriez par exemple utiliser le morceau de code suivant pour éviter les duplications :</p>

<p>
<pre>
def sort
order = params[:order] || "id"
proc = case order
when "author" then lambda { |a| a.user.name.downcase }
when "status", "title" then lambda { |a| a.send(order) }
when "id" then lambda { |a| -a.id }
end
@articles = Article.find(:all).sort_by &amp;proc
end
</pre>
</p>

<p>Notre méthode <code>sort</code> définie une variable <code>order</code> qui contient le paramètre de tri (récupéré dans un formulaire), si aucun paramètre n'a été donné, l'id sera utilisé par défaut. Ensuite la variable <code>proc</code> se voit affecter l'un des trois <code>lambda</code> (un équivalent de <code>Proc</code>) en fonction de l'ordre demandé. Dans le cas où le paramètre de tri est l'auteur, on utilise le champ <code>name</code> en minuscules, pour le status et le titre, on utilise directement le champ en question, sans traitement. C'est pour cela qu'on utilise <code>send</code> qui nous évite de la duplication en appelant la méthode de façon dynamique. Si on fait un tri sur l'identifiant, il nous suffit de faire appel au champ en question en prenant soin de le trier en ordre inverse à l'aide du "-".</p>

<p>Une fois la fonction de tri choisie, on recherche tous les articles disponibles puis on les trie en passant notre <code>Proc</code> à la méthode <code>sort_by</code> qui attend un bloc.</p>

<h2>Conclusion</h2>

<p>Ecrire ce genre de code n'a rien de spectaculaire mais vous permet d'écrire du code plus concis et plus évolutif. Les <code>Proc</code> peuvent être d'une aide très précieuse et vous permettront dans de nombreux cas de limiter le nombre de méthodes à écrire pour des actions quasiment identiques où seule une ou deux opérations de traitement varient.</p></div>

  </div>
