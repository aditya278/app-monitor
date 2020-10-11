# service-monitoring
Designed and developed a REST service backend for monitoring the HTTP/HTTPS services registered by the user.
The user will be notified by SMS when the service that he/she has registered changes it's state from UP to DOWN or vice versa.

This project is made entirely using core Node.js modules. There is no external dependency on any NPM module.
We have used node.js inbuilt ```fs``` module for storing and retrieving the data.

For constantly monitoring the services registred by the users, we have implemented Service Workers, which run parallel to the Web API and constatly hits the service at the specified time. If the state of the service is changed between two hits, then an SMS is triggered to the user with that particular registered check.
We have user Twilio SMS API for implementing the SMS logic.

## How to Use:
- Clone the Repo.
- Add a config.js file inside the "lib" folder having the below code:
```
const configs = {
    'hashingSecret': ~Any String Which Will be Used for Hashing the Password~,
    'maxChecks': ~Please specify the max number of services that a user can register~,
    'twilio': {
        'fromPhone': ~Your registred twilio number~,
        'accountSid': ~Your Twilio Accound SID as a string~,
        'authToken': ~Your Twilio Auth Token as a string~
    }
};

module.exports = configs;
```
- Run the below command in the root folder:
```
node index.js
```

**User Schema:**
  ```
  -firstName
  -lastName
  -phone (unique)
  -email
  -password
  -tosAgreement
  -(checks - unless specified)
  ```
  
**Checks Schema**
  ```
  - protocol : (The protocol on which the service is running on [currently HTTP and HTTPS are supported])
  - url : (The URL of the service to be monitored)
  - method : (Method for which the service needs to be monitored [get, post, put, delete])
  - successCodes : (The status code retured by the request that is considered as to be "UP" [200,201, etc]
  - timeoutSeconds : (waiting after which the request will be considered times out)
  ```
  
Below are the routes for the API:
  - ```POST/users``` with data in the Body of the request in form of above user schema, will register a new user.
  - ```GET/users?phone=<10-digit-number>``` request will allow you to access individual users data. This request requires the Token ID to be generated as part of Authentication.
  - ```PUT/users``` with data in the body of the request will match the phone number and will update the other specified details of that user. This request requires the Auth token in the header.
  - ```DELETE/users?phone=<10-digit-number>``` request will delete the specified user from the file system. This request required the Auth Token in the header. This request will also delete all the "checks" that are registered by the user too.
  - ```POST/tokens``` with phone and password as body to basically login the user and generate the auth token.
  - ```POST/checks``` with data in the body of the request in the form of above checks schema, will register a new check. Need to pass the auth token in the header of the request. This will also update the user schema for the specified user. (These are basically the services that will be monitored by the service worker.
