---
title: setting up my blog
Description: How I set up my blog using pico_cms as a nextcloud plugin and the troubles that paved my way to a working blog.
Author: Jan
Date: 2018/04/23
Robots: index,follow
layout: blog-item
---

## setting up my blog
In the previous blog entry I explained my decision making for how to host my blog. With this decision finally made it was time to get to work on actually setting up the web hosting software stack.

### nextcloud
The first step for me was to enable file syncronization between my devices and the web server. I decided to use [_Nextcloud_](https://nextcloud.com) to do this for three reasons. First of all it has clients for all my devices which allow really painless automatic sync of my documents to the server. Second, it already comes with a plugins that will help me with the other steps. Third, my friend already has a nextcloud instance running on his server.

For testing purposes I set up the nextcloud instance on my homeserver.
This is both to make the deployment onto the actual production server easier for my friend later on and also to have a test server for changes in the future. That way I will not have to make risky changes to the page on the live production installation. 

### docker
I use [_Docker_](https://www.docker.com) for most services running on my homeserver.
The installation of _Nextcloud_ was very easy using _Docker_. I already have a shell script that contains most of what I need to run a container. After I was done adjusting it for nextcloud, it looked like this:

	docker run --name nextcloud -v /opt/nextcloud/data:/config -v /media/data/nextcloud:/data  -d -p 443:443 -e TZ=Europe/Berlin -e PGID=981 -e PUID=981 linuxserver/nextcloud
	
There is not much to this: It...
- calls the docker run command
- gives the container a name
- passes in two host-side folders as volumes to store configuration and database data into
- defines the ports on which it will be reachable
- sets the proper timezone
- sets the user and group accounts that will execute the service
- points to the docker file in the _Docker Hub_ which to use.

After running this shell script I have a running _Nextcloud_ container.

After a minimal amount of configuration in _Nextcloud's_ web frontend I had the synchronization of files between my devices and the server up and running.

### pico

I wanted to write my blog articles in [_Markdown_](https://en.wikipedia.org/wiki/Markdown) since this makes formatting really easy and you can focus on the text and not the design while writing. So I needed a program that would convert my Markdown text to an html page.  
There are plenty of editing programs that could locally save my Markdown as HTML but they would only store the actual text as HTML. There would be no design around the pages. No navigation or page website headers.
This task requires a program that allows me to define the framework of the website and which then takes my Markdown files and converts them into HTML documents that embed my text into the website framework.
[_Pico_](http://picocms.org) does exactly that. It allows me to define the website design via templates and then it takes my Markdown files and generates an HTML page for every file I provide to it.

#### installation

To my great delight _Nextcloud_ has a plugin for _Pico_ called _pico\_cms_. Installing this plugin was as easy as finding it in the plugin list and clicking the install button. Creating your own website is also very easy. You find the settings page for _pico\_cms_ inside _Nextcloud_ and then you select the sub-path of the url for the website. This path will be relative to http://yoururl.tld/sites/. Then you select the folder inside the file repository of _Nextcloud_ where the Markdown documents will be uploaded to.  
If you point your browser to the correct url, you will now be presented with a default website that was automatically added by _pico\_cms_. It contains a document about how to use _pico\_cms_.

#### configuration
For some reason that I am not quite clear on, _Nextcloud_ has two settings areas and pico_cms has a settings page in both of them. When I went to the second one it showed me instructions on how to do a url rewrite with _Apache_ so that the quite long url can be shortened. The url is so long because the _pico\_cms_ output directories are hosted under the _Nextcloud_ web directories in their own sub-folder structure.
There were also inputs for choosing a custom theme and for adding custom templates.  
This is where the difficulties started.

I needed to add another theme, because the default theme really looks horrendous. The instructions for the input said to add a folder for your theme to this directory: `/data/appdata_oc9c9ativ1wi/cms_pico/themes/`. This directory is of course inside the docker container, so it is not available like that on the host machine.  
One way to go about this, is to grab a terminal in the docker container and add folders and files there, but this is not the correct way to use _Docker_. What you want to do is have a folder on the host for all volatile but persistant data. That means all data that you would not want to vanish whenever restart _Nextcloud_ and which you also modify. Then you mount that folder into the container, so that the container can write to and read from it.
If you take a look at the ´docker run´ statement that I provided further up, you can see that I do that with two different directories. One for the configuration stuff and one for the data that is uploaded to the server.  
Due to not paying attention, I only looked inside the folder that contains the configurations. The reason is that the for all my _Docker_ containers I store the mounted volumes containing the configuration files inside the folders that contain the run script somewhere on my system drive. If, however, a program creates significant amounts of data, like you would expect a cloud service to do, I create a folder on my storage partitions. I had forgotten, that I had done this and so I wasn't able to find the folder that _Nextcloud_ told me to add my theme folder to. But I searched and found some 'theme' folder deep down in the folder tree. I hoped that this was down to me using a docker container and there being some confusion for _Nextcloud_. Sadly, adding a new folder here, did not add another theme that I could select inside the settings for _pico\_cms_. I found other theme folders in other locations, but they also wouldn't yield the desired result.  
After a while I remembered that there was a second volume that contained the stored data. It also made sense that the theme files would be inside the data volume, since they are not part of the _Nextcloud_ configuration, but rather part of the data which can be stored in the database. And lo and behold, there was the specific folder I was looking for. Adding a new folder made a new theme show up in the settings.

#### making a theme

With the folder created I could now set on creating my own theme.  
I am capable of writing my own website. In fact, I used to earn a bit of money doing this and writing back-end website code during university, but my knowledge is extremely outdated and my methods very slow. That is why I decided to have a look at the third-party themes that already exist. Many of them are way more flashy than what I like and many use complex frameworks that I want to steer away from, because they add a bunch of complexity that makes it hard for me to debug my site. I really wanted to have total control over my website. A few of the themes I found were quite nice. Very simplistic and focused on the content. But only one of them had blogging functionality.  
The thing about blogging is that you create many pages. You don't want every one of those pages to appear in the main menu, because that would seriously clutter it and make impossible to find anything. So you need a dedicated page that aggregates the blog pages as links.  
One could simply write a Markdown document that contains links to every blog post and then update this page whenever you add a new blog post. This is tedious however and can be solved more elegantly. _Pico\_cms_ exposes variables that you can use inside your Markdown file. One such variable is `pages`. This variable gives you access to all pages you have in your website folder hierarchy in a flattened list.  
By iterating over all pages and filtering out the ones that are blogs entries, you can automatically create an always up-to-date list of your blog entries. By having the blog entries in a `/blog`subfolder and accessing the `page.id` variable that contains the paths you can filter the pages whose id begins with "blog/". You do this for both the main menu, where you omit these pages, and for the blog page where you only display links to those pages.

One theme that wasn't quite so visually appealing contained this method and so I learned from that and took visual inspiration from the [_simpleTwo_](https://github.com/sonst-was/simpleTwo) theme that I found.
This required me to split up the original theme into individual parts. Originally the theme was just one index.html and a css file. _Pico_ uses the index.html file whenever a Markdown file does not explicitly state another template file to use. This index.html file uses the variables provided by _Pico_ to embed the text of the Markdown page that is being rendered to HTML. Using multiple different templates (index, blog, and blog-item) means that we want to reuse a lot of code. I did this by splitting up the index.html file into header.twig, index.twig, and footer,twig. The `.twig` ending is used for [_Twig_](https://twig.symfony.com/) files. Twig is a templating engine based on _PHP_ that _Pico_ uses. The index.twig file includes both the `header.twig` and `foother.twig` files.

In the end my blog.twig (the most interesting file) looked like this:

{% highlight HTML %}
{% raw %}
	{{ include( 'header.twig' ) }}
	<div id="content">
		<div class="inner">
			{{ content }}

			{% set posts_per_page = 10 %}
			{% set cnt = 0 %}
			{% set p = ( TwigGetUrl and TwigGetUrl['p'] ) ? TwigGetUrl['p'] : 0 %}
			{% set first = p * posts_per_page %}
			{% set last = first + posts_per_page %}
			{% for page in pages | sort_by( 'time' ) | reverse %}
				{% if page.id starts with "blog/" %}
					{% set cnt = cnt + 1 %}
					{% if cnt > first and cnt <= last %}
						<section class="post">
							<header class="post-header">
								<h2 class="post-title"><a href="{{ page.url }}">{{ page.title }}</a></h2><div style="float:right">{{ page.date | date( 'j' ) }} {{ page.date | date( 'M' ) }} {{ page.date | date( 'Y' ) }}</div>
								<p class="post-meta">
									Author: <a href="#" class="post-author">{{ page.author }}</a> &ndash;
									{% set tags = page.meta['tags'] | split( ',' ) %}
									{% for tag in tags %}
										{% if not tag is empty %}
											<a href="#{{ tag }}" class="post-tag">{{ tag }}</a>
										{% endif %}
									{% endfor %}
								</p>
							</header>
							<div class="post-description">
								<p> {{ page.description }} </p>
							</div>
						</section>
					{% endif %}
				{% endif %}
			{% endfor %}
			{% if cnt > last %}
				<a href="{{ 'index' | link }}?p={{ p + 1 }}">more posts</a>
			{% endif %}
		</div>
	</div>
	{{ include( 'footer.twig' ) }}
{% endraw %}
{% endhighlight %}

I also didn't like how wide the text blocks would become on displays with higher resolution, so I limited it to 800 pixels.

Then I tried selected my new theme from the settings and excitedly navigated to my page in my browser, aaaaaaand... ugliness. For some reason there was no formatting happening. The website elements were all there but without any formatting.
After checking where the link to the stylesheet pointed to, I noticed that the url was not available on my server.  
The reason for this dawned on me after a while: The entire theme directory is accessible to the parsing process that creates the HTML pages, but the HTML pages themselves are hosted in the www directly of _Nextcloud_ because they have to be served by the _Apache_ service that also hosts my _Nextcloud_ instance.
The theme files are not located in that directory and can therefore not be served to the public. Copying over the stylesheets to the website directory did the trick and everything looked nice and neat after a bit of bugfixing. I am not sure why this step is not mentioned anywhere in the documentation of the _pico\_cms_ plugin, since it seems like this is a necessary step. I would have assumed that these files are automatically copied during the parsing step, but this is not what happens.

After all this the next step was to make sure that the main navigation item for my blog would also stay highlighted whenever I was reading a blog entry and not just the main blog overview.

I achieved this by making sure the item is highlighted when the currently rendering nav item is the blog item and the currently being rendered page is a blog page.  
This is the code:

{% highlight HTML %}
{% raw %}
	{% for page in pages if page.title %}
		{% if not (page.id starts with "blog/") %}
			<li{% if page.id == current_page.id %} id="active"{% endif %}>
				<a href="{{ page.url }}">{{ page.title }} {% if page.id == current_page.id or ((current_page.id starts with "blog/") and page.id == "sub/blog") %} &laquo; {% endif %}</a>
			</li>
		{% endif %}
	{% endfor %}
{% endraw %}
{% endhighlight %}

It filters out the blog pages inside the for loop and then renders a link for every other page on the server. It displays this link as the active link if the link points to the page that is currently being displayed or if the link is the blog link and we are on any blog-related page.

### conclusion

This is by far not the easiest way to setup a website, but it was much simpler than setting up an entire [_LAMP_](https://en.wikipedia.org/wiki/LAMP_(software_bundle)) stack.  
Some follow-up changes will surely occur but for now the biggest task left is moving the page onto the production server.
