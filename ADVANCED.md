# Ligero

[![GitHub license](https://img.shields.io/github/license/mashape/apistatus.svg)](LICENSE.md)


## Advanced Usage

Here we examine usage of the Publisher object in greater detail, explore creating domain contexts and APIs, and discuss other advanced functionalities supported by this package.

[QuickStart](README.md)  |  [Configuration](CONFIGURATION.md)  |  Advanced Usage

- [Publisher Methods](#publisher-methods)
- [Publisher Parameters](#publisher-parameters)
- [Configuration for Multiple Domains](#configuration-for-multiple-domains)
- [Domain-Driven Design Using Contexts](#domain-driven-design-using-contexts)
- [Presenters](#presenters)
- [Handling Relations](#handling-relations)
- [Automatic Handling of Timestamps](#automatic-handling-of-timestamps)
- [Localization](#localization)
- [Unit Conversions](#unit-conversions)


### Publisher Methods

A Publisher object is used to perform CRUD operations and generate a dynamic data-driven display with contextual search and navigation controls. These are the main methods providing such functionality anywhere a Publisher is implemented.

#### Raw Results and UI Data

Get query results, data for dynamic UI controls, and full information on the query. Return records as a collection with `getResults()`, or as a plain array with `getItems()`. Use `getData()` to retrieve all these elements as a bundle.

```php
getQueryInfo()
found()
displayed()
getResults()
getItems()
getPagination()
getKeywordSearch()
getData()
```

#### Presented Results

Return an array of presented records as configured in the [Presenter](#presenters) class. As with `getData()` (above), use `presentData()` to retrieve the full data bundle.

```php
presentItems()
presentData()
```

#### Standard CRUD Actions

As with the raw query methods above, the `find()` and `findBy()` methods can return either a collection (default) or an array of records. The `store()`, `update()` and `delete()` methods can use inputs either previously set (default) or passed explicitly.

```php
find($id, $native = true)
findBy($inputs = [], $native = true)
store($inputs = null)
update($inputs = null)
delete($id = null)
```

#### Multi-Record List Actions

The 'action', 'items' and 'options' query parameters can be used to perform actions on a selection of records. The package supports the actions 'clone' and 'delete'. To add new actions, extend the `BasePublisherRepository`, adding new cases to the `action()` method.

```php
action($inputs = null)
```

Separate from this functionality, using 'select_all' as the 'action' parameter in a query will select all item checkboxes in the results display, via the `$form_item_checked` attribute for each row of results data passed to the views. This is useful when creating a pure HTML front end, where JavaScript can't be used to manipulate UI elements.

### Publisher Parameters



A Publisher accepts standard set of input parameters that are used to configure query operations, and also allows you to define your own custom input parameters. Default values for all query inputs can be set with the fluent setter methods `setQueryDefault()` and `setQueryDefaults()`.

#### Validation Rules are Required

Any query input without a corresponding validation rule will be ignored. See [Input Parameters](CONFIGURATION.md#input-parameters) in the Configuration documentation, and refer to the [example below](#creating-a-domain-context), to understand how rules are set. 

Rules for the standard input parameters are set separately and cannot be modified.

#### Standard Input Parameters

A set of pre-defined input parameters controls the basic operation of Publisher queries. The names of these parameters are reserved, and should not be used for custom inputs or database columns. These are the standard input parameters with their validation rules:

``` php
[
    'id'            => 'numeric|min:1',
    'keyword'       => 'max:32',
    'sort'          => 'max:32',
    'view'          => 'in:list,grid,item',
    'limit'         => 'numeric|min:1',
    'start'         => 'numeric|min:0',
    'action'        => 'max:32',
    'items'         => 'array',
    'options'       => 'array',
    'page'          => 'numeric|min:1'
]
```

### Configuration for Multiple Domains

A domain's table and model, by default, are determined by the Config's `$table_name` and `$model_name` attributes. Supporting multi-domain implementations, `getTableName()` and `getModelName()` will return either the Config attributes themselves, or, if the `ligero.php` config file contains values for that domain (keyed by `$table_name`), those values will be returned instead. The Config also has an array of Context class names, keyed by the context's route parameter, accessed by `getContexts()`.

The config file can be "published" for modification; this will copy it to Laravel's `config` directory, taking precedence over the original package file. See the section on [Customization](README.md#customization).

### Domain-Driven Design Using Contexts

The strategy pattern implemented by this package can be used to quickly scaffold domains with built-in CRUD functions and a nice contextual UI - a good foundation to build on. Of course, most applications or even micro-services require data and functions across several domains.

A **Context** provides a domain with an application service layer, allowing domains to use each other in predictable ways. Contexts can be useful for encapsulating both a domain Publisher and it's dependencies, and additional logic specific to a domain.

When working in a DDD style, take advantage of this package's built-in support for contexts - each domain can be instantiated as a Context (or many different Contexts, as required). A Context that uses data and functions across multiple domains is referred to as an Aggregate Root (or Rich Domain), and will typically use database transactions to ensure data integrity in multi-domain operations.

The package's `BaseContext` and `AggregateContext` classes can be extended to create custom domain Contexts.

#### Creating a Domain Context

This example from the demo illustrates creation of a Context; configuration is essentially the same as with a simple domain, using fluent setter methods (see the [Configuration](CONFIGURATION.md) documentation for a complete list).

```php
namespace Viewflex\Ligero\Publish\Demo\Items;

use Viewflex\Ligero\Base\BaseContext;

class ItemsContext extends BaseContext
{
    
    public function __construct()
    {
        parent::__construct();

        $this
            ->setDomain('Items')
            ->setTranslationFile('items')
            ->setTableName('ligero_items')
            ->setModelName('Viewflex\Ligero\Publish\Demo\Items\Item')
            ->setResultsColumns([
                'id',
                'active',
                'name',
                'category',
                'subcategory',
                'description',
                'price'
            ])
            ->setWildcardColumns([
                'category'
            ])
            ->setControls([
                'pagination'        => true,
                'keyword_search'    => true
            ])
            ->setKeywordSearchColumns([
                'name',
                'category',
                'subcategory',
                'description'
            ])
            ->setQueryRules([
                'active'            => 'boolean',
                'name'              => 'max:60',
                'category'          => 'max:25',
                'subcategory'       => 'max:25'
            ])
            ->setRequestRules([
                'active'            => 'boolean',
                'name'              => 'max:60',
                'category'          => 'max:25',
                'subcategory'       => 'max:25',
                'description'       => 'max:250',
                'price'             => 'numeric'
            ]);
    }

}
```

#### Deploying an API for a Context

The demo included in this package uses the `ContextApiController`, which provides an API for Contexts:

```php
namespace Viewflex\Ligero\Controllers;

use App\Http\Controllers\Controller;
use Viewflex\Ligero\Contracts\ContextInterface as Context;

class ContextApiController extends Controller
{
    use HasContextApi;

    /**
     * @var array
     */
    protected $inputs;
    
    /**
     * @var array
     */
    protected $contexts;

    /**
     * @var Context
     */
    protected $context;

    public function __construct()
    {
        $this->inputs = json_decode(request()->getContent(), true);
        $this->contexts = config('ligero.contexts', []);
    }
    
}
```

You will notice that as with the UI controller, the API controller uses a trait (`HasContextApi`) containing the controller action methods, so they can be easily used in custom API controllers.

##### API Routes

The `ContextApiController` can serve APIs for any number of domains, providing both basic single-domain CRUD actions and more complex CRUD actions performed according to the custom logic of a Context (defaulting to standard CRUD actions where no custom logic is defined).

These are the built-in API routes for Context actions:

```php
// Standard CRUD domain actions...
Route::post('api/ligero/{key}/find', array('as' => 'api.ligero.find', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@find', 'middleware' => 'api'));
Route::post('api/ligero/{key}/findby', array('as' => 'api.ligero.findby', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@findBy', 'middleware' => 'api'));
Route::post('api/ligero/{key}/store', array('as' => 'api.ligero.store', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@store', 'middleware' => 'api'));
Route::post('api/ligero/{key}/update', array('as' => 'api.ligero.update', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@update', 'middleware' => 'api'));
Route::post('api/ligero/{key}/delete', array('as' => 'api.ligero.destroy', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@destroy', 'middleware' => 'api'));

// Custom context actions (defaulting to standard CRUD domain actions)...
Route::post('api/ligero/{key}/context/find', array('as' => 'api.ligero.context.find', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@findContext', 'middleware' => 'api'));
Route::post('api/ligero/{key}/context/findby', array('as' => 'api.ligero.context.findby', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@findContextBy', 'middleware' => 'api'));
Route::post('api/ligero/{key}/context/store', array('as' => 'api.ligero.context.store', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@storeContext', 'middleware' => 'api'));
Route::post('api/ligero/{key}/context/update', array('as' => 'api.ligero.context.update', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@updateContext', 'middleware' => 'api'));
Route::post('api/ligero/{key}/context/delete', array('as' => 'api.ligero.context.destroy', 'uses' => '\Viewflex\Ligero\Controllers\ContextApiController@destroyContext', 'middleware' => 'api'));
```

#### Context Responses

Context CRUD methods for both domain and contextual operations return a response consisting of success flag, message, and data.

### Presenters

A **Presenter** is a commonly used strategy component that provides the ability to attach custom presentation methods to a model, corresponding to any of the model's data columns (plus any calculated values required); the outputs of these methods are then transparently used instead of the column values themselves when outputting presented results.

This package's Presenter is more tightly integrated, providing the additional ability to inject a Publisher's Config, required for supported  advanced operations such as unit conversions and formatting. A Presenter can also be used on a model directly, apart from it's use by a Publisher.

By default a Presenter uses the [`Formatter`](https://github.com/viewflex/ligero/blob/master/src/Utility/Formatter.php) utility class, which includes some typical formatting and conversion methods. To modify or add new methods, extend or clone this class and specify the custom formatter class in the Publisher's Config (or use `setFormatter()` to specify a new formatter directly on a Presenter).

The `dynamicFields()` method of the [`BasePresenter`](https://github.com/viewflex/ligero/blob/master/src/Base/BasePresenter.php) can be overridden to customize the values presented. Here we see the custom Presenter used by the *Items* demo domain. To activate generation and display of the price presented in alternate currency units, enable unit conversions for the demo [as described below](#unit-conversions).

``` php
namespace Viewflex\Ligero\Publish\Demo\Items;

use Viewflex\Ligero\Base\BasePresenter;
use Viewflex\Ligero\Exceptions\PresenterException;

/**
 * Returns an array of dynamic fields for current item.
 *
 * @return array
 * @throws PresenterException
 */
class ItemPresenter extends BasePresenter
{

    public function dynamicFields()
    {
        $this->requireConfig();

        $data['price'] = $this->price();
        $data['alt_price'] = $this->config->getUnitConversions() ? $this->altPrice() : '';

        return $data;
    }

}
```

### Handling Relations

There are several ways to approach related tables, depending on the complexity of your requirements, and whether you need to write related data in addition to reading it. Depending on the database engine and schema you deploy, there may also be actions taken at the database level, such as cascading updates and deletes, and various other data-integrity constraints to consider.

#### Relations using Eloquent and Presenters

Laravel's Eloquent ORM provides a rich set of tools for defining and using relations on a model; using a Presenter we can easily read data using these relations.

##### Reading Eloquent Data

If you define relational methods on your Eloquent model, they can be called via the model's `present()` method. Modify the `dynamicFields()` method in your Presenter to include the output of these relational methods.


##### Writing Eloquent Data

To use `store()` or `update()` to modify data in related tables, you would need to extend the Repository to override those methods, adjusting your configuration and validation rules accordingly. You can also extend the Publisher along with the Repository, adding new methods to handle input with data for related tables.

#### Relations using Contexts

A "rich" Context can read and/or write data to other domains (tables), in addition to it's primary one. Each of the related domains (subcontexts) can be called using the standard Context methods, to manually maintain the arrangement of primary and related data.

### Automatic Handling of Timestamps

The `BasePublisherRepository` `store()`, `update()` and `delete()` methods transparently handle the columns `created_at`, `updated_at`, and `deleted_at` (supporting soft deletes), as well as `creator_id`, if these exist as results columns. This means you don't have to handle them explicitly when using the package CRUD actions.

### Localization

The `ls()` method wraps Laravel's `trans()` and `trans_choice()` helpers to provide more granular control of localization, via namespaces and files. See how the *Items* demo uses the package namespace, 'ligero', and a domain translation file, `items.php` to provide localized strings. This enhanced localization can be accessed within the scope of any Publisher, Publisher controller, or Context:

``` php
$word = $this->ls($key, $option)
```

### Unit Conversions

This package supports definition of dual currencies, ruler units, and weight units, and provides transparent conversion from primary to secondary units.
 Currency conversions are performed using live exchange rates. Before this functionality can be used you must configure the [viewflex/forex](https://github.com/viewflex/forex) package, which is installed as a dependency. The demo *Items* domain can be made to display dual currencies by adding this fluent setter in the demo UI controller or Context:
 
 ``` php
 $this->setUnitConversions(true)
 ```
