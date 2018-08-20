---
title: "Signing Coinbase Pro requests with Postman"
date: "2018-08-20"
categories:
  - "Software Engineering"
  - "Cryptocurrency"
tags:
  - "Coinbase"
  - "Bitcoin"
  - "JavaScript"
  - "Code"
---

For running some tests against the GDAX/Coinbase Pro API, I needed to send properly
signed requests with Postman. To achieve this, I've created the short pre-request script below that
you can easily add to your Postman collection. More information about signing Coinbase Pro API
messages can be found in their [documentation](https://docs.pro.coinbase.com/#signing-a-message).

You might want to adapt the variable names and URL path regex according to your needs. Also, don't
forget that you still need to set the proper headers in your request resource. The code below
obviously also works outside Postman.

<script src="https://gist.github.com/marianzange/354edb2f74e88491f6fa520ea8beb2d8.js"></script>
