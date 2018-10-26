# TP - Les utilisateurs

Dans ce TP, nous allons mettre en place la gestion des utilisateurs.

**REMARQUE**

Laravel inclu par défaut des outils permettant de générer un système d'enregistrement, et de connexion. Nous n'allons pas l'utiliser afin d'apprendre à faire notre propre système de gestion des utilisateurs.

**ETAPE 01 - NOUVEAUX UTILISATEURS**

Nous souhaitons afficher un formulaire permettant l'enregistrement d'un utilisateur lorsque la page demandée est : `/users/create`.

La route existe déjà grâce a `Route::resource("users", "UserController");` : la méthode appelée lors de l'accès à cette page est dans le contrôleur `UserController`, la méthode `create`.

```php
/**
 * Show the form for creating a new resource.
 *
 * @return \Illuminate\Http\Response
 */
public function create()
{
    //
}
```

Lors de l'accès à cette page, nous souhaitons juster afficher une page HTML contenant notre formulaire. Aucun traitement supplémentaire n'est nécessaire. Afficher une page HTML consiste à **retourner une vue**.

**ETAPE 01 - CRÉATION DE LA VUE**

Dans le dossier `resources/views` :
- supprimer le fichier `welcome.blade.php` qui est la page par défaut de Laravel
- créer un dossier `blog`
- créer un dossier `users` dans le dossier `blog`
- créer le fichier `register.blade.php` dans le dossier `users`

