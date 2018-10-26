# Les Contrôleurs

Les contrôleurs possèdent la pluaprt de la logique de notre application. \
Lorsque l'utilisateur essaie d'accéder à une ressource, le `routeur` fait appel à une méthode d'un contrôleur.

**CREATION D'UN CONTRÔLEUR**

Afin de générer un contrôleur, il est possible d'utiliser l'outil `Artisan` :

```bash
php artisan make:controller [nomDuController] [--resource] [--api]
```

**EXPLICATIONS**

La commande génèrera un fichier `app/Http/Controller/nomDuController.php`
- `nomDuController` devrait être au singulier et se finir par Controller (*exemples: `UserController`, `PostController`, ...*)
- `--resource` permet de pre-remplir le contrôleur avec les méthodes usuellement utilisées pour une ressource : `index`, `create`, `store`, `show`, `edit`, `update`, et `destroy`.
- `--api` permet de pre-remplir le contrôleur avec les méthodes usuellement utilisées pour une ressource dans une API : `index`, `store`, `show`,`update`, et `destroy`.

**UTILISATION D'UN CONTRÔLEUR**

Le contrôleur contient la logique de notre application, c'est par exemple lui qui permettra l'ajout, la lecture, la modification et la suppression d'un élément. Les fonctions usuelles pré-générés avec la commande --resource sont :

| Méthode | Description                                                                                              |
| ------- | -------------------------------------------------------------------------------------------------------- |
| index   | Retourne la liste de tout les éléments.                                                                  |
| create  | Retourne le formulaire permettant la création d'un nouvel élément                                        |
| store   | Valide les données envoyées et ajoute le nouvel élément en BDD                                           |
| show    | Retourne un élément précis, dont l'identifiant est donné en paramètre                                    |
| edit    | Retourne le formualire permettant l'édition d'un élément, dont l'identifiant est donné en paramètre      |
| update  | Valide les données envoyées et modifie un élément dans la BDD, dont l'identifiant est donné en paramètre |
| destroy | Supprime l'élément de la BDD                                                                             |

**CAS PRATIQUE**

Nous devons générer 4 contrôleurs : `UserController`, `PostController`, `CategoryController`, et `CommentController`. Chacun de ces contrôleur correspond à un CRUD. Nous pouvons donc utiliser les commandes suivantes :

```bash
php artisan make:controller UserController --resource
php artisan make:controller PostController --resource
php artisan make:controller CategoryController --resource
php artisan make:controller CommentController --resource
```

Nous remplirons ces contrôleurs en temps voulu.