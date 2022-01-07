# Post

![cover-image](./cover-image.jpg)

This week I finished Troy Hunts [What Every Developer Must Know About HTTPS](https://www.pluralsight.com/courses/https-every-developer-must-know) course on Pluralsight. It is a fantastic course that I highly recommend anyone to look at if you want to learn more about HTTPS. I try to use HTTPS with all my HTTP requests. I am taking the precaution not to visit a website if I am unable to use HTTPS. You may ask, what benefits do I get as a visitor/consumer when using HTTPS? According to Troy Hunt, upsides to using HTTPS as a visitor/consumer are

- Confidentiality - You can be confident the data sent from your application is secure.
- Integrity - Data you get back for your request comes from the application you expect(i.e. No one can alter the response).
- Authenticity - You are visiting the application you expected to see.

In the [What Every Developer Must Know About HTTPS](https://www.pluralsight.com/courses/https-every-developer-must-know) course Troy Hunt talks a lot more about these aspects of HTTPS and much more. I would encourage you to take a look if you would like to learn more.

Hopefully, you agree that you want to use HTTPS as a visitor/consumer when browsing the web. What this means, If you have applications that connect to the internet, you will also need to use HTTPS. This is where Cloudflare comes in. Below is from the [Cloudflare](https://www.cloudflare.com/en-au/what-is-cloudflare/) site.

> Cloudflare is a global network designed to make everything you connect to the Internet secure, private, fast, and reliable.

![requests-via-cloudflare.png](./requests-via-cloudflare.png)

Cloudflare sits between the consumer and your web application as a reverse proxy. You can go to **[What is Cloudflare?](https://www.cloudflare.com/en-au/what-is-cloudflare/)** page to learn more about why you should use Cloudflare. Assuming you decided to go with Cloudflare, let me tell you the steps you would need to take to set up Cloudflare to forward requests to your application via Cloudflare.

## Steps

1. Create a Cloudflare account
2. Add a site. (Websites section in navigation bar → Add a site)
3. Select the [Cloudflare plan](https://www.cloudflare.com/en-au/plans/). I am using the free plan, which is sufficient for my needs.
4. Update name servers
   This is to use the Cloudflare name service instead of your current name servers(After this, Cloudflare will be in charge of routing the traffic to your application). For this, you need to go to your domain registrar(e.g. Domain.com, GoDaddy) and update the name servers to use Cloudflare name servers. I am using Google domains. If you are using Google domains, you can do this by going to the DNS section in the navigation bar and selecting Custom name servers.
   Please note, DNS propagation takes time. If you would like to check the status of the DNS propagation, you could do this by visiting [whatsmydns](https://www.whatsmydns.net/) site. While DNS propagates, some users will go to your naked site while others will go via Cloudflare. You could check what your DNS is resolving after this step by using `nslookup` command (e.g. `nslookup your-site-name`)
5. Setup SSL/TLS
   It might take some time for Cloudflare to set up SSL/TLS. As a precaution, wait for a green tick before updating the name servers if you have SSL/TLS already in your application.

![cloudflare-ssl-tls-setup](./cloudflare-ssl-tls-setup.png)

The above steps are all you need to set up Cloudflare. But just because we have set up to use HTTPS does not mean users cannot use HTTP. What we could do is set up Cloudflare so that any HTTP requests coming to your application will result in a `301 Moved Permanently` response when it reaches Cloudflare. If the user made the request using a browser, the browser would automatically send another request to the new location in the `location` header in the response. To redirect any HTTP requests, go to the **_Rules_** section in the navigation bar, then click the **_Create Page Rule_** button. Then you will need to give a URL pattern. Because I wanted all the requests to any resources to use HTTPS, I have set mine to `/*`.

Every time a user uses HTTP to connect to our application Cloudflare will redirect them to use HTTPS. But what this means, users still need to make that first request for Cloudflare to redirect them. This does not stop a user from visiting your application multiple times using HTTP. It would be better if we could tell the browser the first time; please, from now on, use HTTPS for any requests you make for this application. We could do precisely that using [HSTS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security). To set up HSTS, go to the SSL/TLS section of the navigation bar and select **_Edge Certificates_** from the drop-down list. Then go to **_HTTP Strict Transport Security (HSTS)_** and click Change HSTS settings. After that, read and accept the **_Acknowledgement_** from Cloudflare. After that, you will get the configuration for HSTS. You could read more about what those configurations mean by visiting the [Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) page.

Please note if you set HSTS and for some reason, if you decide to use HTTP again, request to your application using HTTP will break until the `max-age` expires.

I hope what I learned from Troy Hunts [What Every Developer Must Know About HTTPS](https://www.pluralsight.com/courses/https-every-developer-must-know) and [Getting Started with Cloudflare Security](https://www.pluralsight.com/courses/cloudflare-security-getting-started) courses are as valuable for you as it was for me. Thanks for reading.

## Credits

- [What Every Developer Must Know About HTTPS](https://www.pluralsight.com/courses/https-every-developer-must-know)
- [Getting Started with Cloudflare Security](https://www.pluralsight.com/courses/cloudflare-security-getting-started)

## Resources

- [SSL Server Test](https://www.ssllabs.com/ssltest/)
- [Cloudflare - The Web Performance & Security Company](https://www.cloudflare.com/en-au/)
- [How to make your Hashnode blog load faster?](https://harshitbudhraja.com/how-to-make-your-hashnode-blog-load-faster)
- [DNS Checker - DNS Propagation Check & DNS Lookup](https://www.whatsmydns.net/)
- [301 Moved Permanently - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/301)
  [Strict-Transport-Security - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
