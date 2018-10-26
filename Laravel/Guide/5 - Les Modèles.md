# Les Modèles

Les modèles sont des fichiers PHP permettant d'ajouter, d'obtenir et modifier des éléments dans notre base de donnée.

De manière général, il est nécessaire de générer 1 modèle par table, **sauf les tables pivots**.

**CRÉATION D'UN MODÈLE**

Afin de générer un modèle, il est possible d'utiliser l'outil `Artisan` :

```bash
php artisan make:model Models/[NomDuModele]
```

**REMARQUES**

- Lors de l'exécution de cette commande, Laravel génèrera les modèles dans le dossier `app`. Afin de mieux organiser notre application, nous allons les générer dans un sous-dossier nommé `Models`.
- Par convention, les modèles sont au singulier

**CAS PRATIQUE**

Pour notre application, nous disposons de 5 tables, mais l'une d'elle : `category_post` est une table **pivot**, c'est-à-dire qu'elle permet la relation entre 2 tables : `categories` et `posts`.

```bash
php artisan make:model Models/Category
php artisan make:model Models/Comment
php artisan make:model Models/Post
php artisan make:model Models/User
```

**LES RELATIONS**

Les modèles permettent également d'établir les différentes relations entre nos tables, permattant ainsi d'accéder à différentes données facilement.

**CRÉATION DES RELATIONS**

Afin de définir une relation, il est nécessaire d'ajouter une méthode à notre modèle. Le nom de la méthode **est important** : elle définie le nom du champ établissant la relation.

De nombreux types de relations existent : `belongsTo`, `belongsToMany`, `hasOne`, `hasMany`, `hasManyThrough`. [Consultez la documentation de Laravel pour en savoir plus.](https://laravel.com/docs/5.7/eloquent-relationships#defining-relationships)

```php
public function maMethode() {
    return $this->belongsTo(Model::class);
}
```

**CAS PRATIQUE**

Un article :
- appartient à un utilisateur
- appartient à plusieurs catégories
- possède plusieurs commentaires

Un commentaire :
- appartient à un utilisateur
- appartient à un article

Une catégorie :
- possède plusieurs articles

Un utilisateur :
- possède plusieurs articles
- possède plusieurs commentaires

`app/Models/Post.php` :
```php
class Post extends Model
{

    protected $fillable = ["title", "content"];

    /**
     * Un article appartient à un utilisateur
     *
     * @return BelongsTo
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }


    /**
     * Un article possède plusieurs commentaires
     *
     * @return HasMany
     */
    public function comments() : HasMany
    {
        return $this->hasMany(Comment::class);
    }


    /**
     * Un article appartient à plusieurs catégories
     *
     * @return BelongsToMany
     */
    public function categories(): BelongsToMany
    {
        return $this->belongsToMany(Category::class);
    }

}
```

`app/Models/Comment.php` :
```php
class Comment extends Model
{
    protected $fillable = ["content"];

    /**
     * Un commentaire appartient à un utilisateur
     *
     * @return BelongsTo
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Un commentaire appartient à un article
     *
     * @return BelongsTo
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

`app/Models/Category.php` :
```php
class Category extends Model
{
    protected $fillable = ["name"];

    /**
     * Une catégories "appartient à" plusieurs posts
     * 
     * @return BelongsToMany
     */
    public function posts(): BelongsToMany
    {
        return $this->belongsToMany(Post::class);
    }

}
```

**REMARQUE**

Il s'agit d'une relation `BelongsToMany` puisqu'on utilise une table pivot.

`app/Models/User.php` :
```php
class User extends Model
{
    protected $fillable = ["username", "email", "password"];

    /**
     * Un utilisateur possède plusieurs articles
     *
     * @return HasMany
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }

    /**
     * Un utilisateur possède plusieurs commentaires
     *
     * @return HasMany
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }

}
```

**EXPLICATIONS**

La variable protégée `protected $fillable` permet à Laravel de savoir quels champs sont modifiables par un utilisateur. Laravel ne permettra pas l'ajout ou la modification des champs n'étant pas présent dans cette variable.

Maintenant que nos modèles sont prêt, nous pourrons les utiliser afin de créer, obtenir, modifier et supprimer des données de nos différentes tables.