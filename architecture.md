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
  - **frontend/** - holds front end specific configurations
    - **routes.xml** - defines router type, route id, and route frontname
    - **events.xml** - defines events to listen for and which observer should respond to them
  - **adminhtml/** - holds admin panel specific configurations
  - **db_schema.xml** -
  - **db_schema_whitelist.json** - 
  - **module.xml** - Adds your module to the list of usable modules in the project, by including a module node with a name value of the fully qualified module name. If the module depends on other modules, you can also add a sequence node to list those modules. *In the future, this may be replaced by composer.json*
  - **di.xml** - DI = dependency injection; tells magento if some files/modules/etc should be replaced by other versions
- **Api/** - interfaces for **repositories** and data api directory will go here
  - **Data/** - interfaces for **Models** will go here
- **Model/** - Holds files that define database models
  - **ResourceModel/**
    - [Model Class].php - Tells magento which table to find data model, and what its ID field is called (basically sets up db connection)
    - **[Model Class]/**
      - Collection.php - Defines methods for interacting with subsets (or all) of the db records for this model
  - **[Model Class].php** - Implements the Model interface from `Api/Data/`. How you get data from records;
  - **[Model Class]Repository.php** - Implements the repository interface from `Api/`. How you save/delete/etc. records;
- **Controller/** - Tells magento what to do when a given route is hit
  - **[ControllerName]**
    - **[Action].php** - This will tell magento how to respond to the url: `website.com/[frontname]/[ControllerName]/[Action]`
  - **Index/**
    - **Index.php** - - This will tell magento how to respond to the url: `website.com/[frontname]`
- **Observer/** - Observer php files go in this directory
- **Setup/**
  - **Patch/**
    - **Data/** - Data patches go in this directory
- **view/**
  - **frontend/**
    - **layout/** - Layout XML files go here
      - [route_id]_[controller]_[action].xml
    - **templates/** - Template .phtml files go here
- **viewModel/** - view models go here
- **i18n/** - translation .csv files go here

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

## Dependency Injection
### Using di.xml
**Dependency Injection (generally)** - Adding a class you need via your class's `__construct()` method instead of instantiating a `new` instance of it.

Similarly, di.xml lets you tell Magento what classes you need so it can give them to you.

In practice, it looks like this:
- `<preference>` lets you substitute one class for another
- `<virtualType>` lets you create a new class

You can use di.xml to specify values of arguments an object receives when constructed.
```
<type name="Vendor\Module\Path\To\Object">
    <arguments>
        <argument name="arg0" xsi:type="string">Argument String</argmument>
        <argument name="arg1" xsi:type="array">
            <argument name="arg1arr0" xsi:type="int">1</argmument>
            <argument name="arg1arr1" xsi:type="object">
                Path\To\Argument\Object
            </argmument>
        </argmument>
    </argument>
</type>
```
Would add values to this Vendor\Module\Path\To\Object object:
```
private $arg0;
private $arg1;

public function __construct(
    string $arg0,
    array $arg1
) { 
    $this->arg0 = $arg0;
    $this->arg1 = $arg1;
}
```
di.xml files may be specific to frontend or adminhtml, in those cases, it can go in `etc/[frontend || adminhtml]/di.xml` and will only apply to that area.

Overriding a class via `preference` should only be done when you need to alter the class's private or protected methods.6

### Factory Classes
By default, when you inject a class in `__construct()` Magento will inject one saved instance of this class. If another class is also injecting that same class it will receive the same instance.

You can get around this with a factory class. Instead of requesting `ClassName` you will request `ClassNameFactory`. 
When you need an instance of class name you will create one like `$instance = $this->classNameFactory->create()`

Magento will create Factory classes for you, automatically (in `generated/`), even for user defined classes. 
Each factory class has one method, `create()`, which returns a unique instance of the class.

### Injecting a class vs injecting its factory
If the object "has state" (i.e. has unique data that only applies to a given instance) ues a factory.

Ex1: A Product object will have a unique name/price/sku etc. This has state and should be created with a factory.

Ex2: A resource model just interacts with a db. There's nothing unique about one instance vs another, so it does not have state and can be injected as a singelton


### Object Manager
The object manager is part of the Magento core code that manages an array of all currently existing objects within an app. 
When you request an object type, it checks if that type is already in the array.
If not, it creates it and adds it to the array. The object also checks for any
preferences defined in di.xml. The OM will also check if any plugins are defined in di.xml
and, if so, will auto-generate the necessary `\Interceptor` 

You will rarely explicitly call the object manager (and if you do, you should NEVER do so in a .phtml file)

## Plugins, preferences, event observers, and interceptors

### How do Plugins work?
Plugins are the best way to change the core functionality of module. If you can use a plugin to achieve your goal you should

There are 3 types of plugin
- **Before**
  - change input parameters, then return parameters as an array
  - Takes in the class it plugs into as `$subject` argument
  - Also takes in `$model` argument that must be returned in an array: `return [$model ...]`
  - By default, have lowest sort order of plugins
  - good for customizing arguments
- **After** 
  - Takes in the class it plugs into as `$subject` argument
  - Takes in `$result` argument that will be returned
  - By default, have highest sort order of plugins
  - good for checking if a method ran
- **Around**
  - Takes in the class it plugs into as `$subject` argument
  - Also takes in `$model` argument
  - Also takes in `callable $proceed` as an argument, which will be returned with `$model` (and any other arguments) as its argument
  - `return $proceed($model, ...)`
  - good for changing a method's output
  - Slow and has unintended consequences, so try not to use this

Plugins can only attach to **PUBLIC** methods

To declare plugins in di.xml:
```
...
<type name="Class\Being\Plugged\In\To">
    <plugin name="PlugInName" type="Path\To\Plugin\Class"/>
</type>
...
```
### Observers
An observer is a class that runs when a specified event occurs.

Observers are registered, disabled, and configured for sharing in an `events.xml` file which looks a bit like this:
```
...
<event name="name_of_event_observed">
    <observer name="ObserverClassName" instance="[Vendor]\[Module]\Observer\ObserverClassName" />
</event>
...
```
The Observer class should go in your modules `Observer/` directory and must implement the `Magento\Framework\Event\ObserverInterface`.
It must have the public function `execute(Magento/Framework/Event/Observer $observer)`. This is where the work the observer does will be defined.

## Controllers

### How do you know what URL a controller will correspond to?
Magento may be broken into 4 parts
- Frontname - This is set in the routes.xml file 
- Controller Name - This is a subdirectory within Controller/
- Action - this is the class that tells Magento how to respond when it hits the url
- Parameters - Additional parameters may be added after the action by appending `/[Param Name]/[Param Value]` i.e. `/id/12345`

EX: If your action class is at the location `Vendor/Module/Controller/Post/Details.php`, and the frontname is set to "blog" it will
correspond to the URL: `website.com/blog/post/details`

If action name is "Index" you may omit that section of the URL, if the controller name **AND** action name are both "Index", you may omit them both.

EX: If your action class is at the location `Vendor/Module/Controller/Index/Index.php`, and the frontname is set to "blog" it will
correspond to the URL: `website.com/blog`

What if the class location is `Vendor/Module/Controller/Index/Details.php`? Is this Valid?

- This URL is valid but the "Index" portion cannot be omitted from the URL. it would be `website.com/blog/index/details` 

URLS requested by users will often not actually map to the given controller, because a user-friendly
URL will be set in the url_rewrite table.

### Routes.xml
This is where your routes are registered. It will look kind of like this:
```
<router id="[router type]"> <!-- usually will be standard -->
    <route id="route_id" frontName="frontname">
        <module name="Vendor_Module"/>
    </route>
</router>
```
The frontname value will relate to the URL, the id value will determine how to name layout files
(which follow the convention `route_id_controller_name_action_name.xml`)

Controllers must extend one of the Magento Framework Action classes
- Get controllers will implement `Magento\Framework\App\Action\HttpGetActionInterface`
- Post controllers will implement `Magento\Framework\App\Action\HttpPostActionInterface`

Controllers constructors must require an instance of `Magento\Framework\App\Action\Context`, which gives us access to 
`getRequest()`, `getResponse`, and `getUrl()`.

Controller response types will come from the `Magento\Frameword\App\Http\ResultInterface`. Which allows for these responses
- Page
  - Returns HTML
  - Looks for a layout corresponding to the controller's [route id]_[controller name]_[action name]
- Redirect
  - Redirects to a provided URL
- JSON
  - Will rarely need this
- Forward
  - Like a redirect, but doesn't change the URL
- Raw
  - will return whatever

Steps to display HTML:
- Create a controller
- Create layout XML for the controller's handle
- Create class to load required information
- Create a view model
- Pass view model to block in template by including it as an argument in the layout XML
- Create template .phtml to display model data

layout XML
```
...
<block name="block.name" template="Vendor_Module::template.phtml">
    <arguments>
        <argument name="view_model" xsi:type="object>
            Vendor\Module\ViewModel\ClassName
        </argument>
    </arguments>
</block>
```

**Additional Notes on Controllers**
- Admin route actions should go into an `adminhtml` directory within `Controller/` before the controller name directory 
- Controllers Should really only return HTML. If you want to return JSON you should use a REST API
 

## Magento CLI
Common commands to be aware of:
- `setup:install` - installs an instance of magento
- `setup:upgrade` - runs the install script and syncs db schema
- `setup:static-content:deploy` - builds contents of pub/static directory
- `admin:user:create` - creates new admin user
- `module:status`/`module:enable` - checks what modules are enabled/enables module
- `dev:source-theme:deploy` - establishes symlink for LESS files
- `cache:flush` - Flushes cache, can take arguments of specific cache to flush
- `cache:clean` - 
- `indexer:reindex` - reindexes indices; can take argument of a specific index

## Indexing

Indexing is a way to improve performance for expensive calculations. Indexing does the calculations and stores them in a table.

Changes that need to be calculated are stored in a change log (tables with a `_cl` prefix) so that
a reindex only has to calculate values that will have changed.

Magento will run the indexing process later on (usually specified by a cron job) to make all the calculations that changed.

You can configure Magento to reindex immediately if necessary (but this could slow things down)

## Localization
Two translation mechanisms:
- Inline translations - Limited tool to translate given phrases directly
- Dictionaries - Extensive tool that can translate everything
  - Provide a dictionary CSV
  - Magento looks for dictionary keys in  `__()` functions
  - Then replaces them with dictionary values

`language.xml` file is used to tell Magento when to use translation packs
- Takes a `language` node
- Which takes
  - `<code>`
  - `<vendor>`
  - `<package>`
  - `<use/>`
    - Multiple use nodes can be given
    - If translation is not found in first, it will look in next

## Cron

## URL Rewrites

## Magento Caching

What information is it important to cache?
- config XML files (like layout, configuration, ui_components)
- Some block HTML outputs
- Critical operation info (like db_schemas, attribute/entity info)

## Scope

Some variables will have different values in different circumstances. Those circumstances are "scope"

Three elements in Scope Structure
- Websites
- Store groups (to business audiences: Stores)
- Stores (to business audiences: Store views)
