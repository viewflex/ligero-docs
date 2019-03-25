# Ligero

[![GitHub license](https://img.shields.io/github/license/mashape/apistatus.svg)](LICENSE.md)


## Configuration

The fluent methods described here support straighforward declarative configuration of a Publisher. See examples in the [QuickStart](README.md) and [Advanced Usage](ADVANCED.md) documentation, and [explore the codebase](https://github.com/viewflex/ligero), to understand how to fully harness this package's capabilities.

[QuickStart](README.md)  |  Configuration  |  [Advanced Usage](ADVANCED.md)

- [Domain Configuration](#domain-configuration)
- [Global Configuration](#global-configuration)
- [Input Parameters](#input-parameters)




### Domain Configuration

These settings are specific to the given domain, getting their defaults from the [`BasePublisherConfig`](https://github.com/viewflex/ligero/blob/master/src/Base/BasePublisherConfig.php) class.

#### Domain, Resource Namespaces, Translation

Specify the domain name used in messages and labels ('Items' in the demo) - this is also used (lower-cased) as part of the the domain view prefix. For multiple-word domain names, name should be in `StudlyCaps` case (no spaces).

```php
setDomain($domain)
```

Specify the resource namespace used to locate published resources in the application's `resources/lang/vendor/[namespace]` and `resources/views/vendor/[namespace]` directories. The package default is 'ligero', but you can set a custom namespace for domain resources - useful if there are many, or if non-canonical names result in naming conflicts between domains.

```php
setResourceNamespace($resource_namespace)
```

Specify a translation filename ('items' in the demo) to get translations from a specific file in the configured resource namespace.

```php
setTranslationFile($translation_file)
```

#### Table and Model

Specify table and model name used in queries.

```php
setTableName($table_name)
setModelName($model_name)
```

#### Query Parameter Defaults, Results Columns, Wildcard Columns

Specify any default values desired for query parameters. This can be useful when you want some parameters to always be there, unseen; unless overridden by explicit inputs, they will not appear in generated URL query strings.

```php
setQueryDefaults([
    'view'              => 'list',
    'sort'              => 'latest'
])
```

```php
setQueryDefault('view' => 'list')
```

Specify all columns to return in query results (default, and optionally, per view).

```php
setResultsColumns([
    'id',
    'author',
    'title',
    'description'
])
```

Specify string columns to treat as wildcards (using `'LIKE'` instead of the `'='` operator) in explicit queries on that column (as opposed to keyword search, which treats all target columns as wildcards).

```php
setWildcardColumns([
    'author',
    'title'
])
```

#### Toggles for UI Controls

Enable or disable generation of individual dynamic UI controls, such as pagination and keyword search..

```php
setControls([
    'pagination'        => true,
    'keyword_search'    => false
])
```

``` php
setControl('keyword_search', false)
```

#### Pagination

This is the specification for a detailed configuration of pagination UI controls.

```php
setPaginationConfig([
	'pager'             =>  [
	    'make'              =>  true,
	    'context'           =>  'logical',
	],
	'page_menu'         =>  [
	    'make'              =>  true,
	    'context'           =>  'logical',
	    'max_links'         =>  5
	],
	'view_menu'         =>  [
	    'make'              =>  true,
	    'context'           =>  'logical',
	],
	'use_page_number'   =>  true
])
```

Specify individually whether to generate data for the **pager**, **page menu**, and **view menu** UI controls.

Specify the navigational context to use in generating links. For the pager and page menu, specify 'logical' (zero-start) or 'relative' (to the current start position, even if it's offset from the logical pagination). The number of pages displayed in the page menu can be controlled by 'max_links'.

With the view menu there is an additional possibility when switching view modes - the 'fresh' context, which returns to the first page, rather than maintaining relative or logical position in the paginated results.

Also specify whether to use the 'page' parameter in links (when generated in 'logical' context) - otherwise the 'start' parameter is used. The 'page' parameter can be useful for SEO and other purposes, but it's essentially an extra pagination parameter that gets converted to a 'start' value, if no 'start' is specified either explicitly or as a default.

#### Keyword Search

Below is a detailed configuration example for the keyword search UI control. Scope can be 'query' (search within current results) or 'global'. Specify whether to persist the sort and view parameters in a query, or omit them and let the defaults be used instead. Also specify whether to persist the keyword as a placeholder in the search text box.

```php
setKeywordSearchConfig([
    'columns'           =>  [
        'author',
        'title',
        'description'
    ],
    'scope'             =>  'query',
    'persist_sort'      =>  true,
    'persist_view'      =>  true,
    'persist_input'     =>  false,
    'on_change'         => 'this.form.submit()'
])
```

Configuring just the search columns:

``` php
setKeywordSearchColumns([
    'author',
    'title',
    'description'
])
```

#### Sorts and View/Limit

Define named sorts available via the 'sort' query parameter, and specify limits for the default and named views (list, grid, item, and any custom views you create). These examples use the package's default values.

```php
setSorts(['default' => ['id' => 'asc']])
```

```php
setViewLimits([
    'default'           =>  10,
    'list'              =>  5,
    'grid'              =>  20,
    'item'              =>  1
])
```

```php
setViewLimit('default', 10)
```

### Global Configuration

These settings get their defaults from the package's [`ligero.php`](https://github.com/viewflex/ligero/blob/master/src/Config/ligero.php) config file (which can be published and customized). Using these methods you can override the global configuration values for a given domain as needed.

#### Caching and Logging

Enable/disable the cache, and specify cache lifetime, passing an array.

```php
setCaching([
    'active'            =>  true,
    'minutes'           =>  10
])
```

Enable/disable the cache, passing a boolean value.

```php
setCaching(true)
```

Enable/disable logging passing an array. Custom elements can be added to provide logging options.

```php
setLogging([
    'active'            =>  true
])
```

Enable/disable logging, passing a boolean value.

```php
setLogging(true)
```

#### URL Format, Paths, Options

Specify whether the URLs generated for search and navigation should be absolute or relative. Paths and options can be defined here for any purpose required.

```php
setAbsoluteUrls($absolute_urls)
setPaths($paths)
setPath($name, $path)
setOptions($options)
setOption($name, $option)
```

#### Unit Formatting and Conversions

Specify formats for currencies, ruler units, and weight units. Enable unit conversions to take advantage of dual currencies and measurement units. Currency conversions are performed based on live exchange rates (the [viewflex/forex](https://github.com/viewflex/forex) package, installed as a dependency, must be configured to take advantage of this feature). Optionally specify a custom formatter class to use in place of the default `Formatter`.

```php
setFormatter($formatter)
setUnitConversions($unit_conversions)
setCurrencies($currencies)
setRulerUnits($ruler_units)
setWeightUnits($weight_units)
```

#### Tables, Models and Contexts for Multi-Domain Implementations

You might find these useful for a multi-domain app where you want to have a single source of truth for all domain tables, models, and contexts, resolved by giving precedence to those listed in the `ligero.php` config file, keyed by the domain's `$table_name`. See the [Advanced Usage](ADVANCED.md) documentation for more about custom multi-domain implementations.

```php
setTables($tables)
setModels($models)
setContexts($contexts)
```

### Input Parameters

#### Validation Rules for GET and POST Inputs

Input parameters are validated according to the appropriate rules array. Specify all valid custom inputs and their rules (no need to specify internally supported parameters). In a resourceful UI controller, typically the GET rules are used for queries and the POST rules are used for store/update/destroy requests.

```php
setQueryRules([
    'author'            => 'max:100',
    'title'             => 'max:100',
])
```

```php
setQueryRule('title', 'max:100')
```

```php
setRequestRules([
    'author'            => 'max:100',
    'title'             => 'max:100',
    'description'       => 'max:250'
]);
```

```php
setRequestRule('description', 'max:250')
```

#### Replace or Merge Inputs

For situations requiring setting the inputs directly on a Publisher.

```php
setInputs($inputs)
mergeInputs($inputs)
```

#### Inputs for Action, Items, and Options

Supporting actions ('clone' and 'delete' are two built-in actions) to perform on a selection of records, with custom options.

```php
setAction($action)
setActionItems($items)
setActionOptions($options)
```

#### Mapping Inputs to Database Columns

Specify the mapping for any input parameters named differently than their corresponding columns in the data source.

``` php
setColumnMap([
    'author'            => 'author_full',
    'title'             => 'title_short',
])
```
