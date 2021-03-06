---
layout: post
title:  API testing using Postman
categories: testing
subcategory: Intermediate
---
All most every web developer or web tester are familiar with [Postman](https://www.getpostman.com/).
Postman is a Chrome add-on and Mac application which is used to fire requests to an API.
It is very lightweight, fast and easy to use. Using this tool we can make different kinds of HTTP requests – GET, POST, PUT, PATCH and DELETE.

In this post, I am going to show some of features of *Postman* that I am using in my everday life.

1. **Collection:**
  A Postman Collection lets you group individual requests together. You can organize these requests in folders.
  For every projct I create new collection.
  
  You can create a new collection from the:

  - Sidebar
  - New button
  
  **Sidebar**

  In the sidebar, select “*Collections*” and click the “*Collections*” icon.
  [new collection from sidebar](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/collections_icon1.png)
  CREATE A NEW COLLECTION modal will appear than fill the require information.
  [create collection modal](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/collections-createcollectionmodal.png)
  
  **New button**
  In the header toolbar, click the New button.
  [New button](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/HeaderToolBar.png)
  The Create New tab appears than select *Collection* and fill require information.
  
  [Collection Documentation](https://www.getpostman.com/docs/postman/collections/creating_collections#collapse-category-0-2)
  
2. **Folder:**
  Folders are a way to organize your API endpoints within a collection into intuitive and logical groups to mirror your       workflow. Next to the collection to which you want to add a folder, click on the ellipses (…) and select “Add Folder”.
  I create different folder for every small section of my app (most of the them for every Controller).
  For example, ***dashboard,***profile*** etc
  [Add Folder Dropdown](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/addFolderDropdown.png)

3. **Sharing collections:**
  This is the feature that I like most. Using this feature you can share your collection to your teammate.
  You must be signed in to your [Postman account](https://www.getpostman.com/docs/postman/launching_postman/postman_account)    to upload or share a collection
  [Share Collection Dropdown](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/shareCollectionDropdown.png)

There are three options for sharing collections.
  - [Team Sharing only for Postman Pro user](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/59137211.png)
  - [Collection link](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/58564829.png)
  - [Embed button](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/58564746.png)

4. **Update Variable data from Response value*:*

postman-main

Variables 
There are two types of variables – global and environment. Global variables are for all requests, environment variables are defined per specific environment which can selected from a drop-down or no environment can be selected. Environments will be discussed in details later in current port. Global variables are editable by small eye-shaped icon in the top right corner. Once defined variables can be used in request with format surrounded by curly brackets: {{VARIABLE_NAME}}.

postman-globals

Pre-Request Script
Postman allows users to do some JavaScript coding with which to manipulate the data being sent with request. One such example is when testing and API with security as explained in How to implement secure REST API authentication over HTTP post – SHA256 hash (build from apiKey + secretKey + timestamp in seconds) is sent as request parameter with the request. Calculating SHA256 hash is done with following pre-request script and then stored as global variable token.

1
2
3
4
5
var timeInSeconds = parseInt((new Date()).getTime() / 1000);
var sigString = postman.getGlobalVariable("apiKey") + 
    postman.getGlobalVariable("secretKey") + timeInSeconds
var token = CryptoJS.SHA256(sigString);
postman.setGlobalVariable("token", token);
Here CryptoJS library is used to create SHA256 hash. All available libraries in Postman are described in Postman Testing Sandbox page. Global variable {{token}} is then send as token request parameter.

postman-pre-request-script

Environments
Code shown above is working fine with just one set of credentials because they are stored as global variables. If you need to switch between different credentials this is where environments come in play. By switching environment and with no change in the request you can send different parameters to API. Environments are managed from Settings icon in the top right corner which opens menu with “Manage Environments” link.

postman-environments

Postman supports so called shared environments, which means whole team can use one and the same credentials managed centrally. It requires sign in and some plan though, but might be good investment in the long run.

In order to use environments pre-request script has to be changed to:

1
2
3
4
var timeInSeconds = parseInt((new Date()).getTime() / 1000);
var sigString = environment.apiKey + environment.secretKey + timeInSeconds
var token = CryptoJS.SHA256(sigString);
postman.setEnvironmentVariable("token", token);
Both apiKey and secretKey are read from environment. Environment can be changed from top right corner.

Nota bene: There is specific behaviour (I would not call it bug as it makes sense) in Postman. If select “No Environment” and fire request above Postman will automatically create environment with name “No Environment”. This is because it actually needs an environment to store variable into. This might be very confusing first time.

Post-Request Script
There is no such term defined in Postman. The idea is that in many cases you will need to do something with the response and extract variable from it in order to use it at later stage. This can be done in “Tests” tab. Example given bellow is to take all persons with API call and then to process response and at random select one id which is stored as global variable and then used in next request. You can put whatever JavaScript code you like in order to fulfil your logic.

1
2
3
4
var jsonData = JSON.parse(responseBody)
var size = jsonData.length
var index = parseInt(Math.random() * size)
postman.setGlobalVariable("userId", index);
postman-post-request

Then in subsequent request you can use GET call to URL: http://localhost:9000/person/get/{{userId}}

Tests
After response is received Postman has functionality to make verifications on it. This is done in “Tests” tab. Bellow is example on different verifications. Most interesting part is in case of JSON response it can be parsed to an array and then elements accessed by index and value jsonData[0].id or even iterated as shown bellow. Format is: tests[“TEST_NAME”] = BOOLEAN_CONDITION.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
tests["Status code is 200"] = responseCode.code === 200;
 
tests["Response time is less than 200ms"] = responseTime < 200;
 
var expected = "email1@email.na"
tests["Body cointains string: " + expected] = responseBody.has(expected);
 
var jsonData = JSON.parse(responseBody);
var expectedCount = 4
tests["Response count is: " + expectedCount] = jsonData.length === expectedCount;
 
for(var i=1; i<=expectedCount; i++) {
    tests["Verify id is: " + i] = jsonData[i-1].id === i;
}
postman-test-response

postman-test-results

Nota bene: if you use responseTime verification you have to know that it measures just the TTFB (time to first bite) it does not measure time needed to transfer the data. If you have API with big responses or network is slow you may fire the request, wait a lot and then Postman shows very small response time which might be confusing.

Run from command line
In order to run Postman tests in command line as part of some CI process there is separate tool called Newman. It requires NodeJS to be installed and runs on NodeJS environment. It is very well described in How to write powerful automated API tests with Postman, Newman and Jenkins.

Code reuse between requests
It is very convenient some piece of code to be re-used between request to prevent copy/paste it. Postman does not support yet code re-use between requests. Good thing is that there is workaround for this. It is possible to do it by defining a helper function with verifications which is saved as global variable in first request from your test scenario:

1
2
3
4
5
6
7
8
9
10
11
12
13
postman.setGlobalVariable("loadHelpers", function loadHelpers() {
    let helpers = {};
 
    helpers.verifyCount = function verifyCount(expectedCount) {
        var jsonData = JSON.parse(responseBody);
        tests["Response count is: " + expectedCount] 
            = jsonData.length === expectedCount;
    }
 
    // ...additional helpers
 
    return helpers;
} + '; loadHelpers();');
Then from other requests helpers are taken from global variables and verification functions can be used:

1
2
var helpers = eval(globals.loadHelpers);
helpers.verifyCount(4);
See more in Reusing pre-request scripts across requests in a collection issue thread.
