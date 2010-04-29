---
layout: post
title: How to write a DSL using Ruby 
category: ruby
tags: ['ruby', 'dsl', 'meta-programming']
---

<div class="post">
<h2 id="p141" class="post-title">Ecriture d'un DSL en Ruby</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le lundi, 22 juin 2009, 12:16        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
    - <a href="index.html">Lien permanent</a>
</p>

    <ul class="post-tags">    <li><a href="../../../../../tag/Documentation/index.html">Documentation</a></li>
            <li><a href="../../../../../tag/DSL/index.html">DSL</a></li>
            <li><a href="../../../../../tag/Ruby/index.html">Ruby</a></li>
</ul>    

      <div class="post-excerpt"><p>Beaucoup d'entre vous connaissent <a href="http://www.ruby-lang.org" hreflang="en">Ruby</a> grâce à son fort potentiel Web au travers du framework <a href="http://www.rubyonrails.org" hreflang="en">Ruby on Rails</a>.  Mais Ruby excelle dans de nombreux domaines et s'avère particulièrement efficace dans l'écriture de <a href="http://fr.wikipedia.org/wiki/Domain-specific_programming_language" hreflang="fr">DSL</a> (Domain Specific Language).</p>


<p>Les DSL existent depuis toujours et vous les utilisez peut-être même sans vous en rendre compte. C'est un concept très à la mode ces derniers temps parce qu'avec les langages modernes, il n'a jamais été aussi facile d'en développer un.</p>


<p>Le but d'un DSL (du moins en Ruby) est de proposer à l'utilisateur un langage simple qui va lui permettre d'accomplir des tâches ciblées. L'utilisateur écriera du Ruby sans même s'en rendre compte.</p>


<p>Nous allons voir dans ce billet comment tirer parti de ces possibilités et créer notre propre DSL.</p></div>
    
<div class="post-content"><h2>Jeu de questions / réponses</h2>


<p>Nous allons pour commencer, mettre en place un exemple simple. Nous allons écrire un logiciel de quiz qui va nous permettre de donner une liste de questions et pour chaque question nous pourrons donner des réponses erronées et une réponse correcte.</p>


<p>Notre programme, côté utilisateur, fonctionnera comme suit&nbsp;:</p>


<blockquote><p>Quel est le langage utilisé par Ruby on Rails&nbsp;?</p>
<p>
1 - C#</p>
<p>
2 - Python</p>
<p>
3 - Ruby</p>
<p>
4 - PHP</p>
<p></p>
<p>
Quel est votre réponse&nbsp;?</p></blockquote>


<p>L'idée derrière ce générateur de Quiz est d'avoir un mini-langage dédié à l'écriture des questions et de ses réponses. Le programme que nous écrirons sera en mesure de lire ce mini-langage pour l'exécuter via l'interpréteur Ruby. Nous aurons donc des fichiers de quiz qui décrirons les questions / réponses&nbsp;:</p>



<blockquote><p>question 'Quel est le langage utilisé par Ruby on Rails ?'</p>
<p>
faux 'C#'</p>
<p>
faux 'Python'</p>
<p>
vrai 'Ruby'</p>
<p>
faux 'PHP'</p></blockquote>


<p>Ces questions pourront être dans un fichier 'mon_test.quiz' par exemple. Il faut comprendre que ce fichier ne sera pas réellement un fichier de données mais plutôt un programme qui sera interprété par Ruby.</p>


<p>Les appels à 'question', 'vrai' et 'faux' seront en fait des appels à des méthodes Ruby que nous allons définir. Nous ne parserons pas un fichier texte pour en déduire quelque chose, nous éxecuterons un vrai programme en Ruby.</p>


<h3>Lire les questions</h3>


<p>Nous allons commencer par implémenter la lecture de notre fichier de questions, voici ce à quoi devrait ressembler quiz.rb&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit"><span style="color:#008000; font-style:italic;">#!/usr/bin/env ruby</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> question<span style="color:#006600; font-weight:bold;">&#40;</span>text<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#CC0066; font-weight:bold;">puts</span> <span style="color:#996600;">&quot;Voici une question: #{text}&quot;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> vrai<span style="color:#006600; font-weight:bold;">&#40;</span>text<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#CC0066; font-weight:bold;">puts</span> <span style="color:#996600;">&quot;Voici une réponse valide: #{text}&quot;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> faux<span style="color:#006600; font-weight:bold;">&#40;</span>text<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#CC0066; font-weight:bold;">puts</span> <span style="color:#996600;">&quot;Voici une réponse incorrecte: #{text}&quot;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#CC0066; font-weight:bold;">load</span> <span style="color:#996600;">'mon_test.quiz'</span></pre>


