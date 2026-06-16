
# Lab 1

## Description

This lab's purchasing flow contains a race condition that enables you to purchase items for an unintended price.

To solve the lab, successfully purchase a **Lightweight L33t Leather Jacket**.

You can log in to your account with the following credentials: `wiener:peter`.

For a faster and more convenient way to trigger the race condition, we recommend that you solve this lab using the [Trigger race conditions](https://github.com/PortSwigger/bambdas/blob/main/CustomAction/ProbeForRaceCondition.bambda) custom action. This is only available in Burp Suite Professional.

## Solution

Add a product to the cart the use the coupon `PROMO20` while having intercepter on. Send the `POST /cart/coupon` request to repeater, then add it to a group and duplicate (how often is arbitrary - but for this lab it needs to fit the store credit of `50$`). Then send the requests in parallel. Even if the requests say `Coupon already applied` the result is a massive reduction on the product. The result is a bit random, it looks like the server accepts about 20 request when on a good run and about 5 on a bad one.