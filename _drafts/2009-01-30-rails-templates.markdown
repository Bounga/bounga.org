---
layout: post
title: Rails Templates
category: rails
tags: ['rails', 'template', 'tips']
---

 <div class="post">
    <h2 id="p136" class="post-title">Rails : Templates</h2>
    
    <p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le vendredi, 30 janvier 2009, 16:27        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
        - <a href="index.html">Lien permanent</a>
    </p>
    
        <ul class="post-tags">    <li><a href="../../../../../tag/Configuration/index.html">Configuration</a></li>
                <li><a href="../../../../../tag/Documentation/index.html">Documentation</a></li>
                <li><a href="../../../../../tag/Edge/index.html">Edge</a></li>
                <li><a href="../../../../../tag/Ruby%20on%20Rails/index.html">Ruby on Rails</a></li>
    </ul>    
    
          <div class="post-excerpt"><p>Une nouvelle fonctionnalité très intéressante vient d'être introduite dans Rails Edge et fera son chemin dans Rails 2.3 à priori. Ce sont les Templates.</p>


<p>Ces templates permettent, via du code Ruby, de créer des gabarits de projet qui permettent d'automatiser la phase de personnalisation de votre projet Rails quasi-obligatoire avant d'écrire la moindre ligne de code.</p>


<p>Les Templates sont donc des fichiers Ruby écrit à l'aide d'un DSL qui permettent d'ajouter dynamiquement des plugins, gems, initializers ou encore de modifier des fichiers existants dans votre projet Rails nouvellement créé.</p></div>
        
    <div class="post-content"><h2>Mais comment ça fonctionne&nbsp;?</h2>


<p>Tout d'abord pour créer un nouveau projet utilisant votre template, vous devez passer par l'option "-m" de la ligne de commande&nbsp;:</p>

<pre class="bash bash" style="font-family:inherit">rails blog <span style="color: #660033;">-m</span> ~<span style="color: #000000; font-weight: bold;">/</span>template.rb
&nbsp;
<span style="color: #666666; font-style: italic;"># ou</span>
&nbsp;
rails blog <span style="color: #660033;">-m</span> http:<span style="color: #000000; font-weight: bold;">//</span>host.com<span style="color: #000000; font-weight: bold;">/</span>template.rb</pre>


<p>Vous pouvez également appliquer un template à un projet existant grâce à une tâche Rake&nbsp;:</p>

<pre class="bash bash" style="font-family:inherit">rake rails:template <span style="color: #007800;">LOCATION</span>=~<span style="color: #000000; font-weight: bold;">/</span>template.rb</pre>


<h2>À quoi ressemble un fichier template&nbsp;?</h2>

<pre class="ruby ruby" style="font-family:inherit"><span style="color:#008000; font-style:italic;"># template.rb</span>
run <span style="color:#996600;">&quot;rm public/index.html&quot;</span>
generate<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#ff3333; font-weight:bold;">:model</span>, <span style="color:#996600;">&quot;News title:string body:text&quot;</span><span style="color:#006600; font-weight:bold;">&#41;</span>
generate<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#ff3333; font-weight:bold;">:controller</span>, <span style="color:#996600;">&quot;News&quot;</span><span style="color:#006600; font-weight:bold;">&#41;</span>
route <span style="color:#996600;">&quot;map.root :controller =&gt; 'news'&quot;</span>
rake<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">&quot;db:migrate&quot;</span><span style="color:#006600; font-weight:bold;">&#41;</span>
&nbsp;
run <span style="color:#996600;">'hg init .'</span>
run <span style="color:#996600;">'hg add'</span>
run <span style="color:#996600;">'hg commit -m &quot;Initial import&quot;'</span></pre>


<h2>Quelques exemples d'utilisation du DSL</h2>


<h3>gem</h3>
<pre class="ruby ruby" style="font-family:inherit">gem <span style="color:#996600;">&quot;hpricot&quot;</span>, <span style="color:#ff3333; font-weight:bold;">:version</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#996600;">'0.6'</span>, <span style="color:#ff3333; font-weight:bold;">:source</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#996600;">&quot;http://code.whytheluckystiff.net&quot;</span></pre>


<p>va ajouter la dépendance au gem hpricot à votre fichier environment.rb.</p>


<p>Il vous suffira ensuite d'appeler</p>

<pre class="bash bash" style="font-family:inherit">rake <span style="color: #ff0000;">&quot;gems:install&quot;</span></pre>


<p>pour que Rails installe vos Gems si ce n'est pas déjà fait.</p>


<h3>plugin</h3>
<pre class="ruby ruby" style="font-family:inherit">plugin<span style="color:#006600; font-weight:bold;">&#40;</span>name, options = <span style="color:#006600; font-weight:bold;">&#123;</span><span style="color:#006600; font-weight:bold;">&#125;</span><span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>permet d'installer un plugin, comme par exemple&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">plugin <span style="color:#996600;">'restful_authentication'</span>, <span style="color:#ff3333; font-weight:bold;">:git</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#996600;">'git://github.com/technoweenie/restful-authentication.git'</span></pre>



