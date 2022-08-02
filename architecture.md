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