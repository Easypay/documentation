## Visa Details 2.0

In order to get the visa details url ready to receive calls from easypay you should first create a small script file and lodge it in a public domain.

Example:

your-domain.com/scripts/visa-detail.php

The following example will be written in PHP code but you can adapt it to other language of your choosing:

```
<?php

/**
 * First we check some params from the GET request
 */

if (!isset($_GET['t_key']) || !isset($_GET['id'])) {
    echo "Error: Not enough params";
    exit();
}

/**
 * You have to fetch your account id, the id you use to authenticate yourself with 2.0 APIs
 * @url https://api.prod.easypay.pt/docs#section/Authentication
 * For safety you should check that it matches
 */

if ($api_auth['account_id'] != $_GET['id']) {
    echo "Error: Data mismatch";
    exit();
}

/**
 * If all checks out start to build the response
 */

// Simple mysqli connection as documented at https://www.php.net/manual/en/mysqli-result.fetch-row.php

$mysqli = new mysqli("localhost", "my_user", "my_password", "world");

/* check connection */
if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}

$xml = '<?xml version="1.0" encoding="ISO-8859-1" ?>' . PHP_EOL;
//Output XML
$xml.= '<get_detail>' . PHP_EOL;

global $result;

try {

    //Fetch Payment Details from your notifications table and order details, you may need more than one query
    $query = 'SELECT * FROM easypay_notifications_table
            WHERE  t_key = ' . $_GET['t_key'];

    $result = $mysqli->query($query)

    $obj = $result->fetch_object()

    //Header
    $xml.= '<ep_status>'      . 'ok'               .'</ep_status>' . PHP_EOL;
    $xml.= '<ep_message>'     . 'success'          .'</ep_message>' . PHP_EOL;
    $xml.= '<ep_value>'       . $obj->ep_value     .'</ep_value>' . PHP_EOL;
    $xml.= '<t_key>'          . $obj->t_key        .'</t_key>' . PHP_EOL;
    //Order Information
    $xml.= '<order_info>' . PHP_EOL;

    $xml.= '<total_taxes>'            . $obj->taxes . '</total_taxes>' . PHP_EOL;
    $xml.= '<total_including_taxes>'  . $obj->total . '</total_including_taxes>' . PHP_EOL;
    $xml.= '<bill_name>'              . $obj->billing_first_name  . ' ' . $obj->billing_first_name . '</bill_name>'  . PHP_EOL;
    $xml.= '<shipp_name>'             . $obj->shipping_first_name . ' ' . $obj->shipping_last_name . '</shipp_name>' . PHP_EOL;
    $xml.= '<bill_address_1>'         . $obj->billing_address_1     . '</bill_address_1>' . PHP_EOL;
    $xml.= '<shipp_adress_1>'         . $obj->shipping_address_1    . '</shipp_adress_1>' . PHP_EOL;
    $xml.= '<bill_address_2>'         . $obj->billing_address_2     . '</bill_address_2>' . PHP_EOL;
    $xml.= '<shipp_adress_2>'         . $obj->shipping_address_2    . '</shipp_adress_2>' . PHP_EOL;
    $xml.= '<bill_city>'              . $obj->billing_city          . '</bill_city>' . PHP_EOL;
    $xml.= '<shipp_city>'             . $obj->shipping_city         . '</shipp_city>' . PHP_EOL;
    $xml.= '<bill_zip_code>'          . $obj->billing_postcode      . '</bill_zip_code>' . PHP_EOL;
    $xml.= '<shipp_zip_code>'         . $obj->shipping_postcode     . '</shipp_zip_code>' . PHP_EOL;
    $xml.= '<bill_country>'           . $obj->billing_country       . '</bill_country>' . PHP_EOL;
    $xml.= '<shipp_country>'          . $obj->shipping_country      . '</shipp_country>' . PHP_EOL;

    $xml.= '</order_info>' . PHP_EOL;

    // Now we get all the items in the order insert them into the XML response
    $xml.= '<order_detail>' . PHP_EOL;
    foreach ( $order->get_items() as $item ) {
        $xml.= '<item>' . PHP_EOL;
        $xml.= '<item_description>'   . $item['name']       . '</item_description>' . PHP_EOL;
        $xml.= '<item_quantity>'      . $item['qty']        . '</item_quantity>' . PHP_EOL;
        $xml.= '<item_total>'         . $item['line_total'] . '</item_total>' . PHP_EOL;
        $xml.= '</item>' . PHP_EOL;
    }
    $xml.= '</order_detail>' . PHP_EOL;

} catch (Exception $ex) {
    $xml.= '<ep_status>'      . 'err'                .'</ep_status>' . PHP_EOL;
    $xml.= '<ep_message>'     . $ex->getMessage()    .'</ep_message>' . PHP_EOL;
    $xml.= '<ep_entity>'      . $obj->ep_entity      .'</ep_entity>' . PHP_EOL;
    $xml.= '<ep_reference>'   . $obj->ep_reference   .'</ep_reference>' . PHP_EOL;
    $xml.= '<ep_value>'       . $obj->ep_value       .'</ep_value>' . PHP_EOL;
}

$xml.= '</get_detail>' . PHP_EOL;
echo $xml;

```

