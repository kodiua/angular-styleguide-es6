# Angular Style Guide ES2015/ES6

This is an ES2015/ES6 fork of the popular Angular 1.x Style Guide by John Papa. It was originally written for use with generator-gulp-angular and Babel, and things that do not apply in that circumstance have been removed.

**Note:** generator-gulp-angular has been deprecated, but the people working on it are now working on [FountainJS](http://fountainjs.io/), which I highly recommend as a starting point. Just make sure to choose Angular 1.x and Babel to stay in line with this guide.

Please be advised that all examples will not be copy/paste working examples. In instances where classes are imported,
it is expected that the imported class was defined correctly, in another file, and imported in.

## Table of Contents

  1. [Single Responsibility](#single-responsibility)
  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [No Factories](#factories)
  1. [Data Services](#data-services)
  1. [Directives](#directives)
  1. [Components](#components)
  1. [Resolving Promises](#route-resolve-promises)
  1. [Minification and Annotation](#minification-and-annotation)
  1. [Exception Handling](#exception-handling)
  1. [Naming](#naming)
  1. [Application Structure LIFT Principle](#application-structure-lift-principle)
  1. [Application Structure](#application-structure)
  1. [Modularity](#modularity)
  1. [Startup Logic](#startup-logic)
  1. [Angular $ Wrapper Services](#angular--wrapper-services)
  1. [Testing](#testing)
  1. [Animations](#animations)
  1. [Comments](#comments)
  1. [ESLint](#eslint)
  1. [Constants](#constants)
  1. [File Templates and Snippets](#file-templates-and-snippets)
  1. [Keoman Generator](#yeoman-generator)
  1. [Routing](#routing)
  1. [Task Automation](#task-automation)
  1. [Filters](#filters)
  1. [Angular Docs](#angular-docs)
  1. [Contributing](#contributing)
  1. [License](#license)

## Single Responsibility

### Rule of 1
###### [Style [K001](#style-k001)]

  - Define 1 module per file.

  The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.

  ```js
  /* avoid */

 class SomeController {
    constructor() { }
  }

  class someFactory{
    constructor() { }
  }

  angular
      .module('app', ['ngRoute'])
      .controller('SomeController', SomeController)
      .factory('someFactory', someFactory);
  ```

  The same entities are now separated into their own files.

  ```js
  /* recommended */

  // app.module.js
  import { SomeController } from './some.controller';
  import { someFactory } from './some.factory';

  angular
      .module('app', ['ngRoute'])
      .controller('SomeController', SomeController)
      .factory('someFactory', someFactory);
  ```

  ```javascript
  /* recommended */

  // some.controller.js
  class SomeController{
    constructor() { }
  }
  ```

  ```javascript
  /* recommended */

  // some.factory.js
  class someFactory{
    constructor() { }
  }
  ```

**[Back to top](#table-of-contents)**

## Modules

### Avoid Naming Collisions
###### [Style [K020](#style-k020)]

  - Use unique naming conventions with separators for sub-modules.

  *Why?*: Unique names help avoid module name collisions. Separators help define modules and their submodule hierarchy. For example `app` may be your root module while `app.dashboard` and `app.users` may be modules that are used as dependencies of `app`.

### Definitions (aka Setters)
###### [Style [K021](#style-k021)]

  - Declare modules without a variable using the setter syntax.

  *Why?*: With 1 entity per file, there is rarely a need to introduce a variable for the module.

  ```javascript
  /* avoid */
  const app = angular.module('app', [
      'ngAnimate',
      'ngRoute',
      'app.shared',
      'app.dashboard'
  ]);
  ```

  Instead use the simple setter syntax.

  ```javascript
  /* recommended */
  angular
      .module('app', [
          'ngAnimate',
          'ngRoute',
          'app.shared',
          'app.dashboard'
      ]);
  ```

### Getters
###### [Style [K022](#style-k022)]

  - When using a module, avoid unnecessarily using variables and instead use chaining with the getter syntax.

  *Why?*: This produces more readable code and avoids variable collisions or leaks.

  ```javascript
  /* avoid */
  import { SomeController } from './some.controller';

  const app = angular.module('app');
  app.controller('SomeController', SomeController);
  ```

  ```javascript
  /* recommended */
  import { SomeController } from './some.controller';

  angular
      .module('app')
      .controller('SomeController', SomeController);
  ```

### Setting vs Getting
###### [Style [K023](#style-k023)]

  - Only set once and get for all other instances.

  *Why?*: A module should only be created once, then retrieved from that point and after.

  ```javascript
  /* recommended */

  // to set a module
  angular.module('app', []);

  // to get a module
  angular.module('app');
  ```

### Named vs Anonymous Functions
###### [Style [K024](#style-k024)]

  - Use named functions instead of passing an anonymous function in as a callback.

  *Why?*: This produces more readable code, is much easier to debug, and reduces the amount of nested callback code.

  ```javascript
  /* avoid */
  angular
      .module('app')
      .controller('Dashboard', () => { })
  ```

  ```javascript
  /* recommended */

  // dashboard.controller.js

  class Dashboard {
    constructor() { }
  }

  // index.module.js
  import { Dashboard } from './dashboard.controller';

  angular
      .module('app')
      .controller('Dashboard', Dashboard);
  ```

**[Back to top](#table-of-contents)**

## Controllers

### controllerAs View Syntax
###### [Style [K030](#style-k030)]

  - Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the `classic controller with $scope` syntax.

  *Why?*: Controllers are constructed, "newed" up, and provide a single new instance, and the `controllerAs` syntax is closer to that of a JavaScript constructor than the `classic $scope syntax`.

  *Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

  *Why?*: Helps avoid using `$parent` calls in Views with nested controllers.

  ```html
  <!-- avoid -->
  <div ng-controller="Customer">
      {{name}}
  </div>
  <div ng-controller="Customer as customer">
      {{customer.name}}
  </div>
  ```

  ```html
  <!-- recommended -->
  Set in the router OR use component oriented approach (.component, .directive)
  $stateProvider
            .state("root.reports", {
                url: "/reports",
                templateUrl: "reports/views/dashboard.html",
                controller: 'reportManagementController',
                controllerAs: 'reports'
            })
  ```

### controllerAs Controller Syntax
###### [Style [K031](#style-k031)]

  - Use the `controllerAs` syntax over the `classic controller with $scope` syntax.

  - The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

  *Why?*: `controllerAs` is syntactic sugar over `$scope`. You can still bind to the View and still access `$scope` methods.

  *Why?*: Helps avoid the temptation of using `$scope` methods inside a controller when it may otherwise be better to avoid them or move the method to a factory, and reference them from the controller. Consider using `$scope` in a controller only when needed. For example when publishing and subscribing events using [`$emit`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$emit), [`$broadcast`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$broadcast), or [`$on`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on) consider moving these uses to a factory and invoke from the controller.

  ```javascript
  /* avoid */
  class Customer {
    constructor($scope) {
      $scope.name = {};
      $scope.sendMessage = function() { };
    }
  }
  ```

  ```javascript
  /* recommended - but see next section */
  class Customer {
    constructor() {
      this.name = {};
    }
    sendMessage(){ }
  }
  ```

### controllerAs with fat arrows
###### [Style [K032](#style-k032)]

  - Using a capture variable for `this` when using the `controllerAs` syntax, is not necessary with ES6. You can simply use a fat arrow to automatically reference the correct `this` context


  ```javascript
  /* avoid */
  let self = this;
  () => {
    self.foo = 'bar';
  }
  ```

  ```javascript
  /* recommended */
  () => {
    this.foo = 'bar';
  }
  ```

### Defer Controller Logic to Services
###### [Style [K035](#style-k035)]

  - Defer logic in a controller by delegating to services and factories.

    *Why?*: Logic may be reused by multiple controllers when placed within a service and exposed via a function.

    *Why?*: Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

    *Why?*: Removes dependencies and hides implementation details from the controller.

    *Why?*: Keeps the controller slim, trim, and focused.

  ```javascript

  /* avoid */
  class Order {
    constructor($http, $q, config, userInfo) {
      this.isCreditOk = false;
      this.total = 0;
    }
    checkCredit() {
      let settings = {};
      // Get the credit service base URL from config
      // Set credit service required headers
      // Prepare URL query string or data object with request data
      // Add user-identifying info so service gets the right credit limit for this user.
      // Use JSONP for this browser if it doesn't support CORS
      return $http.get(settings)
        .then((data) => {
        // Unpack JSON data in the response object
        // to find maxRemainingAmount
          this.isCreditOk = this.total <= maxRemainingAmount
        })
        .catch((error) => {
          // Interpret error
          // Cope w/ timeout? retry? try alternate service?
          // Re-reject with appropriate error for a user to see
        });
      };
  }
  ```

  ```javascript
  /* recommended */
  class Order {
    constructor (creditService) {
      this.isCreditOk;
      this.total = 0;
    }
    checkCredit() {
      return creditService.isOrderTotalOk(this.total)
        .then((isOk) => { this.isCreditOk = isOk; })
        .catch(showError);
    }
  }
  ```

### Keep Controllers Focused
###### [Style [K037](#style-k037)]

  - Define a controller for a view, and try not to reuse the controller for other views. Instead, move reusable logic to factories and keep the controller simple and focused on its view.

    *Why?*: Reusing controllers with several views is brittle and good end-to-end (e2e) test coverage is required to ensure stability across large applications.

### Assigning Controllers
###### [Style [K038](#style-k038)]

  - When a controller must be paired with a view and either entity may be re-used by other controllers or views, define controllers along with their routes.

    Note: If a View is loaded via another means besides a route, then use the `ng-controller="AvengersController as avengers"` syntax.

    *Why?*: Pairing the controller in the route allows different routes to invoke different pairs of controllers and views. When controllers are assigned in the view using [`.component`](https://docs.angularjs.org/guide/component)
 ```javascript
  /* avoid - when using with a route and dynamic pairing is desired */

  // route-config.js
  angular
      .module('app')
      .config(config);

  function config($routeProvider) {
      $routeProvider
          .when('/avengers', {
            templateUrl: 'avengers.html'
          });
  }
  ```

  ```html
  <!-- avengers.html -->
  <div ng-controller="AvengersController as avengers">
  </div>
  ```

  ```javascript
  /* recommended */

  // route-config.js
  angular
      .module('app')
      .config(config);

  function config($routeProvider) {
      $routeProvider
          .when('/avengers', {
              templateUrl: 'avengers.html',
              controller: 'AvengersController',
              controllerAs: 'avengers'
          });
  }
  ```

  ```html
  <!-- avengers.html -->
  <div>
  </div>
  ```

**[Back to top](#table-of-contents)**


## No factories

### Use Services instead of Factories
###### [Style [K050](#style-k050)]

  - [SERVICE VS FACTORY](http://blog.thoughtram.io/angular/2015/07/07/service-vs-factory-once-and-for-all.html)

**[Back to top](#table-of-contents)**

## Data Services

### Separate Data Calls
###### [Style [K060](#style-k060)]

  - Refactor logic for making data operations and interacting with data to a factory. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

    *Why?*: The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

    *Why?*: This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

    *Why?*: Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as `$http`. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

    Note: The following ES6 factory definition uses the `new` operator to instantiate the factory function, and injects the desired services used internally.

    ES6: With Angular 1.x and ES6, use of factories will decrease and `services` should be used moving forward (see angular [services](#data-services))

  ```javascript
  /* recommended */

  // dataservice factory
  angular.module('app.core')
      .factory('dataservice', ['$http', 'logger', ($http, logger)
        => new Dataservice($http, logger)]);

  class Dataservice {
    constructor($http, logger) {
      this.$http = $http;
      this.logger = logger;
    }
    getAvengers() {
      return this.$http.get('/api/maa')
        .then(getAvengersComplete)
        .catch(getAvengersFailed);

      getAvengersComplete(response) => {
        return response.data.results;
      }

      getAvengersFailed(error) => {
        this.logger.error('XHR Failed for getAvengers.' + error.data);
      }
    }
  }
  ```

### Return a Promise from Data Calls
###### [Style [K061](#style-k061)]

  - When calling a data service that returns a promise such as `$http`, return a promise in your calling function too.

    *Why?*: You can chain the promises together and take further action after the data call completes and resolves or rejects the promise.

  ```javascript
  /* recommended */

  activate() {
      /**
       * Step 1
       * Ask the getAvengers function for the
       * avenger data and wait for the promise
       */
      return getAvengers().then(() => {
          /**
           * Step 4
           * Perform an action on resolve of final promise
           */
          logger.info('Activated Avengers View');
      });
  }

  getAvengers() {
        /**
         * Step 2
         * Ask the data service for the data and wait
         * for the promise
         */
        return dataservice.getAvengers()
            .then((data) => {
                /**
                 * Step 3
                 * set the data and resolve the promise
                 */
                this.avengers = data;
                return this.avengers;
        });
  }
  ```

**[Back to top](#table-of-contents)**

## Directives
### Limit 1 Per File
###### [Style [K070](#style-k070)]

  - Create one directive per file. Name the file for the directive.

    *Why?*: It is easy to mash all the directives in one file, but difficult to then break those out so some are shared across apps, some across modules, some just for one module.

    *Why?*: One directive per file is easy to maintain.

    > Note: "**Best Practice**: Directives should clean up after themselves. You can use `element.on('$destroy', ...)` or `scope.$on('$destroy', ...)` to run a clean-up function when the directive is removed" ... from the Angular documentation.

  ```javascript
  /* avoid */
  /* directives.js */

  class orderCalendarRange {
      /* implementation details */
  }

  class salesCustomerInfo {
      /* implementation details */
  }

  class sharedSpinner {
      /* implementation details */
  }
  ```

  ```javascript
  /* recommended */

  /* calendarRange.directive.js */

  /**
   * @desc order directive that is specific to the order module at a company named Acme
   * @example <div acme-order-calendar-range></div>
   */

  class orderCalendarRange {
      /* implementation details */
  }
  ```

  ```javascript
  /* recommended */

  /* customerInfo.directive.js */

  /**
   * @desc sales directive that can be used anywhere across the sales app at a company named Acme
   * @example <div acme-sales-customer-info></div>
   */

  class salesCustomerInfo {
      /* implementation details */
  }
  ```

  ```javascript
  /* recommended */

  /* spinner.directive.js */

  /**
   * @desc spinner directive that can be used anywhere across apps at a company named Acme
   * @example <div acme-shared-spinner></div>
   */

  class sharedSpinner {
      /* implementation details */
  }
  ```

    Note: There are many naming options for directives, especially since they can be used in narrow or wide scopes. Choose one that makes the directive and its file name distinct and clear. Some examples are below, but see the [Naming](#naming) section for more recommendations.

### Manipulate DOM in a Directive
###### [Style [K072](#style-k072)]

  - When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hides and shows, use ngHide/ngShow.

    *Why?*: DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templates)

  ```javascript
  /* avoid */

  /* patient.controller.js */
  class sharedSpinner {
      rowDelete(id) {
        $('.'+id).remove();
    }
  }

  /* recommended */

  Use .directive

  ```

### Provide a Unique Directive Prefix
###### [Style [K073](#style-k073)]

  - Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which would be declared in HTML as `acme-sales-customer-info`.

    *Why?*: The unique short prefix identifies the directive's context and origin. For example a prefix of `cc-` may indicate that the directive is part of a CodeCamper app while `acme-` may indicate a directive for the Acme company.

    Note: Avoid `ng-` as these are reserved for Angular directives. Research widely used directives to avoid naming conflicts, such as `ion-` for the [Ionic Framework](http://ionicframework.com/).

### Restrict to Elements and Attributes
###### [Style [K074](#style-k074)]

  - When creating a directive that makes sense as a stand-alone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when it's stand-alone and as an attribute when it enhances its existing DOM element.

    *Why?*: It makes sense.

    *Why?*: While we can allow the directive to be used as a class, if the directive is truly acting as an element it makes more sense as an element or at least as an attribute.

    Note: EA is the default for Angular 1.3 +

  ```html
  <!-- avoid -->
  <div class="my-calendar-range"></div>
  ```

  ```javascript
  /* avoid */

  class myCalendarRange {
    constructor() {
      this.link = this.linkFunc;
      this.templateUrl = '/template/is/located/here.html';
      this.restrict = 'C';
    }
    linkFunc(scope, element, attrs) {
      /* */
    }
  }
  ```

  ```html
  <!-- recommended -->
  <my-calendar-range></my-calendar-range>
  <div my-calendar-range></div>
  ```

  ```javascript
  /* recommended */

  class myCalendarRange {
    constructor() {
      this.link = this.linkFunc;
      this.templateUrl = '/template/is/located/here.html';
      this.restrict = 'EA';
    }
    linkFunc(scope, element, attrs) {
      /* */
    }
  }
  ```

### Directives and ControllerAs
###### [Style [K075](#style-k075)]

  - Use `controller as` syntax with a directive to be consistent with using `controller as` with view and controller pairings.

    *Why?*: It makes sense and it's not difficult.

    Note: The directive below demonstrates some of the ways you can use scope inside of link and directive controllers, using controllerAs. I in-lined the template just to keep it all in one place.

    Note: Note that the directive's controller is outside the directive's closure. This style eliminates issues where the injection gets created as unreachable code after a `return`.

  ```html
  <div my-example max="77"></div>
  ```

  ```js
 //myExample.directive.js
 class ExampleController {
    constructor($scope) {
      // Injecting $scope just for comparison

      this.min = 3;

      console.log('CTRL: $scope.example.min = %s', $scope.example.min);
      console.log('CTRL: $scope.example.max = %s', $scope.example.max);
      console.log('CTRL: this.min = %s', this.min);
      console.log('CTRL: this.max = %s', this.max);
    }
  }

  class myExample {
    constructor() {
      this.restrict = 'EA';
      this.templateUrl = 'app/feature/example.directive.html';
      this.scope = {
        max: '='
      };
      this.link = this.linkFunc;
      this.controller = ExampleController;
      this.controllerAs = 'example';
      this.bindToController = true; // because the scope is isolated
    }
    linkFunc(scope, el, attr, ctrl) {
      console.log('LINK: scope.min = %s *** should be undefined', scope.min);
      console.log('LINK: scope.max = %s *** should be undefined', scope.max);
      console.log('LINK: scope.example.min = %s', scope.example.min);
      console.log('LINK: scope.example.max = %s', scope.example.max);
    }
  }
  ```

  ```html
  <!-- example.directive.html -->
  <div>hello world</div>
  <div>max={{example.max}}<input ng-model="example.max"/></div>
  <div>min={{example.min}}<input ng-model="example.min"/></div>
  ```

    Note: You can also name the controller when you inject it into the link function and access directive attributes as properties of the controller.

  ```javascript
  // Alternative to above example
  linkFunc(scope, el, attr, example) {
    console.log('LINK: scope.min = %s *** should be undefined', scope.min);
    console.log('LINK: scope.max = %s *** should be undefined', scope.max);
    console.log('LINK: example.min = %s', example.min);
    console.log('LINK: example.max = %s', example.max);
  }
  ```

###### [Style [K076](#style-k076)]

  - Use `bindToController = true` when using `controller as` syntax with a directive when you want to bind the outer scope to the directive's controller's scope.

    *Why?*: It makes it easy to bind outer scope to the directive's controller scope.

    Note: `bindToController` was introduced in Angular 1.3.0.

  ```html
  <div my-example max="77"></div>
  ```

  ```javascript
  //myExample.directive.js
   class ExampleController {
    constructor() {
      this.min = 3;
      console.log('CTRL: this.min = %s', this.min);
      console.log('CTRL: this.max = %s', this.max);
    }
  }

  class myExample {
    constructor() {
      this.restrict = 'EA';
      this.templateUrl = 'app/feature/example.directive.html';
      this.scope = {
        max: '='
      };
      this.link = this.linkFunc;
      this.controller = ExampleController;
      this.controllerAs = 'example';
      this.bindToController = true;
    }
  }
  ```

  ```html
  <!-- example.directive.html -->
  <div>hello world</div>
  <div>max={{example.max}}<input ng-model="example.max"/></div>
  <div>min={{example.min}}<input ng-model="example.min"/></div>
  ```

**[Back to top](#table-of-contents)**

### Route Resolve Promises
###### [Style [K081](#style-k081)]
  - Using route resolve may be cause of freeze interface and move some logic out of controller. Move logic out of controller multiply overhead with refactoring or editing.

  - Use route resolve for nessasery data thats very important and without this data page can't be shown or will change a lot. Forexample claims, user type and so on.

  ```javascript
  /* use it only in

  // route-config.js

  function config($routeProvider) {
    $routeProvider
      .when('/avengers', {
        templateUrl: 'avengers.html',
        controller: 'AvengersController',
        controllerAs: 'avengers',
        resolve: {
          moviesPrepService: function(userService) {
            return userService.gerClaimsForMovies();
          }
        }
      });
  }
  ```

<!-- 
  - When a controller depends on a promise to be resolved before the controller is activated, resolve those dependencies in the `$routeProvider` before the controller logic is executed. If you need to conditionally cancel a route before the controller is activated, use a route resolver.

  - Use a route resolve when you want to decide to cancel the route before ever transitioning to the View.

    *Why?*: A controller may require data before it loads. That data may come from a promise via a custom factory or [$http](https://docs.angularjs.org/api/ng/service/$http). Using a [route resolve](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider) allows the promise to resolve before the controller logic executes, so it might take action based on that data from the promise.

    *Why?*: The code executes after the route and in the controller’s activate function. The View starts to load right away. Data binding kicks in when the activate promise resolves. A “busy” animation can be shown during the view transition (via `ng-view` or `ui-view`)

    Note: The code executes before the route via a promise. Rejecting the promise cancels the route. Resolve makes the new view wait for the route to resolve. A “busy” animation can be shown before the resolve and through the view transition. If you want to get to the View faster and do not require a checkpoint to decide if you can get to the View, consider the [controller `activate` technique](#style-k080) instead.

  ```javascript
  /* avoid */

  class AvengersController {
    constructor(movieService) {
      // unresolved
      this.movies;
      // resolved asynchronously
      movieService.getMovies().then((response) => {
          this.movies = response.movies;
      });
    }
  }
  ```

  ```javascript
  /* better */

  // route-config.js

  function config($routeProvider) {
    $routeProvider
      .when('/avengers', {
        templateUrl: 'avengers.html',
        controller: 'AvengersController',
        controllerAs: 'avengers',
        resolve: {
          moviesPrepService: function(movieService) {
            return movieService.getMovies();
          }
        }
      });
  }

  // avengers.controller.js

  class AvengersController {
    constructor(moviesPrepService) {
      this.movies = moviesPrepService.movies;
    }
  }
  ```

    Note: The example below shows the route resolve points to a named function, which is easier to debug and easier to handle dependency injection.

  ```javascript
  /* even better */

  // route-config.js

  function config($routeProvider) {
    $routeProvider
      .when('/avengers', {
        templateUrl: 'avengers.html',
        controller: 'AvengersController',
        controllerAs: 'avengers',
        resolve: {
          moviesPrepService: moviesPrepService
        }
      });
  }

  function moviesPrepService(movieService) {
      return movieService.getMovies();
  }

  // avengers.controller.js

  class AvengersController {
    constructor(moviesPrepService) {
      this.movies = moviesPrepService.movies;
    }
  }
  ``` 

-->

**[Back to top](#table-of-contents)**

## Components

A Component module is the container reference for all reusable components. The entities required are decoupled from all other entities and thus can be moved into any other application with ease. As with other entites, keeping the template and controller in separate files reduces component clutter.

When creating components, a configuration object is supplied as opposed to a function used by directive modules.  

Further, using the `bindings` property is the prefered method as `scope` which is used in directives. All bindings assume isolate scope.


```javascript
/* avoid */

const compliment = {
  bindings: {
    userName: '=',
    compliment: '@',
  },
  template: '<div class=\"compliment\"><h2>Hello {{$ctrl.userName}} you look {{$ctrl.compliment}}!</h2></div>',
  controller: function () {/*controller*/}
}

angular.module('app')
  .component('compliment', compliment);

```

```javascript
/* recommended */

// import template and controller from individual component directory
import controller  from './compliment.controller'
import template    from './compliment.template.html'

const compliment = {
  bindings: {
    userName: '=',
    compliment: '@',
  },
  template,    // template and controller using ES6 shorthand
  controller   
}

angular.module('app')
  .component('compliment', compliment);

```

**[Back to top](#table-of-contents)**

## Minification and Annotation

### ng-annotate
###### [Style [K100](#style-k100)]

  - Use [ng-annotate](//github.com/olov/ng-annotate) for [Gulp](http://gulpjs.com) or [Grunt](http://gruntjs.com) and comment functions that need automated dependency injection using `'ngInject'`

    *Why?*: This safeguards your code from any dependencies that may not be using minification-safe practices.

    ```javascript
    class Avengers {
      constructor(storage, avengerService) {
        'ngInject';

        this.heroSearch = '';

        this.avengerService = avengerService;
        this.storage = storage;
      }
      storeHero() {
        let hero = this.avengerService.find(this.heroSearch);
        this.storage.save(hero.name, hero);
      }
    }
    ```

    Note: When using a route resolver you can prefix the resolver's function with `/* @ngInject */` and it will produce properly annotated code, keeping any injected dependencies minification safe.

    ```javascript
    // Using @ngInject annotations
    function config($routeProvider) {
        $routeProvider
            .when('/avengers', {
                templateUrl: 'avengers.html',
                controller: 'AvengersController',
                controllerAs: 'avengers',
                resolve: { /* @ngInject */
                    moviesPrepService: function(movieService) {
                        return movieService.getMovies();
                    }
                }
            });
    }
    ```

    > Note: Starting from Angular 1.3 you can use the [`ngApp`](https://docs.angularjs.org/api/ng/directive/ngApp) directive's `ngStrictDi` parameter to detect any potentially missing minification safe dependencies. When present the injector will be created in "strict-di" mode causing the application to fail to invoke functions which do not use explicit function annotation (these may not be minification safe). Debugging info will be logged to the console to help track down the offending code. I prefer to only use `ng-strict-di` for debugging purposes only.
    `<body ng-app="APP" ng-strict-di>`

**[Back to top](#table-of-contents)**

## Exception Handling

### decorators
###### [Style [K110](#style-k110)]

  - Use a [decorator](https://docs.angularjs.org/api/auto/service/$provide#decorator), at config time using the [`$provide`](https://docs.angularjs.org/api/auto/service/$provide) service, on the [`$exceptionHandler`](https://docs.angularjs.org/api/ng/service/$exceptionHandler) service to perform custom actions when exceptions occur.

    *Why?*: Provides a consistent way to handle uncaught Angular exceptions for development-time or run-time.

    Note: Another option is to override the service instead of using a decorator. This is a fine option, but if you want to keep the default behavior and extend it a decorator is recommended.

    ```javascript
    /* recommended */

    function exceptionConfig($provide) {
      'ngInject';
      $provide.decorator('$exceptionHandler', extendExceptionHandler);
    }


    function extendExceptionHandler($delegate, toastr) {
      'ngInject';
      return function(exception, cause) {
        $delegate(exception, cause);
        let errorData = {
          exception: exception,
          cause: cause
        };
        /**
         * Could add the error to a service's collection,
         * add errors to $rootScope, log errors to remote web server,
         * or log locally. Or throw hard. It is entirely up to you.
         * throw exception;
         */
        toastr.error(exception.msg, errorData);
      };
    }
    ```

### Exception Catchers
###### [Style [K111](#style-k111)]

  - Create a factory that exposes an interface to catch and gracefully handle exceptions.

    *Why?*: Provides a consistent way to catch exceptions that may be thrown in your code (e.g. during XHR calls or promise failures).

    Note: The exception catcher is good for catching and reacting to specific exceptions from calls that you know may throw one. For example, when making an XHR call to retrieve data from a remote web service and you want to catch any exceptions from that service and react uniquely.

    ```javascript
    /* recommended */

    class exception {
      constructor(logger) {
        'ngInject';
        this.logger = logger;
      }
      catcher(message) {
        return (reason) => {
          this.logger.error(message, reason);
        };
      }
    }
    ```

### Route Errors
###### [Style [K112](#style-k112)]

  - Handle and log all routing errors using [`$routeChangeError`](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeError).

    *Why?*: Provides a consistent way to handle all routing errors.

    *Why?*: Potentially provides a better user experience if a routing error occurs and you route them to a friendly screen with more details or recovery options.

    ```javascript
    /* recommended */
    let handlingRouteChangeError = false;

    function handleRoutingErrors() {
        /**
         * Route cancellation:
         * On routing error, go to the dashboard.
         * Provide an exit clause if it tries to do it twice.
         */
        $rootScope.$on('$routeChangeError',
            function(event, current, previous, rejection) {
                if (handlingRouteChangeError) { return; }
                handlingRouteChangeError = true;
                let destination = (current && (current.title ||
                    current.name || current.loadedTemplateUrl)) ||
                    'unknown target';
                let msg = 'Error routing to ' + destination + '. ' +
                    (rejection.msg || '');

                /**
                 * Optionally log using a custom service or $log.
                 * (Don't forget to inject custom service)
                 */
                logger.warning(msg, [current]);

                /**
                 * On routing error, go to another route/state.
                 */
                $location.path('/');

            }
        );
    }
    ```

**[Back to top](#table-of-contents)**

## Naming

### Naming Guidelines
###### [Style [K120](#style-k120)]

  - Use consistent names for all entites following a pattern that describes the entities feature then (optionally) its type. My recommended pattern is `feature.type.js`. There are 2 names for most assets:
    * the file name (`avengers.controller.js`)
    * the registered entity name with Angular (`AvengersController`)

    *Why?*: Naming conventions help provide a consistent way to find content at a glance. Consistency within the project is vital. Consistency with a team is important. Consistency across a company provides tremendous efficiency.

    *Why?*: The naming conventions should simply help you find your code faster and make it easier to understand.

### Feature File Names
###### [Style [K121](#style-k121)]

  - Use consistent names for all entities following a pattern that describes the entity's feature then (optionally) its type. My recommended pattern is `feature.type.js`.

    *Why?*: Provides a consistent way to quickly identify individual entities.

    *Why?*: Provides pattern matching for any automated tasks.

    ```javascript
    /**
     * common options
     */

    // Controllers
    avengers.js
    avengers.controller.js
    avengersController.js

    // Services/Factories
    logger.js
    logger.service.js
    loggerService.js
    ```

    ```javascript
    /**
     * recommended
     */

    // controllers
    avengers.controller.js
    avengers.controller.spec.js

    // services/factories
    logger.service.js
    logger.service.spec.js

    // constants
    constants.js

    // module definition
    avengers.module.js

    // routes
    avengers.routes.js
    avengers.routes.spec.js

    // configuration
    avengers.config.js

    // directives
    avenger-profile.directive.js
    avenger-profile.directive.spec.js
    ```

  Note: Another common convention is naming controller files without the word `controller` in the file name such as `avengers.js` instead of `avengers.controller.js`. All other conventions still hold using a suffix of the type. Controllers are the most common type of entities so this just saves typing and is still easily identifiable. I recommend you choose 1 convention and be consistent for your team. My preference is `avengers.controller.js`.

    ```javascript
    /**
     * recommended
     */
    // Controllers
    avengers.js
    avengers.spec.js
    ```

### Test File Names
###### [Style [K122](#style-k122)]

  - Name test specifications similar to the component they test with a suffix of `spec`.

    *Why?*: Provides a consistent way to quickly identify components.

    *Why?*: Provides pattern matching for [karma](http://karma-runner.github.io/) or other test runners.

    ```javascript
    /**
     * recommended
     */
    avengers.controller.spec.js
    logger.service.spec.js
    avengers.routes.spec.js
    avenger-profile.directive.spec.js
    ```

### Controller Names
###### [Style [K123](#style-k123)]

  - Use consistent names for all controllers named after their feature. Use UpperCamelCase for controllers, as they are classes.

    *Why?*: Provides a consistent way to quickly identify and reference controllers.

    *Why?*: UpperCamelCase is conventional for identifying object that can be instantiated using a constructor.

    ```javascript
    /**
     * recommended
     */

    // avengers.controller.js

    class HeroAvengersController{
      constructor() { }
    }
    ```

### Controller Name Suffix
###### [Style [K124](#style-k124)]

  - Append the controller name with the suffix `Controller`.

    *Why?*: The `Controller` suffix is more commonly used and is more explicitly descriptive.

    ```javascript
    /**
     * recommended
     */

    // avengers.controller.js

    class AvengersController{
      constructor() { }
    }
    ```

### Factory and Service Names
###### [Style [K125](#style-k125)]

  - Use consistent names for all factories and services named after their feature. Use camel-casing for services and factories. Avoid prefixing factories and services with `$`. Only suffix service and factories with `Service` when it is not clear what they are (i.e. when they are nouns).

    *Why?*: Provides a consistent way to quickly identify and reference factories.

    *Why?*: Avoids name collisions with built-in factories and services that use the `$` prefix.

    *Why?*: Clear service names such as `logger` do not require a suffix.

    *Why?*: Service names such as `avengers` are nouns and require a suffix and should be named `avengersService`.

    ```javascript
    /**
     * recommended
     */

    // logger.service.js

    class logger {
      constructor() { }
    }
    ```

    ```javascript
    /**
     * recommended
     */

    // credit.service.js

    class creditService {
      constructor() { }
    }

    // customer.service.js

    class customersService {
      constructor() { }
    }
    ```

### Directive Names
###### [Style [K126](#style-k126)]

  - Use consistent names for all directives using camel-case. Use a short prefix to describe the area that the directives belong (some example are company prefix or project prefix).

    *Why?*: Provides a consistent way to quickly identify and reference directives.

    ```javascript
    /**
     * recommended
     */

    // avenger-profile.directive.js

    // usage is <xx-avenger-profile> </xx-avenger-profile>

    class xxAvengerProfile {
      constructor() { }
    }
    ```

### Modules
###### [Style [K127](#style-k127)]

  - When there are multiple modules, the main module file is named `app.module.js` while other dependent modules are named after what they represent. For example, an admin module is named `admin.module.js`. The respective registered module names would be `app` and `admin`.

    *Why?*: Provides consistency for multiple module apps, and for expanding to large applications.

    *Why?*: Provides easy way to use task automation to load all module definitions first, then all other angular files (for bundling).

### Configuration
###### [Style [K128](#style-k128)]

  - Separate configuration for a module into its own file named after the module. A configuration file for the main `app` module is named `app.config.js` (or simply `config.js`). A configuration for a module named `admin.module.js` is named `admin.config.js`.

    *Why?*: Separates configuration from module definition, directives, and active code.

    *Why?*: Provides an identifiable place to set configuration for a module.

### Routes
###### [Style [K129](#style-k129)]

  - Separate route configuration into its own file. Examples might be `app.route.js` for the main module and `admin.route.js` for the `admin` module. Even in smaller apps I prefer this separation from the rest of the configuration.

**[Back to top](#table-of-contents)**

## Application Structure LIFT Principle
### LIFT
###### [Style [K140](#style-k140)]

  - Structure your app such that you can `L`ocate your code quickly, `I`dentify the code at a glance, keep the `F`lattest structure you can, and `T`ry to stay DRY. The structure should follow these 4 basic guidelines.

    *Why LIFT?*: Provides a consistent structure that scales well, is modular, and makes it easier to increase developer efficiency by finding code quickly. Another way to check your app structure is to ask yourself: How quickly can you open and work in all of the related files for a feature?

    When I find my structure is not feeling comfortable, I go back and revisit these LIFT guidelines

    1. `L`ocating our code is easy
    2. `I`dentify code at a glance
    3. `F`lat structure as long as we can
    4. `T`ry to stay DRY (Don’t Repeat Yourself) or T-DRY

### Locate
###### [Style [K141](#style-k141)]

  - Make locating your code intuitive, simple and fast.

    *Why?*: I find this to be super important for a project. If the team cannot find the files they need to work on quickly, they will not be able to work as efficiently as possible, and the structure needs to change. You may not know the file name or where its related files are, so putting them in the most intuitive locations and near each other saves a ton of time. A descriptive folder structure can help with this.

    ```
    /bower_components
    /client
      /app
        /avengers
        /blocks
          /exception
          /logger
        /core
        /dashboard
        /data
        /layout
        /widgets
      /content
      index.html
    .bower.json
    ```

### Identify
###### [Style [K142](#style-k142)]

  - When you look at a file you should instantly know what it contains and represents.

    *Why?*: You spend less time hunting and pecking for code, and become more efficient. If this means you want longer file names, then so be it. Be descriptive with file names and keeping the contents of the file to exactly 1 entity. Avoid files with multiple controllers, multiple services, or a mixture. There are deviations of the 1 per file rule when I have a set of very small features that are all related to each other, they are still easily identifiable.

### Flat
###### [Style [K143](#style-k143)]

  - Keep a flat folder structure as long as possible. When you get to 7+ files, begin considering separation.

    *Why?*: Nobody wants to search 7 levels of folders to find a file. Think about menus on web sites … anything deeper than 2 should take serious consideration. In a folder structure there is no hard and fast number rule, but when a folder has 7-10 files, that may be time to create subfolders. Base it on your comfort level. Use a flatter structure until there is an obvious value (to help the rest of LIFT) in creating a new folder.

### T-DRY (Try to Stick to DRY)
###### [Style [K144](#style-k144)]

  - Be DRY, but don't go nuts and sacrifice readability.

    *Why?*: Being DRY is important, but not crucial if it sacrifices the others in LIFT, which is why I call it T-DRY. I don’t want to type session-view.html for a view because, well, it’s obviously a view. If it is not obvious or by convention, then I name it.

**[Back to top](#table-of-contents)**

## Application Structure

### Overall Guidelines
###### [Style [K150](#style-k150)]

  - Have a near term view of implementation and a long term vision. In other words, start small but keep in mind on where the app is heading down the road. All of the app's code goes in a root folder named `app`. All content is 1 feature per file. Each controller, service, module, view is in its own file. All 3rd party vendor scripts are stored in another root folder and not in the `app` folder. I didn't write them and I don't want them cluttering my app (`bower_components`, `scripts`, `lib`).

    Note: Find more details and reasoning behind the structure at [this original post on application structure](http://www.johnpapa.net/angular-app-structuring-guidelines/).


### Folders-by-Feature Structure
###### [Style [K152](#style-k152)]

  - Create folders named for the feature they represent. When a folder grows to contain more than 7 files, start to consider creating a folder for them. Your threshold may be different, so adjust as needed.

    *Why?*: A developer can locate the code, identify what each file represents at a glance, the structure is flat as can be, and there is no repetitive nor redundant names.

    *Why?*: The LIFT guidelines are all covered.

    *Why?*: Helps reduce the app from becoming cluttered through organizing the content and keeping them aligned with the LIFT guidelines.

    *Why?*: When there are a lot of files (10+) locating them is easier with a consistent folder structures and more difficult in flat structures.

    ```javascript
    /**
     * recommended
     */

    app/
        app.module.js
        app.config.js
        components/
            compliment/
              compliment.component.js
              compliment.template.html
              compliment.controller.js
              compliment.spec.js
        directives/
            calendar.directive.js
            calendar.directive.html
            user-profile.directive.js
            user-profile.directive.html
        people/
            attendees.html
            attendees.controller.js
            people.routes.js
            speakers.html
            speakers.controller.js
            speaker-detail.html
            speaker-detail.controller.js
        services/
            data.service.js
            localstorage.service.js
            logger.service.js
            spinner.service.js
        sessions/
            sessions.html
            sessions.controller.js
            sessions.routes.js
            session-detail.html
            session-detail.controller.js
    ```

      ![Sample App Structure](https://raw.githubusercontent.com/rwwagner90/angular-styleguide/master/assets/modularity-2.png)

      Note: Do not structure your app using folders-by-type. This requires moving to multiple folders when working on a feature and gets unwieldy quickly as the app grows to 5, 10 or 25+ views and controllers (and other features), which makes it more difficult than folder-by-feature to locate files.

    ```javascript
    /*
    * avoid
    * Alternative folders-by-type.
    * I recommend "folders-by-feature", instead.
    */

    app/
        app.module.js
        app.config.js
        app.routes.js
        directives.js
        controllers/
            attendees.js
            session-detail.js
            sessions.js
            shell.js
            speakers.js
            speaker-detail.js
            topnav.js
        directives/
            calendar.directive.js
            calendar.directive.html
            user-profile.directive.js
            user-profile.directive.html
        services/
            dataservice.js
            localstorage.js
            logger.js
            spinner.js
        views/
            attendees.html
            session-detail.html
            sessions.html
            shell.html
            speakers.html
            speaker-detail.html
            topnav.html
    ```

**[Back to top](#table-of-contents)**

## Modularity

### Many Small, Self Contained Modules
###### [Style [K160](#style-k160)]

  - Create small modules that encapsulate one responsibility.

    *Why?*: Modular applications make it easy to plug and go as they allow the development teams to build vertical slices of the applications and roll out incrementally. This means we can plug in new features as we develop them.

### Create an App Module
###### [Style [K161](#style-k161)]

  - Create an application root module whose role is pull together all of the modules and features of your application. Name this for your application.

    *Why?*: Angular encourages modularity and separation patterns. Creating an application root module whose role is to tie your other modules together provides a very straightforward way to add or remove modules from your application.

### Keep the App Module Thin
###### [Style [K162](#style-k162)]

  - Only put logic for pulling together the app in the application module. Leave features in their own modules.

    *Why?*: Adding additional roles to the application root to get remote data, display views, or other logic not related to pulling the app together muddies the app module and make both sets of features harder to reuse or turn off.

    *Why?*: The app module becomes a manifest that describes which modules help define the application.

### Feature Areas are Modules
###### [Style [K163](#style-k163)]

  - Create modules that represent feature areas, such as layout, reusable and shared services, dashboards, and app specific features (e.g. customers, admin, sales).

    *Why?*: Self contained modules can be added to the application with little or no friction.

    *Why?*: Sprints or iterations can focus on feature areas and turn them on at the end of the sprint or iteration.

    *Why?*: Separating feature areas into modules makes it easier to test the modules in isolation and reuse code.

### Reusable Blocks are Modules
###### [Style [K164](#style-k164)]

  - Create modules that represent reusable application blocks for common services such as exception handling, logging, diagnostics, security, and local data stashing.

    *Why?*: These types of features are needed in many applications, so by keeping them separated in their own modules they can be application generic and be reused across applications.

### Module Dependencies
###### [Style [K165](#style-k165)]

  - The application root module depends on the app specific feature modules and any shared or reusable modules.

    ![Modularity and Dependencies](https://raw.githubusercontent.com/rwwagner90/angular-styleguide/master/assets/modularity-1.png)

    *Why?*: The main app module contains a quickly identifiable manifest of the application's features.

    *Why?*: Each feature area contains a manifest of what it depends on, so it can be pulled in as a dependency in other applications and still work.

    *Why?*: Intra-App features such as shared data services become easy to locate and share from within `app.core` (choose your favorite name for this module).

    Note: This is a strategy for consistency. There are many good options here. Choose one that is consistent, follows Angular's dependency rules, and is easy to maintain and scale.

    > My structures vary slightly between projects but they all follow these guidelines for structure and modularity. The implementation may vary depending on the features and the team. In other words, don't get hung up on an exact like-for-like structure but do justify your structure using consistency, maintainability, and efficiency in mind.

    > In a small app, you can also consider putting all the shared dependencies in the app module where the feature modules have no direct dependencies. This makes it easier to maintain the smaller application, but makes it harder to reuse modules outside of this application.

**[Back to top](#table-of-contents)**

## Startup Logic

### Run Blocks
###### [Style [K171](#style-k171)]

  - Any code that needs to run when an application starts should be declared in a factory, exposed via a function, and injected into the [run block](https://docs.angularjs.org/guide/module#module-loading-dependencies).

    *Why?*: Code directly in a run block can be difficult to test. Placing in a factory makes it easier to abstract and mock.

  ```javascript
  angular
      .module('app')
      .run(runBlock);

  function runBlock(authenticator, translator) {
    'ngInject';
    authenticator.initialize();
    translator.initialize();
  }
  ```

**[Back to top](#table-of-contents)**

## Angular $ Wrapper Services

### $document and $window
###### [Style [K180](#style-k180)]

  - Use [`$document`](https://docs.angularjs.org/api/ng/service/$document) and [`$window`](https://docs.angularjs.org/api/ng/service/$window) instead of `document` and `window`.

    *Why?*: These services are wrapped by Angular and more easily testable than using document and window in tests. This helps you avoid having to mock document and window yourself.

### $timeout and $interval
###### [Style [K181](#style-k181)]

  - Use [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) and [`$interval`](https://docs.angularjs.org/api/ng/service/$interval) instead of `setTimeout` and `setInterval` .

    *Why?*: These services are wrapped by Angular and more easily testable and handle Angular's digest cycle thus keeping data binding in sync.

**[Back to top](#table-of-contents)**

## Testing
Unit testing helps maintain clean code, as such I included some of my recommendations for unit testing foundations with links for more information.

### Write Tests with Stories
###### [Style [K190](#style-k190)]

  - Write a set of tests for every story. Start with an empty test and fill them in as you write the code for the story.

    *Why?*: Writing the test descriptions helps clearly define what your story will do, will not do, and how you can measure success.

    ```javascript
    it('should have Avengers controller', function() {
        // TODO
    });

    it('should find 1 Avenger when filtered by name', function() {
        // TODO
    });

    it('should have 10 Avengers', function() {
        // TODO (mock data?)
    });

    it('should return Avengers via XHR', function() {
        // TODO ($httpBackend?)
    });

    // and so on
    ```

### Testing Library
###### [Style [K191](#style-k191)]

  - Use [Jasmine](http://jasmine.github.io/) or [Mocha](http://mochajs.org) for unit testing.

    *Why?*: Both Jasmine and Mocha are widely used in the Angular community. Both are stable, well maintained, and provide robust testing features.

    Note: When using Mocha, also consider choosing an assert library such as [Chai](http://chaijs.com). I prefer Mocha.

### Test Runner
###### [Style [K192](#style-k192)]

  - Use [Karma](http://karma-runner.github.io) as a test runner.

    *Why?*: Karma is easy to configure to run once or automatically when you change your code.

    *Why?*: Karma hooks into your Continuous Integration process easily on its own or through Grunt or Gulp.

    *Why?*: Some IDE's are beginning to integrate with Karma, such as [WebStorm](http://www.jetbrains.com/webstorm/) and [Visual Studio](http://visualstudiogallery.msdn.microsoft.com/02f47876-0e7a-4f6c-93f8-1af5d5189225).

    *Why?*: Karma works well with task automation leaders such as [Grunt](http://www.gruntjs.com) (with [grunt-karma](https://github.com/karma-runner/grunt-karma)) and [Gulp](http://www.gulpjs.com). When using Gulp, use [Karma](https://github.com/karma-runner/karma) directly and not with a plugin as the API can be called directly.

    ```javascript
    /* recommended */

    // Gulp example with Karma directly
    function startTests(singleRun, done) {
        var child;
        var excludeFiles = [];
        var fork = require('child_process').fork;
        var karma = require('karma').server;
        var serverSpecs = config.serverIntegrationSpecs;

        if (args.startServers) {
            log('Starting servers');
            var savedEnv = process.env;
            savedEnv.NODE_ENV = 'dev';
            savedEnv.PORT = 8888;
            child = fork(config.nodeServer);
        } else {
            if (serverSpecs && serverSpecs.length) {
                excludeFiles = serverSpecs;
            }
        }

        karma.start({
            configFile: __dirname + '/karma.conf.js',
            exclude: excludeFiles,
            singleRun: !!singleRun
        }, karmaCompleted);

        ////////////////

        function karmaCompleted(karmaResult) {
            log('Karma completed');
            if (child) {
                log('shutting down the child process');
                child.kill();
            }
            if (karmaResult === 1) {
                done('karma: tests failed with code ' + karmaResult);
            } else {
                done();
            }
        }
    }
    ```

### Stubbing and Spying
###### [Style [K193](#style-k193)]

  - Use [Sinon](http://sinonjs.org/) for stubbing and spying.

    *Why?*: Sinon works well with both Jasmine and Mocha and extends the stubbing and spying features they offer.

    *Why?*: Sinon makes it easier to toggle between Jasmine and Mocha, if you want to try both.

    *Why?*: Sinon has descriptive messages when tests fail the assertions.

### Headless Browser
###### [Style [K194](#style-k194)]

  - Use [PhantomJS](http://phantomjs.org/) to run your tests on a server.

    *Why?*: PhantomJS is a headless browser that helps run your tests without needing a "visual" browser. So you do not have to install Chrome, Safari, IE, or other browsers on your server.

    Note: You should still test on all browsers in your environment, as appropriate for your target audience.

### Code Analysis
###### [Style [K195](#style-k195)]

  - Run ESLint on your tests.

    *Why?*: Tests are code. ESLint can help identify code quality issues that may cause the test to work improperly.

### Organizing Tests
###### [Style [K197](#style-k197)]

  - Place unit test files (specs) side-by-side with your client code. Place specs that cover server integration or test multiple components in a separate `tests` folder.

    *Why?*: Unit tests have a direct correlation to a specific component and file in source code.

    *Why?*: It is easier to keep them up to date since they are always in sight. When coding whether you do TDD or test during development or test after development, the specs are side-by-side and never out of sight nor mind, and thus more likely to be maintained which also helps maintain code coverage.

    *Why?*: When you update source code it is easier to go update the tests at the same time.

    *Why?*: Placing them side-by-side makes it easy to find them and easy to move them with the source code if you move the source.

    *Why?*: Having the spec nearby makes it easier for the source code reader to learn how the component is supposed to be used and to discover its known limitations.

    *Why?*: Separating specs so they are not in a distributed build is easy with grunt or gulp.

    ```
    /src/client/app/customers/customer-detail.controller.js
                             /customer-detail.controller.spec.js
                             /customers.controller.js
                             /customers.controller.spec.js
                             /customers.module.js
                             /customers.route.js
                             /customers.route.spec.js
    ```

**[Back to top](#table-of-contents)**

## Animations

### Usage
###### [Style [K210](#style-k210)]

  - Use subtle [animations with Angular](https://docs.angularjs.org/guide/animations) to transition between states for views and primary visual elements. Include the [ngAnimate module](https://docs.angularjs.org/api/ngAnimate). The 3 keys are subtle, smooth, seamless.

    *Why?*: Subtle animations can improve User Experience when used appropriately.

    *Why?*: Subtle animations can improve perceived performance as views transition.

### Sub Second
###### [Style [K211](#style-k211)]

  - Use short durations for animations. I generally start with 300ms and adjust until appropriate.

    *Why?*: Long animations can have the reverse affect on User Experience and perceived performance by giving the appearance of a slow application.

### animate.css
###### [Style [K212](#style-k212)]

  - Use [animate.css](http://daneden.github.io/animate.css/) for conventional animations.

    *Why?*: The animations that animate.css provides are fast, smooth, and easy to add to your application.

    *Why?*: Provides consistency in your animations.

    *Why?*: animate.css is widely used and tested.

    Note: See this [great post by Matias Niemelä on Angular animations](http://www.yearofmoo.com/2013/08/remastered-animation-in-angularjs-1-2.html)

**[Back to top](#table-of-contents)**

## Comments

### jsDoc
###### [Style [K220](#style-k220)]

  - If planning to produce documentation, use [`jsDoc`](http://usejsdoc.org/) syntax to document function names, description, params and returns. Use `@namespace` and `@memberOf` to match your app structure.

    *Why?*: You can generate (and regenerate) documentation from your code, instead of writing it from scratch.

    *Why?*: Provides consistency using a common industry tool.

    ```javascript
    /**
     * Logger Factory
     * @namespace Factories
     */
    (function() {
      angular
          .module('app')
          .factory('logger', logger);

      /**
       * @namespace Logger
       * @desc Application wide logger
       * @memberOf Factories
       */
      function logger($log) {
          var service = {
             logError: logError
          };
          return service;

          ////////////

          /**
           * @name logError
           * @desc Logs errors
           * @param {String} msg Message to log
           * @returns {String}
           * @memberOf Factories.Logger
           */
          function logError(msg) {
              var loggedMsg = 'Error: ' + msg;
              $log.error(loggedMsg);
              return loggedMsg;
          };
      }
    })();
    ```

**[Back to top](#table-of-contents)**

## ESLint

### Use an Options File
###### [Style [K230](#style-k230)]

  - Use ESLint for linting your JavaScript and be sure to customize the .eslintrc file and include in source control. See the [ESLint docs](http://eslint.org/docs/rules/) for details on the options.

    *Why?*: Provides a first alert prior to committing any code to source control.

    *Why?*: Provides consistency across your team.

    *Why?*: ESLint supports ES6.

    ```javascript
    {
      "globals": {
        "_": false,
        "angular": false,
        "console": false,
        "inject": false,
        "module": false,
        "window": false
      },
      "parser": "babel-eslint",
      "rules": {
        "brace-style": [2, "stroustrup", {"allowSingleLine": false}],
        "camelcase": 1,
        "comma-dangle": [1, "never"],
        "curly": 1,
        "dot-notation": 1,
        "eqeqeq": 1,
        "indent": [1, 2],
        "lines-around-comment": [2, {"allowBlockStart": true, "beforeBlockComment": true, "beforeLineComment": true}],
        "new-parens": 1,
        "no-bitwise": 1,
        "no-cond-assign": 1,
        "no-debugger": 1,
        "no-dupe-args": 1,
        "no-dupe-keys": 1,
        "no-empty": 1,
        "no-invalid-regexp": 1,
        "no-invalid-this": 1,
        "no-mixed-spaces-and-tabs": [1, "smart-tabs"],
        "no-multiple-empty-lines": [1, {"max": 2}],
        "no-undef": 1,
        "no-underscore-dangle": 1,
        "no-unreachable": 1,
        "no-unused-vars": 1,
        "one-var": [1, "never"],
        "quote-props": [1, "as-needed"],
        "semi": [1, "always"],
        "keyword-spacing": 1,
        "space-unary-ops": [1, {"words": true, "nonwords": false}],
        "strict": [1, "function"],
        "vars-on-top": 1,
        "wrap-iife": [1, "outside"],
        "yoda": [1, "never"],
    
        //ES6 Stuff
        "arrow-parens": 1,
        "arrow-spacing": 1,
        "constructor-super": 1,
        "no-class-assign": 1,
        "no-const-assign": 1,
        "no-dupe-class-members": 1,
        "no-this-before-super": 1,
        "no-var": 1,
        "object-shorthand": 1,
        "prefer-arrow-callback": 1,
        "prefer-const": 1
      }
    }

    ```
**[Back to top](#table-of-contents)**

## Constants

### Vendor Globals
###### [Style [K240](#style-k240)]

  - Create an Angular Constant for vendor libraries' global variables.

    *Why?*: Provides a way to inject vendor libraries that otherwise are globals. This improves code testability by allowing you to more easily know what the dependencies of your constants are (avoids leaky abstractions). It also allows you to mock these dependencies, where it makes sense.

    ```javascript
    // constants.js

    /* global toastr:false, moment:false */
    (function() {
        'use strict';

        angular
            .module('app.core')
            .constant('toastr', toastr)
            .constant('moment', moment);
    })();
    ```

###### [Style [K241](#style-k241)]

  - Use constants for values that do not change and do not come from another service. When constants are used only for a module that may be reused in multiple applications, place constants in a file per module named after the module. Until this is required, keep constants in the main module in a `constants.js` file.

    *Why?*: A value that may change, even infrequently, should be retrieved from a service so you do not have to change the source code. For example, a url for a data service could be placed in a constants but a better place would be to load it from a web service.

    *Why?*: Constants can be injected into any angular component, including providers.

    *Why?*: When an application is separated into modules that may be reused in other applications, each stand-alone module should be able to operate on its own including any dependent constants.

    ```javascript
    // Constants used by the entire app
    angular
        .module('app.core')
        .constant('moment', moment);

    // Constants used only by the sales module
    angular
        .module('app.sales')
        .constant('events', {
            ORDER_CREATED: 'event_order_created',
            INVENTORY_DEPLETED: 'event_inventory_depleted'
        });
    ```

**[Back to top](#table-of-contents)**


## Yeoman Generator
###### [Style [K260](#style-k260)]

You can use the [generator-gulp-angular](https://github.com/Swiip/generator-gulp-angular) to create an app that serves as a starting point for Angular that follows this style guide.

1. Install generator-gulp-angular

  ```
  npm install -g generator-gulp-angular
  ```

2. Create a new folder and change directory to it

  ```
  mkdir myapp
  cd myapp
  ```

3. Run the generator

  ```
  yo gulp-angular
  ```

**[Back to top](#table-of-contents)**

## Routing
Client-side routing is important for creating a navigation flow between views and composing views that are made of many smaller templates and directives.

###### [Style [K270](#style-k270)]

  - Use the [AngularUI Router](http://angular-ui.github.io/ui-router/) for client-side routing.

    *Why?*: UI Router offers all the features of the Angular router plus a few additional ones including nested routes and states.

    *Why?*: The syntax is quite similar to the Angular router and is easy to migrate to UI Router.

  - Note: You can use a provider such as the `routerHelperProvider` shown below to help configure states across files, during the run phase.

    ```javascript
    // customers.routes.js
    angular
        .module('app.customers')
        .run(appRun);


    function appRun(routerHelper) {
      'ngInject';
      routerHelper.configureStates(getStates());
    }

    function getStates() {
        return [
            {
                state: 'customer',
                config: {
                    abstract: true,
                    template: '<ui-view class="shuffle-animation"/>',
                    url: '/customer'
                }
            }
        ];
    }
    ```

    ```javascript
    // routerHelperProvider.js
    angular
        .module('blocks.router')
        .provider('routerHelper', routerHelperProvider);


    function routerHelperProvider($locationProvider, $stateProvider, $urlRouterProvider) {
      'ngInject';
      /* jshint validthis:true */
      this.$get = RouterHelper;

      $locationProvider.html5Mode(true);
      function RouterHelper($state) {
        'ngInject';
        var hasOtherwise = false;
        var service = {
          configureStates: configureStates,
          getStates: getStates
        };

        return service;

        ///////////////

        function configureStates(states, otherwisePath) {
          states.forEach((state) => {
            $stateProvider.state(state.state, state.config);
          });
          if (otherwisePath && !hasOtherwise) {
            hasOtherwise = true;
            $urlRouterProvider.otherwise(otherwisePath);
          }
        }

        function getStates() { return $state.get(); }
      }
    }
    ```

###### [Style [K271](#style-k271)]

  - Define routes for views in the module where they exist. Each module should contain the routes for the views in the module.

    *Why?*: Each module should be able to stand on its own.

    *Why?*: When removing a module or adding a module, the app will only contain routes that point to existing views.

    *Why?*: This makes it easy to enable or disable portions of an application without concern over orphaned routes.

**[Back to top](#table-of-contents)**

## Task Automation
Use [Gulp](http://gulpjs.com) or [Grunt](http://gruntjs.com) for creating automated tasks.  Gulp leans to code over configuration while Grunt leans to configuration over code. I personally prefer Gulp as I feel it is easier to read and write, but both are excellent.

> Learn more about gulp and patterns for task automation in my [Gulp Pluralsight course](http://jpapa.me/gulpps)

###### [Style [K400](#style-k400)]

  - Use task automation to list module definition files `*.module.js` before all other application JavaScript files.

    *Why?*: Angular needs the module definitions to be registered before they are used.

    *Why?*: Naming modules with a specific pattern such as `*.module.js` makes it easy to grab them with a glob and list them first.

    ```javascript
    var clientApp = './src/client/app/';

    // Always grab module files first
    var files = [
      clientApp + '**/*.module.js',
      clientApp + '**/*.js'
    ];
    ```

**[Back to top](#table-of-contents)**

## Filters

###### [Style [K420](#style-k420)]

  - Avoid using filters for scanning all properties of a complex object graph. Use filters for select properties.

    *Why?*: Filters can easily be abused and negatively affect performance if not used wisely, for example when a filter hits a large and deep object graph.

**[Back to top](#table-of-contents)**

## Angular docs
For anything else, API reference, check the [Angular documentation](//docs.angularjs.org/api).

## Contributing

Open an issue first to discuss potential changes/additions. If you have questions with the guide, feel free to leave them as issues in the repository. If you find a typo, create a pull request. The idea is to keep the content up to date and use github’s native feature to help tell the story with issues and PR’s, which are all searchable via google. Why? Because odds are if you have a question, someone else does too! You can learn more here at about how to contribute.

*By contributing to this repository you are agreeing to make your content available subject to the license of this repository.*

### Process
    1. Discuss the changes in a GitHub issue.
    2. Open a Pull Request, reference the issue, and explain the change and why it adds value.
    3. The Pull Request will be evaluated and either merged or declined.

## License

_tldr; Use this guide. Attributions are appreciated._

### Copyright

Copyright (c) 2014-2016 [John Papa](http://johnpapa.net)
and 2015 Robert Wagner / Mike Erickson

### (The MIT License)
Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

**[Back to top](#table-of-contents)**
