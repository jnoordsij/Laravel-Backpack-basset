# Basset 🐶

[![Latest Version on Packagist][ico-version]][link-packagist]
[![Total Downloads][ico-downloads]][link-downloads]
[![Build Status][ico-travis]][link-travis]
[![StyleCI][ico-styleci]][link-styleci]

**The dead-simple way to use CSS and JS assets in your Laravel projects.** 

Replace your `<script src='file.js'>` and `<link href='file.css'>` tags with `@basset('file.js')` and this package will internalize the file and make sure that CSS or JS will only be loaded one time per page. This package will internalize assets from CDN, local files and code blocks, it may also internalize zip files. All the files are store on `public` disk by default (`base_path('\storage\app\public\bassets')`) but you may change this in your configs.

## Installation

Install the package via Composer

```bash
composer require backpack/basset
```

If you are using the default disk, and you haven't already, you must create the symbolic link on public to storage:

```bash
php artisan storage:link
```

## Usage

Now you can replace your standard CSS and JS loading HTML with the `@basset()` Blade directive:

```diff
// for files from CDNs
-    <script src="https://cdn.ckeditor.com/ckeditor5/36.0.1/classic/ckeditor.js">
+    @basset('https://cdn.ckeditor.com/ckeditor5/36.0.1/classic/ckeditor.js')
```

Basset will:
- copy that file from the vendor directory to your `storage` directory (aka. internalize the file)
- use the internalized file on all requests
- make sure that file is only loaded once per pageload

## Features

The package provides 4 Blade directives, `@basset()`, `@bassetBlock()`, `@bassetArchive()`, `@bassetDirectory()`, that will allow you to:

### Easily self-host files from CDNs

```diff
-   <script src="http://localhost/storage/basset/cdn.ckeditor.com/ckeditor5/36.0.1/classic/ckeditor.js"></script>
+   @basset('https://cdn.ckeditor.com/ckeditor5/36.0.1/classic/ckeditor.js')
```

Basset will:
- copy that file from the CDN to your `storage/app/public/basset` directory (aka. internalize the file)
- use the local (internalized) file on all requests
- make sure that file is only loaded once per pageload

### Easily use local files from non-public directories

```diff
+ @basset(resource_path('assets/file.css'))
+ @basset(resource_path('assets/file.js'))
```

Basset will:
- copy that file from the vendor directory to your `storage/app/public/basset` directory (aka. internalize the file)
- use the internalized file on all requests
- make sure that file is only loaded once per pageload

### Easily move code blocks to files, so they're cached

```diff
+   @bassetBlock('path/or/name-i-choose-to-give-this')
    <script>
      alert('Do stuff!');
    </script>
+   @endBassetBlock()
```

Basset will:
- create a file with that JS code in your `storage/app/public/basset` directory (aka. internalize the code)
- on all requests, use the local file (using `<script src="">`) instead of having the JS inline
- make sure that file is only loaded once per pageload

### Easily use archived assets (.zip & .tar.gz)

```diff
+    @bassetArchive('https://github.com/author/package-dist/archive/refs/tags/1.0.0.zip', 'package-1.0.0')
+    @basset('package-1.0.0/plugin.min.js')
```

Basset will:
- download the archive to your `storage/app/public/basset` directory (aka. internalize the code)
- unarchive it
- on all requests, use the local file (using `<script src="">`)
- make sure that file is only loaded once per pageload

### Easily internalize and use entire non-public directories

```diff
+    @bassetDirectory(resource_path('package-1.0.0/'), 'package-1.0.0')
+    @basset('package-1.0.0/plugin.min.js')
```

Basset will:
- copy the directory to your `storage/app/public/basset` directory (aka. internalize the code)
- on all requests, use the internalized file (using `<script src="">`)
- make sure that file is only loaded once per pageload

### Easily internalize everything from the CLI, using a command

Copying an asset from CDNs to your server could take a bit of time, depending on the asset size. For large pages, that could even take entire seconds. You can easily prevent that from happening, by internalizing all assets in one go. You can use `php artisan basset:internalize` to go through all your blade files, and internalize everything that's possible. If you ever need it, `basset:clear` will delete all the files.

```bash 
php artisan basset:internalize   # internalizes all @bassets
php artisan basset:clear         # clears the basset directory
```

In order to speed up the first page load on production, we recommend you to add `basset:internalize` command to the deploy script.

## Why does this package exist?

1) Keep a copy of the CDN dependencies on your side.

For many reasons you may want to avoid CDNs, CDNs may fail sometimes, the uptime is not 100%, or your app may need to work offline.

2) Forget about compiling your assets.

Most of the times backend developers end up messing around with npm and compiling dependencies. Backpack has been there, at some point we had almost 100Mb of assets on our main repo.
Basset will keep all that mess away from backend developers.

3) Avoid multiple loads of the same assets.

In Laravel, if your CSS or JS assets are loaded inside a blade file:

```php
// card.blade.php

<div class="card">
  Lorem ipsum
</div>

<script src="path/to/script.js"></script>
```

And you load that blade file multiple times per page (eg. include `card.blade.php` multiple times per page), you'll end up with that `script` tag being loaded multiple times, on the same page. To avoid that, Larvel 8 provides [the `@once` directive](https://laravel.com/docs/8.x/blade#the-once-directive), which will echo the thing only once, no matter how many times that blade file loaded:

```php
// card.blade.php

<div class="card">
  Lorem ipsum
</div>

@once
<script src="path/to/script.js"></script>
@endonce
```

But what if your `script.js` file is not only loaded by `card.blade.php`, but also by other blade templates (eg. `hero.blade.php`, loaded on the same page? If you're using the `@once` directive, you will have the same problem all over again - that same script loaded multiple times.

That's where this package comes to the rescue. It will load the asset just ONCE, even if it's loaded from multiple blade files.

## Change log

Please see the [releases tab](https://github.com/Laravel-Backpack/basset/releases) for more information on what has changed recently.

## Testing

``` bash
$ composer test
```

## Contributing

Please see [contributing.md](contributing.md) for details and a todolist.

## Security

If you discover any security related issues, please email hello@backpackforlaravel.com instead of using the issue tracker.

## Credits

- [Antonio Almeida](https://github.com/promatik)
- [Cristian Tabacitu][link-author]
- [All Contributors][link-contributors]

## License

MIT. Please see the [license file](license.md) for more information.

[ico-version]: https://img.shields.io/packagist/v/backpack/basset.svg?style=flat-square
[ico-downloads]: https://img.shields.io/packagist/dt/backpack/basset.svg?style=flat-square
[ico-travis]: https://img.shields.io/travis/backpack/basset/master.svg?style=flat-square
[ico-styleci]: https://styleci.io/repos/421785142/shield

[link-packagist]: https://packagist.org/packages/backpack/basset
[link-downloads]: https://packagist.org/packages/backpack/basset
[link-travis]: https://travis-ci.org/backpack/basset
[link-styleci]: https://styleci.io/repos/421785142
[link-author]: https://github.com/Laravel-Backpack
[link-contributors]: ../../contributors
