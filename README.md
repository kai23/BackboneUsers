BackboneJS, MySQL et PHP : un tutoriel pour les faire marcher ensemble (1)
=======

J’ai commencé à travailler récemment avec BackboneJS, qui est un framework MVC côté client assez puissant et sympa comme tout.
Le but de ce tutorial est d’expliquer rapidement comment créer le driver MySQL qui va bien, pour pouvoir communiquer avec la base de données, et ce en PHP. Je n’ai malheureusement pas trouvé un tuto décent pour le faire, donc je me lance ! :)
## I/ Introduction

Le but est de créer une application basique qui permettra de gérer les utilisateurs de notre base de données.
### 1)  La base de donnée

Nous allons donc, pour commencer, avoir besoin… d’une base de données. Voici son schéma :

    SET SQL_MODE="NO_AUTO_VALUE_ON_ZERO";
    SET time_zone = "+00:00";
     
    /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
    /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
    /*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
    /*!40101 SET NAMES utf8 */;
     
    CREATE DATABASE `TestBackbone` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
    USE `TestBackbone`;
     
    CREATE TABLE IF NOT EXISTS `users` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `prenom` varchar(25) NOT NULL,
      `nom` varchar(25) NOT NULL,
      `age` int(11) NOT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB  DEFAULT CHARSET=utf8 AUTO_INCREMENT=2 ;

Ceci va nous créer une base de données « BackboneUsers » avec une table « users », qui possède quatre champs :

* id
* prenom
* nom
* age

Quelque chose d’assez simple, donc.

### 2) Structure de notre application.

Pour la simplicité du tutoriel, nous allons avoir deux fichiers, tous les deux étant situés à la racine de notre serveur web :

* <code>index.html</code>, qui contiendra l’ensemble de notre code javascript  + html
* <code>users.php</code>, qui contiendra les différentes requêtes que l’on fera en base

### 3) Rappels sur BackboneJS

Comme dit plus haut, BackboneJS est un framework MVC (enfin, plus MV que MVC), en Javascript. Celui-ci est énormément utile pour faire des applications Web de gestion, par exemple. Le principe est que tout soit fait sur une seule et unique page, tout le reste se faisant par des requêtes AJAX.
Si vous souhaitez un bon starter pour Backbone, et comprendre la notion de route, modèle, vue et collections, je vous conseille d’aller faire un tour sur http://backbonetutorials.com/ , qui propose une excellente introduction à ce framework
Basiquement, BackboneJS utilise des requêtes HTTP pour faire la liaison avec son serveur. Vous connaissez sûrement <code>POST</code> et <code>GET</code>, dont on se sert assez couramment en PHP, et bien il en existe d’autres (plein !), dont <code>PUT</code> et <code>DELETE</code>. Notre projet va fonctionner de la sorte :

* Lorsque Backbone fait une requête <code>POST</code> sur <code>users.php</code>, nous allons sauvegarder en base de données les données contenues dans le POST.
* Lorsque Backbone fait une requête <code>PUT</code> sur <code>users.php</code>, nous mettons à jour l’entrée contenue dans ce PUT
* Lorsque Backbone fait une requête <code>DELETE</code> sur <code>users.php</code>, Ô étonnement, nous allons supprimer l’entrée passée dans le DELETE
* Lorsque Backbone fait une requête <code>GET</code> sur <code>users.php</code>, nous avons deux cas de figure :
  1. Soit le GET ne possède pas de paramètres, auquel cas nous allons chercher toutes nos entrées dans la base
  2. Soit le GET possède un paramètre (id), et nous récupérons uniquement l’utilisateur dont on a besoin

Sachant que BackboneJS, à l’aide de <code>Backbone.sync()</code>, gère quasiment toute la partie client, si vous avez compris ça, vous avez toutes les clefs en main. Allons-y !


## II/ Structure, routes et première vue

