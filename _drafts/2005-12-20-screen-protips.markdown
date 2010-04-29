---
layout: post
title: Screen Protips
category: unix
tags: ['unix', 'command-line', 'screen', 'tips']
---

  <div class="post">
    <h2 id="p68" class="post-title">Screen : les bons tuyaux</h2>
    
    <p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le mardi, 20 décembre 2005, 23:05        - <a href="../../../../../category/Tips/index.html">Trucs et astuces</a>
        - <a href="index.html">Lien permanent</a>
    </p>
    
    
    
          <div class="post-excerpt"><p>Cela fait maintenant plusieurs années que j'utilise <a href="http://www.gnu.org/software/screen/" hreflang="en">Screen</a> dans mes consoles et je m'étonne encore de ses possibilités. Grâce à Screen, j'ai plusieurs fenêtres dans la même console, tous les raccourcis clavier qui vont bien et en plus je peux récupérer mes consoles à distance.</p>


<p>Mais pour optimiser l'utilisation de Screen, il faut encore le configurer correctement ce qui peut être plus difficile qu'il n'y parait tant ce logiciel regorge d'options. Je vais donc vous donner quelques pistes qui pourront vous servir ...</p></div>
        
    <div class="post-content"><p>Toutes ces commandes sont à ajouter à votre <code>~/.screenrc</code></p>


<h3>Utiliser UTF-8</h3>

<p><code>defutf8 on</code></p>


<h3>Permettre le scroll avec Shift + PgUp/PgDw</h3>

<p><code>termcapinfo xterm ti@:te</code>@<br />
<code>bindkey -m "^[[5;2~" stuff ^b</code><br />
<code>bindkey -m "^[[6;2~" stuff ^f</code><br /></p>


<h3>Lancer des applications au démarrage</h3>

<p><code>screen -t SHELL   0 bash</code><br />
<code>screen -t EDITOR  1 vim -X</code><br />
<code>screen -t MAIL    2 mutt</code><br />
<code>screen -t NEWS    3 slrn -n -C -k</code><br />
<code>screen -t IRC     9 irssi</code></p>


<p>Ici on lance un <em>bash</em> dans la fenêtre 0 qui se nommera "SHELL", <em>vim</em> est lancé dans la fenêtre 1 qui s'appellera "EDITOR", etc.</p>


<h3>Quelques variables qui font la différence</h3>
<ul>
<li><code>autodetach on</code>&nbsp;: détache Screen automatiquement quand vous fermez la console</li>
<li><code>startup_message off</code>&nbsp;: supprime le message d'accueil au lancement de Screen</li>
<li><code>altscreen on</code>&nbsp;: permet de restaurer l'état de la console après la fermeture d'une application en ncurses (comme vim)</li>
<li><code>defscrollback 1000</code>&nbsp;: définit le nombre de lignes à mémoriser dans l'historique de Screen</li>
</ul>

<h3>Barre des tâches pratique</h3>

<p><code>caption always "%{wK}%-Lw%{= BW}%50&gt;%n%f* %t%{-}%+Lw%&lt; %=%c %d/%m/%y"</code></p>


<p>Ceci vous affichera, au bas de votre console, la liste des fenêtres ouvertes ainsi que l'heure et la date.</p>


<h3>Raccourcis clavier sympas</h3>

<p><code>bind E screen -t 'MAIL' 2 mutt -y</code><br />
<code>bind F screen -t 'FTP' lftp</code><br />
<code>bind I screen -t 'IRC' 9 irssi</code></p>


<p>Ces commandes permettent de lancer&nbsp;:</p>
<ul>
<li><em>mutt</em> dans la fenêtre 2 et de lui donner le nom "MAIL" à l'aide de la combinaison <code>C-a E</code><br /></li>
<li><em>lftp</em> dans la premiére fenêtre libre et de la nommer "FTP" à l'aide de <code>C-a F</code><br /></li>
<li><em>irssi</em> dans la fenêtre 9 et de la nommer "IRC" à l'aide de <code>C-a I</code></li>
</ul>

<p><code>bindkey -k k8 prev</code><br />
<code>bindkey -k k9 next</code></p>


<p>Et finalement avec cela vous pourrez utiliser les touches F8 et F9 pour passer respectivement à la fenêtre directement à gauche ou à droite de celle en cours.</p>


<p>Il est également à noter que <code>C-a a</code> permet de retourner à la derniére fenêtre utilisée et que <code>C-a n</code> permet d'aller à la fenêtre <em>n</em> où <em>n</em> est le numéro de la fenêtre voulue.</p>


<hr />


<p>Maintenant vous n'avez plus qu'à apprécier le confort. Si vous voulez plus d'informations sur l'utilisation de Screen, je vous conseille son excellent manuel. Vous deviendrez vite accros. <img src="../../../../../themes/default/smilies/wink.png" alt=";)" class="smiley" /></p></div>

      </div>

