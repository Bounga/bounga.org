---
layout: post
title: NGinx config for DotClear in PATH_INFO mode
category: misc
tags: ['nginx', 'conf', 'dotclear', 'path_info']
---

<div class="post">
<h2 id="p137" class="post-title">Configuration de Nginx pour DotClear en mode PATH_INFO</h2>

<p class="post-info">Par <a href="http://www.cavigneaux.net">Bounga</a>    le vendredi,  6 février 2009, 11:11        - <a href="../../../../../category/Documentations/index.html">Documentations</a>
    - <a href="index.html">Lien permanent</a>
</p>

    <ul class="post-tags">    <li><a href="../../../../../tag/Apache/index.html">Apache</a></li>
            <li><a href="../../../../../tag/Configuration/index.html">Configuration</a></li>
            <li><a href="../../../../../tag/Documentation/index.html">Documentation</a></li>
            <li><a href="../../../../../tag/DotClear/index.html">DotClear</a></li>
            <li><a href="../../../../../tag/Nginx/index.html">Nginx</a></li>
            <li><a href="../../../../../tag/Ruby%20on%20Rails/index.html">Ruby on Rails</a></li>
            <li><a href="../../../../../tag/Thin/index.html">Thin</a></li>
</ul>    

      <div class="post-excerpt"><p>J'ai très récemment décidé de switcher du couple <a href="http://httpd.apache.org/" hreflang="en">Apache</a> / <a href="http://mongrel.rubyforge.org/" hreflang="en">Mongrel</a> à <a href="http://nginx.net/" hreflang="en">Nginx</a> / <a href="http://code.macournoyer.com/thin/" hreflang="en">Thin</a> pour mes applis Rails, les différences de performance sont visibles à l'oeil nu et les ressources utilisées (mémoire et CPU) ont été très largement réduites. Un vrai bonheur&nbsp;!</p>


<p>Sur ma lancée, j'ai décidé de migrer également mon blog pour pouvoir me séparer d'Apache. Mon blog, comme vous le savez peut-être, tourne sous <a href="http://fr.dotclear.org/" hreflang="fr">DotClear</a> en mode PATH_INFO ce qui permet d'avoir des URLs plus élégantes.</p>


<p>J'ai donc voulu partager avec vous mon expérience et vous propose une documentation sur la mise en place d'un DotClear en mode PATH_INFO avec Nginx.</p>


<p>Voici ma petite recette qui semble très bien fonctionner (surtout depuis la ré-écriture complète de la config suite aux problèmes liés à l'espace d'administration).</p></div>
    
<div class="post-content"><h2>Installation des paquets</h2>


<p>J'utilise une <a href="http://www.ubuntu.com/products/whatisubuntu/serveredition" hreflang="en">Ubuntu Server</a>, je vais donc passer par <code>apt-get</code> pour installer les paquets nécessaires, vous adapterez donc les commandes en fonction de votre distribution.</p>


<p>Allons-y , installons Nginx et la librairie FastCGI. Nous allons également installer Lighttpd qui propose un utilitaire bien pratique pour la gestion des processus FastCGI&nbsp;:</p>

<pre class="bash bash" style="font-family:inherit"><span style="color: #c20cb9; font-weight: bold;">apt-get</span> <span style="color: #c20cb9; font-weight: bold;">install</span> nginx lighttpd libfcgi0ldbl
update-rc.d <span style="color: #660033;">-f</span> lighttpd remove</pre>


<p>Vous noterez que nous avons retiré Lighttpd des services à démarrer automatiquement puisque de toute façon nous allons utiliser Nginx comme serveur HTTP.</p>


<h2>Mise en place du démarrage automatique des services</h2>


<p>Nous allons créer un script d'init qui va se charger de relancer les processus fastcgi à chaque fois que la machine redémarrera, nous créons donc un fichier <code>/etc/init.d/fastcgi</code></p>


<p>Voici le code qu'il contient&nbsp;:</p>

