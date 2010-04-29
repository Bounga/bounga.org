---
layout: post
title: How to use infinity in your code
category: ruby
tags: ['ruby', 'tips']
---

<div class="post">
<h2 id="p103" class="post-title">Ruby : utiliser la notion d'infini dans votre code</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le lundi, 25 décembre 2006, 11:39        - <a href="../../../../../category/Tips/index.html">Trucs et astuces</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>Il peut arriver dans certains cas que vous ayez besoin d'utiliser la notion d'infini dans votre code.</p>

<p>En <a href="http://www.ruby-lang.org">Ruby</a> c'est possible ! Il suffit de connaître l'astuce.</p></div>
    
<div class="post-content"><p>Supposons que vous avez une méthode <code>next_row</code> qui vous renvoie la ligne suivante d'un tableau calculée à la volée et que vous voulez pouvoir récupérer chaque ligne de ce tableau grâce à <code>each</code>. Il va vous falloir ré-écrire <code>each</code> pour qu'elle itére à l'infini sur votre tableau dynamique.</p>

<p>Voici comment faire :</p>

<p>
<pre>
def each
0.upto(1.0/0.0) { yield next_row }
end
</pre>
</p>

<p>Un rapide test dans IRB nous montre qu'effectivement l'expression <code>1.0/0.0</code> est évaluée comme l'infini :</p>

<p>
<pre>
MacBook:~ nico$ irb
>> 1.0/0.0
=> Infinity
</pre>
</p>

<p>Notre version de <code>each</code> va donc itérer à l'infini ! Magique <img src="../../../../../themes/default/smilies/smile.png" alt=":-)" class="smiley" /></p></div>

  </div>



            <div id="comments">
    <h3>Commentaires</h3>
  <dl>
      <dt id="c95" class=" odd first"><a
  href="#c95" class="comment-number">1.</a>
  Le mercredi, 27 décembre 2006, 15:12      par Underflow_</dt>
  
  <dd class=" odd first">

        
  <p>Plus simple, plus court :<br />
<br />
def each<br />
loop { yield new_row }<br />
end<br />
</p>
        </dd>
              <dt id="c96" class="  "><a
  href="#c96" class="comment-number">2.</a>
  Le mercredi, 27 décembre 2006, 17:00      par <a href="../../../../../index.html" rel="nofollow">Nicolas Cavigneaux</a></dt>
  
  <dd class="  ">

        
  <p>Oui <img src="../../../../../themes/default/smilies/smile.png" alt=":-)" class="smiley" /> mais on ne présente plus la notion d'infini ...</p>
        </dd>
      </dl>
  </div>