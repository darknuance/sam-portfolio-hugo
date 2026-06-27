+++
authors = ["Samuel Rodrigues (dark_nuance)"]
title = "Portfolio Project: Website infrastructure"
date = "2026-06-27"
description = "How I created this website"
tags = [
    "sysadmin",
    "nginx",
    "cloudflare",
    "ubuntu",
	"hugo",
]
+++



The first project to be featured in this site is well... the site itself.
I had originally set out to configure a webpage from scratch in order to learn more about linux production workflows.
So this is a hands-on project to force me to get stuck and troubleshoot things.
So here are some of the problems I encountered and how I solved them:

1. Where to host the website

	I have another project in which I repurpose my old gaming laptop into a homelab, but that's not what's being used here.
	I knew that for a public facing website, I wanted something isolated and more reliable than a home device (which depends on consumer electricity and internet connection uptime),
	so I decided to go with a cloud VPS.
	
	At first I had decided to go with Hetzer due to their price-to-performance, but setting up an account was a nightmare, and their system simply did not want to work
	in verifying my identity. I settled with DigitalOcean, since they have a cheap 4$ plan (more than enough for a site), and a free 5$ credit for new accounts.
	Overall it works well and their panel is easy to operate.
2. Securing my host

	Once I had a VPS, it was the time to choose the server. I went with Ubuntu 24.04 LTS, as 26.04 hadn't received the first point release yet, and for my personal needs,
	Ubuntu seemed like the logical choice.
	
	The first thing I did was install updates, and change the hostname to include my domain.
	Then, I added a non-root user with `adduser` , and gave it sudo permissions with `usermod -aG sudo`.
	Lastly, I tweaked `/etc/ssh/sshd_config` to disable root login, as well as force SSH key-based auth.
3. Adding Cloudflare DNS

	Once the server was stable and secured, I added my domain samuelrodrigues.com to my cloudflare account in order to setup DNS records to my server's IP.
	Cloudflare was the obvious choice here because their free tier is remarkably generous, with automatic HTTPS and great performance.
	I created an A record for my server's IPv4 address, an AAAA record for the IPv6, and a CNAME for www.samuelrodrigues.com, enabling proxying for all three.
4. Making Nginx work with SSL

	This was one of the parts where I had more trouble. I was able to eventually figure out through a [DigitalOcean guide post](https://www.digitalocean.com/community/tutorials/how-to-host-a-website-using-cloudflare-and-nginx-on-ubuntu-22-04) and Cloudflare's [documentation](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/).
	Basically, an Nginx config with the domain as the name, has to be created inside `/etc/nginx/sites-available/`, and have a syslink to `/etc/nginx/sites-enabled/`.
	Inside, the syntax will look something like this:
	```
	server {
    listen 80;
    listen [::]:80;
    server_name your_domain www.your_domain;
    return 301 https://$server_name$request_uri;
	}

	server {

    	# SSL configuration
		# path for the cloudflare origin certificate, and the private key 

	    listen 443 ssl http2;
	    listen [::]:443 ssl http2;
	    ssl_certificate         /etc/ssl/cert.pem; 
	    ssl_certificate_key     /etc/ssl/key.pem;

		# additional path for TLS client auth
		# helps prevent connections directly to nginx that circumvent cloudflare
		ssl_client_certificate /etc/ssl/cloudflare.crt;
    	ssl_verify_client on;

	    server_name your_domain www.your_domain;

		# path of the main html file to serve
	    root /var/www/your_domain/html;
	    index index.html;

	    location / {
            	try_files $uri $uri/ =404;
    	}
	}
	```
5. Creating the site content

	So then my basic infrastructure was up and running, secured, and properly configured. I already had a vibe-coded HTML page as a placeholder, but I knew I had to do better.
	My goal was to avoid spending time learning web stack dev, since that's not within my scope. At the same time, I wanted something efficient and secure; so I went with [Hugo](https://gohugo.io/).

	Hugo is amazing for making a website: you download it, create a directory, run `git init`, then add a template (called "themes") as a git submodule. From there, all you have to do is edit `hugo.toml`
	and the markdown files with your information, and your posts/projects. This also allowed me to understand more about Git and how to maintain a repository, crucial for my website.