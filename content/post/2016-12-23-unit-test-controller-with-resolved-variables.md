---
title: "Unit test controllers with resolved variables"
description: "Step by step guide of how to unit test controllers with resolved variable from routing"
tags: [ "jasmine", "unit test", "angular 1.4", "controllers", "resolve"]
lastmod: 2016-12-23
date: "2016-12-23"
categories:
  - "Development"
  - "VIM"
slug: "unit-test-controllers-with-resolved-variable-from-routing"
image: "/images/cockpit.jpg"
---

### Scenario

Today's post is about testing a controller that uses variables from a resolve function in the routing. 

This is a real life example used in translations, say we want to handle different languages, we need to load the available languages and the selected one from a service.

```javascript
$stateProvider.state('locales', {
  url: '/locales',
  templateUrl: 'views/locales.html',
  controller: 'LocaleCtrl',
  data: {
    secure: true,
    backState: 'settings'
  },
  resolve: {
    availableLocales: function (i18n) {
      return i18n.getAvailableLocales();
    },
    selectedLocale: function (i18n) {
      return i18n.getLocale();
    }
  }
})
```

### Solution

```javascript
(function () {
  'use strict';

  describe('Controller: LocaleCtrl', function () {
    var LocaleCtrl,
      $rootScope,
      authService,
      i18n,
      availableLocales,
      selectedLocale,
      scope;

    beforeEach(function () {
      module('fuji.app.controllers');

      inject(function ($controller, _$rootScope_, _authService_, _i18n_) {
        $rootScope = _$rootScope_;
        authService = _authService_;
        i18n = _i18n_;
        availableLocales = ['en', 'fr'];
        selectedLocale = 'fr';

        authService.getEmployeeId = function () {
          return 2;
        };

        scope = $rootScope.$new();
        LocaleCtrl = $controller('LocaleCtrl', {
          $scope: scope,
          availableLocales: availableLocales,
          selectedLocale: selectedLocale
        });

        $rootScope.$digest();
      });
    });

    // List of assertions
    //.
    //.
    //.
    
  });
})();
```

### Analysis

The solution is to inject `availableLocales` and `selectedLocale` into the scope.

