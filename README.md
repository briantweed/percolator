# laravel-search-builder

*Automatically create and apply a search query to a collection built using model scope methods*

Filter forms can get messy. Checking which fields have selected, piecing together query string snippets; and just when you've got everything working along comes another handful of fields to filter on. The code can get really messy, really quickly. One solution I've seen is to have each filter correspond to a class containing the desired query string. While this works well, the solution I've tried to implement uses one of Laravel's existing features - scopes.

**LaravelSearchBuilder** takes each form field name, finds the corresponding model scope and adds it to the query builder. And that's it. As long as the naming conventions are followed and the scope exists, the filters will be automatically applied when required.

<br/>

### Installation

```
composer require "briantweed/laravel-search-builder"
```
<br>

Once installed you can publish the `config/builder.php` file by referencing the provider:
```
php artisan vendor:publish --provider="briantweed\LaravelSearchBuilder\LaravelSearchBuilderServiceProvider"
```
or via the tag:
```
php artisan vendor:publish --tag="builder"
```

<br>

### How to Use

Initialise the SearchBuilder class by passing an instance of the model you're running the query on and the request:

```php
public function index(Request $request)
{
    $results = (new SearchBuilder(new Model, $request))->apply();
}
```

<br>

### Naming Convention

The `config/builder.php` contains several keywords and values that are used by **LaravelSearchBuilder** to determine the scope method names.

By default the `where_scope` value is `where` and should be used when naming any scope that you want **LaravelSearchBuilder** to use. For example, for a form field with the name `rating` **LaravelSearchBuilder** will look for a scope called `scopeWhereRating`.

Similarly, the `sort_scope` default value is `by` and **LaravelSearchBuilder** will look for a scope called `scopeByRating`.

The `sort` value needs to be the same as the sorting form field name in order for  **LaravelSearchBuilder** to recognise it as such. The same applied to the `order` value.

Finally the `related_table_separator` is used when filtering by fields from related models. The naming convention for this is the realted model name, followed by the `related_table_separator` and the related model field name. For example `location__name`.


---

*Form*
```html 
<form>
  <input type="text" name="location" value="" />
  <input type="text" name="rating" value="" />
  <select name="sort">
    <option value="location">Location</option>
    <option value="rating">Rating</option>
  </select>
  <input type="submit" value="submit" />
</form>
```

*Controller*
```php 
  use App/Model;
  use briantweed/laravel-search-builder;
  
  class ModelController
  {
  
    public function index(Request $request)
    {
      $results = (new SearchBuilder(new Model, $request))->apply()
      return view('index', [
        'results' => $results
      ]);
    }
    
  }
```

*Model*
```php

  class Model
  {
    public function scopeWhereLocation($query, $value)
    {
      return $query->where('location', '=', $value);
    }
    
    public function scopeWhereRating($query, $value)
    {
      return $query->where('rating', '>=', $value);
    }
    
    public function scopeByLocation($query, $direction = 'asc')
    {
    		return $query->orderBy('location', $direction);
    }
    
    public function scopeByRating($query, $direction = 'desc')
    {
    		return $query->orderBy('rating', $direction);
    }
  }
```