<pre class="bash bash" style="font-family:inherit"><span style="color: #666666; font-style: italic;">#!/bin/bash</span>
&nbsp;
<span style="color: #007800;">COMMAND</span>=<span style="color: #000000; font-weight: bold;">/</span>usr<span style="color: #000000; font-weight: bold;">/</span>bin<span style="color: #000000; font-weight: bold;">/</span>spawn-fcgi
<span style="color: #007800;">ADDRESS</span>=127.0.0.1
<span style="color: #007800;">PORT</span>=<span style="color: #000000;">9000</span>
<span style="color: #007800;">USER</span>=www-data
<span style="color: #007800;">GROUP</span>=www-data
<span style="color: #007800;">PHPCGI</span>=<span style="color: #000000; font-weight: bold;">/</span>usr<span style="color: #000000; font-weight: bold;">/</span>bin<span style="color: #000000; font-weight: bold;">/</span>php5-cgi
<span style="color: #007800;">PIDFILE</span>=<span style="color: #000000; font-weight: bold;">/</span>var<span style="color: #000000; font-weight: bold;">/</span>run<span style="color: #000000; font-weight: bold;">/</span>fastcgi-php.pid
<span style="color: #007800;">RETVAL</span>=0
&nbsp;
<span style="color: #000000; font-weight: bold;">case</span> <span style="color: #ff0000;">&quot;$1&quot;</span> <span style="color: #000000; font-weight: bold;">in</span>
start<span style="color: #7a0874; font-weight: bold;">&#41;</span>
  <span style="color: #007800;">$COMMAND</span> <span style="color: #660033;">-a</span> <span style="color: #007800;">$ADDRESS</span> <span style="color: #660033;">-p</span> <span style="color: #007800;">$PORT</span> <span style="color: #660033;">-u</span> <span style="color: #007800;">$USER</span> <span style="color: #660033;">-g</span> <span style="color: #007800;">$GROUP</span> <span style="color: #660033;">-f</span> <span style="color: #007800;">$PHPCGI</span> <span style="color: #660033;">-P</span> <span style="color: #007800;">$PIDFILE</span>
  <span style="color: #007800;">RETVAL</span>=<span style="color: #007800;">$?</span>
;;
stop<span style="color: #7a0874; font-weight: bold;">&#41;</span>
  <span style="color: #c20cb9; font-weight: bold;">killall</span> <span style="color: #660033;">-9</span> php5-cgi
  <span style="color: #007800;">RETVAL</span>=<span style="color: #007800;">$?</span>
;;
restart<span style="color: #7a0874; font-weight: bold;">&#41;</span>
  <span style="color: #c20cb9; font-weight: bold;">killall</span> <span style="color: #660033;">-9</span> php5-cgi
  <span style="color: #007800;">$COMMAND</span>
  <span style="color: #007800;">RETVAL</span>=<span style="color: #007800;">$?</span>
;;
<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
  <span style="color: #7a0874; font-weight: bold;">echo</span> <span style="color: #ff0000;">&quot;Usage: fastcgi {start|stop|restart}&quot;</span>
  <span style="color: #7a0874; font-weight: bold;">exit</span> <span style="color: #000000;">1</span>
;;
<span style="color: #000000; font-weight: bold;">esac</span>      
<span style="color: #7a0874; font-weight: bold;">exit</span> <span style="color: #007800;">$RETVAL</span></pre>


<p>Il nous faut ensuite rendre ce script exécutable et demander son exécution au démarrage de la machine&nbsp;:</p>

<pre class="bash bash" style="font-family:inherit"><span style="color: #c20cb9; font-weight: bold;">chmod</span> u+x <span style="color: #000000; font-weight: bold;">/</span>etc<span style="color: #000000; font-weight: bold;">/</span>init.d<span style="color: #000000; font-weight: bold;">/</span>fastcgi
update-rc.d fastcgi defaults</pre>



<h2>Configuration de FastCGI pour Nginx</h2>


<p>Nous devons nous assurer qu'un certain nombre de paramètres seront passés à FastCGI par Nginx pour que tout fonctionne correctement, nous allons donc créer un nouveau fichier de conf <code>/etc/nginx/fastcgi.conf</code> qui contient&nbsp;:</p>

