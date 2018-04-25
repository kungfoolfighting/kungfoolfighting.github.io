---
title: problems with pico and nextcloud
Description: Issues with running your website with the pico_cms plugin for nextcloud.
Author: Jan
Date: 2018/04/24
Robots: index,follow
layout: blog-item
---

## {{ page.title }}
So I was counting my chickens too early when I declared the test setup of my blog to be finished in the last blog. As it turns out the blog is hosted by the same webserver as [_Nextcloud_](http://nextcloud.com). _Pico\_cms_ just puts your blog in a subdirectory or the web root. This means that my blog is governed by the same [_Content Security Policy_](https://en.wikipedia.org/wiki/Content_Security_Policy) (CSP) as _Nextcloud_. This CSP disallows any external _JavaScript_ sources. The blog theme that I built does however contain a bit of _JavaScript_ for a small effect with the header of the blog. As soon as you scroll down, the header gets a bit smaller to give you more screen space to read. This bit of code requires both inline code execution and loading of a local _JavaScript_ file and the _jquery_ script. Both of these things are disallowed by the _CSP_ set by _Nextcloud_.

### using meta tag
My first attempt to solve this issue was to add a [_meta header tag_](https://www.w3schools.com/tags/tag_meta.asp) in the header template of my blog. These meta tags allow me, among other things, to set my own _CSP_.
The tag that I came up with is:

	<meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline' 'unsafe-eval'; object-src 'self'">
	
This should allow "unsafe" scripts to be loaded and executed. It should also allow my inline-scripts to be executed. Those are the ones that you write in your html code like such:

	<script>
		my JavaScript code here
	</script>
	
But this achieved a whole lot of nothing.

### rewriting the url
My next thought was, that the issues I get with not being allowed to load the local javascript file might stem from the blog url not being recognized as the same [_top level domain_](https://en.wikipedia.org/wiki/Top-level_domain) (TLD). I host the test version of my blog on my homeserver in my LAN. That means that I don't use a TLD to reach it, but rather just the name of the machine.
But even accessing my server via http://homeserver.local/my/blog/url did not solve this issue. I then thought it might have to do with the fact that the _pico\_cms_ websites are served by passing the [uri](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) to the index.php of _Nextcloud_. The resulting urls look like this for example:

	https://homeserver/index.php/apps/cms_pico/pico/techblog/blog/setting_up_my_blog

 But the uri for the javascript file excludes the index.php part of the url:
 
	https://homeserver/apps/cms_pico/pico/themes/simpletwo/css/style.css
	
By using proxy_pass in the webserver ([_NGINX_](https://www.nginx.com/)) settings, I managed to make my website accessible via 

	https://homeserver/sites/techblog/index
	
Doing this required me to go to the configuration volume that I pass into the container for _Nextcloud_ and edit the nginx/site-configs/default file. Here I created the following block:

    location /sites/ {
        rewrite /sites/(.*) /index.php/apps/cms_pico/pico/$1 break;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_pass https://homeserver/;
    }

### removing csp header
Sadly, this changed nothing.
Looking at the resulting _HTML_ code, I could see that the resources were still being loaded from the unmodified url. This is because my header template uses a keyword that is provided by _Pico_ to get the current server url. I changed this to the absolute url that I know, but it still didnt work. I was still being nagged by the CSP.
After a while of googling I figured out that there is a command to suppress specific _HTML_ header entries. I added the following directive to the site location in my _NGINX_ config:

	proxy_hide_header Content-Security-Policy;

This prevented the delivery of this header to the client and the client could happily load any javascript it wants.
This only worked when I navigated to my blog via the /sites/techblog/ url, though. The problem was, that as soon as I clicked a link on my blog, the javascript would be blocked again. The reason is that the links on my blog are being automatically generated from the pages that exist in the content directory. The header.twig uses the `page.url` variable to get the url for a page and create a link. Sadly this variable is filled by _Pico_ with the url that has not been modified by _NGINX_ url rewrite. This means it's the old url that doesn't have `proxy_hide_header` directive, so my script stopped working again.

I tried adding the `proxy_hide_header` directive to the entire server definition in the _NGINX_ config, but that didnt do anything more than I already had. After a bit of pondering I understood that this directive will only work if I am actually proxying to another server, like I am doing in the 'site' location. All the other locations were not being proxied.

So I tried to figure out how to remove header entries without proxying, and the answer was to use `add_header Content-Security-Policy ""` According to the documentation this would drop this header entry.

Nope. Didn't work. Sigh...

Then I tried adding a proxy directive to the other (_Nextcloud_ specific) locations in the _NGINX_ config so that the proxy_hide_header directive would work, but that prevented the server from even starting.
So in the end I made sure that all urls inside my blog use the /sites/ url. I did that by changing the `{{ page.url }}` variable in my header.twig to `/sites/techblog/{{ page.id }}`. This did the trick. Now I was able to use JavaScript on all pages, because all requests to the server were being proxied and then the CSP header was being deleted.

### conclusion
So now I have a running test environment for my blog, but it is not a very easy process to get running and I am anxious to see how the deployment to the production server will go. I am actually not quite sure how great it is to have my blog run via _Nextcloud_ anymore, since every request seems to be passed to index.php, even if it should just be going straight to my blog. Being dependent on the _Nextcloud_ security settings also seems unnecessary. _Pico_ seems to not expose the parsed HTML for me to host by another webserver though. At least I could not find it in any of the mounted volumes.
