---
title: Personal academic websites for the impatient
categories:
- advice
feature_image: "https://picsum.photos/2560/600?image=872"
image: "https://jmtayloruk.github.io/assets/img/undraw_Web_developer_re_h7ie.png"
---

If you are used to working with github, and using [markdown](https://www.markdownguide.org/basic-syntax), it really does just take half an hour to throw a website together.
Obviously writing the content will take a little longer, but once you're over the first hurdle you can add things incrementally.
For me the big hurdle was just figuring out how to get anything up and running at all.
This quick post is intended to let you replicate what I did very quickly.

## Starting from a template

I used [this template](https://github.com/daviddarnes/alembic) - just go to that link and click "Use this template" to make your own copy.
Other templates exist such as [academicpages](https://github.com/academicpages/academicpages.github.io) that may be more tailored towards an academic website. Just go to that link and click "fork".

In principle it should be pretty straightforward to migrate from one template to another, so don't feel you are making a big life decision which which template you pick initially.

To turn your new git respository into a website hosted at xxx.github.io, first rename your git repository so it reflects your own git username, e.g. jmtaylor.github.io. 
Then simply navigate to Settings->Pages on github and select a "source" i.e. a git branch that you want the website to be served from.
The usual choice would be "main" or "master" (whatever the main branch of the repository is called).
And that is literally it. Your website is now published (with the standard content from the template).
Next you need to tweak a few settings so it is your website, rather than the template one. 

## Where does the content come from?

Github will look for markdown files (.md) in your git repository, and turn them into webpages.
Behind the scenes it is using a system called [Jekyll](https://jekyllrb.com) but I can honestly say I haven't got the faintest idea how that works, *and I don't need to know*!
There is one file "_config.yml" which contains configuration information (see below), and once that is filled in you just write markdown files and they are magically turned into webpages.
If you are happy to write content and then just browse the live website to "test" your changes, you don't need to install anything new on your computer.
Just `git push`, wait a few minutes and refresh your website to see the update appear.

## Making the website yours

The things you need to change to make the website work, and make it "yours", can be found in _config.yml. 
Editing this was the part that seemed intimidating to me, and took me a bit of trial and error. But you can just copy the changes I did.
The key bits are:

* Put your own name, social media accounts etc under section 3 "Gem settings"
* Customise section 5 "Collections" for the appearance of the blog posts part of the website
* Customise section 7 "Site settings" for some general settings. The "logo" will appear in the top left of the website pages. I still haven't figured out how to replace the little alembic logo that appears e.g. against the website name in the Safari browser's history!
* Customise section 8 "Site navigation" to add additional pages to the manu bar of your website. Just add a url and write the equivalent .md file.
* Customise "navigation_footer" to reflect the url at which your website is going to appear.
* Customise "social_links" as you wish.

If you want to see all the things I have customised, [look here](https://github.com/jmtayloruk/jmtayloruk.github.io/compare/c61eb64...main#files_bucket).

## Writing pages for the website

Pages are written in [markdown](https://www.markdownguide.org/basic-syntax), which will be familiar to anyone working with e.g. Jupyter notebooks.

To embed images in your pages, save them under "assets" and reference them like this:
``` html
{% raw %}
{% include figure.html image="/assets/heart_sync_trabeculation_MIP_magenta_crop_still.png" alt="Heartbeat-synchronized image" position="right" width="300" height="338" %}
{% endraw %}
```

A link is written like this:
``` html
{% raw %}
[text to appear as the hyperlink](https://website.to.link/page/subpage.html)
{% endraw %}
```

The file index.md is the content for your home page - customise this as you wish.
At the top of the .md file you enter some information like the page title and banner image, then you just start writing.
[Take a look at the source file behind my own home page](https://raw.githubusercontent.com/jmtayloruk/jmtayloruk.github.io/main/index.md).

## Writing blog posts

To add a blog post, just write it as a .md file, save it under "_posts", and make sure the filename starts with a date (that is not in the future).
"feature_image" appears as a banner at the top of the page.
I mostly leave them as the theme default, as I don't have a good, quick source of suitably-shaped images.
"image" appears as part of a social media "share" of the blog post.
I find [undraw.co](https://undraw.co) useful for quickly finding a vaguely relevant image to use for that.

## More advanced stuff

Internal links within a webpage:  I added two lines of mysterious gibberish to my _config.yml file
(look for "kramdown" [in here](https://github.com/jmtayloruk/jmtayloruk.github.io/blob/main/_config.yml))
which is apparently necessary to be able to use internal links within a webpage.
For example [this](#writing-blog-posts) links to the "writing blog posts" heading on this webpage you are currently reading. 
I achieved that by writing a hyperlink to `#writing-blog-posts`, which matches the section heading except with spaces replaced by "-".

If you want your own domain name for the website (rather than e.g. jmtaylor.github.io) then you normally have to pay a company to create it for you.
But if you do, github can still serve the pages for that domain. Just fill in the "custom domain" setting under github Settings->Pages.

If you want to test your edits offline before `git push`-ing them to the live site via github, you should be able to do this if you install Jekyll locally.
Supposedly the instructions for that are [here](https://jekyllrb.com/docs/).
That didn't work for me, so for now I'm happily sticking with the amateur method of checking changes on the live website
(even if that just means [the git history gets a bit messy!](https://github.com/jmtayloruk/jmtayloruk.github.io/commits/main/_posts/2021-11-16-quick-website-building.md))