### 1) Structure de notre document HTML
    
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8" />
        <title></title>
    	<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.3.1/css/bootstrap.css" />
    </head>
    <body>
    
    </body>
    
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/jqueryui/1.10.2/jquery-ui.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.3.1/js/bootstrap.min.js"></script>
    </html>

C’est juste une page classique, à laquelle on rajoute Bootstrap et JQuery. Pour info, sachez que j’ai fait de cette page « de start » un snippet pour Sublime Text 2, que vous trouverez ici : https://github.com/kai23/sublime-snippets/blob/master/html5.sublime-snippet
À cette page, nous allons ajouter UnderscoreJS et BackboneJS.

    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8" />
    	<title></title>
    	<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.3.1/css/bootstrap.css" />
    </head>
    <body>
    	
    </body>
    
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/jqueryui/1.10.2/jquery-ui.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.3.1/js/bootstrap.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/underscore.js/1.4.4/underscore-min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/backbone.js/1.0.0/backbone-min.js"></script>
    </html>

Jusque ici, j’espère que vous arrivez à suivre sans soucis :) Le fait d’utiliser le CDN de CDNJS facilite un peu la tâche.
Faites bien attention de bien inclure BackboneJS APRÈS UnderscoreJS, En effet, Underscore est une dépendance de Backbone.
Dans le body, nous allons ajouter ceci :

    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8" />
    	<title></title>
    	<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.3.1/css/bootstrap.css" />
    </head>
    <body>
    	<div class="container">
    		<h1>Gestion des utilisateurs</h1>
    		<hr>
    		<div class="page"></div>
    	</div>
    </body>
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/jqueryui/1.10.2/jquery-ui.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.3.1/js/bootstrap.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/underscore.js/1.4.4/underscore-min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/backbone.js/1.0.0/backbone-min.js"></script>
    
    
### 2) Première route de BackboneJS

On va donc commencer par écrire la route pour le « home », la page d’accueil. Ouvrez une balise <code>script</code> en dessous des autres, et ajoutez ceci

    var Router = Backbone.Router.extend({
     routes: {
     '' : 'home',
     }
    });
     
    var router = new Router();
    router.on('route:home', function(){
     console.log("Je suis dans le home !");
    });
     
    Backbone.history.start();
        
Une route très classique donc. Lorsque je serai dans la racine de mon site, je ferai appel à un « alias » home. Puis lorsque mon router tombe sur cet alias, je fais un console.log(« je suis dans le home »). Enfin, je start le gestionnaire d’historique de Backbone.
Si vous allez dans votre racine du site, vous allez dans la page d’accueil, ouvrez votre console et rafraîchissez. Si vous voyez « Je suis dans le home ! », vous êtes tout bon ! :)


### 3) Première Vue

Nous allons maintenant créer la première vue pour Backbone. Celle-ci aura pour but de lister tous les utilisateurs de notre base. Sa création aura deux répercussions :
Son instanciation
Le changement de la route, disant qu’on va faire appel à la méthode render de notre vue fraîchement instanciée

On va commencer avec quelque chose de très basique :

    var UserListView = Backbone.View.extend({
     el:".page",
     
    render: function() {
     this.$el.html("Coucou !");
     }
    });
    var Router = Backbone.Router.extend({
     routes: {
     '' : 'home',
     }
    });
     
    // Instanciation des vues
    var userListView = new UserListView();
    // Instanciation de mon router
    var router = new Router();
    router.on('route:home', function(){
     userListView.render();
    });
     
    Backbone.history.start();
    
Vous suivez ? Si l’on parcoure ce que notre navigateur fait :

* Il arrive sur le « / »
* Le routeur lui donne un alias « home »
* « home » redirige ver « render » de notre vue  « UserListView »
* « render » affiche « coucou ! » à notre écran.
* Maintenant, ce qu’il nous « reste à faire », c’est d’afficher les éléments de notre base de données.

## III/ Modèles, collections et base de données MySQL

Afin de gérer les différents utilisateurs, on va créer une collection. Pour rappel, une collection est un ensemble de modèles. Et ce qui tombe bien, c’est que notre modèle est justement un user, que nous allons créer tout de suite :)

