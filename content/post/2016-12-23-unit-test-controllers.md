---
title: "Unit test controllers"
description: "Step by step guide of how to unit test controllers"
tags: [ "jasmine", "unit test", "angular 1.4", "controllers"]
lastmod: 2016-12-23
date: "2016-12-23"
categories:
  - "Development"
  - "VIM"
slug: "unit-test-controllers"
image: "/images/controller.jpg"
---

### Scenario
We want to unit test a very basic controller with Jasmine.

```javascript
(function () {
  'use strict';
  /**
   * @ngdoc controller
   * @name fuji.app.controllers:AboutCtrl
   * @description
   */
  angular
    .module('fuji.app.controllers')
    .controller('AboutCtrl', AboutCtrl);

  function AboutCtrl ($scope, configuration, appVersion) {
    $scope.viewClassName = 'wps-view-about';
    $scope.accountCode = configuration.getAccountServiceConfig();
    $scope.buildVersion = appVersion.buildVersion();
  }
})();
```

This controller is doing one thing.

* Initialize some scope variables `$scope.viewClassName`, `$scope.accountCode` and `$scope.buildVersion`

### Solution
We have 3 test we need to do:

* Check the controller is defined
* Check the scope variable are properly initialized initialized

```javascript
  (function () {
    'use strict';
  
    describe('Controller: AboutCtrl', function () {
      var AboutCtrl,
        configuration,
        appVersion,
        scope;
  
      // Initialize the controller and a mock scope
      beforeEach(function () {
        module('app.controllers');
  
        inject(function ($controller, $rootScope, _configuration_, _appVersion_) {
          configuration = _configuration_;
          appVersion = _appVersion_;
  
          appVersion.buildVersion = function () {
            return '9.9.9b9';
          };
  
          configuration.getAccountServiceConfig = function () {
            return 'hopla'
          };
  
          scope = $rootScope.$new();
  
          AboutCtrl = $controller('AboutCtrl', {
            $scope: scope
          });
  
          $rootScope.$digest();
  
        });
      });
  
      it('is defined', function () {
        expect(AboutCtrl).toBeDefined();
      });
  
      it('is initialised', function () {
        expect(scope.buildVersion).toEqual('9.9.9b9');
        expect(scope.accountCode).toEqual('hopla');
        expect(scope.viewClassName).toEqual('wps-view-about');
      });
    });
  })();
```

### Analysis

##### beforeEach

* First we call a `beforeEach` to set the scene for all or assertion.
* The `beforeEach` begins with injecting our controllers `module('app.controllers')`, without this we can't access the controller we want to test.
* Then we need to inject the services used by the controller.
* We fake the different services called by our scope variables `appVersion.buildVersion` and `configuration.getAccountServiceConfig`.
* We create a `scope` from the injected `$rootScope`.
* We create the controller and use the scope we just created.
* Finally we call `$rootScope.$digest()` to make sure all are changes on the scope have been taken into account.

##### assertion

* Testing the controller is defined is straight forward.
* Testing the scope is initialised is straight forward after the preliminary work we did in the `beforeEach`.


