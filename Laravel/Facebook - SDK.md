# Laravel - Facebook API

Si vous souhaitez uniquement utiliser l'API de Facebook afin de donner à vos utilisateurs une manière rapide de s'inscrire et de se connecter (Login With Facebook), vous devriez utiliser la librairie [Laravel Socialite](https://laravel.com/docs/5.6/socialite), mais si vous avez besoin d'utiliser d'autres fonctionnalités de l'API Facebook, telle que l'envoie de Post, il est nécessaire d'utiliser une autre librairie.

Dans notre exemple, nous souhaitons utiliser l'API de Facebook afin de permettre à notre utilisateur de s'authentifier, mais également d'envoyer automatiquement un Post sur une page Facebook dont l'utilisateur est propriétaire.

**REMARQUE IMPORTANTE**:

* Tout au long du guide, le code des méthodes sera indiqué. Cependant, le code en début de fichier comme celui ci-dessous ne le sera pas. Faites attention à bien ajouter les classes nécessaires.
* Lorsque le code d'une méthode est indiqué, la première ligne correspond à l'emplacement du fichier dans le projet. Elle n'est pas à rajouter dans votre code.
* Lorsque les fichiers sont longs, il se peut que certaines parties ne soient pas affichées. Si cela devait arriver, **`...`** remplacera le code.
* Votre application Laravel doit être **accessible via HTTPS**.

### EXEMPLE 

```php
// app/Http/Controllers/UserController.php

<?php

namespace App\Http\Controllers;

use Facebook\Exceptions\FacebookResponseException;
use Illuminate\Support\Facades\Session;
use Vinkla\Facebook\Facades\Facebook;

class UserController extends Controller
{
    ...
}
```

Initialisation de notre Projet
==============================

## ÉTAPE 01 - Création d'une Application Facebook

* Rendez-vous sur [le site Facebook For Developer](https://developers.facebook.com/)
* Dans la barre de navigation, allez sur `Mes Applications`
* Cliquez sur le bouton `Ajouter une application`
* Donnez-lui un `Nom d'usage` et une `Adresse e-mail de contact`

Dans notre exemple, notre application s'appellera `AirsoftAPI`.

## ÉTAPE 02 - Création d'un Projet Laravel

* Installez correctement [les dépendances de Laravel](https://laravel.com/docs/5.6#server-requirements)
* Installez correctement [composer](https://getcomposer.org/download/)
* Créer un répertoire vide et déplacez-vous à l'intérieur
* Effectuez les commandes suivantes:

```bash
composer create-project --prefer-dist laravel/laravel .
composer require vinkla/facebook
composer require barryvdh/laravel-debugbar --dev
```

## ÉTAPE 03 - Récupération des informations

Afin de faire fonctionner notre application, nous aurons besoin des informations suivantes:

* `APP ID` de votre application, trouvable sur [le site Facebook For Developer](https://developers.facebook.com/)
* `CLÉ SECRÈTE` de votre application, trouvable sur [le site Facebook For Developer](https://developers.facebook.com/)

**Ces données sont propres à votre application et doivent rester secrète !**

## ÉTAPE 04 - Modification du fichier .env

Le fichier `.env` contient toutes les informations importantes relative à votre projet Laravel. Il contient les mots de passe et clés importantes utilisés par votre projet.\
Rajoutez et complétez les lignes suivantes dans votre fichier `.env` avec les informations trouvées précédemment.

```php
// .env

FACEBOOK_APP_ID=
FACEBOOK_APP_SECRET=
```

## ÉTAPE 05 - Modification de la configuration des libraires

Nous avons précédemment installé la librairie `vinkla/facebook`, mais nous ne l'avons pas encore configurée.\
Exécutez la commande suivante et sélectionnez `Provider: Vinkla\Facebook\FacebookServiceProvider`:

```bash
php artisan vendor:publish
```

Cette commande générera le fichier de configuration de la librairie afin que nous puissions le modifier. Il se situera à l'emplacement `config/facebook.php`.\
Modifiez les lignes `app_id` et `app_secret` en indiquant la variable d'environnement correspondante.

```php
<?php
...
    'connections' => [

        'main' => [
            'app_id' => env("FACEBOOK_APP_ID"),
            'app_secret' => env("FACEBOOK_APP_SECRET"),
            'default_graph_version' => 'v2.10',
            //'default_access_token' => null,
        ],
        ...
    ],
    ...
```

FacebookAPI - Authentification d'un utilisateur avec Facebook
=============================================================

Nous souhaitons que notre utilisateur puisse s'inscrire et se connecter rapidement à notre projet Laravel grâce à son compte Facebook. Pour cela, nous aurons besoin des informations suivantes: `identifiant Facebook`, `nom`, `prénom`, `e-mail`, `date de naissance` et `genre`.

## ÉTAPE 01 - Création du contrôleur utilisateur

Nous allons créer un contrôleur vide, nommé `UserController` qui se chargera des utilisateurs de notre projet. (Nous ne montrerons que la connexion)

```bash
php artisan make:controller UserController
```

## ÉTAPE 02 - Gestion des routes

Afin de réaliser la connexion via Facebook à notre projet Laravel, nous aurons besoin de 2 routes:

* La première, `/login/facebook` se chargera de rediriger les utilisateurs vers Facebook
* La seconde, `/login/facebook/callback` récupérera les informations envoyées par Facebook

La gestion des routes de votre projet Laravel se fait dans le fichier `routes/web.php`.

```php
// routes/web.php

<?php

Route::get("/login/facebook", "UserController@facebookLogin");
Route::get("/login/facebook/callback", "UserController@facebookLoginCallback")->name("facebook.login.callback");
```

**Remarque: nous avons donné un nom à la route qui nous sert de callback**.

## ÉTAPE 03 - Configuration de l'application Facebook

Lorsque l'utilisateur se sera authentifié par Facebook, il sera redirigé sur une page de notre application: la route `/login/facebook/callback`. Il existe une sécurité au niveau de Facebook qui empêche la redirection de l'utilisateur sur une page inconnue. Il est donc nécessaire d'indiquer l'URL exacte sur laquelle l'utilisateur sera redirigée.

>Exemple: si notre application répond sur `https://example.com/`, au vu de la configuration des routes réalisée précédemment, l'utilisateur se connectera sur `https://example.com/login/facebook` et sera redirigé par Facebook sur **`https://example.com/login/facebook/callback`**. \
Il est donc nécessaire d'enregistrer cette dernière URL sur le site de Facebook.

* Rendez-vous sur [votre application Facebook](https://developers.facebook.com/)
* Cliquez sur `Facebook Login` en desous de `PRODUITS (+)`
* Dans le champ `URI de redirection OAuth valides` rentrez **le lien de votre site internet sur lequel les visiteurs seront redirigés une fois connectés**
* Enregistrez les modifications

## ÉTAPE 04 - Modification du contrôleur utilisateur

Il est nécessaire de créer les méthodes `facebookLogin` et `facebookLoginCallback` dans le contrôleur `UserController`.

### Méthode **`facebookLogin`**

Afin de trouver les permissions dont vous avez besoin, vous pouvez consulter:

* [La documentation officielle de Facebook](https://developers.facebook.com/docs/facebook-login/permissions/v3.0)
* [L'explorateur de l'API Graph](https://developers.facebook.com/tools/explorer)

Dans notre cas, nous avons besoin des permissions `email`, `user_birthday` et `user_gender`.

```php
// app/Http/Controllers/UserController.php

/**
 * Redirige l'utilisateur sur la page de connexion Facebook
 * @route("/login/facebook/")
 */
public function facebookLogin()
{
    // Démarre la Session afin de stocker des informations
    // Nécessaire afin de faire fonctionner la librairie Facebook
    // https://github.com/facebook/php-graph-sdk/issues/473#issuecomment-158909097
    session_start();

    // Retourne le lien de connexion que l'utilisateur devra utiliser
    // Nécessite une URL de "callback" et les permissions nécessaires

    // Tableau contenant toutes les permissions nécessaires
    // En plus du nom et de l'ID facebook (de base), on souhaite l'E-Mail, la date de naissance et le genre
    // https://developers.facebook.com/tools/explorer
    $permissions = ["email", "user_birthday", "user_gender"];

    // Retourne le lien de connexion via Facebook
    // Nécessite l'URL de callback, et un tableau contenant toutes les permissions
    $link = Facebook::getRedirectLoginHelper()->getLoginUrl(route("login.facebook.callback"), $permissions);

    // Redirige l'utilisateur sur la page de connexion via Facebook
    return redirect($link);
}
```

### Méthode **`facebookLoginCallback`**

Cette méthode nous donne accès aux informations que l'on demande de l'utilisateur. Il suffirait maintenant d'enregistrer ses informations si vous le souhaitiez.

```php
// app/Http/Controllers/UserController.php

/**
 * L'utilisateur s'est authentifié, on reçoit les informations
 * @route("/login/facebook/callback")
 */
public function facebookLoginCallback()
{
    // Démarre la Session afin de stocker des informations
    // Nécessaire afin de faire fonctionner la librairie Facebook
    // https://github.com/facebook/php-graph-sdk/issues/473#issuecomment-158909097
    session_start();


    // Récupère le Token Utilisateur envoyé par Facebook
    // Le Token permet d'accéder aux informations de l'utilisateur et d'effectuer des informations en son nom
    // https://developers.facebook.com/docs/graph-api/using-graph-api#tokens-d-acc-s
    // Instance de FacebookRedirectLoginHelper
    $userToken = Facebook::getRedirectLoginHelper()->getAccessToken();

    // Enregistrement du Token dans la Session
    Session::put("FACEBOOK_USER_TOKEN", $userToken);

    // On souhaite récupérer des information sur l'utilisateur
    // Instance de FacebookResponse
    $response = Facebook::get("me?fields=id,name,birthday,email,gender", $userToken->getValue());

    // On possède les informations de l'utilisateur
    // Tableau
    $user = $response->getDecodedBody():

    // On retourne directement la réponse (format JSON)
    return($user);
}
```

FacebookAPI - Envoie automatique de Posts sur une page Facebook
===============================================================

Maintenant que notre utilisateur est connecté à l'aide de Facebook, nous souhaitons poster un message sur une page Facebook [**dont il est le propriétaire**](https://developers.facebook.com/docs/pages/publishing).

## ÉTAPE 01 - Création d'une page Facebook

* Rendez-vous sur [le site Facebook](https://facebook.com)
* Dans la barre  de navigation, allez sur la flèche vers le bas
* Cliquez sur le bouton `Créer une Page`
* Choisissez entre `Entreprise ou marque` et `Figure locale ou publique`
* Donnez lui un `Nom` et une `Catégorie`

## ÉTAPE 02 - Récupération des informations

Afin de poster des messages sur notre page Facebook, nous aurons besoin des informations suivantes:

* `ID DE LA PAGE`, trouvable dans l'onglet `À Propos` de votre page

## ÉTAPE 03 - Modification du fichier .env

Rajoutez et complétez les lignes suivantes dans votre fichier `.env` avec les informations trouvées précédemment.

```php
// .env

FACEBOOK_PAGE_ID=
```

## ÉTAPE 04 - Création du contrôleur facebook

Nous allons créer un contrôleur vide, nommé `FacebookController` qui se chargera des interactions entre notre projet et l'API de Facebook.

```bash
php artisan make:controller FacebookController
```

## ÉTAPE 05 - Gestion des routes

Pour des raisons de simplicité, nous allons créer une route `/facebook/post` qui, une fois appelée en `GET` postera automatiquement un message sur la page Facebook, si l'utilisateur est connecté et est bien le propriétaire.

```php
// routes/web.php

<?php

Route::get("/facebook/post", "FacebookController@postMessage");
```

## ÉTAPE 06 - Modification du contrôleur utilisateur

Les permissions nécessaires afin de poster un message sur une page Facebook sont différentes de celles nécessaires pour obtenir les informations d'un utilisateur. Nous allons donc devoir modifier notre contrôleur `UserController`.

### Méthode **`facebookLogin`**

Rappel: Afin de trouver les permissions dont vous avez besoin, vous pouvez consulter:

* [La documentation officielle de Facebook](https://developers.facebook.com/docs/facebook-login/permissions/v3.0)
* [L'explorateur de l'API Graph](https://developers.facebook.com/tools/explorer)

Dans notre cas, nous avons besoin des permissions `manage_pages` et `publish_pages`.

```php
// app/Http/Controllers/UserController.php

/**
 * Redirige l'utilisateur sur la page de connexion Facebook
 * @route("/login/facebook/")
 */
public function facebookLogin()
{
    ...
    // Tableau contenant toutes les permissions nécessaires
    // En plus du nom et de l'ID facebook (de base), on souhaite l'E-Mail, la date de naissance et le genre
    // https://developers.facebook.com/tools/explorer
    $permissions = ["email", "user_birthday", "user_gender", "manage_pages", "publish_pages"];
    ...
}
```

### Méthode **`facebookLoginCallback`**

Il est nécessaire de modifier la méthode `facebookLoginCallback` également afin d'enregistrer dès la connexion le `Token de la page` qui nous sera nécessaire plus tard.

```php
// app/Http/Controllers/UserController.php

/**
 * L'utilisateur s'est authentifié, on reçoit les informations
 * @route("/login/facebook/callback")
 */
public function facebookLoginCallback()
{
    ...
    // On cherche toutes les pages dont l'utilisateur est le propriétaire
    // Il faut donc utiliser le Token Utilisateur
    // Instance de FacebookResponse
    $json = Facebook::get("me/accounts", $userToken->getValue());

    // On a récupérer toutes les informations, format JSON
    // On traite le JSON
    $pages = (json_decode($json->getBody()));

    // On cherche dans le JSON la page qui nous intéresse
    // Une fois trouvée, on ajoute le Token de la PAGE à la Session
    // Si l'utilisateur n'est pas propriétaire de la page, il n'y aura pas de FACEBOOK_PAGE_TOKEN dans la Session
    foreach ($pages->data as $page) {
        if ($page->id == env("FACEBOOK_PAGE_ID")) {
            Session::put("FACEBOOK_PAGE_TOKEN", $page->access_token);
        }
    }

    if (Session::has("FACEBOOK_PAGE_TOKEN")) {
        // Maintenant que l'on possède toutes les informations
        // On redirige l'utilisateur sur notre application
        // (Ici, sur une page postant automatiquement un
        // post sur la page Facebook)
        return redirect(route("facebook.post"));
    }

    // L'utilisateur n'était pas propriétaire de la page
    return "Pas de Token de page !";
```

## ÉTAPE 07 - Modification du contrôleur facebook

Il est nécessaire de créer les méthodes `postMessage` dans le contrôleur `FacebookController`. \
Cette méthode vérifiera la présence du `FACEBOOK_PAGE_TOKEN` dans la Session et essaiera d'envoyer le message "Hello World!"

### Méthode **`postMessage`**

```php
// app/Http/Controllers/FacebookController.php

/**
 * Poste un message sur la page Facebook
 * @route("/facebook/post")
 */
public function postMessage()
{
    // FACEBOOK_PAGE_TOKEN n'existe que si l'utilisateur est authentifié et qu'il est propriétaire de la bonne page Facebook. [devrait être dans un Middleware]
    if (Session::has("FACEBOOK_PAGE_TOKEN")) {

        // On récupère le Token de la page
        $pageToken = Session::get("FACEBOOK_PAGE_TOKEN");

        // On essaie de poster sur la page Facebook
        // https://developers.facebook.com/docs/pages/publishing#post
        try {

            // POST https://graph.facebook.com/{FACEBOOK_PAGE_ID}/feed
            // Params: ["message" => "Hello World!"]
            // Token: $pageToken
            $post = Facebook::post(env("FACEBOOK_PAGE_ID") . "/feed", [
                "message" => "Hello World!"
            ], $pageToken);

        } catch (FacebookResponseException $e) {
            // En cas d'erreur (ex: duplicata de Message - on ne peut pas envoyer 2 fois le même message)
            dd($e);
        }

        return "Hello World!";

    } else {
        return "No Facebook page Token!";
    }
}
```

> De la même manière, en utilisant les méthodes `Facebook::get` et `Facebook::post`, il est possible d'utiliser toutes les méthodes de l'API Facebook.