<pre class="ini ini" style="font-family:inherit">fastcgi_intercept_errors on<span style="color: #666666; font-style: italic;">;</span>
fastcgi_connect_timeout <span style="">60</span><span style="color: #666666; font-style: italic;">;</span>
fastcgi_send_timeout <span style="">300</span><span style="color: #666666; font-style: italic;">;</span>
fastcgi_buffer_size 32k<span style="color: #666666; font-style: italic;">;</span>
fastcgi_buffers <span style="">32</span> 32k<span style="color: #666666; font-style: italic;">;</span>
fastcgi_busy_buffers_size 32k<span style="color: #666666; font-style: italic;">;</span>
fastcgi_temp_file_write_size 32k<span style="color: #666666; font-style: italic;">;</span>
&nbsp;
fastcgi_index index.php<span style="color: #666666; font-style: italic;">;</span>
&nbsp;
fastcgi_param SCRIPT_NAME       $fastcgi_script_name<span style="color: #666666; font-style: italic;">;</span>
&nbsp;
fastcgi_param QUERY_STRING      $query_string<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param REQUEST_METHOD    $request_method<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param CONTENT_TYPE      $content_type<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param CONTENT_LENGTH    $content_length<span style="color: #666666; font-style: italic;">;</span>
&nbsp;
fastcgi_param REQUEST_URI       $request_uri<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param DOCUMENT_URI      $document_uri<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param DOCUMENT_ROOT     $document_root<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param SERVER_PROTOCOL   $server_protocol<span style="color: #666666; font-style: italic;">;</span>
&nbsp;
fastcgi_param GATEWAY_INTERFACE CGI/<span style="">1.1</span><span style="color: #666666; font-style: italic;">;</span>
fastcgi_param SERVER_SOFTWARE   nginx/$nginx_version<span style="color: #666666; font-style: italic;">;</span>
&nbsp;
fastcgi_param REMOTE_ADDR       $remote_addr<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param REMOTE_PORT       $remote_port<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param SERVER_ADDR       $server_addr<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param SERVER_PORT       $server_port<span style="color: #666666; font-style: italic;">;</span>
fastcgi_param SERVER_NAME       $server_name<span style="color: #666666; font-style: italic;">;</span>
&nbsp;
fastcgi_param REDIRECT_STATUS   <span style="">200</span><span style="color: #666666; font-style: italic;">;</span></pre>


<h2>Configuration du virtual host</h2>


<p>Nous n'avons plus maintenant qu'à configurer notre virtual host Nginx. Nous créons donc <code>/etc/nginx/sites-available/blog</code> qui contient le code suivant&nbsp;:</p>

<pre class="c c" style="font-family:inherit">server <span style="color: #009900;">&#123;</span>
listen <span style="color: #0000dd;">80</span>;
server_name  bounga.<span style="color: #202020;">org</span> www.<span style="color: #202020;">bounga</span>.<span style="color: #202020;">org</span>;
&nbsp;
root   <span style="color: #339933;">/</span>home<span style="color: #339933;">/</span>prod<span style="color: #339933;">/</span>www<span style="color: #339933;">/</span>blog;
    index  index.<span style="color: #202020;">html</span> index.<span style="color: #202020;">htm</span> index.<span style="color: #202020;">php</span>;
&nbsp;
error_log  <span style="color: #339933;">/</span>var<span style="color: #339933;">/</span>log<span style="color: #339933;">/</span>nginx<span style="color: #339933;">/</span>blog.<span style="color: #202020;">errors</span>.<span style="color: #202020;">log</span> error;
&nbsp;
<span style="color: #339933;"># On redirige toutes les requêtes sur les alias vers notre nom de domaine principal</span>
<span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span>$host <span style="color: #339933;">!=</span> <span style="color: #ff0000;">&quot;www.bounga.org&quot;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
      rewrite <span style="color: #339933;">^</span><span style="color: #009900;">&#40;</span>.<span style="color: #339933;">*</span><span style="color: #009900;">&#41;</span>$ http<span style="color: #339933;">:</span><span style="color: #666666; font-style: italic;">//www.bounga.org$1 permanent;</span>
    <span style="color: #009900;">&#125;</span>
