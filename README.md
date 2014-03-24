Veritrans PHP Wrapper
==============================================

PHP wrapper for Veritrans Payment API. Visit [https://www.veritrans.co.id](https://www.veritrans.co.id) for more information about the product and see documentation at [http://docs.veritrans.co.id](http://docs.veritrans.co.id/vtweb/index.html) for more technical details.

## Installation

### Composer Installation

If you are using [Composer](https://getcomposer.org), add this require line to your `composer.json` file:

```json
"require": {
	"veritrans/veritrans-php": "dev-vtweb-2"
}
```

and run `composer install` on your terminal.

### Manual Instalation

If you are not using Composer, just copy all files in this repository into your project's library.

## How to use

### Setting up basic information

Given you have a cart ready for checkout, the first step you have to do to interact with Veritrans is to create a Veritrans instance and populating it with basic information.

```php

// Create a new Veritrans instance. It will default to 2013 stack and VT-Web payment method.
$veritrans = new Veritrans();

$veritrans->order_id = 'XDF1AA5'; // change the order_id property with your actual order ID.
```

### About Veritrans stack

Veritrans API is always evolving. As a result, there are several stacks that is improved in each subsequent version.
Veritrans stack is transparent to the API. Veritrans defined the stacks inside the `veritrans.php` file, and the code fragment below displays the available stacks of the current Veritrans API:

```php
class Veritrans {
	// ...
	const STACK_2013 = 1;
	const STACK_2014 = 2;
	const STACK_STABLE = self::STACK_2013;
	// ...	
}
```

At initialization, the stack will be always initialized to `STACK_STABLE`, which points to `STACK_2013` in this case.
You can set the stack you want to use by manipulating the `version` property.

```php
// Set the stack to STACK_2014 instead of the STACK_STABLE
$veritrans->version = Veritrans::STACK_2014;
```

### Environment

There are two environments in Veritrans:

1. __Development__ environment, which is defined as `Veritrans::ENVIRONMENT_DEVELOPMENT` and
2. __Production__ environment, which is defined as `Veritrans::ENVIRONMENT_PRODUCTION`.

Veritrans PHP will default to the __Development__ environment. You can set the environment by accessing the `environment` property.

```php
// Set the environment to production
$veritrans->environment = Veritrans::ENVIRONMENT_PRODUCTION;
```

### Payment types

Independently from the stacks, you may define the payment type which will be used by the library. The available payment types are also defined in the `veritrans.php` file.

```php
class Veritrans {
	// ...
	/***
	  *
	  * VT-Web payment type
	  *
	  */
	const VT_WEB = 0;

	/***
	  *
	  * VT-Direct payment type
	  *
	  */
	const VT_DIRECT = 1;
	// ...
}
```

At initialization, Veritrans defaults to `VT_WEB` type. You can change the payment method by accessing the `payment_type` method.

```php
$veritrans->payment_type = Veritrans::VT_DIRECT;
```

### Setting up customer's billing information

The handling of customer information is different in each stack.

- __2013 Stack:__ It is optional to set-up the customer information, but if you are developing plugins using this library, it is __highly__ recommended to set it up to give the best user experience. Otherwise, your customer have to fill them in the VT-Web page.

- __2014 Stack:__ Some of the customer information fields become obligatory in the 2014 stack, which will be displayed in the example below.

To set up your customer information, you can manipulate the following properties:

```php
$veritrans->first_name = "Andri"; // obligatory in 2014 stack
$veritrans->last_name = "Setiawan";
$veritrans->email = "customer@email.com"; // obligatory in 2014 stack
$veritrans->city = "Jakarta"; // obligatory in 2014 stack
$veritrans->country_code = "IDN";
$veritrans->postal_code = "12345"; // obligatory in 2014 stack
$veritrans->phone = "08123123123123";

// To define the customer's billing address, you can either set the address1 and address2 properties...
$veritrans->address1 = "Karet Belakang"; // obligatory in 2014 stack
$veritrans->address2 = "Setiabudi";

// ...or by setting the address property
$veritrans->address = "Karet Belakang Setiabudi";
```

### Do you need a shipping address?

Whether you need a shipping address or not depends on the type of the merchandise your customer orders. 
For example, if your customer orders an electornic airline ticket that will be sent online, you do not
need to define a shipping address. In other case, if you want to ship the merchandise, you need to tell Veritrans the
the shipping address of the order.

State that you want to send shipping address information by setting the `required_shipping_address` property.

```php
$veritrans->required_shipping_address = 1; // Set '0' if shipping address is not required
```

Now where would you ship your order?

- If it is the same as the billing address, set the `billing_different_with_shipping` flag to `FALSE`.

	```php
	$veritrans->billing_different_with_shipping = FALSE; // Set FALSE if shipping address == billing address
	```

- Otherwise, set it to `FALSE` and complete your shipping information.
	
	```php
	$veritrans->billing_different_with_shipping = TRUE; // Set TRUE if shipping address != billing address

	$veritrans->shipping_first_name = "John";
	$veritrans->shipping_last_name = "Watson";
	$veritrans->shipping_address1 = "Bakerstreet 221B";
	$veritrans->shipping_address2 = "Tebet";
	$veritrans->shipping_city = "Jakarta";
	$veritrans->shipping_country_code = "IDN";
	$veritrans->shipping_postal_code = "12346";
	$veritrans->shipping_phone 	= "082313123131";
	```

### Setting your order detail information

Next, you need to tell Veritrans the detail of the order. The following code illustrates the method to do it.

```php
// Set commodity items
$items = array(
			array(
				"item_id" => 'itemsatu',
				"price" => 250000,
				"quantity"   => 1,
				"item_name1" => 'sepatu',
				"item_name2" => 'Shoes' // item_name2 is only obligatory in 2013 Stack's VT-Web method
			),
			array(
				"item_id" => 'itemdua',
				"price" => 500000,
				"quantity"   => 2,
				"item_name1" => 'Tas',
				"item_name2" => 'Bag' // item_name2 is only obligatory in 2013 Stack's VT-Web method
			),
		);
$veritrans->items = $items;
```

### Setting up your payment options

There are myriads of options to be set with Veritrans. Please consult [this page](http://docs.veritrans.co.id/vtweb/other_features.html) to see the optional features that can be set with Veritrans.

- __3-D Secure:__ Enable a more powerful authentication for your customer. You (or your merchant) must sign additional documents with Veritrans though to activate it. 
  
  ```php
  $veritrans->enable_3d_secure = TRUE;
	```

- __Promo:__ Set promotion.
  
  ```php
  $veritrans->promo_bins = array("411111", "444444");
  ```

- __Acquiring bank:__ Set the acquiring bank if you have multiple accounts registered with Veritrans.
  
  ```php
  $veritrans->bank = "bni";
  ```  

- __Installment transactions:__ 

	```php

	$veritrans->installment_banks = array("bni", "cimb");
	$veritrans->installment_terms = array(
		'bni' => array(3, 12),
		'cimb' => array(3, 6, 12)
		);
	```

- __Transaction with shopping points:__
	```php
	$veritrans->point_banks	= array("bni", "cimb");
	```

- __Setting the available payment methods for the VT-Web:__
	```php
	$veritrans->payment_methods	= array("credit_card", "mandiri_clickpay");
	```

## Step 2: Setting up your API keys

As a final step, you have to set your API keys in order to let yourself get authenticated by Veritrans API. The methods to set the keys are different for each stack.

### Current Stack

In the current stack (2013), set the `merchant_id` property with your Merchant ID and `merchant_hash_key` with your Merchant Hash Key. Both of them are available [here](https://payments.veritrans.co.id/map).

```php
//TODO: Change with your actual merchant id and merchant hash key
$veritrans->merchant_id 		= 'T100000000000001000001';
$veritrans->merchant_hash_key 	= '305e0328a366cbce8e17a385435bb7eb3f0cbcfbfc0f1c3ef56b658';
```

### 2014 Stack

If you set the `version` to `Veritrans::STACK_2014`, you have to set your keys by setting the `server_key` property with the Server Key from your account. The server key can be obtained [here](https://my.sandbox.veritrans.co.id/settings/config_info)

```php
//TODO: Change with your actual server key
$veritrans->server_key 		= 'eebadfec-fa3a-496a-8ea0-bb5795179ce6';
```

## Step 3: Authenticate with Veritrans API

### VT-Web

```php
//Call Veritrans VT-Web API Get Token
try {
	$keys = $veritrans->getTokens();

	if(!$keys) {
		print_r($veritrans->errors);
		exit();
	} else {
		//Save this token_merchant on your database to be used for checking veritrans notification response.
		$token_merchant = $keys['token_merchant'];

		//Use this token_browser for redirecting customer to Veritrans payment page.
		$token_browser = $keys['token_browser'];
	}
} catch (Exception $e) {
	var_dump($e);
}
```

### VT-Direct

TODO

## STEP 4:  Redirecting user to Veritrans payment page

**Prepare the FORM to redirect the customer**
	
```html
<!DOCTYPE html>
<html>
<head>
	<script language="javascript" type="text/javascript">
	<!--
	function onloadEvent() {
	  document.form_auto_post.submit();
	}
	//-->
	</script>
</head>

<body onload="onloadEvent();">
<form action="<?php echo Veritrans::PAYMENT_REDIRECT_URL ?>" method="post" name='form_auto_post'>
<input type="hidden" name="MERCHANT_ID" value="<?php echo $veritrans->merchant_id ?>" />
<input type="hidden" name="ORDER_ID" value="<?php echo $veritrans->order_id ?>" />
<input type="hidden" name="TOKEN_BROWSER" value="<?php echo $token_browser ?>" />
<span>Please wait. You are being redirected to Veritrans payment page...</span>
</form>

</body>
```

### STEP 3 : Responding Veritrans payment notification
After the payment process is completed, Veritrans will send HTTP(S) POST notification to merchant's web server.
As a merchant, you need to process this POST paramters to update order status in your database server. Veritrans will send 3 POST parameters: `orderId`, `mStatus`, and `TOKEN_MERCHANT`.

```php
$notification = new VeritransNotification();

if($notification->mStatus == "fatal")
{
	// Veritrans internal system error. Please contact Veritrans Support if this occurs.
}
else
{
	// TODO: Retrieve order info from your database to check the token_merchant for security purpose.
	// The token_merchant that you get from this http(s) POST notification must be the same as token_merchant that you get previously when requesting token (before redirecting customer to Veritrans payment page.)

	//$order = Order::find('order_id' => $notification->orderId);

	if($order['TOKEN_MERCHANT'] == $notification->TOKEN_MERCHANT )
	{
		// TODO: update order payment status on your database. 3 possibilities $notification->mStatus responses from Veritrans: 'success', 'failure', and 'challenge'
		// $order['payment_status'] = $notification->mStatus; 
		// $order->save();
	}
	else
	{
		// If token_merchant that you get from this http(s) POST request is different with token_merchant that you get previously when requesting token (before redirecting customer to Veritrans payment page.), there is a possibility that the http(s) POST request is not coming from Veritrans. 
		// Don't update your payment status. 
		// To make sure, check your Merchant Administration Portal (MAP) at https://payments.veritrans.co.id/map/
	}
}
```

## Contributing

### Developing e-commerce plug-ins

### New Veritrans stack

## Credits
