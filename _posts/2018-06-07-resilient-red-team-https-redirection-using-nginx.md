---
title: Resilient Red Team HTTPS Redirection Using Nginx
category: Security
tags: [red-team, hacking, security, tutorial]
---


On a typical red team assessment, a redirector is a crucial part of the infrastructure in use. A redirector is basically a box that sits out on the internet (usually in some type of cloud service provider's network) and forwards traffic for the red team so that the blue team can never see the IP address of the attacker's command and control (C2) server. Using redirectors, if a C2 channel gets burned, the red team can just spin up a new redirector, rather than rolling their entire infrastructure. When HTTPS gets involved, you have to deal with certificates, categorization, and a whole gambit of problems. This technique aims to solve that.

[Click here](#the-solution) to jump straight to the solution.


## The Old Way

On past assessments, we would usually have a setup like the one seen below. Our C2 would leave a compromised workstation, and call out to a redirector in the cloud. The redirector would be running [socat](http://www.dest-unreach.org/socat/doc/socat.html), forwarding traffic from port 443 on the redirector to port 443 on our C2 server.

{% include figure image_path="/assets/images/2018-08-07-resilient-red-team-https-redirectors/basic-redirector.png" %}

This is all completely fine, until you start to add in three things: proxies, free SSL certificates, and 120-day long assessments.

## The Perfect Storm

Our engagements are typically 90 days, plus an out-brief. That means that we potentially need to have infrastructure stood up and working for around 120 days just to be safe. When we're using HTTPS, this quickly becomes a problem. Let's start with the proxy problem.


### Proxies

Most organizations we assess have some type of web proxy in place. This means they won't be able to browse the whole web, but only pieces of the web that are marked safe by their particular proxy or web filter. One example of a filter is provided by [Palo Alto Networks](https://urlfiltering.paloaltonetworks.com/). With their appliance in place, only sites that are set up properly, and that are deemed by Palo Alto to fit within certain categories are allowed to be accessed over HTTP/s. This means two things for us, we need a valid SSL certificate, and we need to get our C2 domain categorized.

##### Categorization

We'll talk more on SSL certificates later, but for now, let's focus on what it takes to get a site categorized. Basically, it's just necessary to host content on the site that fits the categorization you are going for. Example, I mostly try to categorize as a Business site because it's vague, and generally marked as a safe category by customers. So, I just need to have some type of content that makes the site seem like it could be a legitimate business. See my [github](https://github.com/audrummer15/cat-sites) for some sites I've used to categorize domains in the past.

Categorization is hit or miss, sometimes an analyst will deny your categorization request, and sometimes they'll accept it. Just try a few times, and customize content to make it seem more legitimate.

Once a site would get categorized, we would take down the web server and start up our C2 server in it's place. However, on one assessment we noticed that our C2 channel got burned because a blue team analyst saw traffic flowing to our domain, visited it, and saw no legitimate content being served. It was an empty site, so they blocked it. This was one problem that led to the creation of the solution to come.

### Free SSL Certificates

Another thing proxies might disallow are HTTPS requests to sites with invalid SSL certificates. Let's be real, if you're truly performing a red team assessment, you need to have your ducks in a row. For us, we like to spend as little money as possible. So, we use [Let's Encrypt](https://letsencrypt.org/). These free certificates last for 3 months, and so do our assessments.

Seems like a perfect match at first, but we've found that we typically need to keep infrastructure up for about 120 days just to be safe. So at some point during the assessment we need to renew our SSL certificates, which means rebuilding our C2 profiles and restarting our C2 servers to load the new certificates. This isn't ideal, and can cause some issues with communications if not done carefully.


## The Solution

How do we host valid content, with free and valid SSL certificates, for 120 days without rolling our infrastructure then? Simple, we replace socat with Nginx, and we let Nginx do the heavy decision making for us!

What do I mean? Well, one very common use case for Nginx is to act as a [reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/). What this meant for our team is that we could host content on our redirector, and if someone tried to visit content that existed, Nginx would serve that content out. However, if the content requested did not exist, it was potentially a compromised workstation trying to communicate with us, so the request would be forwarded back to our C2 server. Let's take a look at an Nginx configuration that accomplishes this.

```nginx
server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;

        root /var/www/html;

        index index.html;

        server_name *.example.org;

        location / {
                try_files $uri $uri/ @c2;
        }

        location @c2 {
                proxy_pass https://c2-ip-address;
                proxy_redirect off;
                proxy_ssl_verify off;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

The main line to pay attention to is the `try_files` line, which tells nginx to try to serve files in that order. If the uri does not exist, then the request is handled by the `@c2` block. This block passes the traffic back to our c2 server, and is completely transparent to the client.


#### Updating SSL Certs

Another great thing about this, is that we only need to keep the SSL certificates updated within the context of Nginx. Our compromised workstation only establishes a secure session with our redirector running Nginx. Nginx then takes care of establishing a secure session with our C2 server that the workstation knows nothing about. As long as we define `proxy_ssl_verify off;` (thanks to [xan7r](https://github.com/xan7r) for this crucial find) in our Nginx config, if the C2 server is found to have an expired or self-signed SSL certificate, no one will care! Luckily, the `proxy_ssl_verify` option is defaulted to off for some reason, or we may have never noticed this solution.

With this information known, we can let [certbot](https://certbot.eff.org/) take care of keeping our SSL certificates up-to-date, and run our assessment for as long as we need.


## Coffee Break!

This solution really saved our team on our last assessment. Admittedly, there was a little luck involved in noticing how great this solution really was, but a huge shout out again to [xan7r](https://github.com/xan7r) for helping me walk through just how great this solution could be. If anyone is interested, I've began work on a script for setting this up that I will link below. Pull-requests are welcome, and I hope your teams are able to take this and run with it! Let me know your thoughts in the comments below!

##### Repositories
* [Site Categorization Templates](https://github.com/audrummer15/cat-sites)
* [C2 Server Setup Script](https://github.com/audrummer15/now-you-see-me)