## 1) Modèles et collections

Voici donc notre modèle de User :

    var User = Backbone.Model.extend({
      url: function() {
        if (this.isNew()) {
          return "./users.php";
        } else {
          return "./users.php?id=" + this.id;
        }
      }
    });
    
Typiquement, on vérifie si l’utilisateur existe déjà sur le serveur. C’est pas le cas ? Dans ce cas là, on fait un « GET » sur « users.php », sans paramètre. Il existe ? Eh bien, on va faire un « GET », avec, en paramètre, l’ID de l’utilisateur.
En ce qui concerne la collection :

    var UserCollection = Backbone.Collection.extend({
      url:"users.php",
    });
    
Notre collection se contente de faire un « GET » sur tous les utilisateurs. 
Ainsi, lorsque l’on appellera la méthode « fetch », l’ensemble des objets retournés seront stockés sont forme de modèle dans notre collection. Pour être honnête, dans cet exemple, la collection n’est pas hyper hyper utile, mais elle pourrait peut être le devenir pour un autre tutoriel… :)
Une fois notre collection prête ainsi que notre modèle, on va pouvoir changer la méthode render de notre vue, pour qu’elle récupère la liste des utilisateurs. Pour faire les choses proprement, on va instancier notre collection lors de la création de notre vue :

    var UserListView = Backbone.View.extend({
      el: ".page",
     
      initialize: function() {
        this.users = new UserCollection();
      },
     
      render: function() {
        this.$el.html("Coucou !");
      }
    });
    
Puis nous allons appeler cette collection dans le render, en faisant un fetch. La methode fetch()va faire un « GET » sur « users.php », sans paramètre, puis prendre chaque objet du JSON retourné pour l’insérer dans la collection.

    var UserListView = Backbone.View.extend({
      el: ".page",
     
      initialize: function() {
        this.users = new UserCollection();
      },
     
      render: function() {
        this.users.fetch({
          success: function(users){
            // On affiche les utilisateurs.
            console.log(users.models);
          }
        });
      }
    });
    
Si vous executez vous aurez… Rien pour l’instant. En revanche, si vous êtes sous Chrome, dans l’onglet Network, vous aurez pu voir une requête « GET » passer sous cette forme.

    Request URL:http://localhost/xxxxxx/users.php
    Request Method:GET
    Status Code:200 OK
    
### 2) Le fichier « users.php »

Dans users.php, nous allons juste « écouter » quel type de requête est faite, puis dispatcher vers les bonnes fonctions. La variable $_SERVER est justement là pour nous aider. Celle-ci possède un index REQUEST_METHOD qui nous dit justement quel type de requête. Nous allons donc faire ceci :
    <?php
    switch($_SERVER['REQUEST_METHOD']){
        case 'POST':
            save();
        break;
     
        case 'GET':
            fetch();
        break;
     
        case 'PUT':
            update();
        break;
     
        case 'DELETE':
            delete();
        break;
     
        default:
            echo "erreur dans la méthode requise par le seveur";
        break;
    }
    ?>
Bien entendu, ces fonctions n’existent pas, et nous allons faire la fonction fetch() qui s’exécute lors d’un GET sur le serveur.
Dans cette fonction, nous allons avoir plusieurs étapes :

    switch($_SERVER['REQUEST_METHOD']){
        case 'POST':
            save();
        break;
     
        case 'GET':
            fetch();
        break;
     
        case 'PUT':
            update();
        break;
     
        case 'DELETE':
            delete();
        break;
     
        default:
            echo "erreur dans la méthode requise par le seveur";
        break;
    }
 
    function fetch() {
        // Connexion à la base de données
     
        // La requête à effectuer
     
        // Execution de la requête
     
        // récupération des resultats
     
        // affichage du resultat en json
    }
    
