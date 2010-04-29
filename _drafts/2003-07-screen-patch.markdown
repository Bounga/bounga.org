---
layout: post
title: Know the command-line which has launched Screen
category: contribution
tags: ['screen', 'c', 'patch']
---

<p>Bonsoir&nbsp;!</p>\
\
\
<p>Voila, j'ai enfin décidé d'écrire une fonction qui était manquante à mon goût dans le logiciel de gestion de fenêtres en mode console <a href=\"http://www.gnu.org/software/screen/\" hreflang=\"en\">Screen</a>.</p>","En effet, lorsque vous lancez la commande @@screen -ls@@, vous obtenez la liste des processus lancés dans screen. Mais ceux-ci sont listés par leur PID et la commande qui a lancé ce processus n'est listée nulpart. Il est donc parfois difficile de savoir à quelle commande correspond tel PID. Il y a évidemment la solution du @@ps aux | grep PID@@ mais ça fait une commande en plus et je suis feignant :-) , en plus la commande @@screen -ls@@ existe alors pourquoi s'en priver ? Elle avait juste un manque à combler.
\

\
Mon code fonctionne depuis avant-hier. Après quelques tests il ne semble amener avec lui aucun bug flagrant. J'ai donc créé un patch et je l'ai envoyé aux deux principaux auteurs de screen. J'attend quelques semaines pour voir si j'aurai une réponse et si les auteurs vont décider d'inclure ce patch dans la version officielle. Toutefois, je vais mettre le patch à disposition [ICI|http://stuff.bounga.org/patches/16-screen-know-which-command-started-screen].
\

\
Maintenant vous savez tout :-) .
\

\
Bye, à plus tard.