---
layout: post
title: Google CTF - Ernst Echidna
comments: true
---

Let's talk about the [Ernst Echidna](https://ernst-echidna.ctfcompetition.com/) web challenge of the [Google CTF](https://capturetheflag.withgoogle.com).

<div class="danger" markdown="1">
  **Warning:** This post contains spoilers.
</div>

The challenge description goes like that:

>Can you hack this website? The robots.txt sure looks interesting.

Let's start by loading the website.

![Ernst Echidna homepage](/public/img/2016/05/04/Google-CTF-Ernst-Echidna/homepage.png)

A good practice (especially on web CTF) is to follow the trails that are given, and then explore around them. Let's follow the **robots.txt** trail.

<!--more-->

```
Disallow: /admin
```

This is really informative, nothing we could have guessed. Here is the admin page:

![Blocked administration page](/public/img/2016/05/04/Google-CTF-Ernst-Echidna/admin-not-logged-in.png)

On the homepage, there is a link to register, let's try it.

![Register page with username and password](/public/img/2016/05/04/Google-CTF-Ernst-Echidna/register-form.png)

We are redirected to a [welcome](https://ernst-echidna.ctfcompetition.com/welcome) page that state there is no content.

![Welcome page](/public/img/2016/05/04/Google-CTF-Ernst-Echidna/register-confirmation.png)

Now that we are registered (and apparently logged in), we should have a **cookie** to reflect that.

``` js
document.cookie; // display the cookies in the developer console
```

And we have a very interesting cookie:

```
md5-hash=be62e165534615fb9bfbda456f2e12a8
```

It looks like it's a md5-hash (the name is obvious, but some people are evil enough to put us on the wrong trail). I wonder what is hashed... There is a good website where you can paste a hash and it will try to match it. It will only work if the hash is not [salted](https://en.wikipedia.org/wiki/Salt_(cryptography)). The website is [CrackStation](https://crackstation.net/)

Enter your hash in the CrackStation and you should obtain your username. This is an example where it is useful to take a simple username when registering in a CTF, so that the hash is easily found.

![CrackStation website with the hash and the username found](/public/img/2016/05/04/Google-CTF-Ernst-Echidna/crackstation.png)

We are logged in, and our cookie contain our username. Let's try the admin page again.

![Admin page with restricted message](/public/img/2016/05/04/Google-CTF-Ernst-Echidna/admin-restricted.png)

We need to become an admin. Since the token is only the md5 hash of the username, we can try to hash `admin` and set our cookie with the new hash. Let's try it.

``` bash
$ echo -n "admin" | md5 # Get the hash in your terminal
> 21232f297a57a5a743894a0e4a801fc3
```

``` js
// Set the cookie in your browser console
document.cookie = "md5-hash=21232f297a57a5a743894a0e4a801fc3";
```

Refresh the admin page and you have your flag!
