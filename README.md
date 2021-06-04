# php-projects-Google APIs Client Library for PHP
Reference Docs
https://googleapis.github.io/google-api-php-client/master/
License
Apache 2.0
The Google API Client Library enables you to work with Google APIs such as Gmail, Drive or YouTube on your server.

These client libraries are officially supported by Google. However, the libraries are considered complete and are in maintenance mode. This means that we will address critical bugs and security issues but will not add any new features.

Google Cloud Platform
For Google Cloud Platform APIs such as Datastore, Cloud Storage, Pub/Sub, and Compute Engine, we recommend using the Google Cloud client libraries. For a complete list of supported Google Cloud client libraries, see googleapis/google-cloud-php.

Requirements
PHP 5.6.0 or higher
Developer Documentation
The docs folder provides detailed guides for using this library.

Installation
You can use Composer or simply Download the Release

Composer
The preferred method is via composer. Follow the installation instructions if you do not already have composer installed.

Once composer is installed, execute the following command in your project root to install this library:

composer require google/apiclient:"^2.7"
Finally, be sure to include the autoloader:

require_once '/path/to/your-project/vendor/autoload.php';
This library relies on google/apiclient-services. That library provides up-to-date API wrappers for a large number of Google APIs. In order that users may make use of the latest API clients, this library does not pin to a specific version of google/apiclient-services. In order to prevent the accidental installation of API wrappers with breaking changes, it is highly recommended that you pin to the latest version yourself prior to using this library in production.

Cleaning up unused services
There are over 200 Google API services. The chances are good that you will not want them all. In order to avoid shipping these dependencies with your code, you can run the Google\Task\Composer::cleanup task and specify the services you want to keep in composer.json:

{
    "require": {
        "google/apiclient": "^2.7"
    },
    "scripts": {
        "post-update-cmd": "Google\\Task\\Composer::cleanup"
    },
    "extra": {
        "google/apiclient-services": [
            "Drive",
            "YouTube"
        ]
    }
}
This example will remove all services other than "Drive" and "YouTube" when composer update or a fresh composer install is run.

IMPORTANT: If you add any services back in composer.json, you will need to remove the vendor/google/apiclient-services directory explicity for the change you made to have effect:

rm -r vendor/google/apiclient-services
composer update
NOTE: This command performs an exact match on the service name, so to keep YouTubeReporting and YouTubeAnalytics as well, you'd need to add each of them explicitly:

{
    "extra": {
        "google/apiclient-services": [
            "Drive",
            "YouTube",
            "YouTubeAnalytics",
            "YouTubeReporting"
        ]
    }
}
Download the Release
If you prefer not to use composer, you can download the package in its entirety. The Releases page lists all stable versions. Download any file with the name google-api-php-client-[RELEASE_NAME].zip for a package including this library and its dependencies.

Uncompress the zip file you download, and include the autoloader in your project:

require_once '/path/to/google-api-php-client/vendor/autoload.php';
For additional installation and setup instructions, see the documentation.

Examples
See the examples/ directory for examples of the key client features. You can view them in your browser by running the php built-in web server.