<p>Pas beaucoup de code me direz-vous, et bien oui mais pourtant avec si peu de code, nous avons déjà un programme fonctionnel qui va être capable de lire les questions et les réponses. Vous avez, avec ce petit morceau de code, toute la logique d'écriture d'un DSL sous les yeux. Nous avons simplement définit 3 méthodes qui vont nous permettre de lire notre fichier de quiz.</p>


<p>Il ne nous reste plus qu'à lire notre fichier de quiz&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit"><span style="color:#CC0066; font-weight:bold;">load</span> <span style="color:#996600;">'mon_test.quiz'</span></pre>


<p>Ici encore, nous utilisons du code Ruby conventionnel qui nous sert à charger un fichier qui sera interprété comme étant du code Ruby. Notre fichier 'mon_test.quiz' n'est en fait rien d'autre qu'un fichier Ruby qui fait de multiples appels aux méthodes 'question', 'vrai' et faux' que nous avons définit avant.</p>


<p>Si nous lancions notre programme, nous obtiendrions&nbsp;:</p>


<blockquote><p>Voici une question: Quel est le langage utilisé par Ruby on Rails&nbsp;?</p>
<p>
Voici une réponse incorrecte: C#</p>
<p>
Voici une réponse incorrecte: Python</p>
<p>
Voici une réponse valide: Ruby</p>
<p>
Voici une réponse incorrecte: PHP</p></blockquote>



<h2>Création d'un quiz</h2>


<p>Maintenant que nous sommes capable de lire les questions et réponses, il ne nous reste plus qu'à implémenter le fonctionnement de chaque méthode. Il faut donc décrire ce que feront réellement les appels à 'question', 'vrai' et faux'.</p>


<p>Dans notre exemple de quiz, cela reste très facile à faire puisqu'il s'agit simplement de mettre en place une structure de données. Nous allons donc créer une classe Quiz qui va nous permettre d'encapsuler le fonctionnement.</p>


<pre class="ruby ruby" style="font-family:inherit"><span style="color:#9966CC; font-weight:bold;">class</span> Quiz
<span style="color:#9966CC; font-weight:bold;">def</span> initialize
<span style="color:#0066ff; font-weight:bold;">@questions</span> = <span style="color:#006600; font-weight:bold;">&#91;</span><span style="color:#006600; font-weight:bold;">&#93;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> ajouter_question<span style="color:#006600; font-weight:bold;">&#40;</span>question<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#0066ff; font-weight:bold;">@questions</span> <span style="color:#006600; font-weight:bold;">&lt;&lt;</span> question
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> derniere_question
<span style="color:#0066ff; font-weight:bold;">@questions</span>.<span style="color:#9900CC;">last</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> lancer
count=0
<span style="color:#0066ff; font-weight:bold;">@questions</span>.<span style="color:#9900CC;">each</span> <span style="color:#006600; font-weight:bold;">&#123;</span> |q| count <span style="color:#006600; font-weight:bold;">+</span>= <span style="color:#006666;">1</span> <span style="color:#9966CC; font-weight:bold;">if</span> q.<span style="color:#9900CC;">poser</span> <span style="color:#006600; font-weight:bold;">&#125;</span>
<span style="color:#CC0066; font-weight:bold;">puts</span> <span style="color:#996600;">&quot;Vous avez #{count} bonnes réponses sur #{@questions.size}.&quot;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
<span style="color:#9966CC; font-weight:bold;">end</span></pre>


<p>Cette classe va simplement nous servir à créer une collection de questions. Il nous faut maintenant une classe Question qui va nous permettre de poser une question et de savoir si la réponse donnée est bonne ou non.</p>

