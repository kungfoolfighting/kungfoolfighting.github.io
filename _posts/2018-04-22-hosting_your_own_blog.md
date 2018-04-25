---
title: hosting my own blog
Description: My thoughts on and solutions for hosting your own blog.
Author: Jan
Date: 2018/04/22
Robots: index,follow
layout: blog-item
---

## first steps in blogging
I am a man of little patience. It comes as no surprise then, that I have a hard time sticking with projects. So I accumulate a lot of sort-of finished, just started or running little projects, which fade out of memory. 
Not infrequently do I find myself wanting to go back to a previous project or needing to solve some issue I had already solved a while back.
This is always a nuisance.

I have long contemplated creating my own little blog that would let me recall all those nifty little tricks I had learned and also share with the world my own solutions or simply what I am up to.

So I decided to finally tackle the task of setting up my blog.

As with most things this is a task that can range in complexity from '_your average Joe is gonna be blogging away by the end of the day_' to '_I wish I had gotten that PhD after all_'. 

In my case it falls somewhere in between.


## my blogging needs
On the surface my needs are quite simple.
- I need a way to post blog entries without too much hassle.  I don't need a bunch of fancy stuff but a bit of text formatting would be nice and and an easy way to input it to the website.
- I also need to serve a tiny amount of page views.
- I don't really need any social features or commenting. Feedback via email will suffice. As such there is no need for a full LAMP stack with some server side processing.
- I want this to be secure. When I started out thinking about how to do this, I wasn't quite sure where the site would be hosted. That means I wanted to make absolutely sure that it would be very difficult to compromise the serving part of the blog. Otherwise other services or documents on the hosting infrastructure would be at risk. In my experience secure means low attack surface. Which in turn means that you want to have the smallest software stack you can find to serve your needs using the simplest software.
- I also needed this to be a cheap venture. I don't have any readership to speak of, and I don't earn any money with this, so more than very few (below 5) euros a month would be a deal-breaker.
- I also abhor ads on websites and can not with good conscience serve them on my own site.
Putting all these preferences together made it a bit more complicated.

## finding solutions
### hosting options
#### self hosting
The first question I tackled was:  _where do I want to host the site?_
I do have my own server, but it is located in my home network and as such it would expose my infrastructure to any attacker who managed the penetrate the hosting stack and gain control over the server. This is something I don't want. Obviously. There are ways to isolate the server from the rest of the network, but I am not an IT expert per se and wouldn't want to risk it. Also this would have severely limited the server's ability to communicate with all the other Home-Automation hardware.
#### online hosting
Another alternative would have been to host the site on some hosting service online. 
I looked into having a dedicated server or a virtual private server that shares hardware resources with other users. Being responsible for another entire linux machine carries the burden of having to update that server and configure it correctly and securely. This is a tedious task, especially considering that I am already on the hook for maintaining a slew of other machines in my own home.
Many hosters also offer packages where you get to use gui-created pre-built websites directly from the hosting menu.
Some also offer free website hosting with their CMS software but they display ads on your page.
#### amazon
Amazon offers a very simple way to create 'buckets' in their cloud that let you statically serve websites with very little maintanence work. The price for such a bucket instance is also almost free.

### hosting pick

So instead of trusting my ability to securely host a website in my own network. I decided to go for online hosting. This turned out to be a decision that really didn't reduce the number of questions I had. I was now left with figuring out the best prices for the tiny resource needs I had.
It turns out that for hosting your own LAMP stack you can't really get anything cheaper than 4 to 5 Euros. That would have been my upper limit but with the cost of a domain name this would surpass what I was willing to pay, so this was out.
The cheap and free versions were also too limiting for me or displayed ads which I wouldn't accept.

Finally the Amazon offer had pretty much all I needed. Simple bucket creation without maintenance overhead and an absolutely rock bottom price of an estimated 50ct./ month. The only issue: they expect you to give them your credit card info even if you haven't even accrued any cost yet. If someone were to abuse my site and create a ton of traffic, I would be on the hook for it. There did not seem to be an option to set hard limits for cost beforehand. I didn't want to expose myself to that.

So finally none of these hosting solutions quite hit the sweet spot. Luckily, however, I have a friend who already rents a VPS and agreed to put up my few pages on it.

### hosting software
Now that I had decided to entrust my website to my friends server, I was relatively limited in software decisions, since he had to agree to host anything that I would need.
#### serving
For security reasons I knew that I really only wanted to statically serve some html pages. This would massively limit the amount of software needed to serve my site, as serving static html is a stupidly simple thing to do. You could theoretically do this in bash, if you ever wanted to.
My friend runs a very lightweight webserver already and adding my few html pages was no problem.

#### building
Now that the issue of serving my static html was solved I had to figure out how to get from thoughts to html.
I knew that I really didn't want to have to format my text with html every time I wrote a blog post, so I either needed one of those fancy input forms that provide formatting tools and a webserver that converts all that into html or an entirely different formatting language.

Markdown just so happens to be a pretty easy, minimal and comfortable choice. 
So I thought about a tool to turn markdown, which I can use to write my texts, into html. Luckily I had just installed nextcloud into my local server.
#### nextcloud & pico_cms
Nextcloud is an open source software that lets you host a dropbox-like cloud solution on your own server. It comes with a ton of plugins. One of them was pico_cms, which, surprise, converts markdown into html. It also allows you to specify templates and themes to customize the look of your website and it also allows you to use rudimentary scripts for more complicated pages.
While pico_cmd can be hosted as a standalone application the availability of the nextcloud plugin was serendipitous because my friend already had a nextcloud instance installed on his server.
## result
So I will be using a markdown editor on a desktop or my phone to write a blog post. I will then move it into a nextcloud folder inside the nextcloud app. This will automatically sync it to my friends server, where pico_cms will parse it and create html pages using the templates and themes that I configured, which will be served by a simple webserver.

An added benefit of this method is that I can write blog posts even when I am offline because the nextcloud app will just sync when I eventually go back online.

## conclusion

The use of my friends server is a bit of a cop-out since it allowed me to circumvent some of the issues Inna way that not everyone could. It also doesn't address all the issues about security since the server exposes nextcloud to the internet which of course presents a pretty huge attack surface.
InI had been forced to solve these issues for myself I would have gone with Amazon and taken the payment risks. I would have used the nextcloud instance that runs on my homeserver to upload generated html to the Amazon cloud via plugin and I would have used an ssh tunnel to allow my smartphone to access the nextcloud instance in my home network from the internet.
