# Insecure Deserialization

## Prerequisites

**Serialization** converts complex data structures such as objects and their fields into a format that can be sent and received as a sequential stream of bytes. Serialized data format can be either binary or string.  

**Deserialiaztion** restors the byte stream to a fully functional replica of the original object, in the exact state as when it was serialized. 

## What Is Insecure Deserialization?
Insecure Deserialization occurs when untrusted data is used to abuse the logic of an application's deserialialization process, allowing an attacker to execute code, manipulate objects, or perform injection attacks. In simpler words, attacker-controlled data is deserialized by the server.

### Example
Java application uses the native Java serialization to save a cookie object on the user's hard drive. The cookie object contains the user's session ID. 

```java
import java.io.*;

public class SerializeCookie {
  public static void main(String[] args) {
    Cookie cookieObj = new Cookie();
    cookieObj.setValue("alice");
    
    try {
      FileOutputStream fos = new FileOutputStream("cookies.ser");
      ObjectOutputStream oos = new ObjectOutputStream(fos);
      oos.writeObject(cookieObj);
      oos.close(); 
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

The writeObject() function serializes the cookieObj to a file called cookies.ser.

Now suppose the application later reads this serialized object from the file:

```java
import java.io.*;

public class DeserializeCookie {
  public static void main(String[] args) {
    try {
      FileInputStream fis = new FileInputStream("cookies.ser");
      ObjectInputStream ois = new ObjectInputStream(fis);
      Cookie cookieObj = (Cookie) ois.readObject();
      System.out.println(cookieObj.getValue());
    } catch (IOException | ClassNotFoundException e) {
      e.printStackTrace();
    }
  }
}
```

If an attacker is able to replace the `cookie.ser` file or its content with a malicious serialized object, they could potentially execute arbitrary code when the object is deserialized. For example, the attacker could create a serialized object that executes OS commands when deserialized.

## How To Find Insecure Deserialization Vulnerabilities?
* Look at all data being passed into the website and try to identify anything that looks like serialized data. 
* Serialized data can be identified easily if you know the format of the different languages. 
* Once serialized data has been identified, test if it can be controlled.
* Tools exist for automated finding
  * Tracing by hand is only practical in small applications
* **Ysoserial** is the main tool for exploiting code

### Insecure PHP Deserialization 
* PHP uses mostly human-readable string format
* Letters represent data types
* Numbers represent the length of each entry

#### Example
Consider the following "User" object with the attributes: 

```html
$user->name = "carlos";
$user->isLoggedIn = true;
```

When serialized, the object may look like: 

`O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}`

This can be interpreted as follows: 

* `P:4:"User"` - An object with the 4-character class name "User"
* `2` - the object has 2 attributes
* `s:4:"name"` - The key of the first attribute is the 4-character string "name"
* `s:6:"carlos"` - The value of the first attribute is the 6-character string "carlos"
* `s:10:"isLoggedIn"` - The key of the second attribute is the 10-character string "isLoggedIn"
* `b:1` - The value of the second attribute is the boolean value `true`

* The native methods for PHP serialization are `serialize()` and `unserialize()`
* If you have source code access, start looking for `unserialize()` anywhere in the code and investigating further. 

### Insecure Java Deserialization
* Serialized Java objects always begin with the same bytes, encoded as `ac ed` in hexadecimal and `RO0` in Base64.
* Any class that implements the `java.io.Serializable` can be serialized and deserialized. 
* If you have source code access, take note of any code that uses the `readObject()` method, which is used to erad and deserialize data from an `InputStream`.

## How To Exploit Insecure Deserialization?

### Initial step for a basic deserialization exploit
* Can be as easy as changing an attribute in a serialized object
* Study the serialized data to identify and edit interesting attribute values
* You can then pass the malicious object into the website via its deserialization process

### Approaches for manipulating serialized objects
* Edit the object directly in its byte stream form
* Write a short script in the corresponding language to create and serialize the new object yourself

### Manipulating object attributes
#### Example
A website uses a serialized `User` object to store data about a user's session in a cookie.
#### Decoded Byte stream
`O:4:"User":2:{s:8:"username";s:6:"carlos";s:7:"isAdmin";b:0;}`
#### Exploit
* Change the `isAdmin` boolean value to `1 (true)`. 
* Re-encode the object and overwrite current cookie

```php
$user = unserialize($_COOKIE);
if ($user->isAdmin === true) {
// allow access to admin interface
}
```
Not a common scenario in the wild.

### Using Application Functionality

#### Example
Suppose an application has the functionality of deleting your account and the application uses a serialization-based session mechanism. 

##### Cookie session
`Tzo0OiJVc2VyIjozOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJ4Z2UyZ3hvYWhra2Q5Y2UxeXJ5bzE3c2UxZ210bnU5biI7czoxMToiYXZhdGFyX2xpbmsiO3M6MTk6InVzZXJzL3dpZW5lci9hdmF0YXIiO30%3d`

##### Decoded Cookie Session
`O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"xge2gxoahkkd9ce1yryo17se1gmtnu9n";s:11:"avatar_link";s:19:"users/wiener/avatar";}`
* We can take advantage of this and delete a user's file from the home directory

##### Modified Cookie Session
`O:4:"User":3:{s:8:"username";s:5:"gregg";s:12:"access_token";s:32:"lcdra3ckgcmecgg1xy7gh8ebob9uczka";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}`

Once the request with the new cookie has been sent, your account along with Carlos's `morale.txt` file will be deleted.

### Arbitrary Injection



## How to Prevent Insecure Deserialization?
