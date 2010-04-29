---
layout: post
title: Generate a pretty URL from any string
category: ruby
tags: ['ruby', 'string', 'tips', 'url', 'seo']
---

<div class="post">
<h2 id="p80" class="post-title">Créer de belles URL à partir d'un texte en Ruby</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le samedi, 10 juin 2006, 17:58        - <a href="../../../../../category/Tips/index.html">Trucs et astuces</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>Suite à une discussion sur #rubyonrails.fr@freenode.net, j'ai découvert un morceau de code sympa et bien pratique pour qui veut construire une <acronym title="Uniform Resource Locator">URL</acronym> qui tient la route à partir d'un titre ou d'un texte.</p></div>
    
<div class="post-content"><pre><code>
require 'iconv'

def build_nice_url(str)
str = Iconv.new('us-ascii//TRANSLIT', 'utf-8').iconv(str).strip.downcase
str = str.gsub(/[^a-z0-9\-\.,\*]/, '-').gsub(/([\-\.,\*]){2,}/, '\1').gsub(/^[^a-z0-9]|[^a-z0-9]$/, '')
end
</code></pre>


<p>petite explication pour les néophytes&nbsp;:</p>
<ul>
<li>la chaîne passée en argument est convertie en chaîne <acronym title="American Standard Code for Information Interchange">ASCII</acronym> valide grâce à <a href="http://www.ruby-doc.org/stdlib/libdoc/iconv/rdoc/index.html" hreflang="en">ICONV</a></li>
<li>les espaces sont enlevés</li>
<li>la chaîne est passée en minuscules</li>
<li>tous les caractères qui ne sont pas
<ul>
<li>des lettres de l'alphabet</li>
<li>des chiffres</li>
<li>-</li>
<li>.</li>
<li>, </li>
<li>* </li>
</ul>
sont transformés en "-"</li>
<li>les caractères 
<ul>
<li>-</li>
<li>.</li>
<li>,</li>
<li>*</li>
</ul>
qui se répétent sont réduits à un seul, par exemple "salut-,comment ça va" devient "salut,comment-ca-va"</li>
<li>et finalement on enlève les caractères non-pertinents qui pourraient se trouver en début et fin de chaîne</li>
</ul>

<p>Un petit exemple d'utilisation maintenant&nbsp;:</p>

<code> build_nice_url("$*mon texte avec des virgules, points., accent éàû et répétition----hop !--")<br />
=&gt; "mon-texte-avec-des-virgules-points-accent-eau-et-repetition-hop"</code>


<p>Il ne vous reste plus qu'à intégrer ça dans vos applications <a href="http://www.ruby-lang.org" hreflang="en">Ruby</a>(<a href="http://www.rubyonrails.org" hreflang="en">OnRails</a>) <img src="../../../../../themes/default/smilies/wink.png" alt=";-)" class="smiley" /></p>


<p>Vous noterez toutefois que ce code ne fonctionne correctement avec les caractères accentués qu'avec un interpréteur Ruby &gt;= 1.8.4.</p></div>

  </div>