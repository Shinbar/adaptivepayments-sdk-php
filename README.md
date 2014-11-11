
# PayPal PHP Adaptive Payments SDK
## POODLE Update
- Because of the Poodle vulnerability, PayPal has disabled SSLv3. 
- To enable TLS encryption, the changes were made to [PPHttpConfig.php](https://github.com/paypal/sdk-core-php/blob/namespace-5.3/lib/PayPal/Core/PPHttpConfig.php#L11) in [SDK Core](https://github.com/paypal/sdk-core-php/tree/namespace-5.3) to use a cipher list specific to TLS encryption.
``` php
    /**
	 * Some default options for curl
	 * These are typically overridden by PPConnectionManager
	 */
	public static $DEFAULT_CURL_OPTS = array(
		CURLOPT_SSLVERSION => 1,
		CURLOPT_CONNECTTIMEOUT => 10,
		CURLOPT_RETURNTRANSFER => TRUE,
		CURLOPT_TIMEOUT        => 60,	// maximum number of seconds to allow cURL functions to execute
		CURLOPT_USERAGENT      => 'PayPal-PHP-SDK',
		CURLOPT_HTTPHEADER     => array(),
		CURLOPT_SSL_VERIFYHOST => 2,
		CURLOPT_SSL_VERIFYPEER => 1,
		CURLOPT_SSL_CIPHER_LIST => 'TLSv1',
	);
```
- There are two primary changes done to curl options: 
    - CURLOPT_SSLVERSION is set to 1 . See [here](http://curl.haxx.se/libcurl/c/CURLOPT_SSLVERSION.html) for more information
    - CURLOPT_SSL_CIPHER_LIST was set to TLSv1, See [here](http://curl.haxx.se/libcurl/c/CURLOPT_SSL_CIPHER_LIST.html) for more information

All these changes are included in the recent release, along with many other bug fixes. We highly encourage you to update your versions, by either using `composer` or running this command shown below:

```
curl -L https://raw.github.com/paypal/adaptivepayments-sdk-php/stable-php5.3/samples/install.php | php    
        OR        
wget  https://raw.github.com/paypal/adaptivepayments-sdk-php/stable-php5.3/samples/install.php
php install.php
```

## Prerequisites

PayPal's PHP Adaptive Payments SDK requires 

   * PHP 5.3 and above 
   * curl/openssl PHP extensions 

## Running the sample

To run the bundled sample, first copy the samples folder to your web server root. You will then need to install the SDK as a dependency using either composer (PHP V5.3+ only).


run `composer update` from the samples folder.

## Using the SDK


To use the SDK,

   * Create a composer.json file with the following contents.
```json
{
    "name": "me/shopping-cart-app",
    "require": {
        "paypal/adaptivepayments-sdk-php":"v3.6.106"
    }
}
```

   * Install the SDK as a dependency using composer. 
   * Require `vendor/autoload.php` OR `PPBootStrap.php` in your application.
   * Choose how you would like to configure the SDK - You can either
	  * Create a hashmap containing configuration parameters and pass it to the service object OR
      * Create a `sdk_config.ini` file and set the PP_CONFIG_PATH constant to point to the directory where this file exists.
   * Instantiate a service wrapper object and a request object as per your project's needs.
   * Invoke the appropriate method on the service object.

For example,

```php
	// Sets config file path(if config file is used) and registers the classloader
    require("PPBootStrap.php");
	
	// Array containing credentials and confiuration parameters. (not required if config file is used)
	$config = array(
       'mode' => 'sandbox',
       'acct1.UserName' => 'jb-us-seller_api1.paypal.com',
       'acct1.Password' => 'WX4WTU3S8MY44S7F'
       .....
    );

    $payRequest = new PayRequest($requestEnvelope, $actionType, $cancelUrl, 
                                  $currencyCode, $receiverList, $returnUrl);
    // Add optional params
    if($_POST["feesPayer"] != "") {
	   $payRequest->feesPayer = $_POST["feesPayer"];
    }
	......

	$service = new AdaptivePaymentsService($config);
	$response = $service->Pay($payRequest);	
	if(strtoupper($response->responseEnvelope->ack == 'SUCCESS') {
		// Success
	}
```  
  
## Authentication

The SDK provides multiple ways to authenticate your API call.

```php
	$service = new AdaptivePaymentsService($config);
	
	// Use the default account (the first account) configured in sdk_config.ini or config hashmap
	$response = $service->Pay($payRequest);	

	// Use a specific account configured in sdk_config.ini or config hashmap
	$response = $service->Pay($payRequest, 'jb-us-seller_api1.paypal.com');	
	 
	// Pass in a dynamically created API credential object
    $cred = new PPCertificateCredential("username", "password", "path-to-pem-file");
    $cred->setThirdPartyAuthorization(new PPTokenAuthorization("accessToken", "tokenSecret"));
	$response = $service->Pay($payRequest, $cred);	
```

## SDK Configuration


The SDK allows you to configure the following parameters. 

   * Integration mode (sandbox / live)
   * (Multiple) API account credentials.
   * HTTP connection parameters
   * Logging 

   Dynamic configuration values can be set by passing a map of credential and config values (if config map is passed the config file is ignored)
```php
    $config = array(
       'mode' => 'sandbox',
       'acct1.UserName' => 'jb-us-seller_api1.paypal.com',
       'acct1.Password' => 'WX4WTU3S8MY44S7F'
       .....
    );
	$service  = new AdaptivePaymentsService($config);
```
Alternatively, you can configure the SDK via the sdk_config.ini file. 
```php
    define('PP_CONFIG_PATH', '/directory/that/contains/sdk_config.ini');
    $service  = new AdaptivePaymentsService();
```

You can refer full list of configuration parameters in [wiki](https://github.com/paypal/sdk-core-php/wiki/Configuring-the-SDK) page.

## Instant Payment Notification (IPN)

Please refer to the IPN-README in the 'samples/IPN' directory.

## Links

   * API Reference - https://developer.paypal.com/webapps/developer/docs/classic/api/#ap
   * If you need help using the SDK, a new feature that you need or have a issue to report, please visit https://github.com/paypal/adaptivepayments-sdk-php/issues 
