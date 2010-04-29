---
layout: post
title: How to validate a model object which is not an ActiveRecord object
category: rails
tags: ['rails', 'tips', 'ActiveRecord', 'validation', 'model']
---

 <h2 id="p142" class="post-title">Valider un modèle qui ne descend pas de ActiveRecord::Base sans plugin</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le lundi, 21 décembre 2009, 15:19        - <a href="../../../../../category/Tips/index.html">Trucs et astuces</a>
    - <a href="index.html">Lien permanent</a>
</p>



      <div class="post-excerpt"><p>Après bien longtemps sans le moindre signe de vie, voici un petit billet rapide pour vous faire savoir que je suis toujours présent et que je continu à développer en Ruby&nbsp;!</p>


<p>Vous est-il déjà arrivé d'avoir besoin, en Rails, d'une classe qui se comporte comme un modèle ActiveRecord mais qui n'en ai pas un&nbsp;? Typiquement, un modèle auquel aucune table n'est associée en base.</p>


<p>Le cas le plus simple où l'on peut avoir besoin de ce genre de classe serait par exemple un modèle qui gère les prises de contact (formulaire de contact) depuis le site.</p>


<p>Votre formulaire de contact doit valider les infos qui lui sont passées (est-ce que les champs obligatoires sont remplis, est-ce que l'email semble valide, …) avant d'envoyer par mail cette prise de contact à l'admin mais vous n'avez absolument pas besoin d'enregister cette prise de contact en base et donc aucune envie de créer une table dédiée pour simplement satisfaire ActiveRecord&nbsp;!</p>


<p>Il existe des plugins qui font ça me direz-vous. Certes oui, c'est vrai. Mais j'aime comprendre comment fonctionne les choses et j'aime autant me passer d'un plugin pour une chose qui me paraît aussi simple.</p></div>
    
<div class="post-content"><p>On pourrait penser qu'il suffit d'inclure le module de validation dans notre classe, comme ceci&nbsp;:</p>

<pre class="rails rails" style="font-family:inherit">class Contact
include ActiveRecord::Validations
attr_accessor :name, :email, :message
&nbsp;
validates_presence_of :email, :message
end</pre>


<p>mais ce n'est pas le cas, les validations dépendent de plusieurs méthodes d'instance de votre classe et vous serez donc obligé de les définir si vous voulez avoir accès aux validations dans votre modèle.</p>

<pre class="rails rails" style="font-family:inherit">class Contact
include ActiveRecord::Validations
attr_accessor :name, :email, :message
&nbsp;
def initialize&#40;attrs = &#123;&#125;&#41;
attrs.each do |key, value|
  update_attribute&#40;key, value&#41;
end
@new_record = true
end
&nbsp;
def save
# Ici votre logique de sauvegarde
# Dans notre cas, un envoi d'email ...
&nbsp;
@new_record = false
return true # Si notre mail a pu être envoyé
end
&nbsp;
alias :save! :save
&nbsp;
def update_attribute&#40;key, value&#41;
send &quot;#{key}=&quot;, value
end
&nbsp;
def new_record?&#40;&#41;
@new_record
end
&nbsp;
# Et enfin nos validations
validates_presence_of :email, :message
end</pre>


<p>Nous pouvons maintenant créer un objet Contact&nbsp;:</p>

<pre class="rails rails" style="font-family:inherit">contact = Contact.new&#40;:name =&gt; &quot;Nico&quot;, :email =&gt; &quot;nico@test.com&quot;&#41;
# =&gt; #&lt;Contact:0x132d3a0 @new_record=true, @name=&quot;Nico&quot;, @email=&quot;nico@test.com&quot;&gt;
&nbsp;
contact.valid?
# =&gt; false</pre>


<p>Nos validations fonctionnent, notre objet n'est pas valide.</p>

<pre class="rails rails" style="font-family:inherit">contact = Contact.new&#40;:name =&gt; &quot;Nico&quot;, :email =&gt; &quot;nico@test.com&quot;, :message =&gt; &quot;Hey !&quot;&#41;
# =&gt; #&lt;Contact:0x134d3c3 @new_record=true, @name=&quot;Nico&quot;, @email=&quot;nico@test.com&quot;, @message=&quot;Hey !&quot;&gt;
&nbsp;
contact.valid?
# =&gt; true</pre>


<p>L'objet est maintenant valide.</p>


<p>Vous avez bien évidemment accès à toutes les méthodes mises à disposition par le module de validation, vous êtes donc en mesure de parcourir les erreurs pour savoir ce qui est à l'origine d'un objet non-valide.</p>


<p>Il serait très simple, à partir de ce code, de créer une classe de base dont hériteraient vos modèles non-ActiveRecord qui ont besoin des validations. Il est également très facile d'en extraire un plugin pour encore faciliter la mise en place de cette architecture.</p></div>
