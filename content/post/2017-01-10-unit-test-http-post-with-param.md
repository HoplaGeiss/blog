---
title: "Unit test $http.post with parameters"
description: "Step by step guide of how to unit test $http.post with parameters"
tags: [ "jasmine", "unit test", "angular 1.4", "$http.get",]
lastmod: 2017-01-10
date: "2017-01-10"
categories:
  - "Development"
  - "VIM"
slug: "unit-test-http-post-with-param"
image: "/images/donald.jpg"
---

### Scenario

In this example I will demonstrate how to unit `$http.post` when used with parameters. 

This is a real life example of an application, calling the backend with `$http.post` to register a new device.
The difficulty here is, that the post includes a parameter, we need to find a way to access it.

```javascript
registerDevice: function (token) {
    return $http.post(configuration.getFrontEndUrl('notification_devices'), {
      notification_device: {
        device_token: token
      }
    });
  }
}
```

There are two assertions we need to check:
* A call has been made to `configuration.getFrontEndUrl` with the correct parameter
* A call has been made to the backend with the correct parameter

### Solution

```javascript
describe('registerDevice function', function () {
  it('is defined', function () {
    expect(notificationApi.registerDevice).toBeDefined();
  });

  it('it calls the configuration', function () {
    notificationApi.registerDevice('fake-token');
    expect(configuration.getFrontEndUrl).toHaveBeenCalledWith('notification_devices');
  });

  it('makes a post to the backend', function () {
    var hasResponse = false;

    $httpBackend
      .expectPOST('fake-api-call', {
        notification_device: {
          device_token: 'fake-token'
        }
      })
      .respond(200);

    notificationApi.registerDevice('fake-token').then(function () {
      hasResponse = true;
    });

    $httpBackend.flush();
    expect(hasResponse).toBeTruthy();
  });
});
```

### Analysis

The trick is to use `.expectPOST` instead of `.when('POST')`. Then we use `hasResponse` to make sure the call to the backend has been made.


