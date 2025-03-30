## StuntCoders WordPress &amp; WooCommerce Developer Guideline

This guideline provides an overview of the best practices, resources, and tools essential for building, customizing, and maintaining WordPress and WooCommerce projects. It covers environment setup, coding standards, theme & plugin development, working with popular tools like ACF and LocalWP, localization, and more.

### Prerequisites

- **LocalWP**: For WordPress-specific projects, LocalWP is a user-friendly option for setting up WP projects locally. [Download LocalWP](https://localwp.com/).
- **Poedit**: Is a popular tool to edit .po and .mo files. It simplifies the translation process for themes and plugins. [Download Poedit](https://poedit.net/).
- **Basic Command Line Knowledge**: Familiarity with terminal or command prompt commands is required.
- **Code Editor**: Use any code editor like Visual Studio Code, Sublime Text, etc.

## Setting up WordPress projects via LocalWP

To set WordPress projects locally in a more user friendly way you can use [LocalWP](https://localwp.com/).

- Download the Local WP to your computer
- Add new local project with default settings and `project_name`, and set site domain to `project_name.local`
- Clone this project to your `Sites` folder, and from there run this command:

    ```bash
    rsync -avzhp --progress --exclude 'wp-config.php' . ~/Local\ Sites/project_name/app/public/
    ```

    Alternatively you can delete the WordPress files added by Local WP, and clone this project in that place(~/Local Sites/project_name/app/public), and then update the wp-config.php file with these:

    ```bash
    'DB_NAME': 'local'
    'DB_USER': 'root'
    'DB_PASSWORD': 'root'
    ```

- Import demo database from the project. But to do so first unzip the database:

    ```bash
    unzip -o database_name.sql.zip > database_name.sql
    rm -r __MACOSX
    ```

    Import the database included in the project using the Adminer tool provided in Local. (This is a simple phpMyAdmin alternative.). Or if there are any issues with this you can open the site SSH shell and type:

    ```bash
    mysql -uroot -proot local < database_name.sql
    ```

    In case if there are issues with loading the database, you can try to delete and recreate the local wp database from the site shell. First login to mysql with:

    ```bash
    mysql -uroot -proot
    ```

    Then delete and recreate WordPress local database, with:

    ```bash
    drop database local;
    create database local;
    exit
    ```

- Sort out the SSL certificate trust issues by following this [guide](https://localwp.com/help-docs/getting-started/managing-local-sites-ssl-certificate-in-macos/)
- Test the site by opening WP Admin from Local.
- Happy coding!

### LocalWP CLI

To use the WP CLI in your project set with Local WP, you will need to open the SSH shell, and from there you can use WP CLI commands. Some useful commands:

- Export the local database

  ```bash
  wp db export backup.sql
  ```

- Import a local database

  ```bash
  wp db import backup.sql
  ```

- Search and Replace in project database

  ```bash
  # Useful if you have loaded the database from staging or production
  wp search-replace '.com' '.local'
  wp search-replace '.websitetotal.com' '.local'
  ```

- Ggenerate `.pot` file from theme or plugin folder

  ```bash
  # Make sure this one is run from the correct folder
  wp i18n make-pot . languages/pot_file_name.pot
  ```

- List all the plugins

  ```bash
  wp plugin list
  ```

- Update the plugin

  ```bash
  wp plugin update plugin_name
  ```

- Activate the plugin

  ```bash
  wp plugin activate plugin_name
  ```

- Deactivate the plugin

  ```bash
  wp plugin deactivate plugin_name
  ```

- Show only a list of all active plugins which have updates available

  ```bash
  wp plugin list --field=name --status=active --update=available
  ```

- List all users

  ```bash
  wp user list
  ```

- List admin users

  ```bash
  wp user list --role=administrator
  ```

- Create a new administrator user

  ```bash
  wp user create admin name@stuntcoders.com --role=administrator --user_pass=sc123123
  ```

- Update existing administrator user password

  ```bash
  wp user update USER_ID  --user_pass=sc123123
  ```

- Delete the user and reassign that user’s posts to any other user

  ```bash
  wp user delete USER_ID --reassign=OTHER_USER_ID
  ```

- Regenerate all media attachments. Useful when changing image sizes

  ```bash
  wp media regenerate
  ```

- Flushe the WordPress site cache

  ```bash
  wp cache flush
  ```

- List all meta data associated with the specified post

  ```bash
  wp post meta list POST_ID --format=table
  ```

- Get a Specific Meta Field for a Post

  ```bash
  wp post meta get POST_ID META_KEY
  ```

- List All Meta Data for a User

  ```bash
  wp user meta list USER_ID --format=table
  ```

- Get a Specific Meta Field for a User

  ```bash
  wp user meta get USER_ID META_KEY
  ```

- List Rewrite Rules

  ```bash
  wp rewrite list --format=table
  ```

- Export content to an XML file

  ```bash
  wp export --dir=/path/to/export --user=author --start_date=2021-01-01 --end_date=2021-12-31
  ```

## Debugging in WordPress

Debugging in WordPress is a crucial skill for developers, as it helps identify and resolve issues within themes, plugins, or the WordPress core itself. Here are some general rules and advice for debugging in WordPress:

### Enabling Debug Mode

- Enable WP_DEBUG:
  - Edit the `wp-config.php` file in your WordPress root directory.
  - Set `WP_DEBUG` to `true`. This enables the debug mode in WordPress.

    ```php
    define('WP_DEBUG', true);
    ```
  - This will start displaying PHP errors, notices, and warnings.
- Log Errors to a File:
  - In `wp-config.php`, you can also enable `WP_DEBUG_LOG` to save these errors to a `debug.log` file within the `wp-content` directory.

    ```php
    define('WP_DEBUG_LOG', true);
    ```
- Display Errors on the Page:
  - To display errors directly on the web page (useful in a local development environment), set `WP_DEBUG_DISPLAY` to `true`.

    ```php
    define('WP_DEBUG_DISPLAY', true);
    ```
  - In a production environment, it's recommended to set this to `false` to prevent visitors from seeing errors.

Usual combination will look, something like this:

```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true );
define( 'WP_DEBUG_DISPLAY', false );
@ini_set( 'display_errors', 0 );
```

This will ensure that WordPress will save errors to `wp-content/debug.log`, and no errors will be shown on the web page. **Regularly check this file for any new warnings or errors.**

### Debugging with Plugins

- Query Monitor:
  - Install the [Query Monitor plugin](https://wordpress.org/plugins/query-monitor/). It provides a detailed analysis of database queries, hooks, conditionals, HTTP requests, redirects, and more.
  - It's particularly useful for identifying slow queries or hooks that may be affecting performance.
  - More in depth guide, can be found in this [article](https://kinsta.com/blog/query-monitor/).
- WP Crontrol:
  - [WP Crontrol](https://wordpress.org/plugins/wp-crontrol/) plugin allows you to view and control what's happening in the WP-Cron system. It's useful for debugging issues with scheduled events in WordPress.
- User Switching:
  - [User Switching](https://wordpress.org/plugins/user-switching/) plugin is a handy tool for developers and administrators. It allows you to quickly switch between user accounts in WordPress at the click of a button. It's great for testing different user roles and capabilities.

Have in mind that these plugins in most cases should be installed only when needed, and removed after there is no more need for them.

### Code-Level Debugging PHP

- Using `error_log()`:
  - You can use the `error_log()` function in PHP to add custom error messages or variable values to the WordPress `debug.log` file.

    ```php
    error_log( print_r( $your_variable, true ) );
    ```
  - This is useful for tracking the values of variables at specific points in your code.
- Var Dump and Print_R:
  - Use `var_dump()` or `print_r()` to output the contents of a variable, usually for debugging purposes directly on the web page.

    ```php
    var_dump( $your_variable );
    print_r( $your_variable );
    ```
  - Have in mind that `pring_r()` displays information about a variable in a way that's readable by humans, which means that output will be better formatted, and the data we are trying to see will be presented in a more readable format. We can do something similar with `var_dump()`, by using something like this:

    ```php
    echo '<pre>' , var_dump( $your_variable ) , '</pre>';
    ```
  - Remember to remove or comment out these calls before pushing your code to production.

### Code-Level Debugging JS

- Using `console.log()`:
  - Use `console.log()` to output variable values or simple messages to the browser’s console. This is useful for tracking the state of your variables as your code executes. Find more about it [here](https://developer.mozilla.org/en-US/docs/Web/API/console).
    ```js
    console.log(your_variable);
    ```
  - Use `console.dir()` which recognizes the object as an object and prints it’s properties in the form of a clean expandable list
  - Use `console.table()` to print a visually nice table representation of an object with labeled rows for each the objects properties
  - Use `console.trace()` when debugging deeply nested objects or functions, we may want to print the stack trace of the code.
  - Use `console.log({your_variable})` with curly brackets, which will output a JavaScript object, with the key as the variable name and the value in a console message.
  - Use `console.time()` in combination with `console.timeEnd()` to benchmark loops. It can be super really to know exactly how long something has taken to execute, especially when debugging slow loops.
- Using the `debugger` Keyword - Insert the `debugger;` statement in your code to automatically pause execution when the Developer Tools are open. This allows you to inspect variables and the call stack at that moment.
   ```js
    function testFunction() {
        debugger; // Execution will pause here if DevTools is open.
        // Additional code...
    }
    testFunction();
    ```
- Setting Breakpoints in Developer Tools - More about this can be found [here](https://developer.chrome.com/docs/devtools/javascript).

### Debugging Themes and Plugins

- Deactivate All Plugins:
  - If you're encountering issues, try deactivating all plugins. If the issue resolves, reactivate them one by one to identify the culprit.
- Switch to a Default Theme:
  - Temporarily switch to a default WordPress theme (like Twenty Twenty-One) to see if the issue is theme-related.

### Debugging Best Practices

- Cleanup Before Production:
Always remove or comment out `error_log()`, `console.log()`, `debugger;`, and other debugging statements before deploying code to production to avoid leaking internal state information.
- Leverage Documentation:
Regularly refer to updated resources and tutorials to stay current with debugging techniques. Developer tools are continuously evolving, and new features are added that can simplify your debugging workflow.

### LocalWP tools

- Xdebug:
  - Using Xdebug for profiling is a powerful way to identify performance bottlenecks in your PHP code, including WordPress and other web applications. Xdebug is a PHP extension that provides debugging and profiling capabilities. If you are using LocalWP for setting up your local WP projects, you can easily enable or disable this extension with one click.
  - You can find more about how to use and set this extension on LocalWP, in this user guide: [Xdebug within Local](https://localwp.com/help-docs/advanced/using-xdebug-within-local/)
  - **Have in mind that Xdebug can make your site really slow, so enable this extension only if you need it.** Since Xdebug hooks into every part of PHP, it can cause complex sites to run slower than it would without Xdebug. If you are not able to disable Xdebug directly from the LocalWP user interface, to disable it follow [these](https://community.localwp.com/t/how-can-i-disable-the-xdebug-php-extension/18120) instructions. To make LocalWP even faster, you can update `max_execution_time`, `max_input_time`, `max_input_vars` and `memory_limit` in the same file mentioned in the instructions above.
- Mailhog:
  - To make debugging easier, LocalWP uses Mailhog modules to log all the emails for use in debugging and troubleshooting. You don't need an extra mail server to test the mail functionality.
  - To view the email logs, you can go to the `Utilities` tab, then select `Open Mailhog`. You can see email logs.
  - You can find more about Mailhog in this Kinsta [article](https://kinsta.com/blog/mailhog/).

## WordPress and WooCommerce best practices

[WordPress Coding Standards](https://developer.wordpress.org/coding-standards/) lay the foundation for writing code that is readable, maintainable, and secure. They cover a broad spectrum of coding styles—from PHP and HTML to CSS and JavaScript—emphasizing the importance of consistent formatting, naming conventions, and thorough documentation. By following these standards, developers can ensure that their contributions seamlessly integrate with the wider WordPress ecosystem and can be easily understood by other team members.

Similarly, [WooCommerce Customizing Best Practices](https://woocommerce.com/document/customizing-woocommerce-best-practices/) provide tailored recommendations for extending and modifying WooCommerce functionalities without compromising stability or upgrade compatibility. These practices highlight the proper use of hooks, filters, and template overrides, and emphasize performance optimization and security. Whether you’re adjusting the default behavior of WooCommerce or building custom features for an e-commerce site, adhering to these guidelines helps maintain a clean separation of core functionality and custom modifications.

### Additional Points

- Validate and Sanitize All User Inputs - Ensure that every piece of user-provided data is both validated and sanitized before processing or saving it. Detailed guidelines can be found in the [Validating, Sanitizing, and Escaping](https://docs.wpvip.com/security/validating-sanitizing-and-escaping/) resource.
- Escape Output Rigorously - Always escape output when rendering data on the front end to prevent Cross-Site Scripting (XSS) vulnerabilities. This practice is thoroughly explained in the [WordPress Security: Escaping](https://developer.wordpress.org/apis/security/escaping/) documentation. All of this is well summarized in [Rudrastyh’s guide](https://rudrastyh.com/wordpress/sanitize-escape-validate.html), ensuring robust security throughout your codebase.
- Implement Nonces for Secure Actions - Always use nonces (number used once) to secure forms, AJAX requests, and other actions. Nonces help protect against Cross-Site Request Forgery (CSRF) by verifying that the request comes from a trusted source. For best practices on implementing nonces, refer to the [WordPress Nonces Documentation](https://developer.wordpress.org/apis/security/nonces/).
- Avoid adding jQuery in Templates - Embedding jQuery code directly in your PHP templates is not recommended because:
  - It prevents the jQuery library from being deferred or asynchronously loaded.
  - It increases the risk of blocking rendering, which negatively impacts page speed.
- Eliminate Deprecated Methods jQuery methods - Since our front-end does not load the jQuery Migrate script, it’s crucial to avoid any deprecated jQuery methods.
- Proper Script Enqueuing - Use `wp_enqueue_script()` to load your custom scripts. This approach ensures that scripts are loaded in the proper order and can take advantage of deferring or async attributes.

## PHPCS and WPCS

Below is a step-by-step guide to installing PHP CodeSniffer (PHPCS) and the WordPress Coding Standards (WPCS) on macOS using Composer.

- Ensure that Composer is installed on your Mac. You can verify this by running:
```bash
composer --version
```

- Open your Terminal and run the following command to install PHPCS globally:
```bash
composer global require "squizlabs/php_codesniffer=*"
```

- The global installation places PHPCS in your Composer vendor bin directory. You will need to add this directory to your system PATH. Open your shell configuration file with this:
```bash
vim ~/.zshrc
```

- Add this at the end of the file:

```bash
export PATH="$PATH:$HOME/.composer/vendor/bin"
```

`which phpcs` should now return you something like `/Users/jovanvukovic/.composer/vendor/bin/phpcs`

- Install WordPress Coding Standards (WPCS) Globally:
```bash
composer global require "wp-coding-standards/wpcs"
```

- Configure PHPCS to Use WPCS (replace `jovanvukovic` with your user)
```bash
phpcs --config-set installed_paths /Users/jovanvukovic/.composer/vendor/wp-coding-standards/wpcs
```

- Run the following command to see a list of installed coding standards:
```bash
phpcs -i
```

- You should see something like this:
```bash
The installed coding standards are MySource, PEAR, PSR1, PSR2, Squiz, Zend, WordPress, WordPress-Extra, WordPress-Docs.
```

- Try to use phpcs from the command line in some WP project with this command for example:
```
phpcs wp-config.php
```

- If it's set properly you should see something like this:
```
FILE: /Users/jovanvukovic/Local Sites/imi-new/app/public/wp-config.php
---------------------------------------------------------------------------------------------------------------
FOUND 80 ERRORS AND 11 WARNINGS AFFECTING 35 LINES
---------------------------------------------------------------------------------------------------------------
  17 | ERROR   | [ ] The tag in position 1 should be the @package tag
  20 | WARNING | [ ] PHP version not specified
  20 | ERROR   | [ ] Missing @category tag in file comment
  20 | ERROR   | [ ] Missing @author tag in file comment
  20 | ERROR   | [ ] Missing @license tag in file comment
  23 | ERROR   | [x] The open comment tag must be the only content on the line
```

### Debugging PHPCS setup

- If you are getting errors like this one:
```bash
phpcs: Referenced sniff "Universal.NamingConventions.NoReservedKeywordParameterNames" does not exist
```

- First check if these are installed with `phpcs -i`, and if these are listed like this:
```
The installed coding standards are MySource, PEAR, PSR1, PSR2, PSR12, Squiz, Zend, WordPress, WordPress-Core, WordPress-Docs, WordPress-Extra, PHPCSUtils, Modernize, NormalizedArrays, Universal and PHPCompatibility
```

- We will have to set paths (Replace jovanvukovic with your user):
```bash
phpcs --config-set installed_paths /Users/jovanvukovic/.composer/vendor/wp-coding-standards/wpcs,/Users/jovanvukovic/.composer/vendor/phpcsstandards/phpcsutils,/Users/jovanvukovic/.composer/vendor/phpcsstandards/phpcsextra,/Users/jovanvukovic/.composer/vendor/phpcompatibility/php-compatibility
```

- But if you are missing something, for example `PHPCompatibility`, you will have to install it first, and then set the paths:
```bash
composer global require --dev phpcompatibility/php-compatibility
```

- `composer global show` should now show all the packages you have installed.

### Integrating PHPCS (with WPCS) into VS Code

- Install these two extensions for VS Code:
  - phpcs - by [shevaua](https://marketplace.visualstudio.com/items?itemName=shevaua.phpcs)
  - phpcbf - by [Per Soderlind](https://marketplace.visualstudio.com/items?itemName=persoderlind.vscode-phpcbf)

- Once these are installed and activated, add this to VS Code `settings.json` file (Replace jovanvukovic with your user):
```json
"phpcs.enable": true,
"phpcs.executablePath": "/Users/jovanvukovic/.composer/vendor/bin/phpcs",
"phpcs.standard": "WordPress",
"phpcbf.enable": true,
"phpcbf.executablePath": "/Users/jovanvukovic/.composer/vendor/bin/phpcbf",
"phpcbf.documentFormattingProvider": true,
"phpcbf.onsave": false,
"phpcbf.standard":"WordPress",
"php.validate.enable": false,
```

- You can see yours `executablePath` by running:
```bash
which phpcs
which phpcbf
```

- Restart VS Code, open some file in one of yours WP projects, and it should automatically report code style errors now.