&nbsp;
    <span style="color: #339933;"># On s'assure que les fichiers statiques seront servis directement sans traitement particulier</span>
    location ~<span style="color: #339933;">*</span> <span style="color: #339933;">^</span>.<span style="color: #339933;">+</span>.<span style="color: #009900;">&#40;</span>jpg|jpeg|gif|png|ico|css|zip|tgz|gz|rar|bz2|doc|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|js|swf|flv<span style="color: #009900;">&#41;</span>$ <span style="color: #009900;">&#123;</span>
      access_log off;
      expires 30d;
      <span style="color: #000000; font-weight: bold;">break</span>;
    <span style="color: #009900;">&#125;</span>
&nbsp;
    <span style="color: #339933;"># On empêche l'accès aux .htaccess</span>
    location ~ <span style="color: #339933;">/</span>\.<span style="color: #202020;">ht</span> <span style="color: #009900;">&#123;</span>
      deny all;
    <span style="color: #009900;">&#125;</span>
&nbsp;
    <span style="color: #339933;"># Le vrai travail commence ici</span>
location <span style="color: #339933;">/</span> <span style="color: #009900;">&#123;</span>
        access_log  <span style="color: #339933;">/</span>var<span style="color: #339933;">/</span>log<span style="color: #339933;">/</span>nginx<span style="color: #339933;">/</span>blog.<span style="color: #202020;">access</span>.<span style="color: #202020;">log</span>;
&nbsp;
        set $good_path_info <span style="color: #ff0000;">&quot;&quot;</span>;
&nbsp;
        <span style="color: #339933;"># On récupére le bon path_info qui va permettre à DotClear d'utiliser les ré-écritures en mode PATH_INFO</span>
        <span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span>$uri ~ <span style="color: #ff0000;">&quot;^(/.*)&quot;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
          set $good_path_info  $<span style="color: #0000dd;">1</span>;
        <span style="color: #009900;">&#125;</span>
&nbsp;
        <span style="color: #339933;"># Si on se toruve dans l'espace admin, il ne faut pas modifier cette variable (pas de ré-écriture dans l'admin)</span>
        <span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span>$uri ~ <span style="color: #ff0000;">&quot;^/admin.*&quot;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
          set $good_path_info <span style="color: #ff0000;">&quot;&quot;</span>;
        <span style="color: #009900;">&#125;</span>
&nbsp;
        <span style="color: #339933;"># On inclut nos paramètres FastCGI communs</span>
        include <span style="color: #339933;">/</span>etc<span style="color: #339933;">/</span>nginx<span style="color: #339933;">/</span>fastcgi.<span style="color: #202020;">conf</span>;
&nbsp;
        <span style="color: #339933;"># On inclut nos paramètres FastCGI spécifiques à DotClear</span>
        fastcgi_pass  127.0.0.1<span style="color: #339933;">:</span><span style="color: #0000dd;">9000</span>;
        fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
        fastcgi_param  PATH_INFO          $good_path_info;
&nbsp;
        <span style="color: #339933;"># Redirige toutes les requetes sur des pseudos fichiers (URL rewrite) vers index.php pour traitement</span>
        <span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span><span style="color: #339933;">!-</span>e $request_filename<span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
          rewrite . <span style="color: #339933;">/</span>index.<span style="color: #202020;">php</span> <span style="color: #000000; font-weight: bold;">break</span>;
        <span style="color: #009900;">&#125;</span>
   <span style="color: #009900;">&#125;</span>
<span style="color: #009900;">&#125;</span></pre>


<h2>Lancer les serveurs</h2>


<p>On peut maintenant activer notre vhost, lancer Nginx et les processus FastCGI pour que notre application soit accessible</p>

<pre class="bash bash" style="font-family:inherit"><span style="color: #7a0874; font-weight: bold;">cd</span> <span style="color: #000000; font-weight: bold;">/</span>etc<span style="color: #000000; font-weight: bold;">/</span>nginx<span style="color: #000000; font-weight: bold;">/</span>sites-enabled<span style="color: #000000; font-weight: bold;">/</span>
<span style="color: #c20cb9; font-weight: bold;">ln</span> <span style="color: #660033;">-s</span> <span style="color: #000000; font-weight: bold;">/</span>etc<span style="color: #000000; font-weight: bold;">/</span>nginx<span style="color: #000000; font-weight: bold;">/</span>sites-available<span style="color: #000000; font-weight: bold;">/</span>blog 
&nbsp;
<span style="color: #000000; font-weight: bold;">/</span>etc<span style="color: #000000; font-weight: bold;">/</span>init.d<span style="color: #000000; font-weight: bold;">/</span>fastcgi start
<span style="color: #000000; font-weight: bold;">/</span>etc<span style="color: #000000; font-weight: bold;">/</span>init.d<span style="color: #000000; font-weight: bold;">/</span>nginx restart</pre>