<h3>initializer</h3>
<pre class="ruby ruby" style="font-family:inherit">initializer<span style="color:#006600; font-weight:bold;">&#40;</span>filename, data = <span style="color:#0000FF; font-weight:bold;">nil</span>, <span style="color:#006600; font-weight:bold;">&amp;</span>block<span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>permet de créer un fichier initializer dans le répertoire 'config/initializers', par exemple&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">initializer <span style="color:#996600;">'globalize.rb'</span>, 
<span style="color:#006600; font-weight:bold;">%</span>q<span style="color:#006600; font-weight:bold;">&#123;</span><span style="color:#9966CC; font-weight:bold;">include</span> Globalize
Locale.<span style="color:#9900CC;">set_base_language</span><span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">'fr-FR'</span><span style="color:#006600; font-weight:bold;">&#41;</span>
LOCALES = <span style="color:#006600; font-weight:bold;">&#123;</span><span style="color:#996600;">'en'</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#996600;">'en-US'</span>,
           <span style="color:#996600;">'fr'</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#996600;">'fr-FR'</span><span style="color:#006600; font-weight:bold;">&#125;</span>.<span style="color:#9900CC;">freeze</span>
<span style="color:#006600; font-weight:bold;">&#125;</span></pre>


<p>Vous pouvez également utiliser <code>lib</code> qui permet de créer un fichier dans le répertoire 'lib' et <code>vendor</code> qui ajoute un fichier dans le répertoire 'vendor'.</p>


<h3>file</h3>

<p>Pour créer des fichiers à un autre endroit de l'arborescence, vous pouvez utiliser <code>file</code>.</p>

<pre class="ruby ruby" style="font-family:inherit">file 'app/views/shared/_flash.rb', 
%q{&lt;div id=&quot;flash&quot;&gt;
  <span style="color:#006600; font-weight:bold;">&lt;%</span> flash.<span style="color:#9900CC;">each</span> <span style="color:#9966CC; font-weight:bold;">do</span> |key, value| <span style="color:#006600; font-weight:bold;">-%&gt;</span>
    &lt;div id=&quot;flash_<span style="color:#006600; font-weight:bold;">&lt;%</span>= key <span style="color:#006600; font-weight:bold;">%&gt;</span>&quot;&gt;<span style="color:#006600; font-weight:bold;">&lt;%</span>=h value <span style="color:#006600; font-weight:bold;">%&gt;</span>&lt;/div&gt;
  <span style="color:#006600; font-weight:bold;">&lt;%</span> <span style="color:#9966CC; font-weight:bold;">end</span> <span style="color:#006600; font-weight:bold;">-%&gt;</span>
&lt;/div&gt;
}</pre>


<p>vous aurez donc un nouveau répertoire 'shared' contenant le fichier '_flash.rb'.</p>


<h3>rakefile</h3>
<pre class="ruby ruby" style="font-family:inherit">rakefile<span style="color:#006600; font-weight:bold;">&#40;</span>filename, data = <span style="color:#0000FF; font-weight:bold;">nil</span>, <span style="color:#006600; font-weight:bold;">&amp;</span>block<span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>permet de créer un nouveau fichier Rake dans 'lib/tasks'&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">rakefile<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">&quot;bounga.rake&quot;</span><span style="color:#006600; font-weight:bold;">&#41;</span>,
<span style="color:#006600; font-weight:bold;">%</span>q<span style="color:#006600; font-weight:bold;">&#123;</span>namespace <span style="color:#ff3333; font-weight:bold;">:bounga</span> <span style="color:#9966CC; font-weight:bold;">do</span>
      task <span style="color:#ff3333; font-weight:bold;">:hello</span> <span style="color:#9966CC; font-weight:bold;">do</span>
        <span style="color:#CC0066; font-weight:bold;">puts</span> <span style="color:#996600;">&quot;Kawa Bounga!&quot;</span>
      <span style="color:#9966CC; font-weight:bold;">end</span>
    <span style="color:#9966CC; font-weight:bold;">end</span>
<span style="color:#006600; font-weight:bold;">&#125;</span>
<span style="color:#9966CC; font-weight:bold;">end</span></pre>


<p>Vous avez donc maintenant un nouveau fichier 'lib/tasks/bounga.rake' et une nouvelle tâche disponible dans Rake.</p>


<h3>generate</h3>
<pre class="ruby ruby" style="font-family:inherit">generate<span style="color:#006600; font-weight:bold;">&#40;</span>what, args<span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>permet d'appeler un générateur&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">generate<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#ff3333; font-weight:bold;">:model</span>, <span style="color:#996600;">&quot;Actor&quot;</span>, <span style="color:#996600;">&quot;name:string&quot;</span>, <span style="color:#996600;">&quot;age:number&quot;</span><span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<h3>run</h3>

