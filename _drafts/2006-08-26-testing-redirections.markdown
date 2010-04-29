---
layout: post
title: Efficient redirection testing
category: ruby
tags: ['ruby', 'test', 'redirection', 'tips']
---

  <div class="post">
    <h2 id="p87" class="post-title">Tester efficacement les redirections en Ruby on Rails</h2>
    
    <p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le samedi, 26 août 2006, 17:32        - <a href="../../../../../category/Tips/index.html">Trucs et astuces</a>
        - <a href="index.html">Lien permanent</a>
    </p>
    
    
    
          <div class="post-excerpt"><p><a href="http://www.rubyonrails.org" hreflang="en">Ruby on Rails</a> permet de mettre facilement en place des tests fonctionnels qui nous permetttent de savoir si nos controlleurs se comportent comme on le souhaite.</p>


<p>Un test fréquent est de voir si, dans certaines conditions, notre controlleur nous redirige bien vers une page donnée. Malheureusement la méthode <code>assert_redirected_to</code> qui nous permet de faire cela ne fonctionne pas comme on pourrait s'y attendre.</p></div>
        
    <div class="post-content"><p>En effet, un code de test tel que le suivant&nbsp;:</p>


<pre> def test_index_redirection
   get :index
   assert_response :redirect
   assert_redirected_to :controller =&gt; 'user', :action =&gt; 'login'
 end</pre>


<p>doit vérifier que lorsque la page d'index est appelée, une redirection est faite vers le controlleur <code>user</code> et l'action <code>login</code>. Cette redirection est mise en place dans le controlleur avec un simple <code>redirect_to :controller =&gt; 'user', :action =&gt; 'login'</code>.</p>


<p>Lorsque nous lançons notre procédure de test, l'erreur suivante est détectée&nbsp;:</p>

<pre> 1) Failure:
 test_index_redirection(Admin::ConfigurationControllerTest) <a href="../test/functional/admin/configuration_controller_test.rb_20">./test/functional/admin/configuration_controller_test.rb:20</a>:
 response is not a redirection to all of the options supplied  (redirection is &lt;{:action=&gt;"login", :controller=&gt;"user"}&gt;),  difference: &lt;{}&gt;</pre>


<p>Le test échoue donc en nous indiquant que la différence entre ce qu'y était attendu et le résultat effectif est nulle. Je soupçonne donc un bug au niveau de la génération ou de la comparaison des routes. Toujours est-il qu'une solution existe pour contourner ce problème et faire en sorte que les tests soient exécutés correctement. Il suffit de créer l'url attendue à l'aide de la méthode <code>url_for</code>. Il faut donc procéder de la façon suivante&nbsp;:</p>


<pre> assert_redirected_to @controller.url_for(:controller =&gt; 'user', :action =&gt; 'login')</pre>


<p>De cette manière le test passe sans problème et vous pouvez vérifier que votre code vous envoit bien au controlleur et à l'action voulue.</p>


<p>J'ai ouvert un rapport de bug sur le <a href="http://dev.rubyonrails.org/ticket/5921" hreflang="en">Trac de Ruby on Rails</a> en espérant que les développeurs prendront le temps de se pencher sur la question.</p></div>

      </div>