Note that you may not have all the information to fill in the XML, in any case you should pass all that you can leaving the other fields empty.


###.

## Visa Forward 2.0

When your client finalizes his credit card authorization he will be redirected to "visa-fwd" landing page.
You can configure this URL on easypay's backoffice under "URL Configurations"

In order to get the visa forward url ready to receive calls from easypay with query parametrs you should first create a small script file and lodge it in a public domain.

Example:

your-domain.com/scripts/visa-forward.php
###### Note: Easypay will redirect to visa-fwd url adding the a query parameters, as you can see below

The following example :
```
http://www.your-domain.com/visa-forward.php?e=10611&r=999907995&v=15.25&k=CCCA9DB585706CB435164C2D6A06C3D27AB1ABA&s=ok&t_key=3
```
#### Parameters
| Parameters  | Descriptions |
| ------------ | -------------|
|___________|__________________________________________________________|              
|           |ENTITY of the transaction the client is paying.|
| e         |Example value: 10611 |
|___________|__________________________________________________________|              
|           | REFERENCE of the transaction the client is paying.|
| r         | Example value: 999907995 |
|___________|__________________________________________________________|              
|           | Value that the client is paying.|
| v         |            Example value: 15.25 |
|___________|__________________________________________________________|              
| k         | Authorization key.|
|           |  Example value: CCCA9DB585706CB435164C2D6A06C3D27AB1ABA |
|___________|__________________________________________________________|              
|           | Status of authorization.|
| s         |    Available values: ok ( authorized ) or err ( not authorized ) |
|___________|__________________________________________________________|              
|           | Your order identification.|
|  t_key    |         Example value: 3 |
|___________|__________________________________________________________|              


#### Last step
Now if "s" is ok you have to store the authorization key with your order record in order to request payment in the future.

If "s" is err you should show a message like:
"Your transaction was not authorized by your credit card issuer. Please try again with another card or use one of the following payment methods.".

#### Example Code:

The following example will be written in PHP code but you can adapt it to other language of your choosing:
```
<?php

/**
 * First we check some params from the GET request
 */

if (!isset($_GET['e']) || !isset($_GET['r']) || !isset($_GET['v']) || !isset($_GET['s']) || !isset($_GET['t_key'])){
    echo "Error: Not enough params";
    exit();
}


// Simple mysqli connection as documented at https://www.php.net/manual/en/mysqli-result.fetch-row.php

$mysqli = new mysqli("localhost", "my_user", "my_password", "world");

/* check connection */
if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}

global $result;


//Fetch Payment Details from your notifications table and order details, you may need more than one query
   
$query = 'SELECT * FROM <order_table>
           WHERE  t_key = ' . $_GET['t_key'];

$result = $mysqli->query($query)

$orderObject = $result->fetch_object()

if ($_GET['s'] != 'ok') {
    $result = $mysqli->query('UPDATE <order_table> SET <status> ='failed' WHERE <id>='.$orderObject->id);
    If($result !== TRUE){
        $response = sprintf("Error updating order record: %s\n", mysqli_connect_error());
    }
        
    /* Here you should show the error message to your customer. 
     * But remember to make sure that your customer know that he can try again to pay with another credit card.
     * Show a message like: "Your transaction was not authorized by your credit card issuer. Please try again with another card or use one of the following payment methods.
     */  
}
  
/*
* Here you have to store the authorization key with your order record in order to request payment in the future and prepare the order shipment.
*/  
$result = $mysqli->query('UPDATE <order_table> SET <status> ='payed' WHERE <id>='.$orderObject->id);
If($result !== TRUE){
   $response =  sprintf("Error updating record: %s\n", mysqli_connect_error());
}


 //Here you should show the success message to your customer. 
```

