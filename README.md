Laravel Translator
==================

> Please **NOTE** the master branch is under development for Laravel 5.

![image](https://raw.githubusercontent.com/vinkla/vinkla.github.io/master/images/laravel-translator.png)

This package gives you an easy way to translate Eloquent models into multiple languages.

```php
// Display the default title for an Eloquent object.
echo $foo->title;

// Change the current language to Swedish.
App::setLocale('sv');

// Display the translated title in Swedish.
echo $foo->title;
```
Read more about how this package was created and why it exists [in this blog post](http://vinkla.com/2014/11/laravel-translator/).

[![Build Status](https://img.shields.io/travis/vinkla/translator/master.svg?style=flat)](https://travis-ci.org/vinkla/translator)
	[![Latest Stable Version](http://img.shields.io/packagist/v/vinkla/translator.svg?style=flat)](https://packagist.org/packages/vinkla/translator)
	[![License](https://img.shields.io/packagist/l/vinkla/translator.svg?style=flat)](https://packagist.org/packages/vinkla/translator)

## Installation

Require this package, with [Composer](https://getcomposer.org/), in the root directory of your project.

```bash
composer require vinkla/translator
```

Add the service provider to `config/app.php` in the providers array.

```bash
'Vinkla\Translator\TranslatorServiceProvider'
```

#### Looking for a Laravel 4 compatible version?

Please use `1.0` branch instead. Installable by requiring:

```bash
composer require "vinkla/translator=~1.0"
```

## Configuration

To add the configuration files to the `app/config/packages` directory, run the command below.
```bash
php artisan vendor:publish
```

## Documentation

Below we have examples of [migrations](#migrations), [models](#models), [seeds](#seeds) and [templates](#templates).

## Migrations

Here's an example of the localisations migration.

```php
Schema::create('locales', function(Blueprint $table)
{
	$table->increments('id');
	$table->string('language', 2); // en, sv, da, no, etc.
	$table->timestamps();
});
```

This example migration comes out of the box with this package. Run the command below to add it in your database.
```bash
php artisan migrate --package="vinkla/translator"
```

Add the Laravel migration for the base table which you want to translate.

```php
Schema::create('articles', function(Blueprint $table)
{
	$table->increments('id');
	$table->string('thumbnail');
	$table->timestamps();
});
```

Add the Laravel migration for the translatable relation table.

```php
Schema::create('article_translations', function(Blueprint $table)
{
	$table->increments('id');

	// Translatable attributes
	$table->string('title');
	$table->string('content');
	// Translatable attributes

	$table->integer('article_id')->unsigned()->index();
	$table->foreign('article_id')->references('id')->on('articles')->onDelete('cascade');

	$table->integer('locale_id')->unsigned()->index();
	$table->foreign('locale_id')->references('id')->on('locales')->onDelete('cascade');

	$table->unique(['article_id', 'locale_id']);

	$table->timestamps();
});
```

## Models
Firstly you'll need to setup the `Locale` Eloquent model. Then add the `Locale` model path to the configuration file.

```php
<?php namespace Acme\Locales;

use Illuminate\Database\Eloquent\Model;

class Locale extends Model {

	/**
	 * @var array
	 */
	protected $fillable = ['language'];

}
```

Here's an example of a translatable Laravel Eloquent model. Remember to fill the `$fillable` array the translatable attributes.


```php
<?php namespace Acme\Articles;

use Illuminate\Database\Eloquent\Model;
use Vinkla\Translator\Translatable;

class Article extends Model {

	use Translatable;

	/**
	 * @var array
	 */
	protected $fillable = ['title', 'content', 'thumbnail'];

	/**
	 * @var string
	 */
	protected $translator = 'Acme\Articles\ArticleTranslation';

	/**
	 * @var array
	 */
	protected $translatedAttributes = ['title', 'content'];

}
```

The ArticleTranslation basically is an empty Eloquent object.
```php
<?php namespace Acme\Articles;

use Illuminate\Database\Eloquent\Model;

class ArticleTranslation extends Model {}
```

## Seed
Before you start to populate your database with translations you'll need to add languages to the locales table that you want to support. Below is an example seeder.

```php
<?php

use Acme\Locales\Locale;
use Illuminate\Database\Seeder;

class LocaleTableSeeder extends Seeder {

	public function run()
	{
		$languages = ['en', 'sv', 'no'];

		foreach ($languages as $language)
		{
			Locale::create(compact('language'));
		}
	}
}
```

### Templates

That's it! You're done. Now you can do:
```php
<h1>{{ $article->title }}</h1>
<img src="{{ $article->thumbnail }}">
<p>{{ $article->content }}</p>
```

If you want to fetch a specific translation that isn't the current one you can specify it in the translate method as in the example below.
```php
<h1>{{ $article->translate('sv')->title }}</h1>
<img src="{{ $article->thumbnail }}">
<p>{{ $article->translate('sv')->content }}</p>
```

## Contributing

Thank you for considering contributing to the Translator package. If you are submitting a bug-fix, or an enhancement that is **not** a breaking change, submit your pull request to the the latest stable release of the package, the `master` branch. If you are submitting a breaking change or an entirely new component, submit your pull request to the `develop` branch.

## License

The Translator package is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).
