---
layout: post
title: Observe file creations
category: misc
tags: ['sysadmin', 'ruby', 'script']
---

  <div class="post">
    <h2 id="p58" class="post-title">Surveiller la création de fichiers</h2>
    
    <p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le mercredi,  8 juin 2005, 13:14        - <a href="../../../../../category/Developpement/index.html">Développement</a>
        - <a href="index.html">Lien permanent</a>
    </p>
    
    
    
          <div class="post-excerpt"><p>Il arrive parfois qu'on ait besoin d'être prévenu lorsqu'un nouveau fichier est créé dans un répertoire donné. On retrouve ce besoin, par exemple, lorsqu'on a un serveur <acronym>FTP</acronym> et qu'on veut être tenu au courantde l'ajout de fichiers dans le répertoire d'upload. Ça peut également être utile dans le cas d'un répertoire partagé où plusieurs utilisateurs sont en mesure d'écrire. C'est suite à ces besoins que j'ai décidé d'écrire un petit script qui remplirait cette tâche sans avoir recourt à des outils comme <a href="https://savannah.nongnu.org/projects/fam/" hreflang="en">FAM</a> bien trop puissant et trop lourd à mettre en place pour un besoin aussi limité.</p></div>
        
    <div class="post-content"><p>J'ai donc développé ce script qui devrait pouvoir tourner sous tous les systèmes disposants d'un interpréteur <a href="http://www.ruby-lang.org/" hreflang="en">Ruby</a>.</p>


<p>Il permet de choisir un répertoire à surveiller, d'exclure certains sous-répertoires à ne pas surveiller, de choisir une ou plusieurs adresses e-mail pour l'envoi des rapports, de choisir le serveur <acronym title="Simple Mail Tranfer Protocol">SMTP</acronym> à utiliser pour l'envoi des messages ainsi que le temps à attendre entre chaque vérification.</p>


<p>Il est disponible à cette adresse&nbsp;: <a href="http://stuff.bounga.org/scripts/20-monitor" title="http://stuff.bounga.org/scripts/20-monitor">http://stuff.bounga.org/scripts/20-...</a></p>


<p>Pour le moment, les différentes options sont à changer directement dans le code. Elles sont disponibles en début de fichier et facilement repérables. S'il s'avère que le script est utilisé par d'autres personnes et que le besoin d'options en ligne de commande se fait resentir, je les ajouterai.</p>


<p>En ce qui concerne l'utilisation, c'est tout simple. Vous pouvez lancer le script des façons suivantes&nbsp;:</p>
<ul>
<li><code>./monitor.rb</code>&nbsp;: auquel cas le répertoire surveillé sera celui en cours</li>
<li><code>./monitor.rb /mon/rép/à/surveiller/</code>&nbsp;: et dans ce cas <code>/mon/rép/à/surveiller/</code> sera surveillé quelque-soit le répertoire depuis lequel vous lancez le script.</li>
</ul>

<p>Maintenant à chaque ajout de fichiers dans les répertoires surveillés, les adresses indiquées lors de la configuration recevront un mail de notification.</p>


<p>En espérant que cet utilitaire, sous licence <a href="http://www.gnu.org/licenses/gpl.html" hreflang="en">GPL</a> comme à l'habitude, pourra vous servir. N'hésitez surtout pas à me donner des retours. <img src="../../../../../themes/default/smilies/wink.png" alt=";-)" class="smiley" /></p></div>

      </div>
