---
layout: post
title: Easily add verbose / silent mode to your software
category: ruby
tags: ['ruby', 'debug', 'tips']
---

<div class="post">
<h2 id="p74" class="post-title">Ajouter facilement les modes verbeux / silencieux à votre application Ruby</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le lundi, 20 mars 2006, 19:07        - <a href="../../../../../category/Tips/index.html">Trucs et astuces</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>Il est parfois intéressant d'ajouter à son application les modes verbeux / silencieux. Les utilisateurs peuvent ainsi contrôler facilement la quantité d'informations qu'ils souhaitent avoir à propos du fonctionnement de l'application. Ces modes sont tout particulièrement utiles lorsque vous utilisez des applications en mode texte. Si vous développez votre application en <a href="http://www.ruby-lang.org/" hreflang="en">Ruby</a>, vous pouvez tirer partie des fonctionnalités disponibles dans le langage pour mettre facilement en place ces modes.</p></div>
    
<div class="post-content"><p>Lorsqu'un interpréteur Ruby est lancé, il crée automatiquement un certain nombre de variables globales. Celle à laquelle nous nous intéresserons est <code>$VERBOSE</code>. Par défaut cette variable se voit affecter la valeur <code>false</code>.</p>


<p>La méthode <code>warn()</code><sup>[<a href="#pnote-86-1" name="rev-pnote-86-1">1</a>]</sup> de la librairie standard permet d'envoyer un message sur la sortie d'erreur standard<sup>[<a href="#pnote-86-2" name="rev-pnote-86-2">2</a>]</sup>. Cette méthode n'affiche la chaîne passée en argument que si la variable <code>$VERBOSE</code> est différente de <code>nil</code>. Pour activer notre mode verbeux il nous suffit donc d'utiliser <code>warn()</code> à la place de <code>puts()</code><sup>[<a href="#pnote-86-3" name="rev-pnote-86-3">3</a>]</sup> lorsqu'on veut afficher un avertissement, un message d'erreur, etc. Ensuite, à l'éxecution de notre programme, on changera le contenu de la variable <code>$VERBOSE</code>, selon que l'utilisateur veuille beaucoup d'informations ou non.</p>


<p>Concrétement, dans notre code, il n'y a que quelques étapes simples à suivre pour activer cette fonctionnalité. On considérera dans un premier temps que le comportement par défaut de notre programme est d'être silencieux. Dans notre code, avant tout affichage on ajoutera donc&nbsp;: <code>$VERBOSE = nil</code>. L'activation du mode verbeux pourra se faire à l'aide des options de la ligne de commande. On pourra utiliser, par exemple, la librairie <strong>OptParse</strong> qui est livrée en standard avec l'interpréteur. Dans ce cas, Vous pourrez procéder de la façon suivante&nbsp;:
<code>opts.on("-V", "--verbose", "Active le mode verbeux") { $VERBOSE = true }</code><sup>[<a href="#pnote-86-4" name="rev-pnote-86-4">4</a>]</sup></p>


<p>Vous voilà donc, en quelques lignes de code, avec un petit système de gestion des modes verbeux / silencieux qui pourra vous être très utile.</p>
<div class="footnotes"><h4>Notes</h4>
<p>[<a href="#rev-pnote-86-1" name="pnote-86-1">1</a>] <code>warn()</code> fait partie de la classe <strong>Kernel</strong></p>
<p>[<a href="#rev-pnote-86-2" name="pnote-86-2">2</a>] STDERR</p>
<p>[<a href="#rev-pnote-86-3" name="pnote-86-3">3</a>] On parle ici de <code>puts()</code> mais aussi de toutes les autres fonctions d'affichage</p>
<p>[<a href="#rev-pnote-86-4" name="pnote-86-4">4</a>] L'utilisation d'optparse est laissé comme exercice pour le lecteur <img src="../../../../../themes/default/smilies/wink.png" alt=";-)" class="smiley" /></p></div>
</div>

  </div>