<pre class="ruby ruby" style="font-family:inherit">run<span style="color:#006600; font-weight:bold;">&#40;</span>command<span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>permet d'appeler une commande système exactement comme le ferait l'appel à <code>system()</code>.</p>

<pre class="ruby ruby" style="font-family:inherit">run <span style="color:#996600;">&quot;rm public/index.html&quot;</span></pre>


<h3>rake</h3>

<pre class="ruby ruby" style="font-family:inherit">rake<span style="color:#006600; font-weight:bold;">&#40;</span>command, options = <span style="color:#006600; font-weight:bold;">&#123;</span><span style="color:#006600; font-weight:bold;">&#125;</span><span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>exécute une tâche Rake&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">rake <span style="color:#996600;">&quot;db:migrate&quot;</span>
&nbsp;
<span style="color:#008000; font-style:italic;"># ou</span>
&nbsp;
rake <span style="color:#996600;">&quot;db:migrate&quot;</span>, <span style="color:#ff3333; font-weight:bold;">:env</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#996600;">'production'</span>
&nbsp;
<span style="color:#008000; font-style:italic;"># ou encore</span>
&nbsp;
rake <span style="color:#996600;">&quot;gems:install&quot;</span>, <span style="color:#ff3333; font-weight:bold;">:sudo</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#0000FF; font-weight:bold;">true</span></pre>


<h3>route</h3>

<pre class="ruby ruby" style="font-family:inherit">route<span style="color:#006600; font-weight:bold;">&#40;</span>routing_code<span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>permet d'ajouter une nouvelle règle pour vos routes dans le fichier 'config/routes.rb'&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">route <span style="color:#996600;">&quot;map.root :controller =&gt; :actors&quot;</span></pre>


<h3>inside</h3>

<pre class="ruby ruby" style="font-family:inherit">inside<span style="color:#006600; font-weight:bold;">&#40;</span>dir<span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>permet d'effectuer une ou plusieurs actions dans un répertoire donné&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">inside<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">'vendor'</span><span style="color:#006600; font-weight:bold;">&#41;</span> <span style="color:#9966CC; font-weight:bold;">do</span>
  run <span style="color:#996600;">&quot;git clone git://github.com/rails/rails.git&quot;</span>
<span style="color:#9966CC; font-weight:bold;">end</span></pre>


<h3>ask</h3>

<pre class="ruby ruby" style="font-family:inherit">ask<span style="color:#006600; font-weight:bold;">&#40;</span>question<span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>vous permet de poser une question à l'utilisateur pour ensuite d'utiliser sa réponse&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">name = ask<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">&quot;Quel est votre nom ?&quot;</span><span style="color:#006600; font-weight:bold;">&#41;</span>
&nbsp;
file <span style="color:#996600;">'COPYRIGHT'</span>, <span style="color:#006600; font-weight:bold;">&lt;&lt;-</span>CODE
 Copyright <span style="color:#008000; font-style:italic;">#{name} #{Time.now.year}</span>
CODE</pre>


<h3>Oui ou non&nbsp;?</h3>

<pre class="ruby ruby" style="font-family:inherit">yes?<span style="color:#006600; font-weight:bold;">&#40;</span>question<span style="color:#006600; font-weight:bold;">&#41;</span>
no?<span style="color:#006600; font-weight:bold;">&#40;</span>question<span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<p>permet de demander à l'utilisateur de faire un choix concernant une action donnée&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">rake<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">&quot;rails:freeze:gems&quot;</span><span style="color:#006600; font-weight:bold;">&#41;</span> <span style="color:#9966CC; font-weight:bold;">if</span> yes?<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">&quot;Voulez-vous freezer Rails ?&quot;</span><span style="color:#006600; font-weight:bold;">&#41;</span></pre>


<h3>git</h3>


<p>Et voici comment faire des appels à <code>git</code>&nbsp;:</p>

<pre class="ruby ruby" style="font-family:inherit">git <span style="color:#ff3333; font-weight:bold;">:init</span>
git <span style="color:#ff3333; font-weight:bold;">:add</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#996600;">&quot;.&quot;</span>
git <span style="color:#ff3333; font-weight:bold;">:commit</span> <span style="color:#006600; font-weight:bold;">=&gt;</span> <span style="color:#996600;">&quot;-a -m 'Initial commit'&quot;</span></pre>


<h2>Des exemples complets&nbsp;?</h2>


<p>Si cette nouvelle fonctionnalité vous intéresse, vous pouvez déjà trouver un <a href="http://github.com/jeremymcanally/rails-templates/tree/master" hreflang="en">repository Git</a> sur lequel des contributeurs s'échangent des templates.</p>


<h2>Un récapitulatif des changements dans Rails 2.3</h2>


<p>Je vous conseille de jeter un oeil aux <a href="http://guides.rubyonrails.org/2_3_release_notes.html" hreflang="en">Release Notes de Rails 2.3</a> pour plus de détails sur les nouveautés</p>


<p>Il ne me reste plus qu'à écrire mon propre template maintenant&nbsp;!</p>


<p>Bon amusement à vous.</p></div>