<pre class="ruby ruby" style="font-family:inherit"><span style="color:#9966CC; font-weight:bold;">class</span> Question
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> initialize<span style="color:#006600; font-weight:bold;">&#40;</span>text<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#0066ff; font-weight:bold;">@text</span> = text
<span style="color:#0066ff; font-weight:bold;">@reponses</span> = <span style="color:#006600; font-weight:bold;">&#91;</span><span style="color:#006600; font-weight:bold;">&#93;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> ajouter_reponse<span style="color:#006600; font-weight:bold;">&#40;</span>reponse<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#0066ff; font-weight:bold;">@reponses</span> <span style="color:#006600; font-weight:bold;">&lt;&lt;</span> reponse
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> poser
<span style="color:#CC0066; font-weight:bold;">puts</span> <span style="color:#996600;">&quot;&quot;</span>
<span style="color:#CC0066; font-weight:bold;">puts</span> <span style="color:#996600;">&quot;Question: #{@text}&quot;</span>
<span style="color:#0066ff; font-weight:bold;">@reponses</span>.<span style="color:#9900CC;">size</span>.<span style="color:#9900CC;">times</span> <span style="color:#9966CC; font-weight:bold;">do</span> |i|
  <span style="color:#CC0066; font-weight:bold;">puts</span> <span style="color:#996600;">&quot;#{i+1} - #{@reponses[i].text}&quot;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
<span style="color:#CC0066; font-weight:bold;">print</span> <span style="color:#996600;">&quot;Quel est votre réponse ? &quot;</span>
reponse = <span style="color:#CC0066; font-weight:bold;">gets</span>.<span style="color:#9900CC;">to_i</span> <span style="color:#006600; font-weight:bold;">-</span> <span style="color:#006666;">1</span>
<span style="color:#0000FF; font-weight:bold;">return</span> <span style="color:#0066ff; font-weight:bold;">@reponses</span><span style="color:#006600; font-weight:bold;">&#91;</span>reponse<span style="color:#006600; font-weight:bold;">&#93;</span>.<span style="color:#9900CC;">correct</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
<span style="color:#9966CC; font-weight:bold;">end</span></pre>


<p>Une question représente simplement une chaîne de caractères associées à un ensemble de réponses.</p>


<p>Pour finir, nous aurons besoin d'une classe Reponse pour gérer les réponses utilisateur&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit"><span style="color:#9966CC; font-weight:bold;">class</span> Reponse
attr_reader <span style="color:#ff3333; font-weight:bold;">:text</span>, <span style="color:#ff3333; font-weight:bold;">:correct</span>
<span style="color:#9966CC; font-weight:bold;">def</span> initialize<span style="color:#006600; font-weight:bold;">&#40;</span> text, correct <span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#0066ff; font-weight:bold;">@text</span> = text
<span style="color:#0066ff; font-weight:bold;">@correct</span> = correct
<span style="color:#9966CC; font-weight:bold;">end</span>
<span style="color:#9966CC; font-weight:bold;">end</span></pre>


<p>Nous n'avons plus qu'à rassembler ces éléments pour avoir un programme fonctionnel.</p>


<p>Et voici la touche finale qui nous permettra d'avoir un programme utilisable&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit"><span style="color:#008000; font-style:italic;">#!/usr/bin/env ruby</span>
&nbsp;
<span style="color:#CC0066; font-weight:bold;">require</span> <span style="color:#996600;">'quiz'</span>
&nbsp;
<span style="color:#0066ff; font-weight:bold;">@quiz</span> = Quiz.<span style="color:#9900CC;">new</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> question<span style="color:#006600; font-weight:bold;">&#40;</span>text<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#0066ff; font-weight:bold;">@quiz</span>.<span style="color:#9900CC;">ajouter_question</span> Question.<span style="color:#9900CC;">new</span><span style="color:#006600; font-weight:bold;">&#40;</span>text<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> vrai<span style="color:#006600; font-weight:bold;">&#40;</span>text<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#0066ff; font-weight:bold;">@quiz</span>.<span style="color:#9900CC;">derniere_question</span>.<span style="color:#9900CC;">ajouter_reponse</span> Reponse.<span style="color:#9900CC;">new</span><span style="color:#006600; font-weight:bold;">&#40;</span>text,<span style="color:#0000FF; font-weight:bold;">true</span><span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#9966CC; font-weight:bold;">def</span> faux<span style="color:#006600; font-weight:bold;">&#40;</span>text<span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#0066ff; font-weight:bold;">@quiz</span>.<span style="color:#9900CC;">derniere_question</span>.<span style="color:#9900CC;">ajouter_reponse</span> Reponse.<span style="color:#9900CC;">new</span><span style="color:#006600; font-weight:bold;">&#40;</span>text,<span style="color:#0000FF; font-weight:bold;">false</span><span style="color:#006600; font-weight:bold;">&#41;</span>
<span style="color:#9966CC; font-weight:bold;">end</span>
&nbsp;
<span style="color:#CC0066; font-weight:bold;">load</span> <span style="color:#996600;">'mon_test.quiz'</span>
&nbsp;
<span style="color:#0066ff; font-weight:bold;">@quiz</span>.<span style="color:#9900CC;">lancer</span></pre>


