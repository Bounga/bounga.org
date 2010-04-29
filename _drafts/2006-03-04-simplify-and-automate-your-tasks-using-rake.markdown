---
layout: post
title: Simplify and automate your tasks using rake
category: ruby
tags: ['ruby', 'rake', 'productivity']
---

  <div class="post">
    <h2 id="p72" class="post-title">Rake : simplifier et automatiser la maintenance de vos projets</h2>
    
    <p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le samedi,  4 mars 2006, 13:10        - <a href="../../../../../category/General/index.html">Général</a>
        - <a href="index.html">Lien permanent</a>
    </p>
    
    
    
          <div class="post-excerpt"><p>Pour ceux qui ne connaissent pas <a href="http://rake.rubyforge.org/" hreflang="en">Rake</a> est l'équivalent <a href="http://www.ruby-lang.org" hreflang="en">Ruby</a> de <a href="http://www.gnu.org/software/make/" hreflang="en">GNU Make</a>. Équivalent est un faible mot puisque Rake apporte en plus toute la puissance de Ruby dans vos fichiers d'automatisation. Rake vous permet par exemple de créer facilement vos <a href="http://docs.rubygems.org" hreflang="en">Gems</a>, vos fichiers de traduction, la documentation de vos classes, etc.</p></div>
        
    <div class="post-content"><p>Rake est donc un moyen pratique de se simplifier la tâche lorsqu'on est développeur et qu'on veut automatiser une partie du processus de développement.</p>


<p>L'exemple de Rakefile qui suit est la base que j'utilise pour tous mes projets en Ruby. S'il manque des tâches, vous pouvez facilement étendre ce fichier en utilisant la magie de Ruby ;-).</p>


<h3>Fichier Rakefile à placer à la racine de votre projet</h3>


<pre># On charge les librairies nécessaires à la création de Gem, de documentation, des fichiers de traduction et des suites de tests
require 'rake/gempackagetask'
require 'gettext/utils'
require 'gettext/rmsgfmt'
require 'rake/rdoctask'
require 'rake/testtask'</pre>



<pre># Constante qui contient la version de notre projet
VER = '0.3.0'</pre>


<pre># Tâche à éxecuter par défaut si on tape simplement rake
desc "Create the gem and install it" # Description de la tâche dans l'aide donnée par Rake -T
task :default =&gt; [:install] # La tâche :default dépend de la tâche :install</pre>


<pre># Spécifications pour le gem
SPEC = Gem::Specification.new do |s|
   s.name      = 'nom_du_projet'
   s.version   = VER
   s.author    = 'Mon nom'
   s.email     = 'mon@mail.org'
   s.homepage  = 'http://mon.site.org'
   s.summary   = 'Résumé de l'utilité du logiciel'
   s.description = 'Description plus longue'
   candidates  = Dir.glob("{bin,data,doc,lib,po,locale,test}/**/*")
   s.files     = candidates.delete_if do |item|
                   item.include?("CVS") || item.include?("rdoc")
               end
   s.executables           = ['nom_de_l_executable']
   s.test_file             = 'test/ts_mestests.rb' #Chemin vers la suite de tests
   s.has_rdoc              = true # On a de la RDoc et on veut générer une sortie HTML
   # Fichiers qui contiennent de la RDoc mais qui ne sont pas des fichiers source
   s.extra_rdoc_files      = ['README', 'doc/FAQ', 'doc/HACKING', 'doc/TODO', 'doc/AUTHORS', 'doc/BUGS', 'doc/ChangeLog']
   s.rdoc_options          = ['--title', 'Mon titre pour la doc HTML', '--main', 'README', '--line-numbers']
   s.required_ruby_version = '&gt;= 1.8.1'
   s.add_dependency('gettext') # Libs requises et qui existent en Gem
   s.requirements          = ['Libglade 2 bindings'] # Libs requises qui ne sont pas disponibles en Gem
end</pre>


<pre># Permettre la création d'un Gem : rake gem
task :gem =&gt; [:create_mo]
Rake::GemPackageTask.new(SPEC) do |pkg|
   pkg.need_zip = true
   pkg.need_tar_bz2 = true
end</pre>


<pre># Installation du logiciel pour utilisation : rake install
task :install =&gt; [:gem] do
   sh "gem install pkg/Mon-logiciel-#{VER}.gem"
end</pre>


<pre># Création de la documentation HTML : rake rdoc
Rake::RDocTask.new do |rd|
   rd.main = "README"
   rd.rdoc_files.include('README', 'doc/FAQ', 'doc/HACKING', 'doc/TODO', 'doc/AUTHORS',
                       'doc/BUGS', 'doc/ChangeLog', 'bin/*', 'lib/*')
end</pre>


<pre># Lancer les tests unitaires : rake test
Rake::TestTask.new do |t|
   t.libs &lt;&lt; "test"
   t.test_files = FileList['test/ts*.rb']
   t.verbose = true
end</pre>


<pre># Créer le .mo à partir des .po : rake create_mo
desc "Generate MO files from PO files"
task :create_mo do
   GetText.create_mofiles(true, "po", "locale")
end

# Met à jour les fichiers de traduction en relisant le sources
desc "Update PO files"
task :update_po do
   GetText.update_pofiles("domaine", ['bin/fichier', 'lib/fichier1.rb', 'lib/fichier2.rb'], VER)
end</pre>


<pre># Effacer tous les fichiers .mo : rake clobber_mo
desc "Remove generated MO files"
task :clobber_mo do
   sh "rm -r locale"
end</pre>


<pre># Effacer tous les fichiers auto-générés : rake clobber
desc "Remove any generated file"
task :clobber =&gt; [:clobber_package, :clobber_rdoc, :clobber_mo] do
end</pre>


<p>En plus des ses tâches, plusieurs autres sont disponibles grâce à l'inclusion des librairies en début de fichier. Pour une liste exhaustive des tâches supportées essayez <code>rake -T</code>.</p>


<p>Cet exemple vous permettra de commencer confortablement, n'hésitez pas à l'enrichir si certaines de vos tâches courantes n'y figurent pas. Voilà, maintenant vous n'avez plus d'excuse si vous n'utilisez pas Rake ;-).</p></div>

      </div>

