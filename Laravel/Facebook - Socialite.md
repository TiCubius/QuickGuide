# LARAVEL - Socialite Guide - Facebook Login

Remarques:

* Votre application Laravel doit être **accessible via HTTPS**.

## INSTALLATION DE SOCIALITE

A l'aide de composer, vous devrez installer la dépendance suivante:

```bash
composer require laravel/socialite
```

## CONFIGURATION DE SOCIALITE

Dans le fichier `config/services.php`, rajouter la clé `facebook`:

```php
<?php

    return [
        ...

        'facebook' => [
            'client_id' => env("FACEBOOK_APP_ID"),
            "client_secret" => env("FACEBOOK_APP_SECRET"),
            "redirect" => env("FACEBOOK_URL"),
        ],

        ...

    ];
```

## CREATION D'UN CONTROLLER POUR L'AUTHENTIFICATION

A l'artisan, générer le controller puis éditez-le :

```bash
php artisan generate:controller FacebookController
```

Éditez le fichier `app/Http/Controllers/FacebookController.php`:

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Laravel\Socialite\Facades\Socialite;

class FacebookController extends Controller
{
    /**
     * Redirige sur la page d'authentification Facebook
     */
    public function redirect() {

        return Socialite::driver("facebook")->redirect();
    }

    /**
     * Récupère les informations de l'utilisateur une fois connecté
     */
    public function callback() {

        $user = Socialite::driver("facebook")->user();

        // Ici, réalisez les actions
        // avec votre utilisateur...
        dd($user);
    }
}
```

## CREATION DES ROUTES

Éditez votre fichier `routes/web.php`:

```php
Route::get("/facebook", "FacebookController@redirect");
Route::get("/facebook/callback", "FacebookController@callback");
```

## CONFIGURATION DU FICHIER .ENV

Dans le fichier `.env`, rajoutez les lignes suivantes:

```php
FACEBOOK_ID=123456
FACEBOOK_SECRET=abcdef
FACEBOOK_URL=https://192.168.43.100/LaravelSocialite/public/facebook/callback
```

La ligne `FACEBOOK_ID` correspond à l'ID de votre application Facebook, trouvable sur [cette page](https://developers.facebook.com/apps/). \
La ligne `FACEBOOK_SECRET` correspond à la clé secrète de votre application Facebook, trouvable dans l'onglet Paramètre Général, de votre **Tableau de bord** de votre application. \
La ligne `FACEBOOK_URL` correspond à l'URL sur laquelle les utilisateurs seront redirigés une fois connectés via Facebook.\
**Elle doit correspondre à une URL de votre application** (ici la route `/facebook/callback`)
