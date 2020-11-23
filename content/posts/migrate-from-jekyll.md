---
author:
  name: "Hugo Authors"
date: 2014-03-10
linktitle: Migrating from Jekyll
title: Migrate to Hugo from Jekyll
type:
- post
- posts
weight: 10
series:
- Hugo 101
aliases:
- /blog/migrate-from-jekyll/
---

## Move static content to `static`
Jekyll has a rule that any directory not starting with `_` will be copied as-is to the `_site` output. Hugo keeps all static content under `static`. You should therefore move it all there.
With Jekyll, something that looked like

    ▾ <root>/
        ▾ images/
            logo.png

should become

    ▾ <root>/
        ▾ static/
            ▾ images/
                logo.png

Additionally, you'll want any files that should reside at the root (such as `CNAME`) to be moved to `static`.

## Create your Hugo configuration file
Hugo can read your configuration as JSON, YAML or TOML. Hugo supports parameters custom configuration too. Refer to the [Hugo configuration documentation](/overview/configuration/) for details.

## Set your configuration publish folder to `_site`
The default is for Jekyll to publish to `_site` and for Hugo to publish to `public`. If, like me, you have [`_site` mapped to a git submodule on the `gh-pages` branch](http://blog.blindgaenger.net/generate_github_pages_in_a_submodule.html), you'll want to do one of two alternatives:

1. Change your submodule to point to map `gh-pages` to public instead of `_site` (recommended).

        git submodule deinit _site
        git rm _site
        git submodule add -b gh-pages git@github.com:your-username/your-repo.git public

2. Or, change the Hugo configuration to use `_site` instead of `public`.

        {
            ..
            "publishdir": "_site",
            ..
        }

## Convert Jekyll templates to Hugo templates
That's the bulk of the work right here. The documentation is your friend. You should refer to [Jekyll's template documentation](http://jekyllrb.com/docs/templates/) if you need to refresh your memory on how you built your blog and [Hugo's template](/layout/templates/) to learn Hugo's way.

As a single reference data point, converting my templates for [heyitsalex.net](http://heyitsalex.net/) took me no more than a few hours.

## Convert Jekyll plugins to Hugo shortcodes
Jekyll has [plugins](http://jekyllrb.com/docs/plugins/); Hugo has [shortcodes](/doc/shortcodes/). It's fairly trivial to do a port.

### Implementation
As an example, I was using a custom [`image_tag`](https://github.com/alexandre-normand/alexandre-normand/blob/74bb12036a71334fdb7dba84e073382fc06908ec/_plugins/image_tag.rb) plugin to generate figures with caption when running Jekyll. As I read about shortcodes, I found Hugo had a nice built-in shortcode that does exactly the same thing.

Jekyll's plugin:

    module Jekyll
      class ImageTag < Liquid::Tag
        @url = nil
        @caption = nil
        @class = nil
        @link = nil
        // Patterns
        IMAGE_URL_WITH_CLASS_AND_CAPTION =
        IMAGE_URL_WITH_CLASS_AND_CAPTION_AND_LINK = /(\w+)(\s+)((https?:\/\/|\/)(\S+))(\s+)"(.*?)"(\s+)->((https?:\/\/|\/)(\S+))(\s*)/i
        IMAGE_URL_WITH_CAPTION = /((https?:\/\/|\/)(\S+))(\s+)"(.*?)"/i
        IMAGE_URL_WITH_CLASS = /(\w+)(\s+)((https?:\/\/|\/)(\S+))/i
        IMAGE_URL = /((https?:\/\/|\/)(\S+))/i
        def initialize(tag_name, markup, tokens)
          super
          if markup =~ IMAGE_URL_WITH_CLASS_AND_CAPTION_AND_LINK
            @class   = $1
            @url     = $3
            @caption = $7
            @link = $9
          elsif markup =~ IMAGE_URL_WITH_CLASS_AND_CAPTION
            @class   = $1
            @url     = $3
            @caption = $7
          elsif markup =~ IMAGE_URL_WITH_CAPTION
            @url     = $1
            @caption = $5
          elsif markup =~ IMAGE_URL_WITH_CLASS
            @class = $1
            @url   = $3
          elsif markup =~ IMAGE_URL
            @url = $1
          end
        end
        def render(context)
          if @class
            source = "<figure class='#{@class}'>"
          else
            source = "<figure>"
          end
          if @link
            source += "<a href=\"#{@link}\">"
          end
          source += "<img src=\"#{@url}\">"
          if @link
            source += "</a>"
          end
          source += "<figcaption>#{@caption}</figcaption>" if @caption
          source += "</figure>"
          source
        end
      end
    end
    Liquid::Template.register_tag('image', Jekyll::ImageTag)

is written as this Hugo shortcode:
``` css
@import url(https://fonts.googleapis.com/css?family=Questrial);
@import url(https://fonts.googleapis.com/css?family=Arvo);

@font-face {
	src: url(https://lea.verou.me/logo.otf);
	font-family: 'LeaVerou';
}

/*
 Shared styles
 */

section h1,
#features li strong,
header h2,
footer p {
	font: 100% Rockwell, Arvo, serif;
}

/*
 Styles
 */

* {
	margin: 0;
	padding: 0;
}

body {
	font: 100%/1.5 Questrial, sans-serif;
	tab-size: 4;
	hyphens: auto;
}

a {
	color: inherit;
}

section h1 {
	font-size: 250%;
}

	section section h1 {
		font-size: 150%;
	}

	section h1 code {
		font-style: normal;
	}

	section h1 > a,
	section h2[id] > a {
		text-decoration: none;
	}

	section h1 > a:before,
	section h2[id] > a:before {
		content: '§';
		position: absolute;
		padding: 0 .2em;
		margin-left: -1em;
		border-radius: .2em;
		color: silver;
		text-shadow: 0 1px white;
	}

	section h1 > a:hover:before,
	section h2[id] > a:hover:before {
		color: black;
		background: #f1ad26;
	}

p {
	margin: 1em 0;
}

section h1,
h2 {
	margin: 1em 0 .3em;
}

h2 {
	font-weight: normal;
}

dt {
	margin: 1em 0 0 0;
	font-size: 130%;
}

	dt:after {
		content: ':';
	}

dd {
	margin-left: 2em;
}

strong {
	font-weight: bold;
}

code, pre {
	font-family: Consolas, Monaco, 'Andale Mono', 'Lucida Console', monospace;
	hyphens: none;
}

pre {
	max-height: 30em;
	overflow: auto;
}

pre > code.highlight {
	outline: .4em solid red;
	outline-offset: .4em;
}

header,
body > section {
	display: block;
	max-width: 900px;
	margin: auto;
}

header, footer {
	position: relative;
	padding: 30px -webkit-calc(50% - 450px); /* Workaround for bug */
	padding: 30px calc(50% - 450px);
	color: white;
	text-shadow: 0 -1px 2px black;
	background: url(img/spectrum.png) fixed;
}

header:before,
footer:before {
	content: '';
	position: absolute;
	bottom: 0; left: 0; right: 0;
	height: 20px;
	background-size: 20px 40px;
	background-repeat: repeat-x;
	background-image: linear-gradient(45deg, transparent 34%, white 34%, white 66%, transparent 66%),
	                  linear-gradient(135deg, transparent 34%, white 34%, white 66%, transparent 66%);
}

header {

}

	header .intro,
	html.simple header {
		overflow: hidden;
	}

	header h1 {
		float: left;
		margin-right: 30px;
		color: #7fab14;
		text-align: center;
		font-size: 140%;
		text-transform: uppercase;
		letter-spacing: .25em;
	}

	header h2 {
		margin-top: .5em;
		color: #f1ad26;
	}

		header h1 a {
			text-decoration: none;
		}

		header img {
			display: block;
			width: 150px;
			height: 128px;
			margin-bottom: .3em;
			border: 0;
		}

	header h2 {
		font-size: 300%;
	}

	header .intro p {
		margin: 0;
		font: 150%/1.4 Questrial, sans-serif;
		font-size: 150%;
	}

	#features {
		width: 66em;
		margin-top: 2em;
		font-size: 80%;
	}

		#features li {
			margin: 0 0 2em 0;
			list-style: none;
			display: inline-block;
			width: 27em;
			vertical-align: top;
		}

		#features li:nth-child(odd) {
			margin-right: 5em;
		}

			#features li:before {
				content: '✓';
				float: left;
				margin-left: -.8em;
				color: #7fab14;
				font-size: 400%;
				line-height: 1;
			}

				#features li strong {
					display: block;
					margin-bottom: .1em;
					font-size: 200%;
				}

	header .download-button {
		float: right;
		margin: 0 0 .5em .5em;
	}

	#theme {
		position: relative;
		z-index: 1;
		float: right;
		margin-right: -1em;
		text-align: center;
		text-transform: uppercase;
		letter-spacing: .2em;
	}

		#theme > p {
			position: absolute;
			left: 100%;
			transform: translateX(50%) rotate(90deg) ;
			transform-origin: top left;
			font-size: 130%;
		}

		#theme > label {
			position: relative;
			display: flex;
			justify-content: center;
			align-items: center;
			width: 8.5em;
			height: 8.5em;
			line-height: 1em;
			border-radius: 50%;
			background: hsla(0,0%,100%,.5);
			cursor: pointer;
			font-size: 90%;
			padding: 0;
		}

		#theme > label:before {
			content: '';
			position: absolute;
			top: 0; right: 0; bottom: 0; left: 0;
			z-index: -1;
			border-radius: inherit;
			background: url(img/spectrum.png) fixed;
		}

		#theme > label:nth-of-type(n+2) {
			margin-top: -2.5em;
		}

		#theme > input:not(:checked) + label:hover {
			background: hsla(77, 80%, 60%, .5);
		}

		#theme > input {
			position: absolute;
			clip: rect(0,0,0,0);
		}

		#theme > input:checked + label {
			background: #7fab14;
		}

footer {
	margin-top: 2em;
	background-position: bottom;
	color: white;
	text-shadow: 0 -1px 2px black;
}

	footer:before {
		bottom: auto;
		top: 0;
		background-position: bottom;
	}

	footer p {
		font-size: 150%;
	}

	footer ul {
		column-count: 3;
	}

.download-button {
	display: block;
	padding: .2em .8em .1em;
	border: 1px solid rgba(0,0,0,0.5);
	border-radius: 10px;
	background: #39a1cf;
	box-shadow: 0 2px 10px black,
	   inset 0 1px hsla(0,0%,100%,.3),
	   inset 0 .4em hsla(0,0%,100%,.2),
	   inset 0 10px 20px hsla(0,0%,100%,.25),
	   inset 0 -15px 30px rgba(0,0,0,0.3);
	color: white;
	text-shadow: 0 -1px 2px black;
	text-align: center;
	font-size: 250%;
	line-height: 1.5;
	text-transform: uppercase;
	text-decoration: none;
	hyphens: manual;
}

.download-button:hover {
	background-color: #7fab14;
}

.download-button:active {
	box-shadow: inset 0 2px 8px rgba(0,0,0,.8);
}

#toc {
	position: fixed;
	bottom: 15px;
	max-width: calc(50% - 450px - 40px);
	font-size: 80%;
    z-index: 999;
    background: white;
    color: rgba(0,0,0,.5);
    padding: 0 10px 10px;
	border-radius: 0 3px 3px 0;
	box-sizing: border-box;
}

@media (max-width: 1200px) {
	#toc {
		 display: none;
	}
}

#toc:hover {
    color: rgba(0,0,0,1);
}

	#toc h1 {
		font-size: 180%;
		margin-top: .75rem;
	}

	#toc li {
		list-style: none;
	}

#logo:before {
	content: '☠';
	float: right;
	font: 100px/1.6 LeaVerou;
}

.used-by-logos {
	overflow: hidden;
}
	.used-by-logos > a {
		float: left;
		width: 33.33%;
		height: 100px;
		text-align: center;
		background: #F5F2F0;
		box-sizing: border-box;
		border: 5px solid white;
		position: relative;
	}
		.used-by-logos > a > img {
			max-height: 100%;
			max-width: 100%;
			position: absolute;
			top: 50%;
			left: 50%;
			transform: translate(-50%, -50%);
		}

label a.owner {
	margin: 0 .5em;
}

label a.owner:not(:hover) {
	text-decoration: none;
	color: #aaa;
}

#languages-list ul {
	column-count: 3;
}
	#languages-list li {
		padding: .2em;
	}
	#languages-list li[data-id="javascript"] {
		border-bottom: 1px solid #aaa;
		padding-bottom: 1em;
		margin-bottom: 1em;
		margin-right: 1em;
	}

ul.plugin-list {
	column-count: 2;
}
	ul.plugin-list > li {
		break-inside: avoid;
		page-break-inside: avoid;
	}
	ul.plugin-list > li > a {
		font-size: 110%;
	}
	ul.plugin-list > li > div {
		margin-bottom: .5em;
	}

/*
 * Fix for Toolbar's overflow issue
 */

div.code-toolbar {
	display: block;
	overflow: auto;
}

```
### Usage
I simply changed:

    {% image full http://farm5.staticflickr.com/4136/4829260124_57712e570a_o_d.jpg "One of my favorite touristy-type photos. I secretly waited for the good light while we were "having fun" and took this. Only regret: a stupid pole in the top-left corner of the frame I had to clumsily get rid of at post-processing." ->http://www.flickr.com/photos/alexnormand/4829260124/in/set-72157624547713078/ %}

to this (this example uses a slightly extended version named `fig`, different than the built-in `figure`):

    {{%/* fig class="full" src="http://farm5.staticflickr.com/4136/4829260124_57712e570a_o_d.jpg" title="One of my favorite touristy-type photos. I secretly waited for the good light while we were having fun and took this. Only regret: a stupid pole in the top-left corner of the frame I had to clumsily get rid of at post-processing." link="http://www.flickr.com/photos/alexnormand/4829260124/in/set-72157624547713078/" */%}}

As a bonus, the shortcode named parameters are, arguably, more readable.

## Finishing touches
### Fix content
Depending on the amount of customization that was done with each post with Jekyll, this step will require more or less effort. There are no hard and fast rules here except that `hugo server --watch` is your friend. Test your changes and fix errors as needed.

### Clean up
You'll want to remove the Jekyll configuration at this point. If you have anything else that isn't used, delete it.

## A practical example in a diff
[Hey, it's Alex](http://heyitsalex.net/) was migrated in less than a _father-with-kids day_ from Jekyll to Hugo. You can see all the changes (and screw-ups) by looking at this [diff](https://github.com/alexandre-normand/alexandre-normand/compare/869d69435bd2665c3fbf5b5c78d4c22759d7613a...b7f6605b1265e83b4b81495423294208cc74d610).
