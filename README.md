Laravel Widgetize
=================

###Widget Objects help you to have __cleaner code__ and __easy caching__.

* [Introduction](#introduction)
    - [When to use the _widget_ concept?](#when-to-use-the-widget-concept)
    - [The Problems](#what-is-our-problems)
    - [The Solution](#what-is-the-solution)
* [Installation](#installation)
* [Configuration](#configuration)
    - [Global](#global-config)
    - [Per Widget](#per-widget-config)
* [Usage and Example](#example)
    - [Guideline](#guideline)
    - [How to make a widget class](#how-to-make-a-widget)
    - [How to use a widget class](#how-to-use-a-widget-class)
* [Behind the curtain](#behind-the-curtain)

This page may look long and boring to read at first, but bear with me!!!

I bet if you read through it you won't get disappointed at the end.So let's Go...


###Introduction
####When to use the _widget_ concept?

>This package (this design pattern) helps you in situations that you want to create crowded web pages with multiple widgets (on sidebar, menu, carousels ...) and each widget needs seperate sql queries and php logic to be provided with data for its template. If you need a small application with low traffic this package is not much of a help. Anyway installing it has minimal overhead since surprisingly it is just a small abstract class and Of course you can use it to __refactor your monster code and tame it__ into managable pieces or __boost the performance 4x-10x__ times faster. ;)



### What is our problems?

#### Problem 1 : Controllers easily get crowded :(
>Imagine An online shop like amazon which shows the list of products, popular products, etc (in the sidebar), user data and basket data in the navbar and a tree of product categories in the menu and etc... In traditional good old MVC model you have a single controller method to provide all the widgets with data. You can immidiately see that you are violating the SRP (Single Responsibility Priciple)!!! The trouble is worse when the client changes his mind over time and asks the deveploper to add, remove and modify those widgets on the page. And it always happens. Clients do change their minds.The developoer's job is to be ready to cope with that as effortlessly as possible.

#### Problem 2 : Page caching is always hard :( 
>Trying to cache the pages which include user specific data (for example the username on the top menu) is a often fruitless. Because each user sees slightly different page from other users. Or in cases when we have some parts of the page which update frequently and some other parts which change rarly... we have to expire the entire page cache to match the most frequently updated one. :(
AAAAAAAAAhh...


#### Problem 3 : View templates easily get littered with if/else blocks (&_&)
>We ideally want our view files to be as logic-less as possible and very much like the final output HTML.Don't we ?! if/else blocks and other computations are always irritating within our views. specially for static page designers in our team. We just want to print out already defined variables wiout the to decide what to print. Anyway the data we store in database are sometimes far from ready to be printed on the page.

========================

### What is the solution?

So, How to fight against those ? ;(
>__The main idea is simple, Instead of one controller method to handle all widgets of the page, Each widget should have it's own `controller class`, `view partial`, `view presenter class` and `cache config`, isolated from others.__
>That's it !! :)
>This idea originally comes from the client-side js frameworks and is somewhat new in server-side world.

- Ok, but How this package is going to help us ? (@_@)

1. It helps you to reach SRP (`single responsibility principle`) in your controllers (Because each widget class is only responsible for one and only one widget of the page but before you had a single controller method that was resposible for all the widgets. Effectively exploding one controller method into multiple widget classes.)
2. It helps you to conforms to `Open-closed principle`. (Because if you want to add a widget on your page you do not need to touch the controller code. Instead you create a new widget class from scratch.)
3. It optionally `caches the output` of each widget. (which give a very powerful, flexible and easy to use caching opportunity) You can set different cache config for each part of the page. Similar to `ESI` standard.
4. It executes the widget code `Lazily`. Meaning that the widget's data method `public function data(){` is hit only and only after the widget object is forced to be rendered in the blade file like this: `{!! $widgetObj !!}`, So for example if you comment out `{!! $widgetObj !!}` from your blade file then all database queries will be disabled automatically. No need to comment out the controller codes anymore...
5. It optionally `minifies` the output of the widget. (In order to save cache storage space and bandwidth)
6. It support the `nested widgets` tree structure. (Use can inject and use widgets within widgets.)
7. It can help you generate widget class boilerplate with artisan command. 
8. It helps you to have a dedicated presenter class of each widget to clean up your views.

### Installation:

`composer require imanghafoori/laravel-widgetize`

>Add `Imanghafoori\Widgets\WidgetsServiceProvider::class` to the providers array in your config/app.php

>And you will be on fire!

>Now you are free to extend the `Imanghafoori\Widgets\BaseWidget` abstract class and implement the `public data` method in your sub-class or use the `php artisan make:widget`.

## Configuration:

### Global Config:
You can set the variables in your .env file to globally set some configs for you widgets and override them if needed.

__WIDGET_MINIFICATION=true__ (you can globally turn off HTML minification for development)

__WIDGET_CACHE=true__ (you can turn caching on and off for all widgets.)

__WIDGET_IDENTIFIER=true__ (you can turn off widget identifiers in production)

__WIDGET_DEFAULT_CACHE_LIFETIME__=1 (You can set a global default lifetime for all widgets and override it per widget if needed)


### Per Widget Config:

- _protected $template_

>If you do not set it,By default, it refers to app/Widgets folder and looks for the 'widgetNameView.blade.php'
(Meaning that if your widget is `app/Widgets/home/recentProducts.php` the default view for that is `app/Widgets/home/recentProductsView.blade.php`)
Anyway you can ovrride it to point to any partial in views folder.(for example: `protected $template='home.footer'` will look for resource/views/home/footer.blade.php)


| app\Widgets\Homepage\RecentProductsWidget.php

| app\Widgets\Homepage\RecentProductsWidgetView.blade.php


So the entire widget lives in one folder.


- _protected $controller_

>If you do not want to put your _data_ method on your widget class, you can set `protected $controller = App\Some\Class\MyController::class` and put your `public data` method on a dedicated class.(instead od having it on your widget class)


- _protected $presenter_

>If you do not want to put your _present_ method on your widget class, you can set `protected $presenter = App\Some\Class\MyPresenter::class` and put your `public present` method on a dedicated class.The data retured from your controller is first piped to your presenter and then to your view.(So if you specify a presenter your view file gets its data from the presenter and not the controller.)



- _protected $cacheLifeTime_

>If you want to override the global cache life time (which is set in your .env file) for a specific widget, you can set $cacheLifeTime on your widget class.


- _protected $cacheTags_

>If you want you can set `protected $cacheTags = ['tag1','tag2']` to easily target them for cache expiration.(Note that  _database_ and _file_ cache driver do not support cache tags.)




##Example

### Guideline

>1. So we first extract each widget into it's own partial. (app/Widgets/recentProducts.blade.php)
>2. Use `php artisan make:widget` command to create your widget class.
>3. Set configurations like __$cacheLifeTime__ , __$template__, etc on your widget class.
>4. Set your controller class and implement the `data` method.
>5. Your widget is ready to be instanciated and be used in your view files. (see example below)

### How to make a Widget?

>__You can use : `php artisan make:widget MyWidget` to make your widget class.__

Sample widget class :
```php
namespace App\Widgets;

use Imanghafoori\Widgets\BaseWidget;


class RecentProductsWidget extends BaseWidget
{
    protected $template = 'widgets.recentProducts.blade.php'; // referes to: views/widgets/recentProducts.blade.php
    protected $cacheLifeTime = 1; // 1(min) ( 0 : disable, -1 : forever)
    protected $context_as = '$recentProducts'; // you can access $recentProducts in view file (default: $data)

    // The data returned here would be available in widget view file automatically.
    // You can use dependancy injection here like you do in your typical controllers.
    public function data($param1=5)
    {
        // It's the perfect place to query the database for your widget...
        return Product::orderBy('id', 'desc')->take($param1)->get();

    }
}
```

=================

recentProducts.blade.php

```blade
<ul>
  @foreach($recentProducts as $product)
    <li>
      <h3> {{ $product->title }} </h3>
      <p>$ {{ $product->price }} </p>
    </li>
  @endforeach
</ul>
```

Ok, Now it's done! We have a ready to use widget. let's use it...

__Tip:__ If you decide to use some other template engine instead of Blade it would be no problem.

### How to use a widget class?

>First we should instanciate our widget class.

In your typical controller methods (or somewhere else) we may instanciate our widget classes and pass the resulting object to our view like this:
```php

use \App\Widgets\RecentProductsWidget;

public function index()
{
    $recentProductsWidget = new RecentProductsWidget();
    
    return view('home', compact('recentProductsWidget'));
}
```

And then you can force the object to render (home.blade.php) like this `{!! $recentProductsWidget !!}`:
```blade
<div class="container">
    <h1>Hello {{ auth()->user()->username }} </h1> <!-- not cached -->
    <br>
    {!! $recentProductsWidget !!} <!-- cached part -->
    <p> if you need to pass parameters to data method :</p>
    {!! $recentProductsWidget(10) !!} <!-- cached part -->
</div>
```

=============

In order to easily understand what's going on here...
Think of `{!! $recentProductsWidget !!}` as `@include('widgets.recentProductsWidget')` but more sophisticated.
The final result is the same piece of HTML, which is the result of rendering the partial.
but widget object are __self contained__ and __self cached__

=============

You may want to look at the BaseWidget source code and read the comments for more information.

==============


### Behind the Curtain


####How the data method on the widget's controller is called then? (0_o)

>Ok, now we know that we do not call widget controller actions from our routes or any where else, how the data method on the widget's controller is called then ???


>Think of widget controllers as laravel view composers which get called automatically when a specific partial is included. Under the hood, After `{!! $myWidget('param1') !!}` is executed in your view file by php, then `public data` method is called on your widget class with the corresponding parameters.
`But only if it is Not already cached` or the `protected $cacheLifeTime` is set to 0.
If the widget HTML output is already in the cache it prints out the HTML without executing `data` method 
(hence avoids performing database queries or even rendering the blade file.)
