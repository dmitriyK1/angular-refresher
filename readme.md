# AngularJS Refresher

This is a quick refresher of AngularJS concepts compiled from various articles online.

**Table of Contents**

![](modularity-1.png)

**Use revealing module pattern to expose interface of services:**

Why?: Placing the callable members at the top makes it easy to read and helps
you instantly identify which members of the service can be called and must be
unit tested (and/or mocked).
Why?: This is especially helpful when the file gets longer as it helps avoid
the need to scroll to see what is exposed.

```javascript
.factory('dataService', dataService);

function dataService() {
    var service = {
        getData: getData,
        pushDta: pushData
    };

    return service;

    // keep implementation at bottom via function declarations

    function getData() {}
    function pushData() {}
}
```

================================================================================

**Use function declarations instead of anonymous functions to keep a structure of code more flat**

```javascript
angular
  .module('app', [])
  .controller('MainCtrl', MainCtrl)
  .service('SomeService', SomeService);

function MainCtrl () { }
function SomeService () { }

MainCtrl.$inject = [];
SomeService.$inject = [];
```

================================================================================

**Send all data in directive isolate scope via one object if possible:**

```html
<input ng-model="user.name">
<input ng-model="user.role">

<user-card user="user"></user-card>
```

================================================================================

**Use one-time binding to show rarely updated data to decrease performance overhead:**

{{:: page.title  }}

================================================================================

**Use bindToController option in directive definition object in order to set
properties of directive isolate scope on controller instance instead of $scope:**


```javascript
.directive('tabs', tabsDirective);

function tabsDirective() {
    var directiveDefinitionObject = {
        bindToController: true,
        link: link
    };

    function link() {}

    return directiveDefinitionObject;
}
```

================================================================================

**Use Controller suffix instead of Ctrl for better readability:**

.controller('MainController', MainController);

function MainController() {}

================================================================================

**Reuse logic via services instead of controllers reuse**

.factory('dataService', dataService);

function FirstController(dataService) { }

function SecondController(dataService) { }

================================================================================

**Return promise from a services:**

.factory('dataService', function($http) {
    var service = {
        getData: getData
    };

    function getData() {
        return $http
                .get('data.json')
                .then(function(data) {
                    return data * 2;
                });
    }
})

.controller('MainController', function(dataService) {
    dataService
        .getData()
        .then(function(data) {
            this.data = data;
        });
});

================================================================================

**Avoid using ng-controller ( use controllers via directives instead )**

.directive('usersList', userListDirective);

function usersListDirective() {
    var ddo = {
        controller: UsersListController,
        controllerAs: 'usersListController',
        bindToController: true
    };

    return ddo;
}

================================================================================

**Avoid using ng-controller in views for routes ( use controllers in config for
routes instead ):**

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
<!-- avengers.html -->
<div ng-controller="AvengersController as vm">
</div>


/* recommended */

// route-config.js
angular
    .module('app')
    .config(config);

function config($routeProvider) {
    $routeProvider
        .when('/avengers', {
            templateUrl: 'avengers.html',
            controller: 'Avengers',
            controllerAs: 'vm'
        });
}
<!-- avengers.html -->
<div>
</div>

================================================================================

**Controllers are classes so define controllers on prototypes in order to extend
controllers via inheritance:**

.controller('BaseController', BaseController);

function BaseController() { }

BaseController.prototype.log = function() {};

function AppController() {}

AppController.prototype = Object.create( BaseController.prototype );

================================================================================

**Use consistent file names:**

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

================================================================================

**Avoid using ng- prefix for directives and $ for custom component names because they are reserved**

================================================================================

**Keep project directory structure as flat as possible; use folders-by-feature structure**

/**
 * recommended
 */

app/
    app.module.js
    app.config.js
    components/
        calendar.directive.js
        calendar.directive.html
        user-profile.directive.js
        user-profile.directive.html
    layout/
        shell.html
        shell.controller.js
        topnav.html
        topnav.controller.js
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

================================================================================

**Config block**

Inject code into module configuration that must be configured before running
the angular app. Ideal candidates include providers and constants.
Why?: This makes it easier to have less places for configuration.

angular
    .module('app')
    .config(configure);

configure.$inject =
    ['routerHelperProvider', 'exceptionHandlerProvider', 'toastr'];

function configure (routerHelperProvider, exceptionHandlerProvider, toastr) {
    exceptionHandlerProvider.configure(config.appErrorPrefix);
    configureStateHelper();

    toastr.options.timeOut = 4000;
    toastr.options.positionClass = 'toast-bottom-right';

    ////////////////

    function configureStateHelper() {
        routerHelperProvider.configure({
            docTitle: 'NG-Modular: '
        });
    }
}

================================================================================

**Run Blocks**

Any code that needs to run when an application starts should be declared in
a factory, exposed via a function, and injected into the run block.

Why?: Code directly in a run block can be difficult to test. Placing in
a factory makes it easier to abstract and mock.

angular
    .module('app')
    .run(runBlock);

runBlock.$inject = ['authenticator', 'translator'];

function runBlock(authenticator, translator) {
    authenticator.initialize();
    translator.initialize();
}

================================================================================

**Use Angular $ Wrapper Services instead of native implementations because they
trigger $digest cycle as needed thus keeping data binding in sync**

$timeout
$interval
$http
$window
$document

================================================================================

**Use constants to expose vendor libraries**

Why?: Provides a way to inject vendor libraries that otherwise are globals.
This improves code testability by allowing you to more easily know what the
dependencies of your components are (avoids leaky abstractions). It also allows
you to mock these dependencies, where it makes sense.

// constants.js

/* global toastr:false, moment:false */
(function() {
    'use strict';

    angular
        .module('app.core')
        .constant('toastr', toastr)
        .constant('moment', moment);
})();

When constants are used only for a module that may be reused in multiple
applications, place constants in a file per module named after the module.
Until this is required, keep constants in the main module in a constants.js
file.

Why?: Constants can be injected into any angular component, including providers.

================================================================================

**Unbind event listeners on $scope.$destroy**
