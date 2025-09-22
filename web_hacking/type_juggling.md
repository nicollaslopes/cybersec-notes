# Type Juggling

Type juggling vulnerability in PHP applications occurs when a loose comparison operator (`==` or `!=`) is used in the place of a strict comparison operator (`===` or`!==`) in a situation where the attacker has access to one of the variables being compared. For more details, we can see how comparison operators in PHP works in the following example:

PHP Comparisons: Loose

<figure><img src="../.gitbook/assets/type_juggling-1.jpg" alt=""><figcaption></figcaption></figure>

PHP Comparisons: Strict

<figure><img src="../.gitbook/assets/type_juggling-2.jpg" alt=""><figcaption></figcaption></figure>

## Scenarios of Exploitation

This vulnerability, though, is not always exploitable, frequently requires the use of a deserialization bug. This is due to the fact that POST, GET, and cookie values are typically supplied to programs as strings or arrays.

PHP would compare two strings if the program received the POST parameter from the previous example as a string, therefore no type conversion would be required. And it goes without saying that “0” and “password” are separate strings.

```php
var_dump("password" == "0");  // false
```

The problem is that in PHP versions < 7.4, when comparing a string with an int 0 for example, it returns true.

```php
var_dump("password" == 0);  // true
```

If the application accepts the input through functions like **json\_decode()** or **unserialize()**, type juggling issues can be exploited. The end-user would be able to specify the kind of input that is passed in in this way.

Let's take a look at the following example, we send a string `api123` to check if the key is correct, and obviously it is not.

<figure><img src="../.gitbook/assets/type-juggling-3.png" alt=""><figcaption></figcaption></figure>

However, when we edit and put an int 0, we can notice that the application responds as true and thus we can bypass the comparison.

<figure><img src="../.gitbook/assets/type-juggling-4.png" alt=""><figcaption></figcaption></figure>

As mentioned earlier, in PHP < 7.4, `0` is equal as `a2kml49nm2nvo4jy` which is the API key. We can take a look in the following code.

```php
<?php
header("Content-Type: application/json");

if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    $raw = file_get_contents('php://input');

    $data = json_decode($raw, true);

    $api_key = $data["api_key"];

    if ($api_key == "a2kml49nm2nvo4jy"){
        echo json_encode([
            "valid" => true,
            "message" => "✅ API Key is valid!"
        ]);
    } else {
        echo json_encode([
            "valid" => false,
            "message" => "❌ Invalid API Key."
        ]);
    }
    exit;
}
```

In this scenario, it is really important to use `===` comparison operator to check the type.
