# AngularJS styleguide

*Opinionated AngularJS styleguide for teams by [@toddmotto](//twitter.com/toddmotto)*

From my experience with [Angular](//angularjs.org), [several talks](https://speakerdeck.com/toddmotto) and working in teams, here's my opinionated styleguide for syntax, building and structuring Angular applications.

> See the [original article](http://toddmotto.com/opinionated-angular-js-styleguide-for-teams)

## Table of Contents
  
  1. [Folder structure](#folder-structure)
  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [Services](#services)
  1. [Directives](#directives)
  1. [Filters](#filters)
  1. [Routing resolves](#routing-resolves)
  1. [Publish and subscribe events](#publish-and-subscribe-events)
  1. [Angular wrapper references](#angular-wrapper-references)
  1. [Comment standards](#comment-standards)
  1. [Minification and annotation](#minification-and-annotation)

## Folder structure

```
app/
|_module1/
  |_configs/
    |_module1RoutesConfig.js
  |_controllers/
    |_module1BarController.js
    |_module1FooController.js
  |_directives/
    |_module1Foo.js
  |_factories/
    |_module1FooFactory.js
  |_runs/
    |_module1Baz.js
  |_app.js
|_module2/
  |_...
  |_app.js
|_...
|_app.js
```

- Put all angular blocks (controllers, directives, services, etc.) in their own files and `require()` them from their module's app.js
- Prefix all files with the module name to avoid name collisions when multiple modules are running within the same app
- Suffix config, controller, factory, provider, run, and service file names with the type of angular construct (eg. `Factory`, `Service`, etc.)

```

## Modules

  - **Definitions**: Declare modules without a variable using the setter and getter syntax

    ```javascript
    // bad
    var app = angular.module('app', []);
    app.controller();
    app.factory();

    // good
    angular
      .module('app', [])
      .controller()
      .factory();
    ```

  - Note: Using `angular.module('app', []);` sets a module, whereas `angular.module('app');` gets the module. Only set once and get for all other instances.

  - **Methods**: Pass requires into module methods rather than assign as an inline callback

    ```javascript
    // bad
    angular
      .module('app', [])
      .controller('MainCtrl', function () {
        ...
      })
      .service('SomeService', function () {
        ...
      });

    // good
    angular
      .module('app', [])
      .controller('MainCtrl', require('./controllers/foo'))
      .service('SomeService', require('./services/foo'));
    ```

  - This aids with readability and reduces the volume of code "wrapped" inside the Angular framework

**[Back to top](#table-of-contents)**

## Controllers (*angular 1.2+ only*)

  - **controllerAs syntax**: Controllers are classes, so use the `controllerAs` syntax at all times

    ```html
    <!-- bad -->
    <div ng-controller="MainCtrl">
      {{ someObject }}
    </div>

    <!-- good -->
    <div ng-controller="MainCtrl as main">
      {{ main.someObject }}
    </div>
    ```

  - In the DOM we get a variable per controller, which aids nested controller methods avoiding any `$parent` calls

  - The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

    ```javascript
    // bad
    function MainCtrl ($scope) {
      $scope.someObject = {};
      $scope.doSomething = function () {

      };
    }

    // good
    function MainCtrl () {
      this.someObject = {};
      this.doSomething = function () {

      };
    }
    ```

  - Only use `$scope` in `controllerAs` when necessary, for example publishing and subscribing events using `$emit`, `$broadcast`, `$on` or using `$watch`, try to limit the use of these, however and treat `$scope` uses as special

**[Back to top](#table-of-contents)**

## Services

  - Prefer services over factories and providers.

    ```javascript
    function SomeService () {
      this.someMethod = function () {

      };
    }
    ```

**[Back to top](#table-of-contents)**

## Directives

  - **Declaration restrictions**: Only use `custom element` and `custom attribute` methods for declaring your Directives (`restrict: 'EA'`) depending on the Directive's role

    ```html
    <!-- bad -->
    <div class="my-directive"></div>

    <!-- good -->
    <my-directive></my-directive>
    ```

  - Comment and class name declarations are confusing and should be avoided.

  - **Templating**: Prefer external templates over inlined HTML

    ```javascript
    // bad
    function someDirective () {
      return {
        template: '<div class="some-directive">' +
          '<h1>My directive</h1>' +
        '</div>'
      };
    }

    // good
    function someDirective () {
      return {
        templateUrl: './templates/some-directive.html'
      };
    }
    ```

  - **DOM manipulation**: Only takes place inside Directives, never a controller/service

    ```javascript
    // bad
    function UploadCtrl () {
      $('.dragzone').on('dragend', function () {
        // handle drop functionality
      });
    }
    angular
      .module('app')
      .controller('UploadCtrl', UploadCtrl);

    // good
    function dragUpload () {
      return {
        restrict: 'EA',
        link: function (scope, element, attrs) {
          element.on('dragend', function () {
            // handle drop functionality
          });
        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

  - **Naming conventions**: Never `ng-*` prefix custom directives, they might conflict future native directives.

    ```javascript
    // bad
    // <div ng-upload></div>
    function ngUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('ngUpload', ngUpload);

    // good
    // <div drag-upload></div>
    function dragUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

  - Directives and Filters are the _only_ providers that we have the first letter as lowercase, this is due to strict naming conventions in Directives due to the way Angular translates `camelCase` to hyphenated, so `dragUpload` will become `<div drag-upload></div>` when used on an element.

  - **controllerAs**: Use the `controllerAs` syntax inside Directives also (*angular 1.2+ only*)

    ```javascript
    // bad
    function dragUpload () {
      return {
        controller: function ($scope) {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);

    // good
    function dragUpload () {
      return {
        controllerAs: 'dragUpload',
        controller: function () {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

**[Back to top](#table-of-contents)**

## Filters

  - **Global filters**: Create global filters only using `angular.filter()` never use local filters inside Controllers/Services

    ```javascript
    // bad
    function SomeCtrl () {
      this.startsWithLetterA = function (items) {
        return items.filter(function (item) {
          return /$a/i.test(item.name);
        });
      };
    }
    angular
      .module('app')
      .controller('SomeCtrl', SomeCtrl);

    // good
    function startsWithLetterA () {
      return function (items) {
        return items.filter(function (item) {
          return /$a/i.test(item.name);
        });
      };
    }
    angular
      .module('app')
      .filter('startsWithLetterA', startsWithLetterA);
    ```

  - This enhances testing and reusability

**[Back to top](#table-of-contents)**

## Routing resolves

  - **Promises**: Resolve dependencies of a Controller in the `$routeProvider` not the Controller itself

    ```javascript
    // bad
    function MainCtrl (SomeService) {
      var _this = this;
      // unresolved
      _this.something;
      // resolved asynchronously
      SomeService.doSomething().then(function (response) {
        _this.something = response;
      });
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

    // good
    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        resolve: {
          // resolve here
        }
      });
    }
    angular
      .module('app')
      .config(config);
    ```

  - **Controller.resolve property**: Never bind logic to the router itself, reference a `resolve` property for each Controller to couple the logic

    ```javascript
    // bad
    function MainCtrl (SomeService) {
      this.something = SomeService.something;
    }

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controllerAs: 'main',
        controller: 'MainCtrl'
        resolve: {
          doSomething: function () {
            return SomeService.doSomething();
          }
        }
      });
    }

    // good
    function MainCtrl (SomeService) {
      this.something = SomeService.something;
    }

    MainCtrl.resolve = {
      doSomething: function () {
        return SomeService.doSomething();
      }
    };

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controllerAs: 'main',
        controller: 'MainCtrl'
        resolve: MainCtrl.resolve
      });
    }
    ```

  - This keeps resolve dependencies inside the same file as the Controller and the router free from logic

**[Back to top](#table-of-contents)**

## Publish and subscribe events

  - **$scope**: event emitters should be avoided when possible (prefer `$watch`es and 2-way binding instead). When necessary, use the `$emit` and `$broadcast` methods to trigger events to direct relationship scopes only.

    ```javascript
    // up the $scope
    $scope.$emit('customEvent', data);

    // down the $scope
    $scope.$broadcast('customEvent', data);
    ```

  - **$rootScope**: use only `$emit` as an application-wide event bus and remember to unbind listeners

    ```javascript
    // all $rootScope.$on listeners
    $rootScope.$emit('customEvent', data);
    ```

  - Hint: `$rootScope.$on` listeners are different than `$scope.$on` listeners and will always persist so they need destroying when the relevant `$scope` fires the `$destroy` event

    ```javascript
    // call the closure
    var unbind = $rootScope.$on('customEvent'[, callback]);
    $scope.$on('$destroy', unbind);
    ```

**[Back to top](#table-of-contents)**

## Angular wrapper references

  - **$document and $window**: Use `$document` and `$window` at all times to aid testing and Angular references

    ```javascript
    // bad
    function dragUpload () {
      return {
        link: function (scope, element, attrs) {
          document.addEventListener('click', function () {

          });
        }
      };
    }

    // good
    function dragUpload () {
      return {
        link: function (scope, element, attrs, $document) {
          $document.addEventListener('click', function () {

          });
        }
      };
    }
    ```

  - **$timeout and $interval**: Use `$timeout` and `$interval` (*angular 1.2+ only*) over their native counterparts to automatically trigger digests when the timer fires

    ```javascript
    // bad
    function dragUpload () {
      return {
        link: function (scope, element, attrs) {
          setTimeout(function () {
            scope.$apply(function(){
              ...
            });
          }, 1000);
        }
      };
    }

    // good
    function dragUpload () {
      return {
        link: function (scope, element, attrs, $timeout) {
          $timeout(function () {
            ...
          }, 1000);
        }
      };
    }
    ```

**[Back to top](#table-of-contents)**

## Comment standards

  - **jsDoc**: Use jsDoc syntax to document function names, description, params and returns

    ```javascript
    /**
     * @name SomeService
     * @desc Main application Controller
     */
    function SomeService (SomeService) {

      /**
       * @name doSomething
       * @desc Does something awesome
       * @param {Number} x First number to do something with
       * @param {Number} y Second number to do something with
       * @returns {Number}
       */
      this.doSomething = function (x, y) {
        return x * y;
      };

    }
    angular
      .module('app')
      .service('SomeService', SomeService);
    ```

**[Back to top](#table-of-contents)**

## Minification and annotation

  - **ng-annotate**: Use [ng-annotate](//github.com/olov/ng-annotate) and comment functions that need automated dependency injection using `/* @ngInject */`

    ```javascript
    // controller.js
    module.exports = /* @ngInject */ function (SomeService) {
      this.doSomething = SomeService.doSomething;
    }

    // app.js
    angular
      .module('app')
      .controller('MainCtrl', require('./controllers/controller'));
    ```

  - Which ng-annotate compiles to the following annotated code

    ```javascript
    // controller.js
    module.exports = /* @ngInject */ ['SomeService', function (SomeService) {
      this.doSomething = SomeService.doSomething;
    }];

    // app.js
    angular
      .module('app')
      .controller('MainCtrl', require('./controllers/controller'));
    ```

**[Back to top](#table-of-contents)**

## Angular docs
For anything else, API reference, check the [Angular documentation](//docs.angularjs.org/api).

## Contributing

Open an issue first to discuss potential changes/additions.

## License

#### (The MIT License)

Copyright (c) 2014 Todd Motto

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
