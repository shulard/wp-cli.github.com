---
layout: doc
title: Common issues and their fixes
description: In case of fire, break glass.
category: Guides
---

### Error: Can't connect to the database

A few possibilities:

a) you're using MAMP, but WP-CLI is not using the MAMP PHP binary.

You can check which PHP WP-CLI is using by running `wp --info`.

If you need to specify an alternate PHP binary, see [using a custom PHP binary](http://wp-cli.org/docs/installing/#using-a-custom-php-binary).

b) it's a WordPress multisite install.

c) the database credentials in `wp-config.php` are actually incorrect.

### PHP Fatal error: Cannot redeclare wp_unregister_GLOBALS()

If you get this fatal error running the `wp` command, you may have moved or edited `wp-config.php` beyond what wp-cli supports:

```
PHP Fatal error: Cannot redeclare wp_unregister_GLOBALS() (previously declared in /var/www/foo.com/wp-includes/load.php:18) in /var/www/foo.com/wp-includes/load.php on line 33
```

One of WP-CLI's requirements is that the line:

```php
require_once(ABSPATH . 'wp-settings.php');
```

remains in the `wp-config.php` file, so if you've modified or moved it, put it back there. It gets matched by a regex when WP-CLI runs.

See also: [#1631](https://github.com/wp-cli/wp-cli/issues/1631)

### PHP Fatal error: Call to undefined function cli\posix_isatty()

Please ensure you have the php-process extension installed. For example for Centos 6: `yum install php-process`

### Error: YIKES! It looks like you're running this as root.

Running WP-CLI as root is extremely dangerous. When you execute WP-CLI as root, any code within your WordPress instance (including third-party plugins and themes you've installed) will have full privileges to the entire server. This can enable malicious code within the WordPress instance to compromise the entire server.

The WP-CLI project strongly discourages running WP-CLI as root.

See also: [#973](https://github.com/wp-cli/wp-cli/pull/973#issuecomment-35842969)

### PHP notice: Undefined index on `$_SERVER` superglobal

The `$_SERVER` superglobal is an array typically populated by a web server with information such as headers, paths, and script locations. PHP CLI doesn't populate this variable, nor does WP-CLI, because many of the variable details are meaningless at the command line.

Before accessing a value on the `$_SERVER` superglobal, you should check if the key is set:

    if ( isset( $_SERVER['HTTP_X_FORWARDED_PROTO'] ) && 'https' === $_SERVER['HTTP_X_FORWARDED_PROTO'] ) {
      $_SERVER['HTTPS']='on';
    }

When using `$_SERVER['HTTP_HOST']` in your `wp-config.php`, you'll need to set a default value in WP-CLI context:

    if ( defined( 'WP_CLI' ) && WP_CLI && ! isset( $_SERVER['HTTP_HOST'] ) ) {
        $_SERVER['HTTP_HOST'] = 'wp-cli.org';
    }

See also: [#730](https://github.com/wp-cli/wp-cli/issues/730)

### Can't find wp-content directory / use of `$_SERVER['document_root']`

`$_SERVER['document_root']` is defined by the webserver based on the incoming web request. Because this type of context is unavailable to PHP CLI, `$_SERVER['document_root']` is unavailable to WP-CLI. Furthermore, WP-CLI can't safely mock `$_SERVER['document_root']` as it does with `$_SERVER['http_host']` and a few other `$_SERVER` values.

If you're using `$_SERVER['document_root']` in your `wp-config.php` file, you should instead use `dirname( __FILE__ )` or similar.

See also: [#785](https://github.com/wp-cli/wp-cli/issues/785)

### Conflict between global parameters and command arguments

All of the [global parameters](http://wp-cli.org/config/) (e.g. `--url=<url>`) may conflict with the arguments you'd like to accept for your command. For instance, adding a RSS widget to a sidebar will not populate the feed URL for that widget:


    $ wp widget add rss sidebar-1 1 --url="http://www.smashingmagazine.com/feed/" --items=3
    Success: Added widget to sidebar.

* Expected result: widget has the feed URL set.
* Actual result: widget is added with the number of items set to 3, but with empty feed URL.

Use the `WP_CLI_STRICT_ARGS_MODE` environment variable to tell WP-CLI to treat any arguments before the command as global, and after the command as local:

    WP_CLI_STRICT_ARGS_MODE=1 wp --url=wp.dev/site2 widget add rss sidebar-1 1 --url="http://wp-cli.org/feed/"

In this example, `--url=wp.dev/site2` is the global argument, setting WP-CLI to run against 'site2' on a WP multisite install. `--url="http://wp-cli.org/feed/"` is the local argument, setting the RSS feed widget with the proper URL.

See also: [#3128](https://github.com/wp-cli/wp-cli/pull/3128)

### Warning: Some code is trying to do a URL redirect

Most of the time, it's some plugin or theme code that disables wp-admin access to non-admins.

Quick fix, other than disabling the protection, is to pass the user parameter: `--user=some_admin`

See also: [#477](https://github.com/wp-cli/wp-cli/issues/477)

### The installation hangs

If the installation seems to hang forever while trying to clone the resources from GitHub, please ensure that you are allowed to connect to Github using SSL (port 443) and Git (port 9418) for outbound connections.

### W3 Total Cache Error: some files appear to be missing or out of place.

W3 Total Cache object caching can cause this problem. Disabling object caching and optionally removing `wp-content/object-cache.php` will allow WP-CLI to work again.

See also: [#587](https://github.com/wp-cli/wp-cli/issues/587)

### The automated updater doesn't work for versions before 3.4

The `wp core update` command is designed to work for WordPress 3.4 and above. To be able to update an older website to latest WordPress, you could try one of the following alternatives:

1. **Fully-automated:** Run `wp core download --force` to download latest WordPress and replace it with your files (don't worry, `wp-config.php` will remain intact). Then, run `wp core update-db` to update the database. Since the procedure isn't ideal, run once again `wp core download --force` and the new version should be available.
2. **Semi-automated:** Run `wp core download --force` to download all files and replace them in your current installation, then navigate to `/wp-admin/` and run the database upgrade when prompted.
