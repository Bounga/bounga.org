---
layout: post
title: Dynamic 404 and 500 pages
category: rails
tags: ['rails', 'meta-programming', 'tips']
---

<div class="post">
<h2 id="p100" class="post-title">Rails : Gérer des pages 404 et 500 dynamiques</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le lundi, 11 décembre 2006, 16:40        - <a href="../../../../../category/Tips/index.html">Trucs et astuces</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>Par défaut, dans <a href="http://www.rubyonrails.org">Rails</a>, les pages 404 (page inexistante) et 500 (erreur de l'application) sont des pages statiques. Vous ne pouvez donc pas y utiliser vos layouts ou même y insérer du contenu de façon dynamique en fonction de l'erreur qui s'est produite.</p>
<p>Une solution existe, vous permettant de générer une page aussi dynamique que vous le souhaitez. Cette solution passe par l'utilisation de <code>rescue_action_in_public</code>. Voyons comment la mettre en place.</p></div>
    
<div class="post-content"><p>La méthode <code>rescue_action_in_public</code> définie dans <code>ActionController::Rescue</code> peut être réécrite, comme toutes les méthodes <a href="http://www.ruby-lang.org">Ruby</a> d'ailleurs, pour en changer son comportement. Elle est appelée à chaque fois qu'une exception est levée suite au chargement d'une page. Les chargements locaux (lorsque vous êtes en développement par exemple) n'appellent pas cette méthode. Si vous voulez mettre en place le même système pour les chargements locaux, il faudra passer par la méthode <code>rescue_action_locally</code>.</p>
<p>Par défaut, cette méthode attrape les exceptions <code>RoutingError</code> et <code>UnknownAction</code> et affiche la page statique disponible dans "public/404.html", pour toutes les autres exceptions, elle affiche la page statique disponible dans "public/500.html".</p>
<p>Nous allons faire en sorte que ces exceptions soient attrapées par notre code et qu'elles affichent nos pages dynamiques ce qui nous permettra d'utiliser nos layouts, CSS et du contenu dynamique plutôt qu'une simple page statique. Pour cela, il suffit d'ajouter le code suivant dans votre fichier "app/controller/application.rb" :</p>
<pre>def rescue_action_in_public(exception)
case exception
when ::ActionController::UnknownAction, ::ActionController::RoutingError
render :file => File.join("#{RAILS_ROOT}", "public", "404.rhtml"), :layout => true, :status => "404 not found"
else
render :file => File.join("#{RAILS_ROOT}", "public", "500.rhtml"), :layout => true, :status => "500 error"
SystemNotifier.deliver_exception_notification(self, request, exception)
end
end
</pre>
<p>L'exception levée par le chargement de la page est passée à notre méthode, nous la stockons dans <code>exception</code>. Ensuite, selon le type d'exception, note méthode se comportera de différentes façons.</p>
<p>Si une exception de type <code>ActionController::UnknownAction</code> ou <code>ActionController::RoutingError</code> est levée, alors nous allons afficher notre page stockée dans "public/404.rhtml". Ce fichier est un template RHTML typique, vous pouvez donc y faire ce que vous voulez. Nous faisons également attention de bien affecter le statut "404" lors du rendu de cette page.</p>
<p>Pour toutes les autres erreurs (qui seront souvent des erreurs fatales qui remettent en cause notre code), nous afficherons le fichier "public/500.html" avec le statut "500". Nous allons également faire en sorte qu'un mail soit envoyé aux admins pour leur signaler l'erreur et leur donner l'ensemble des éléments (URL appelée, paramètres passés à la requête, etc) qui ont fait apparaître cette erreur. Nous utilisons pour cela <code>SystemNotifier</code>, une classe du module <code>ActionMailer</code> qui nous permet d'avoir un e-mail joliment formaté.</p>
<p>Grâce à ce petit bout de code, vous pourrez très facilement rendre vos pages d'erreurs plus attractives et utiles. Vous gagnerez par la même occasion en souplesse.</p>
<p>UPDATE :</p>
<p>Depuis Rails 2.1.0, une méthode plus rapide et plus propre permet d'arriver au même résultat (et même un peu mieux en fait) :</p>
<code>rescue_from ActionController::RoutingError, ActionController::UnknownAction, ActiveRecord::RecordNotFound do |exception|</code><div><code>  respond_to do |type|</code></div><div><code>    type.html { render :template => "shared/404", :layout => 'application', :status => 404 }</code></div><div><code>    type.all  { render :nothing => true, :status => 404 }</code></div><div><code>  end</code></div><div><code>end</code></div></div>

  </div>