<p>Et voilà, le tour est joué, votre blog DotClear est accessible et peut utiliser le PATH_INFO. J'ai pour ma part noté une amélioration impressionnante du nombre de req/s possible, une baisse de la consommation mémoire et CPU depuis mon passage a Nginx.</p>


<p>J'espère que ces informations pourront vous être utiles.</p></div>

  </div>



            <div id="comments">
    <h3>Commentaires</h3>
  <dl>
      <dt id="c128" class=" odd first"><a
  href="#c128" class="comment-number">1.</a>
  Le lundi,  6 avril 2009, 13:30      par aldo</dt>
  
  <dd class=" odd first">

        
  <p>l'url d'admin de dotclear<br />
nomdedomain/admin/  ne fonctionne pas. on doit forcer le <a href="http://nomdedomain/admin/auth.php" title="http://nomdedomain/admin/auth.php" rel="nofollow">http://nomdedomain/admin/auth.php</a></p>


<p>location ~ \.php(/|$) {</p>

<pre>  include fastcgi_params;
set $path_info "";
if ($uri ~ \.php/.) {
  set $path_info $uri;
}
fastcgi_param PATH_INFO $path_info;</pre>

<p>}</p>


<p>fonctionne mieux que le fastcgi_parms PATH_INFO request_uri; il evite le rewrite nom de domaine.</p>


<p>il y a aussi ds le php.ini la variable cgi.fix.path_info = 1 au lieu du 0</p>


<p>sans oublié le fastcgi_parms HTTPS on; quand on veut utiliser le https pour se connecter sur l'admin dotclear.</p>
        </dd>
              <dt id="c129" class="me  "><a
  href="#c129" class="comment-number">2.</a>
  Le lundi,  6 avril 2009, 14:12      par <a href="http://www.cavigneaux.net" rel="nofollow">Bounga</a></dt>
  
  <dd class="me  ">

        
  <p>@<a href="#c128" rel="nofollow">aldo</a> : Merci beaucoup Aldo ! Tu m'enlèves un épine du pied, je cherchais depuis des semaines !</p>
        </dd>
              <dt id="c130" class=" odd "><a
  href="#c130" class="comment-number">3.</a>
  Le lundi,  6 avril 2009, 17:37      par Aldo Reset</dt>
  
  <dd class=" odd ">

        
  <p>voila la soluce qui marche dans tout les cas:<br />
php.ini mettre le cgi.fix.path_info à 1.</p>


<p>et ce petit morceaux de code sans rewrite:<br />
location ~ \.php($|/) {</p>


<pre>    set $good_script     $uri;
set $good_path_info  "";</pre>


<pre>    if ($uri ~ "^(.+\.php)(/.+)") {
    set $good_script     $1;
    set $good_path_info  $2;
              }</pre>


<pre>    fastcgi_pass   127.0.0.1:9000;
fastcgi_index  index.php;
fastcgi_param  SCRIPT_FILENAME    $document_root$good_script;
fastcgi_param  SCRIPT_NAME        $good_script;
fastcgi_param  PATH_INFO          $good_path_info;
}</pre>



<p>quelques astuces:<br />
location ~* ^.+.(jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|pdf|ppt|txt|tar|mid|midi|wav|bmp|rtf|js)$<br />
{</p>

<pre>           expires 10y;</pre>

<p>access_log off;<br />
}</p>


<p>location ~* ^.+.(css|swf)$<br />
{</p>

<pre>           expires 15m;</pre>

<p>access_log off;<br />
}</p>


<p>location ~ /\.ht {</p>

<pre>     deny  all;
}</pre>


<p>besoin de rien d'autre pas de fastcgi_params include non plus.</p>
        </dd>
      </dl>
  </div>
