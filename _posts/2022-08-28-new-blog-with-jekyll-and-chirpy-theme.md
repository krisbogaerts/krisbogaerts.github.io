---
title: "New Github Pages blog with Jekyll and the Chirpy theme"
author: Kris Bogaerts
date: 2022-08-28
category: [Github, Blog, Jekyll]
tags: [github, blog, writeup, jekyll]
---

## Getting started
### installation Jekyll on MacOS
This post is about the installation and setup of a new blog/documentation site on GitHub Pages with Jekyll and the Chirpy theme. Technical notes on the initial settings of https://krisbogaerts.github.io.

If followed the article https://jekyllrb.com/docs/installation/macos/ for the installation of Jekyll and requirements on Monterey (macOS 12).

### Site Creation
For the site creation the Chirpy has been used and initiated with "option 1 using the chirpy starter".
https://chirpy.cotes.page/posts/getting-started/#option-1-using-the-chirpy-starter

This will create a new repository from the 'Chirpy Starter' and you can name it <GH_USERNAME>.github.io, where GH_USERNAME represents your GitHub username. 
In this case krisbogaerts.github.io

After creating a site based on the template, clone your repo:
```bash
git clone git@<YOUR-USER-NAME>/<YOUR-REPO-NAME>.git
```
then install your dependencies

```bash
cd your-repo-name
bundle
```

After making the first changes to your site you commit and push then up to git

```bash
git add .
git commit -m "made some changes"
git push
```
***

## Site Configuration


### _config.yml [(Configuration)](https://chirpy.cotes.page/posts/getting-started/#configuration)
>Edit with your own information

This is my starter configuration. Everything below has not been changed from the original theme configuration

```yaml
# The Site Configuration
theme: jekyll-theme-chirpy
baseurl: ''

lang: en
prefer_datetime_locale:
timezone: Europe/Brussels

title: Tech Notes
tagline: IT, (Cyber) Security & Smart Home 

description: >-
  There is no future, no past. There is only the present.
url: 'https://krisbogaerts.github.io'

github:
  username: KrisBogaerts

twitter:
  username: BogaertsKris

social:
  name: Kris Bogaerts
  links:
    - https://twitter.com/BogaertsKris
    - https://github.com/KrisBogaerts
    - https://www.linkedin.com/in/bogaertskris

google_site_verification:

theme_mode: dark

img_cdn:

avatar: https://media-exp1.licdn.com/dms/image/C5603AQELKDvai-z10A/profile-displayphoto-shrink_200_200/0/1633932732690?e=1666828800&v=beta&t=Ro_IDU6cQJuIgWaZrtKoXYIR3pDmRw_yYi1BWfzRN8Y

toc: true

comments:
  active: disqus
  disqus:
    shortname: <shortname>
```

### Contact Icons

I also removed the email icon from the left bottom as i don't want to use this
edit _data/contact.yml and comment the following
```console
#-
#  type: email
#  icon: 'fas fa-envelope'
#  noblank: true
```
***

## Creating a Post

### Naming Conventions

Jekyll uses a naming [convention for pages and posts](https://jekyllrb.com/docs/posts/)

Create a file in `_posts` with the format

```file
YEAR-MONTH-DAY-title.md
```

For example:

```file
2022-05-23-homelab-docs.md
2022-05-34-hardware-specs.md
```

> Jekyll can delay posts which have the date/time set for a point in the future determined by the "front matter" section at the top of your post file. Check the date & time as well as time zone if you don't see a post appear shortly after re-build.
{: .prompt-tip }

### Local Linking of Files

Image from asset:

```markdown
... which is shown in the screenshot below:
![A screenshot](/assets/screenshot.jpg)
```

Linking to a file

```markdown
... you can [download the PDF](/assets/diagram.pdf) here.
```

See more post formatting rules on the [Jekyll site](https://jekyllrb.com/docs/posts/)

### Markdown Examples

If you need some help with markdown, check out the [markdown cheat sheet](https://www.markdownguide.org/cheat-sheet/)

For more neat syntax for the Chirpy theme check their demo page on making posts <https://chirpy.cotes.page/posts/write-a-new-post/>


***

### Locally serving your site for validation

```bash
bundle exec jekyll s
```

```shell
Configuration file: /Users/kbo/Documents/kbo.github.io/_config.yml
 Theme Config file: /Users/kbo/.gem/ruby/3.1.2/gems/jekyll-theme-chirpy-5.2.1/_config.yml
            Source: /Users/kbo/Documents/kbo.github.io
       Destination: /Users/kbo/Documents/kbo.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.229 seconds.
 Auto-regeneration: enabled for '/Users/kbo/Documents/kbo.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

![Running Local](/assets/img/Screenshot_runninglocal.png)

### Building your site in production mode

```bash
JEKYLL_ENV=production bundle exec jekyll b
```

This will output the production site to `_site`


***
# Deploy on GitHub Pages

For security reasons, GitHub Pages build runs on safe mode, which restricts us from using tool scripts to generate additional page files. Therefore, we can use GitHub Actions to build the site, store the built site files on a new branch, and use that branch as the source of the Pages service.

> I am running a M1 Mac and this resulted in an error when building the site via Github Actions. 
> The following command fixed the error for me ```bundle lock --add-platform x86_64-linux```

Push any commit to origin/master to trigger the GitHub Actions workflow. Once the build is complete, a new remote branch called gh-pages will appear, which is used to store the built site files.

Choose branch gh-pages as your GitHub Pages source.

![GitHub Pages Settings](/assets/img/Screenshot_GitHubPages.png)

This should result is a build in the GitHub Actions on you repository

![GitHub Actions Result](/assets/img/Screenshot_GitHubActions.png)
you can view the details or results in case of any problems to see what is going wrong

Visit your website at the address indicated by GitHub.






