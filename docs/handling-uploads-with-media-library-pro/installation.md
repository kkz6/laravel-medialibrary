---
title: Installation
weight: 2
---

[Media Library Pro](medialibrary.pro) is a paid add-on package for Laravel Media Library. In order to use it, you must have the base version of media library installed in your project. Here are [the installation instructions for the base version](/docs/laravel-medialibrary/v11/installation-setup).

## Installing the base package

## Getting a license

You must buy a license on [the Media Library Pro product page](https://spatie.be/products/media-library-pro) at spatie.be

Single application licenses maybe installed in a single Laravel app. In case you bought the unlimited application license, there are no restrictions. A license comes with one year of upgrades. If a license expires, you are still allowed to use Media Library Pro, but you won't get any updates anymore.

## Current version

The current version of Media Library Pro is v4.

You will find upgrade instructions [here](/docs/laravel-medialibrary/v11/handling-uploads-with-media-library-pro/upgrading).

## Requiring Media Library Pro

After you've purchased a license, add the `satis.spatie.be` repository in your `composer.json`.

```php
"repositories": [
    {
        "type": "composer",
        "url": "https://satis.spatie.be"
    }
],
```

Next, you need to create a file called `auth.json` and place it either next to the `composer.json` file in your project, or in the Composer home directory. You can determine the Composer home directory on \*nix machines by using this command.

```bash
composer config --list --global | grep home
```

This is the content you should put in `auth.json`:

```php
{
    "http-basic": {
        "satis.spatie.be": {
            "username": "<YOUR-SPATIE.BE-ACCOUNT-EMAIL-ADDRESS-HERE>",
            "password": "<YOUR-MEDIA-LIBRARY-PRO-LICENSE-KEY-HERE>"
        }
    }
}
```


To be sure you can reach `satis.spatie.be`,  clean your autoloaders before using this command:

```bash
composer dump-autoload
```

To validate if Composer can read your auth.json you can run this command:

```bash
composer config --list --global | grep satis.spatie.be
```

If you are using [Laravel Forge](https://forge.laravel.com), you don't need to create the `auth.json` file manually. Instead, you can set the credentials on the Composer Package Authentication screen of your server. Fill out the fields with these values:

- Repository URL: `satis.spatie.be`
- Username: Fill this field with your spatie.be account email address
- Password: Fill this field with your Media Library Pro license key

![screenshot](/docs/laravel-medialibrary/v11/images/forge.png)

With the configuration above in place, you'll be able to install the Media Library Pro into your project using this command:

```bash
composer require "spatie/laravel-medialibrary-pro:^3.0.0"
```

## Prepare the database

Media Library Pro tracks all temporary uploads in a table called `temporary_uploads`.

To create the table you need to publish and run the migration:

```bash
php artisan vendor:publish --provider="Spatie\MediaLibraryPro\MediaLibraryProServiceProvider" --tag="media-library-pro-migrations"
php artisan migrate
```

## Automatically removing temporary uploads

If you are using Media Library Pro, you must schedule this artisan command in `app/Console/Kernel` to automatically delete temporary uploads

```php
// in app/Console/Kernel.php

protected function schedule(Schedule $schedule)
{
    $schedule->command('media-library:delete-old-temporary-uploads')->daily();
}
```

## Add the route macro

To accept temporary uploads via React and Vue, you must add this macro to your routes file. 
You do not need to register this endpoint when using the Blade/Livewire components.

```php
// Probably routes/web.php

Route::mediaLibrary();
```

This macro will add the routes to controllers that accept file uploads for all components.

#### Only allow authenticated users to upload files

If in your project, you only want authenticated users to upload files, you can put the macro in a group that applies authentication middleware.

```php
Route::middleware('auth')->group(function() {
    Route::mediaLibrary();
});
```

We highly encourage you to do this, if you only need authenticated users to upload files.

#### Validating mime types

For security purposes, only files that pass [Laravel's `mimes` validation](https://laravel.com/docs/master/validation#rule-mimetypes) with the extensions [mentioned in this class](https://github.com/spatie/laravel-medialibrary-pro/blob/ba6eedd5b2a7f743909b441c0b6fd111d1a73794/src/Support/DefaultAllowedExtensions.php#L5) are allowed by the temporary upload controllers.

If you want your components to accept other mimetypes, add a key `temporary_uploads_allowed_extensions` in the `media-library.php` config file.

```php
// in config/medialibrary.php

return [
   // ...
   
   'temporary_uploads_allowed_extensions' => [
        // your extensions
        ... \Spatie\MediaLibraryPro\Support\DefaultAllowedExtensions::all(), // add this if you want to allow the default ones too
   ],
],
]
```

#### Rate limiting

To protect you from users that upload too many files, the temporary uploads controllers are rate limited. By default, only 10 files can be upload per minute per ip iddress.

To customize rate limiting, add [a rate limiter](https://laravel.com/docs/master/rate-limiting#introduction) named `medialibrary-pro-uploads`. Typically, this would be done in a service provider.

Here's an example where's we'll allow 15 files:

```php
// in a service provider

use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('medialibrary-pro-uploads', function (Request $request) {
    return [
        Limit::perMinute(10)->by($request->ip()),
    ];
});
```

## Using the CSS

You have a couple of options for how you can use the UI components' CSS, depending on your and your project's needs:

### Using Laravel Mix or Webpack with css-loader

You can import the built CSS in your own CSS files using `@import "vendor/spatie/laravel-medialibrary-pro/resources/js/media-library-pro-styles";`.

This isn't a very pretty import, but you can make it cleaner by adding this configuration to your Webpack config:

**laravel-mix >6**

```js
mix.override((webpackConfig) => {
    webpackConfig.resolve.modules = [
        "node_modules",
        __dirname + "/vendor/spatie/laravel-medialibrary-pro/resources/js",
    ];
});
```

**laravel-mix <6**

```js
mix.webpackConfig({
    resolve: {
        modules: [
            "node_modules",
            __dirname + "/vendor/spatie/laravel-medialibrary-pro/resources/js",
        ],
    },
});
```

This will force Webpack to look in `vendor/spatie/laravel-medialibrary-pro/resources/js` when resolving imports, and allows you to shorten your import to this:

```css
@import "media-library-pro-styles";
```

If you're using PurgeCSS, you might have to add a rule to your whitelist patterns.

```js
mix.purgeCss({ whitelistPatterns: [/^media-library/] });
```

### Directly in Blade/HTML

You should copy the built CSS from `vendor/spatie/laravel-medialibrary-pro/resources/js/media-library-pro-styles/dist/styles.css` into your `public` folder, and then use a `link` tag in your blade/html to get it: `<link rel="stylesheet" href="{{ asset('css/main.css') }}">`.

If you would like to customize the CSS we provide, head over to [the section on Customizing CSS](/docs/laravel-medialibrary/v11/handling-uploads-with-media-library-pro/customizing-css).

## Usage in a frontend repository

If you can't install the package using `composer` because, for example, you're developing an SPA, you can download the packages from GitHub Packages.

### Registering with GitHub Packages

You will need to create a Personal Access Token with the `read:packages` permission on the GitHub account that has access to the [spatie/laravel-medialibrary-pro](https://github.com/spatie/laravel-medialibrary-pro) repository. We suggest creating an entirely new token for this and not using it for anything else. You can safely share this token with team members as long as it has only this permission. Sadly, there is no way to scope the token to only the Media Library Pro repository.

Next up, create a `.npmrc` file in your project's root directory, with the following content:

_.npmrc_

```
//npm.pkg.github.com/:_authToken=github-personal-access-token-with-packages:read-permission
@spatie:registry=https://npm.pkg.github.com
```

Make sure the plaintext token does not get uploaded to GitHub along with your project. Either add the file to your `.gitignore` file, or set the token in your `.bashrc` file as an ENV variable.

_.bashrc_

```
export GITHUB_PACKAGE_REGISTRY_TOKEN=token-goes-here
```

_.npmrc_

```
//npm.pkg.github.com/:_authToken=${GITHUB_PACKAGE_REGISTRY_TOKEN}
@spatie:registry=https://npm.pkg.github.com
```

Alternatively, you can use `npm login` to log in to the GitHub Package Registry. Fill in your GitHub credentials, using your Personal Access Token as your password.

```
npm login --registry=https://npm.pkg.github.com --scope=@spatie
```

If you get stuck at any point, have a look at [GitHub's documentation on this](https://docs.github.com/en/free-pro-team@latest/packages/publishing-and-managing-packages/installing-a-package).

### Downloading the packages from GitHub Packages

Now, you can use `npm install --save` or `yarn add` to download the packages you need.

```
yarn add @spatie/media-library-pro-styles @spatie/media-library-pro-vue3-attachment
```

**You will now have to include the `@spatie/` scope when importing the packages**, this is different from examples in the documentation.

```
import { MediaLibraryAttachment } from '@spatie/media-library-pro-vue3-attachment';
```

You can find a list of all the packages on the repository: https://github.com/orgs/spatie/packages?repo_name=laravel-medialibrary-pro.

## What happens when your license expires?

A few days before a license expires, we'll send you a reminder mail to renew your license.

Should you decide not to renew your license, you won't be able to use composer anymore to install this package. You won't get any new features or bug fixes.

Instead, you can download a zip containing the latest version that your license covered. This can be done on  [your purchases page on spatie.be](https://spatie.be/profile/purchases). You are allowed to host this version in a private repo of your own.
