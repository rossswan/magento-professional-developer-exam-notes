# Architecture Notes

## Magento File Structure
- **app/code** - This is where your custom modules go
- **app/design** - This is where custom themes will go
  - **./frontend** - This is for themes your customer will see
  - **./adminhtml** - This is for themes affecting the /admin site
- **app/i18n** - i18n = internationalization; translation/language packages go here
- **app/etc/** - This is where configuration files will live
  - **env.php** - holds all your environment variables (database info, ect). Keep this out of git
  - **config.php** - Tells Magento which modules are enabled/disabled. Do commit this to git.
- **bin/** - You will mainly be concerned with bin/magento because this is where CLI commands are defined
- **dev/** - Contains built in tools for compiling code and testing (i.e. Grunt)
- **generated/** - Home for code built during the deployment process. Sometimes you have to clear this out to get things working right
- **lib/** - Home for internal libraries
- **pub/** - When your app is served to the web, this should be the root directory. This prevents unauthorized access to the /var directory
  - **index.php** - entry-point into your web app (this is the file that gets served up when a user accesses your URL)
  - **./media** - Images are stored here
  - **./static** - static CSS/JS/HTML files are stored here
    - **.htaccess** - Used to download your theme files. DO NOT DELETE THIS.
- **setup/** - Contains files for installing Magento
- **var/** - Contains temporary files used by your app (logs, cache, reports), things that you probably don't want outsiders to have access too. Also, don't store anything here because it gets deleted often.
- **vendor/** - This is where composer installed modules live. Created when `composer install` is run. Don't change anything in here. Those changes will get overwritten.

### Some notes on the pub/static directory
- In developer mode, the files in this directory will largely be symlinked to the pre-processed versions (i.e. LESS, JS, Etc. from the app/design/frontend dir)
- In production mode, the files in this directory will be processed versions of the files (i.e. fully ran through grunt and not linked to the app/ dir)
- Basically, in dev mode files in pub/static will reflect the latest changes to your codebase (in fact, they will be symlinks to those files)
- In prod mode, files in pub/static will reflect the state of your codebase at last deploy
- To deploy static files to **dev**: `bin/magento dev:source-theme:deploy`
- To deploy static files to **prod**: `bin/magento setup:static-content:deploy`

## Modules

### Module Names
The location of a given module will be: `app/code/YourVendorName/YourModule`. 
A module with this file path will have the name `YourVendorName_YourModule`. 
Modules in the vendor/ directory will follow a bit of a different convention. 
This will mainly come up when looking for Magento files.

ex: `Magento/Catalog` => `vendor/magento/module-catalog`

### Module File Structure
- **registration.php** - Boilerplate that tells Magento it can use this modules
- **composer.json** - Necessary if module will be installed through composer. Tells composer how to find your module.
- **etc/** 
  - **module.xml** - Adds your module to the list of usable modules in the project, by including a module node with a name value of the fully qualified module name. If the module depends on other modules, you can also add a sequence node to list those modules. *In the future, this may be replaced by composer.json*

At a bare minimum, a module just needs registration.php and etc/module.xml. That will register the module with your app, but it will have no functionality.

To install modules to the db: `bin/magento setup:upgrade`
To enable a module: `bin/magento module:[enable/disable] YourVendorName_YourModule`

### When should I use a custom module?
- If you need view models for a new theme
- If you are adding a larger new feature
- If you are modifying a 3rd party module
- If you are making customizations to an existing module

### Using composer.json 
- Composer package types:
  - *magento2-module* - for modules in `app/code`
  - *magento2-theme* - for theme modules in `app/design/[frontend/adminhtml]`
  - *magento2-language* - for i18n packages
  - *magento2-component* - relates to `vendor/magento/magento2-base`
  - *magento2-library* - relates to core framework `vendor/magento/framework`