# Laravel Core Package
Luezoid came up with a compact way to help one creating the APIs very fast & without much hassle. Using this package, one case easily create simple CRUD operations in Laravel framework in couple of minutes with just creating a few files & configuring the components. With a lot of efforts and deep analysis of day-to-day problems (our) developers faced in the past, we came up with a sub-framework of it's own kind to simplify & fasten up the REST APIs creating process.

A few cool features of this package are:
 1. Simplest & fastest way to create [CRUD](#1-creating-crud)s.
 2. Pre-built support to define table columns which are to be [specifically excluded](#2-exclude-columns-for-default-post--put-requests) before creating/updating a record(in default CRUD).
 3. Pre-built [Search & Filters](#3-searching--filters) ready to use with just configuring components.
 4. Pre-built [Pagination](#4-pagination) ready.
 5. [Relationship's data](#5-relationships-data) in the APIs(GET) is just a config thing.
 6. Better way to correctly [fire an event](#6-attach-event-on-an-action-success) upon successful completion of an action.
 7. File uploads has never been easy before.
 8. Pre-built feature rich Service classes eg. [EnvironmentService](src/services/EnvironmentService.php), [RequestService](src/services/RequestService.php), [UtilityService](src/services/UtilityService.php), etc.
 9. Nested Related models can be queried with simple config based approach from the code components.
 10. On the go filters can be passed as JSON in query params to select particular columns from a table(and related objects defined in models) making the API's response with less garbage data instead of writing custom query every time a new endpoint is created.
 11. On the go searching over the related objects with simple Array based config. Much more effective when a generic search has to be made over a couple of related tables.
>**Note:** For a complete working example of all these feature with core package pre-configured is available on this repository [luezoidtechnologies/laravel-core-base-repo-example](https://github.com/luezoidtechnologies/laravel-core-base-repo-example "laravel-core-base-repo-example").

## Installation
We recommend using [Composer](https://getcomposer.org) to install this package. This package supports [Laravel](https://laravel.com) versions >= 5.x.

    composer require "luezoid/laravel-core"		# For latest version of Laravel (>=7.x)
    composer require "luezoid/laravel-core:^6.0"	# For Laravel version 6.x
    composer require "luezoid/laravel-core:^5.0"	# For Laravel version 5.x
Next, configure your `app/Exceptions/Handler.php` and extend it with `Luezoid\Laravelcore\Exceptions\Handler`. Sample file can be seen [here](/examples/Exceptions/Handler.php).

## 1. Creating CRUD
Using this packages adds an extra entity between the Controller & Model paradigm in a normal MVC architecture. This extra entity we are referring here is called **Repository**. A **Repository** is a complete class where your whole business logic resides from getting the processed data from a **Controller** and saving it into database using **Model**(s).
By using **Repository** as an intermediate between **Controller** & **Model**, we aim at maintaing clean code at **Controller's** end and making it a mediator which only receives data(from View, typically a REST route), validate it against the defined validation rules(if any, we uses **Request** class to define such rules), pre-process it(for eg. transform ***camelCased*** data from front-end into ***snake_case***) & sending business cooked data back the View.

Let's start with creating a simple **Minions** CRUD.

We have sample [migration](examples/migrations/2020_04_24_175321_create_minions_table.php) for table `minions`, model [`Minion`](/examples/Models/Minion.php), controller [`MinionController`](/examples/Controllers/MinionController.php) and repository [`MinionRepository`](/examples/Repositories/MinionRepository.php).
Add these files into your application & adjust the namespaces accordingly. Then create a Route resource as below in [`routes/api.php`](examples/routes/api.php) and we are all ready:

    Route::resource('minions', 'MinionController', ['parameters' => ['minions' => 'id']]);
Assuming your local server is running on port: 7872, try hitting REST endpoints as below:

 1. POST /minions

	     curl -X POST \
	      http://localhost:7872/api/minions \
	      -H 'Content-Type: application/json' \
	      -H 'cache-control: no-cache' \
	      -d '{
	    	"name": "Stuart",
	    	"totalEyes": 2,
	    	"favouriteSound": "Grrrrrrrrrrr",
	    	"hasHairs": true
	    }'
 2. PUT /minions/1

	    curl -X PUT \
	      http://localhost:7872/api/minions/1 \
	      -H 'Content-Type: application/json' \
	      -H 'cache-control: no-cache' \
	      -d '{
	    	"name": "Stuart - The Pfff",
	    	"totalEyes": 2,
	    	"favouriteSound": "Grrrrrrrrrrr Pffffff",
	    	"hasHairs": false
	    }'
 3. DELETE /minions/1

	    curl -X DELETE \
	      http://localhost:7872/api/minions/1 \
	      -H 'cache-control: no-cache'
 4. GET /minions

	    curl -X GET \
	      http://localhost:7872/api/minions \
	      -H 'cache-control: no-cache'
 5. GET /minions/2
 
     	curl -X GET \
    	  http://localhost:7872/api/minions/2 \
    	  -H 'cache-control: no-cache'

## 2. Exclude columns for default POST & PUT request(s)
Refer the [`Minon`](examples/Models/Minion.php "Minon") model. We have the below public properties which can be used to define an array containing list of the columns to be specifically excluded for default **POST** & **PUT** requests of [CRUD](#1-creating-crud "CRUD"):

	// To exclude the key(s) if present in request body for default POST CRUD routes eg. POST /minions
    public $createExcept = [
    	'id'
    ];

	// To exclude the key(s) if present in request body for default PUT CRUD routes eg. PUT /minions/1
    public $updateExcept = [
    	'total_eyes',
    	'has_hairs'
    ];
The major advantage for using such config in the model in the first place is: to provide a clean & elegant way & to simply reduce the coding efforts to be done just before saving the whole data into table. Typical examples could be:
1. You don't want to save a column value say ***is_email_verified*** if an attacker sends it the request body of POST /users; just add it into `$createExcept`. You need not to exclude this specifically in the codes/or request rules.
2. You don't want to update a column say ***username*** if an attacker sends it the request body of PUT /users/{id}; just add it into `$updateExcept`. You need not to exclude this specifically in the codes/or request rules.

## 3. Searching & Filters
You can simply search over the list of available column(s) in the table for all GET requests. Let's begin with examples:
- **General Searching**

    By default all the available columns in the tables are ready to be queried over GET request just by passing the key(s)-value(s) pair(s) in the query params. But to specifically mention it in the Model itsef, just define a public property `$searchable` which is an array containing the columns allowed to be searched.
Let's say you want to search for all minions whose favourite sound is ***Pchhhh***.

		curl -X GET \
		  'http://localhost:7872/api/minions?favouriteSound=Pchhhh' \
		  -H 'cache-control: no-cache'
    Response should contain all minions having ***Pchhhh*** string available in the column `favourite_sound` in the table `minions`.
    > Searching is using `LIKE` operator as `'%-SEARCH-STRING%'`.
- **General Filtering**

    Similar to searching, you need to define public property `$filterable` which is again an array containing the columns allowed to be filtered.

		public $filterable = [
			'id',
			'total_eyes',
			'has_hairs'
		];
    Exact match will performed against these columns if present in the query params.

    Example: To find all the minions having `totalEyes` equal to 1:

    	curl -X GET \
    	  'http://localhost:7872/api/minions?totalEyes=1' \
    	  -H 'cache-control: no-cache'
    > Filtering is using `=` operator as `'total_eyes'=1`.
- **Date Filters**

    Add the column which you want to be used for date filtering into `$filterable` property in the model class. Once done, you can now simply pass the query params **from** (AND/OR) **to** with the date(or datetime) values in standard MySQL format(`Y-m-d H:i:s`).
Example: We want to search for all the minions which are created after **2020-04-25  09:25:20**:

    	curl -X GET \
    	  'http://localhost:7872/api/minions?createdAt=2020-04-25%20%2009:25:20' \
    	  -H 'cache-control: no-cache'
    You can also pass the column name in the query params over which you want the date search to be applied for. Just pass the key **dateFilterColumn** with the column name you want to use date search on (but note that the **$filterable** property must have this column specified in order to make things work).
> Notes:
>1. You may specify multiple key-value pairs in the query params & all the conditions will be queried with `AND` operators.
>2. Pass all the variables in **camelCasing** & all will be transferred into **snake_casing** internally. You may configure this transformation **turning off** by **overriding properties** `$isCamelToSnake` & `$isSnakeToCamel` and setting them to `false` in [ApiCotroller](src/Http/Controllers/ApiController.php "ApiCotroller").

## 4. Pagination
Did you notice the response of GET endpoint we just created above? Let's take a look in brief. Refer the [response](examples/Responses/get-minions-paginated-response.json "response") of **GET /minions** API.

    {
      "message": null,
      "data": {
        "items": [
          {},
          {},
          ...
        ],
        "page": 1,      // tells us about the current page
        "total": 6,     // tells us the total results available (matching all the query params for searching/filtering applied)
        "pages": 1,     // total pages in which the whole result set is distributed
        "perpage": 15   // total results per page
      },
      "type": null
    }
Pretty self explanatory, eh?

You can pass query param **perpage=5** (to limit the per page size). Similarly, the **page=2** will grab the results of page 2.

Paginating the results has never been so easy before :)
> Note: Any GET(index) route retrieving results from a Repository (eg.[MinionRepository](src/examples/Repositories/MinionRepository.php "MinionRepository")) extending `\Luezoid\Laravelcore\Repositories\EloquentBaseRepository::getAll()` is all ready with such pagination. Make sure to use this pre-built feature & save time for manually implementing pagination & grab a pint of beer to chill.

## 5. Relationship's data
Let's assume each **Minion** leads a mission operated by **Gru** i.e. there is one-to-one relationship exists between Minion & Missions. See the `missions` table migrations([1](examples/migrations/2020_04_25_193714_create_missions_table.php),[2](examples/migrations/2020_04_25_193715_add_foreign_keys_to_missions_table.php)) and Model [`Mission`](examples/Models/Mission.php). To retrieve the leading mission by each Minion in GET requests(index & show), just add the relationship name in the [`MinionController`](/examples/Controllers/MinionController.php) properties as follows:
- GET /minions

      protected $indexWith = [
          'leading_mission'	// name of the hasOne() relationship defined in the Minion model
      ];
- GET /minions/2

      protected $showWith = [
          'leading_mission'	// name of the hasOne() relationship defined in the Minion model
      ];

That's it. Just a config thingy & you can see in the response each **Minion** object contains another object **leadingMission** which is an instance of [`Mission`](examples/Models/Mission.php) model lead by respective Minion.

> Note: For nested relationships, you can define them appending dot(.) operator eg. `employee.designations`.

## 6. Attach Event on an action success
Let's arrange an [Event](#) to get triggered to bring a **Minion** to **Gru's** lab whenever a new [`Mission`](examples/Models/Mission.php) is created leading by the **Minion**. Create a POST route in [`routes/api.php`](examples/routes/api.php):

    Route::post('missions', 'MissionController@createMission')->name('missions.store');

We need to have [`MissionController`](examples/Controllers/MissionController.php), [`MissionRepository`](examples/Repositories/MissionRepository.php), [`MissionCreateRequest`](examples/Requests/MissionCreateRequest.php) and [`MissionCreateJob`](examples/Jobs/MissionCreateJob.php) ready for this route to work.
Also we need to have an Event say [`BringMinionToLabEvent`](examples/Events/BringMinionToLabEvent.php) ready to be triggered & the same to be configured into job [`MissionCreateJob`](examples/Jobs/MissionCreateJob.php) under property `public $event = BringMinionToLabEvent::class;`
Now try hitting the route POST /missions as follows:

    curl -X POST \
      http://localhost:7872/api/missions \
      -H 'Content-Type: application/json' \
      -H 'cache-control: no-cache' \
      -d '{
    	"name": "Steal the Moon! Part - 4",
    	"description": "The first moon landing happened in 1969. Felonius Gru watched this historic moment with his mother and was inspired by the landing to go to outer space just like his idol Neil Armstrong.",
    	"minionId": 2
    }'
You should be able to see a log entry under file `storage/logs/laravel.log` which is the action we had set in the event [`BringMinionToLabEvent`](examples/Events/BringMinionToLabEvent.php).

## FILTERS - SELECT PARTICULAR FIELDS
**k** is keys, **r** is relation, **cOnly** is flag to set count is needed or the relational data  
  
**cOnly** flag can be used in **r** relations nestedly

    {
      "cOnly": true,
      "k": [
        "id",
        "firstName",
        "lastName"
      ],
      "r": {
        "designation": {
          "k": [
            "id",
            "name",
            "departmentId"
          ],
          "r": {
            "department": {
              "k": [
                "id",
                "name"
              ],
              "r": null
            }
          }
        },
        "salaryBreakups": {
          "k": [
            "id"
          ],
          "r": null
        },
        "salaryBreakup": {
          "k": [],
          "r": null
        }
      }
    }

  
## Search  
Need to send key `search_key` via query params along with value.  
In repo, define config along with models & column names to be searched on.  
**Usage:**  
  

    $searchConfig = [  
      'abc' => [  
      'model' => Abc::class,  
      'keys' => ['name', 'field']  
     ],  
      'groups' => ['model' => ProductGroup::class,  
      'keys' => ['name', 'code', 'hsn_code']  
     ]];  
    return $this->search($searchConfig, $params);

  
## License  
Laravel-core is released under the MIT License. See the bundled [LICENSE](LICENSE) for details.