L'extension `blade.php` permet d'indiquer à Laravel que nous allons utiliser `Blade`, un [`moteur de template`](https://laravel.com/docs/5.7/blade) permettant d'inclure facilement du PHP dans nos vues.

**ÉTAPE 02 - AJOUT DU FORMULAIRE**

Afin de nous faciliter la tâche, nous ne ferons pas de JavaScript ni de CSS durant ce guide. Nous utiliserons le framework CSS [Bootstrap](https://getbootstrap.com)

Notre formulaire d'enregistrement devra contenir :
- un champ email
- un champ username
- deux champs password
- un bouton de validation

```html
<!doctype html>
<html lang="en">
    <head>
        <!-- Required meta tags -->
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Bootstrap CSS -->
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">

        <title>Blog - Register</title>
    </head>
    <body>
        <div class="container">
            <div class="d-flex align-items-center" style="height: 100vh;">
                <form class="col-12" method="POST" action="{{ route("users.store") }}">
                    {{ csrf_field() }}

                    <h4 class="text-center">Register a new user</h4>
                    <div class="form-group">
                        <label for="email">Email address</label>
                        <input id="email" class="form-control" type="email" name="email" placeholder="E.g : email@example.com">
                        <small id="emailHelp" class="form-text text-muted">We'll never share your email with anyone else.</small>
                    </div>

                    <div class="form-group">
                        <label for="username">Username</label>
                        <input id="username" class="form-control" type="text" name="username" placeholder="E.g : TiCubius">
                    </div>

                    <div class="form-group">
                        <label for="password">Password</label>
                        <input id="password" class="form-control" type="password" name="password" placeholder="E.g : My$up3rP4550rD">
                    </div>

                    <div class="form-group">
                        <label for="password_confirm">Password confirmation</label>
                        <input id="password_confirm" class="form-control" type="password" name="password_confirmation" placeholder="E.g : My$up3rP4550rD">
                    </div>

                    <button type="submit" class="btn btn-outline-primary float-right">Submit</button>
                </form>
            </div>
        </div>
    </body>
</html>
```

**ÉTAPE 03 - MODIFICATION DU CONTRÔLEUR**

Maintenant que notre vue existe, nous pouvons modifier la méthode `create` afin de la retourner :

```php
/**
 * GET - Affiche le formulaire de création d'un utilisateur
 *
 * @return View
 */
public function create(): View
{
    return view("blog.users.register");
}
```

**REMARQUE**

- Le lien du fichier se fait a partir du dossier `resources/views`
- Les `/` sont remplacés par des `.`
- Il n'y a pas besoin d'indiquer l'extension

**ÉTAPE 04 - VÉRIFICATION**

Vous pouvez maintenant accéder à la page `/users/create` et vous pourrez voir votre formulaire d'enregistrement.

**ÉTAPE 05 - ENVOIE DU FORMUALIRE**

Nous n'avons pour le moment pas indiquer où envoyer les informations renseignés par l'utilisateur : 

``` html
<form class="col-12">
    ...
</form>
```

Il s'agit d'un envoi de nouvelle donnée permettant la **création d'un utilisateur**, la requête devra donc être en **POST**. La **route** existe déjà et est déjà nommé correctement : **`users.store`**

**CAS PRATIQUE**

``` html
<form class="col-12" method="POST" action="{{ route("users.store") }}">
    ...
</form>
```

**EXPLICATIONS**

- `Blade` permet d'utiliser des méthodes PHP, des boucles, des conditions, ... facilement grâce à `{{  }}` et `@`.
- La méthode `route` permet de récupérer l'`URL` associé au nom d'une route.
- La route `users.store` a été généré par `Route::resource("users", "UserController");` et correspond a l'URL `/users`

**ÉTAPE 06 - TENTATIVE D'ENVOI**

Remplissez le formulaire et tentez de le soumettre. \
Laravel vous affiche un erreur `419: Sorry, your session has expired.`

**EXPLICATIONS**

Par défaut, Laravel inclu un système de protection contre les attaques [**Cross-Site Request Forgery**](https://fr.wikipedia.org/wiki/Cross-site_request_forgery).

Afin de faire fonctionner **tout les formulaire de notre site**, il est nécessaire d'ajouter un champ :

```html
<form class="col-12" method="POST" action="{{ route("users.store") }}">
    {{ csrf_field() }}
</form>
```

Après avoir ré-actualiser votre page d'enregistrement, soumettez de nouveau votre formulaire.

**ÉTAPE 07 - TRAITEMENT DES DONNÉES**

Lors de la soumission du formulaire, toutes les données sont envoyées en `POST` sur `/users`. \
Le routeur fait donc appel à la méthode `store` de notre `UserController` :

```php
/**
 * Store a newly created resource in storage.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */
public function store(Request $request)
{
    //
}
```

Avant d'insérer ces données dans notre base de donnée, il est nécessaire de les **valider**.

Laravel inclu [un outil permettant d'effectuer ces validations automatiquement](https://laravel.com/docs/5.7/validation)

```php
$request->validate([
    "username" => "required|max:255|unique:users,username",
    "email"    => "required|max:255|email|unique:users,email",
    "password" => "required|min:8|confirmed"
]);
```

**EXPLICATIONS**

- Tout les champs sont requis
- Les champs `username` et `email` doivent être unique
- Les champs `username` et `email` ne doivent pas être plus long que 255 caractères (limitation du VARCHAR dans MySQL)
- Le champ `email` doit être correctement formaté (`X@Y.Z`)
- Le champ `password` doit être au moins de 8 caractères (sécurité)
- La champ `password` doit être identique au `password_confirmation`
- Les règles sont séparés par des `|`

Si les données envoyées sont incorrectes, Laravel redirige automatiquement l'utilisateur sur le formulaire d'ajout.

Si les données sont correctes, nous pouvons utiliser le modèle `User` afin d'ajouter un utilisateur dans notre base de donnée :

```php
User::create([
    "username" => $request->input("username"),
    "email"    => $request->input("email"),
    "password" => Hash::make($request->input("password"))
]);
```

**EXPLICATIONS**

- `$request->input()` permet d'accéder à une donnée envoyée grâve au nom du champ HTML
- `Hash::make` permet d'enregistrer le HASH du mot de passe : **NE JAMAIS ENREGISTRER LES MOTS DE PASSE EN CLAIR !** 

Exemple de mot de passe : *`$2y$10$mlY0xSTQjc4uHnPoJG94eeEt/luXDy0GvYxXHAkhsU7RiUiEU3BU2`*

La création d'utilisateurs est fonctionne mais :
- aucun message d'erreur n'apparait en cas de problème
- l'utilisateur est redirigé sur une page blanche après enregistrement