<p>La méthode 'question' nous permet d'ajouter un nouvel objet Question dans notre instance de Quiz. Les réponses sont collectées (pour la dernière question posée) par les méthodes 'vrai' et 'faux'. Ces deux méthodes sont identiques, seul un booléen nous permet de savoir si la réponse est correcte ou non.</p>


<p>Les 2 dernières lignes de notre programme nous permettent de charger les questions et de lancer le quiz. Nous aurions d'ailleurs pu mettre en place une routine qui chargerait tous les quiz présents dans un répertoire ou faire en sorte que le quiz soit un paramètre de la ligne de commande.</p>


<p>Si vous tester quiz.rb, vous devriez obtenir quelque chose de ce genre&nbsp;:</p>


<blockquote><p>Question: Quel est le langage utilisé par Ruby on Rails&nbsp;?</p>
<p>
1 - C#</p>
<p>
2 - Python</p>
<p>
3 - Ruby</p>
<p>
4 - PHP</p>
<p>
Quel est votre réponse&nbsp;? 3</p>
<p>
Vous avez 1 bonnes réponses sur 1.</p></blockquote>


<p>Cet exemple représente parfaitement la structure d'un DSL typique.</p>


<p>Nous avons commencer par définir la structure d'un quiz du plus général au plus spécifique&nbsp;:</p>

<ul>
<li>Quiz</li>
<li>Question</li>
<li>Reponse</li>
</ul>

<p>Nous avons ensuite définit quelques méthodes de premier niveau (accessibles directement depuis Object) qui vont nous permettre d'utiliser notre DSL (méthodes 'question', 'vrai', faux'). Après quoi nous pouvons charger les questions / réponses utilisateur (load 'mon_test.quiz') pour finalement lancer le test.</p>


<h2>Avantages et inconvénients des DSLs internes</h2>


<p>Les DSLs internes (le fichier de donnés est écrit dans le même langage que son interpréteur) ont des avantages certains&nbsp;:</p>

<ul>
<li>concision (70 lignes) et clarté du code DSL</li>
<li>Possibilité d'utiliser les capacités de ruby depuis le fichier de données pour ajouter des commentaires dans le fichier de données, générer des questions posées au hasard, afficher les réponses dans un ordre aléatoire, implémenter de nouvelles fonctionnalités avancées</li>
</ul>

<p>Voici un exemple d'une utilisation avancée de Ruby dans le fichier de données&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">a = <span style="color:#CC0066; font-weight:bold;">rand</span><span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#006600; font-weight:bold;">&#41;</span>
b = <span style="color:#CC0066; font-weight:bold;">rand</span><span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#006600; font-weight:bold;">&#41;</span>
question <span style="color:#996600;">&quot;Quel est la somme de #{a} et #{b}&quot;</span>
faux <span style="color:#996600;">&quot;#{a + b + 12}&quot;</span>
faux <span style="color:#996600;">&quot;#{a + (b * 2)}&quot;</span>
faux <span style="color:#996600;">&quot;#{a + b - 0.5}&quot;</span>
vrai <span style="color:#996600;">&quot;#{a + b}&quot;</span></pre>


<p>Il y a bien évidemment des inconvénients à utiliser un DSL interne. L'avantage cité ci-dessus pourrait vite devenir un désanvatage si vous deviez limiter les possibilités côté utilisateur. L'utilisateur est ici en mesure (pour le peu qu'il connaisse Ruby) de faire tout ce que bon lui semble dans le fichier de configuration des questions / réponses. Il lui serait donc possible d'ouvrir des connexions réseau, d'écrire des fichiers, …</p>


<p>Voici qui clôture cette introduction sur les DSL en Ruby. Avez-vous déjà écrit des DSL&nbsp;? Voyez-vous des applications concrètes et utiles de DSL&nbsp;?</p>


<p>Bon amusement.</p></div>

  </div>

        <div id="attachments">
  <h3>Annexes</h3>
  <ul>
      <li class="package">
            	   	   	<a href="../../../../../public/code/Quiz_DSL.zip"
	title="Quiz_DSL.zip (1.64 KB)">DSL Ruby Quiz</a>
          </li>
      </ul>
  </div>