Commençons pas la connexion à la base de données. On va utiliser PDO pour faire quelque chose de « classique » :

    function fetch() {
        // Connexion à la base de données
        $pdo=new PDO("mysql:dbname=BackboneUsers;host=localhost","xxxxx","xxxxx");
        // La requête à effectuer
     
        // Execution de la requête
     
        // récupération des resultats
     
        // affichage du resultat en json
    }
La requête à effectuer est vraiment classique.

    function fetch() {
        // Connexion à la base de données
        $pdo=new PDO("mysql:dbname=BackboneUsers;host=localhost","xxxxx","xxxxx");
        // La requête à effectuer
        $sql = "SELECT * FROM users";
        // Execution de la requête
     
        // récupération des resultats
     
        // affichage du resultat en json
    }
    
Puis on execute notre requête :

    function fetch() {
        // Connexion à la base de données
        $pdo=new PDO("mysql:dbname=BackboneUsers;host=localhost","xxxxx","xxxxx");
     
        // La requête à effectuer
        $sql = "SELECT * FROM users";
     
        // Execution de la requête
        $statement=$pdo->prepare($sql);
        $statement->execute();
     
        // récupération des resultats
     
        // affichage du resultat en json
    }
    
On récupère les résultats

    function fetch() {
        // Connexion à la base de données
        $pdo=new PDO("mysql:dbname=BackboneUsers;host=localhost","xxxxx","xxxxx");
        // La requête à effectuer
        $sql = "SELECT * FROM users";
     
        // Execution de la requête
        $statement=$pdo->prepare($sql);
        $statement->execute();
     
        // récupération des resultats
        $results=$statement->fetchAll(PDO::FETCH_ASSOC);
     
        // affichage du resultat en json
        die(json_encode($results));
    }
    
Puis, afin d’avoir une réponse, on l’affiche en json. Pour pouvoir tester, il faut insérer quelques users en base, rafraichir la page et chercher une requête GET de users.php. Localisez le et affichez la réponse : si tout va bien, c’est tous les utilisateurs de la base :)
Il ne manque plus qu’à les afficher dans le html. Pour ça, on va utiliser UnderscoreJS

### Retour vers index.html

On va retourner dans notre vue, et plutôt que de faire un console.log, on va ajouter ceci. Explication en suivant !

    var UserListView = Backbone.View.extend({
      el: ".page",
     
      initialize: function() {
        this.users = new UserCollection();
      },
     
      render: function() {
        var that = this;
        this.users.fetch({
          success: function(users){
            // On affiche les utilisateurs.
              var template = _.template($('#user-list-template').html(), {users: users.models});
              that.$el.html(template);
          }
        });
      }
      
Ici, on crée un template. Ce template va utiliser la div qui possède un ID #user-list-template (qui n’existe pas et que l’on va créer), à laquelle on va transférer les modèles de notre collection dans la variable « users ».
Enfin, on va insérer le template créé dans notre el 
En ce qui concerne le var that = this , c’est pour transférer le this(notre view) de notre modèle vers le success de notre fetch
Commençons donc par créer cet #user-list-template. En fait, c’est un script de type « template », auquel on donne l’ID donné plus haut. Copiez ce code en dessous du body.

    <script type="text/template" id="user-list-template">
    <table class="table striped">
        <thead>
            <tr>
                <th>Prénom</th>
                <th>Nom</th>
                <th>Age</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <% _.each(users, function(user) {%>
                <tr>
                    <td><%= user.get('prenom') %></td>
                    <td><%= user.get('nom') %></td>
                    <td><%= user.get('age') %></td>
                    <td></td>
                </tr>
            <% }); %>
        </tbody>
    </table>
    </script>

Vous avez remarqué les <% %> ? C’est dedans qu’on exécute le code de Underscore. C’est ici que l’on récupère la variable users, qu’on a transféré dans notre vue, et sur laquelle on fait un foreach
Retournez sur votre serveur, actualisez et… tadam ! :)

## Conclusion

C’est ici que je vais terminer ce premier tutoriel sur BackboneJS, PHP et MySQL. C’est déjà une très grande introduction.
Le code est dispo sur Github ici : https://github.com/kai23/BackboneUsers
