---
layout: post
title: Simple web gallery generator
category: misc
tags: ['ruby', 'html', 'gallery', 'generator', 'script']
---

  <div class="post">
    <h2 id="p42" class="post-title">Un script ruby pour créer un page web contenant vos images</h2>
    
    <p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le vendredi,  4 février 2005, 20:49        - <a href="../../../../../category/Developpement/index.html">Développement</a>
        - <a href="index.html">Lien permanent</a>
    </p>
    
    
    
          <div class="post-excerpt"><p>J'avais un besoin bien spécifique, pouvoir afficher toutes les images d'un répertoire suivies de leur nom. J'ai donc décidé de développer un petit script en <a href="http://www.ruby-lang.org" hreflang="en">Ruby</a> qui se chargerait de générer une page <acronym>HTML</acronym> pour moi.</p></div>
        
    <div class="post-content"><p><a href="http://stuff.bounga.org/scripts/21-img2html">img2html.rb</a> permet donc de générer une page <acronym>XHTML</acronym> valide et facilement adaptable puisqu'elle utilise les <acronym>CSS</acronym> pour gérer l'aspect de la page.</p>


<p>Par défaut, le répertoire utilisé est le répertoire en cours, six images sont affichées par ligne, le titre de la page est <em>My pictures</em> et le fichier produit se nomme <em>index.html</em>. Bien évidemment tous ces paramètres sont modifiables directement depuis la ligne de commande. Pour plus d'informations, je vous propose d'essayer <code>img2html.rb --help</code>.</p>


<p>La page générée est en UTF-8.</p>


<p>J'espère que ce script <sup>[<a href="#pnote-42-1" id="rev-pnote-42-1">1</a>]</sup> pourra vous être utile :present: .</p>
<div class="footnotes"><h4>Notes</h4>
<p>[<a href="#rev-pnote-42-1" id="pnote-42-1">1</a>] distribué sous <a href="http://www.gnu.org/copyleft/gpl.html" hreflang="en">licence GPL</a></p></div>
</div>

      </div>