$ php -S localhost:8000 -t examples/
And then browsing to the host and port you specified (in the above example, http://localhost:8000).

Basic Example
// include your composer dependencies
require_once 'vendor/autoload.php';

$client = new Google\Client();
$client->setApplicationName("Client_Library_Examples");
$client->setDeveloperKey("YOUR_APP_KEY");

$service = new Google_Service_Books($client);
$optParams = array(
  'filter' => 'free-ebooks',
  'q' => 'Henry David Thoreau'
);
$results = $service->volumes->listVolumes($optParams);

foreach ($results->getItems() as $item) {
  echo $item['volumeInfo']['title'], "<br /> \n";
}
Authentication with OAuth
An example of this can be seen in examples/simple-file-upload.php.

Follow the instructions to Create Web Application Credentials

Download the JSON credentials

Set the path to these credentials using Google\Client::setAuthConfig:

$client = new Google\Client();
$client->setAuthConfig('/path/to/client_credentials.json');
Set the scopes required for the API you are going to call

$client->addScope(Google_Service_Drive::DRIVE);
Set your application's redirect URI

// Your redirect URI can be any registered URI, but in this example
// we redirect back to this same page
$redirect_uri = 'http://' . $_SERVER['HTTP_HOST'] . $_SERVER['PHP_SELF'];
$client->setRedirectUri($redirect_uri);
In the script handling the redirect URI, exchange the authorization code for an access token:

if (isset($_GET['code'])) {
    $token = $client->fetchAccessTokenWithAuthCode($_GET['code']);
}
Authentication with Service Accounts
An example of this can be seen in examples/service-account.php.

Some APIs (such as the YouTube Data API) do not support service accounts. Check with the specific API documentation if API calls return unexpected 401 or 403 errors.

Follow the instructions to Create a Service Account

Download the JSON credentials

Set the path to these credentials using the GOOGLE_APPLICATION_CREDENTIALS environment variable:

putenv('GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json');
Tell the Google client to use your service account credentials to authenticate:

$client = new Google\Client();
$client->useApplicationDefaultCredentials();
Set the scopes required for the API you are going to call

$client->addScope(Google_Service_Drive::DRIVE);
If you have delegated domain-wide access to the service account and you want to impersonate a user account, specify the email address of the user account using the method setSubject:

$client->setSubject($user_to_impersonate);
How to use a specific JSON key
If you want to a specific JSON key instead of using GOOGLE_APPLICATION_CREDENTIALS environment variable, you can do this:

$jsonKey = [
   'type' => 'service_account',
   // ...
];
$client = new Google\Client();
$client->setAuthConfig($jsonKey);
Making Requests
The classes used to call the API in google-api-php-client-services are autogenerated. They map directly to the JSON requests and responses found in the APIs Explorer.

A JSON request to the Datastore API would look like this:

POST https://datastore.googleapis.com/v1beta3/projects/YOUR_PROJECT_ID:runQuery?key=YOUR_API_KEY

{
    "query": {
        "kind": [{
            "name": "Book"
        }],
        "order": [{
            "property": {
                "name": "title"
            },
            "direction": "descending"
        }],
        "limit": 10
    }
}
Using this library, the same call would look something like this:

// create the datastore service class
$datastore = new Google_Service_Datastore($client);

// build the query - this maps directly to the JSON
$query = new Google_Service_Datastore_Query([
    'kind' => [
        [
            'name' => 'Book',
        ],
    ],
    'order' => [
        'property' => [
            'name' => 'title',
        ],
        'direction' => 'descending',
    ],
    'limit' => 10,
]);

// build the request and response
$request = new Google_Service_Datastore_RunQueryRequest(['query' => $query]);
$response = $datastore->projects->runQuery('YOUR_DATASET_ID', $request);
However, as each property of the JSON API has a corresponding generated class, the above code could also be written like this:

// create the datastore service class
$datastore = new Google_Service_Datastore($client);

// build the query
$request = new Google_Service_Datastore_RunQueryRequest();
$query = new Google_Service_Datastore_Query();
//   - set the order
$order = new Google_Service_Datastore_PropertyOrder();
$order->setDirection('descending');
$property = new Google_Service_Datastore_PropertyReference();
$property->setName('title');
$order->setProperty($property);
$query->setOrder([$order]);
//   - set the kinds
$kind = new Google_Service_Datastore_KindExpression();
$kind->setName('Book');
$query->setKinds([$kind]);
//   - set the limit
$query->setLimit(10);

// add the query to the request and make the request
$request->setQuery($query);
$response = $datastore->projects->runQuery('YOUR_DATASET_ID', $request);
The method used is a matter of preference, but it will be very difficult to use this library without first understanding the JSON syntax for the API, so it is recommended to look at the APIs Explorer before using any of the services here.

Making HTTP Requests Directly
If Google Authentication is desired for external applications, or a Google API is not available yet in this library, HTTP requests can be made directly.

If you are installing this client only to authenticate your own HTTP client requests, you should use google/auth instead.

The authorize method returns an authorized Guzzle Client, so any request made using the client will contain the corresponding authorization.

// create the Google client
$client = new Google\Client();

/**
 * Set your method for authentication. Depending on the API, This could be
 * directly with an access token, API key, or (recommended) using
 * Application Default Credentials.
 */
$client->useApplicationDefaultCredentials();
$client->addScope(Google_Service_Plus::PLUS_ME);

// returns a Guzzle HTTP Client
$httpClient = $client->authorize();

// make an HTTP request
$response = $httpClient->get('https://www.googleapis.com/plus/v1/people/me');
Caching
It is recommended to use another caching library to improve performance. This can be done by passing a PSR-6 compatible library to the client:

use League\Flysystem\Adapter\Local;
use League\Flysystem\Filesystem;
use Cache\Adapter\Filesystem\FilesystemCachePool;

$filesystemAdapter = new Local(__DIR__.'/');
$filesystem        = new Filesystem($filesystemAdapter);

$cache = new FilesystemCachePool($filesystem);
$client->setCache($cache);
In this example we use PHP Cache. Add this to your project with composer:

composer require cache/filesystem-adapter
Updating Tokens
When using Refresh Tokens or Service Account Credentials, it may be useful to perform some action when a new access token is granted. To do this, pass a callable to the setTokenCallback method on the client:

$logger = new Monolog\Logger();
$tokenCallback = function ($cacheKey, $accessToken) use ($logger) {
  $logger->debug(sprintf('new access token received at cache key %s', $cacheKey));
};
$client->setTokenCallback($tokenCallback);
Debugging Your HTTP Request using Charles
It is often very useful to debug your API calls by viewing the raw HTTP request. This library supports the use of Charles Web Proxy. Download and run Charles, and then capture all HTTP traffic through Charles with the following code:

// FOR DEBUGGING ONLY
$httpClient = new GuzzleHttp\Client([
    'proxy' => 'localhost:8888', // by default, Charles runs on localhost port 8888
    'verify' => false, // otherwise HTTPS requests will fail.
]);

$client = new Google\Client();
$client->setHttpClient($httpClient);
Now all calls made by this library will appear in the Charles UI.

One additional step is required in Charles to view SSL requests. Go to Charles > Proxy > SSL Proxying Settings and add the domain you'd like captured. In the case of the Google APIs, this is usually *.googleapis.com.

Controlling HTTP Client Configuration Directly
Google API Client uses Guzzle as its default HTTP client. That means that you can control your HTTP requests in the same manner you would for any application using Guzzle.

Let's say, for instance, we wished to apply a referrer to each request.

use GuzzleHttp\Client;

$httpClient = new Client([
    'headers' => [
        'referer' => 'mysite.com'
    ]
]);

$client = new Google\Client();
$client->setHttpClient($httpClient);
Other Guzzle features such as Handlers and Middleware offer even more control.

Service Specific Examples
YouTube: https://github.com/youtube/api-samples/tree/master/php

How Do I Contribute?
Please see the contributing page for more information. In particular, we love pull requests - but please make sure to sign the contributor license agreement.

Frequently Asked Questions
What do I do if something isn't working?
For support with the library the best place to ask is via the google-api-php-client tag on StackOverflow: https://stackoverflow.com/questions/tagged/google-api-php-client

If there is a specific bug with the library, please file an issue in the GitHub issues tracker, including an example of the failing code and any specific errors retrieved. Feature requests can also be filed, as long as they are core library requests, and not-API specific: for those, refer to the documentation for the individual APIs for the best place to file requests. Please try to provide a clear statement of the problem that the feature would address.

I want an example of X!
If X is a feature of the library, file away! If X is an example of using a specific service, the best place to go is to the teams for those specific APIs - our preference is to link to their examples rather than add them to the library, as they can then pin to specific versions of the library. If you have any examples for other APIs, let us know and we will happily add a link to the README above!

Why does Google_..._Service have weird names?
The _Service classes are generally automatically generated from the API discovery documents: https://developers.google.com/discovery/. Sometimes new features are added to APIs with unusual names, which can cause some unexpected or non-standard style naming in the PHP classes.

How do I deal with non-JSON response types?
Some services return XML or similar by default, rather than JSON, which is what the library supports. You can request a JSON response by adding an 'alt' argument to optional params that is normally the last argument to a method call:

$opt_params = array(
  'alt' => "json"
);
How do I set a field to null?
The library strips out nulls from the objects sent to the Google APIs as its the default value of all of the uninitialized properties. To work around this, set the field you want to null to Google\Model::NULL_VALUE. This is a placeholder that will be replaced with a true null when sent over the wire.

Code Quality
Run the PHPUnit tests with PHPUnit. You can configure an API key and token in BaseTest.php to run all calls, but this will require some setup on the Google Developer Console.

phpunit tests/
Coding Style
To check for coding style violations, run

vendor/bin/phpcs src --standard=style/ruleset.xml -np
To automatically fix (fixable) coding style violations, run

vendor/bin/phpcbf src --standard=style/ruleset.xml
