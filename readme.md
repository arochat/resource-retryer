##resource-retryer: retry failed REST API calls automatically.

This AngularJS module decorates the [$resource](https://docs.angularjs.org/api/ngResource/service/$resource) service so that when creating a resource all actions are wrapped in a function that will retry any failed http calls. To get started follow the installation instructions below and then have a read through the configuration notes. You can also clone this repo and try out the demo.

###installation

**bower:** 
```
bower install  angular-resource-retryer --save
```

Once installed add the script reference and then add a dependendency to the module to make sure it gets loaded.
 
```
var app = angular.module('demo', ['resourceRetryer']);
``` 

(NPM and nuget packages coming soon)

###usage

To enable retrying when calling the $resource factory add a object named retry to the options object.

```
var retryOptions ={
	minWait: 20,
    maxWait: 100,
    retries: 3
};

var User = $resource('/user/:userId', {userId:'@id'},{ retry: retryOptions});
```

With that done all the resource actions will get wrapped with the retryer the $resource service can be used as normal.

**callback syntax**

```
var User = $resource('/user/:userId', {userId:'@id'});

User.get({userId:123}, function(u, getResponseHeaders){
  u.abc = true;
  u.$save(function(u, putResponseHeaders) {
    //u => saved user object
    //putResponseHeaders => $http header getter
  });
});
```

**promise syntax**

```
var User = $resource('/user/:userId', {userId:'@id'});

User.get({userId:123})
    .$promise.then(function(user) {
      $scope.user = user;
    });
```

###configuration

When calling $resource if you pass in an options.retry configuration object the returned resource will have it's actions (get, save,query, remove, delete and any custom actions) wrapped in a retryer function.
Depending on the retry configuration actions will be retried again.

The default retry options are:

```
{
	strategy: 'progressiveBackOff',
	minWait: 50,
	maxWait: 500,
	retries: 5,
	base: 2,
	startSequenceAfter: 1                                
}
```

3 retry strategies are available:
randomizedRetry: the wait before a retry will be a random value of milliseconds between the minWait and maxWait values
progressiveBackOff: follow an exponential sequence of retry wait times starting from the value of startSequenceAfter. (note the minWait or maxWait value will be used if the sequence goes outside of these values)
customs: use a custom function (options.retry.delayCalculator) to calculate the wait time between retries

###options

|	property 	| 	value	 | 	details	|
|---------------|------------|-------------
|	strategy	|	"randomizedRetry", "progressiveBackOff", "custom"	| select the strategy to use when calculating the wait time between each retry	|
|	minWait	|	integer	| minimum time in milliseconds to wait before trying a failed action	|	
|	maxWait	|	integer	| maximum time in milliseconds to wait before trying a failed action	|
|	retries	|	integer	| number of times to retry before failing and rejecting the action promise	|
|	base	|	integer	|	used with **progressiveBackOff** strategy only set the base for the exponential sequence (eg 2 would give a sequence of 1,2,4,8,16,32,64...)	|
|	startSequenceAfter	|	integer	|	used with **progressiveBackOff** strategy only set the starting point for the sequence (eg value of 4 with base 2 would mean sequence starts at 8	|
|	delayCalculator	| function(options, retryCount)	|	used with **custom** strategy only. Custom function that will be called to calculate the wait time between retries. Should return an integer between the minWait and maxWait times	|
|	retryCallback	| function(result, retries, delay) | function called after every failure before the next try. Passes the result a count of the number of retries already made and the delay until the next try	|
    		
###running the demo

Under the demo folder in the repo is a Node.js test server and a simple client. 

To run the server open a command prompt under the demo directory and run:

```
	node testServer
```

, then open **demo.html** in a browser to run the demo client application.
