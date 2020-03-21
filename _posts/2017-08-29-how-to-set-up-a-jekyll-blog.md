---
title: How to Set Up a Jekyll Blog
category: Gist
tags: [jekyll, web, github]
---

So, you want to start blogging, and you should! Blogging is a great way to relax, share information, and reinforce new topics that you may be learning about. Jekyll is a simple, blog-aware, static site generator, and you can use it to easily stand up your own blog and have it hosted for free using Github Pages. Today, we'll be using the [Jekyll-Now](www.jekyllnow.com) theme to style our newly-created blog, and we'll host it on Github Pages with a custom domain name.

To follow along with us you will need the following:
* An installation of [Ruby >= 2.1](https://rvm.io/rvm/basics)
* An account with [Github](https://github.com)
* Only basic knowledge of [Jekyll](https://jekyllrb.com)

## Getting Started

#### Creating the Project

To get started, let's create a folder on our local computer, and download the base project for jekyll-now.

```bash
[coffeegist: ~]$ mkdir -p Development/blog
[coffeegist: ~]$ cd Development/blog
[coffeegist: blog]$ curl -O https://raw.githubusercontent.com/jekyll/jekyll/master/.gitignore
[coffeegist: blog]$ curl -o ~/Downloads/jekyll.zip https://codeload.github.com/barryclark/jekyll-now/zip/master
[coffeegist: blog]$ unzip ~/Downloads/jekyll.zip
[coffeegist: blog]$ mv jekyll-now-master/* .
[coffeegist: blog]$ rmdir jekyll-now-master
[coffeegist: blog]$ rm ~/Downloads/jekyll.zip
```

#### Configuration Changes

In order to make this site your own, you'll need to make some changes to the `_config.yml` file. This is where settings such as your sites name and description as stored. It also acts as a configuration point for social media profiles, as well as add-ons like Google-Analytics and Disqus.

At a minimum, you should change the name and description fields.

```yml
# _config.yml

# Name of your site (displayed in the header)
name: The Coffeegist

# Short bio or description (displayed in the header)
description: Pragmatic Pour-Overs
```

#### Serving the Project Locally

Once these changes are made, you're ready to view your site! Jekyll will serve your site at <http://localhost:4000/> in development mode by default. Open a terminal and type the following:

```bash
[coffeegist: blog]$ gem install github-pages # You only need to do this once!
[coffeegist: blog]$ jekyll serve -w
```

## Tweaking the Theme

Before we init, commit, and deploy our new blog, we can tweak parts of the layout to get exactly what we want. A lot of the styles to tweak will be found in the `styles.scss` file. I will make the following changes to mine. To skip this section, [click here](#committing-and-deploying).

* [Sticky Footer](#sticky-footer)
* [Widen the Layout](#widen-the-layout)
* [Code-Highlighting Changes](#code-highlighting-changes)

#### Sticky Footer

```scss
/* styles.scss */

html {
  font-size: 100%;
  height: 100%;
}

body {
  background: $white;
  font: 18px/1.4 $helvetica;
  color: $darkGray;
  height: 100%;
}

.wrapper {
  min-height: 100%;
  margin-bottom: -138px;
}

.push {
  height: 138px;
}
```

```html
<!-- _layouts/default.html -->

<div class="wrapper">
  <div class="wrapper-masthead">
    <div class="container">
      <header class="masthead clearfix">
        <a href="{{ '{{ site.baseurl ' }}}}/" class="site-avatar"><img src="{{ '{{ site.avatar ' }}}}" /></a>

        <div class="site-info">
          <h1 class="site-name"><a href="{{ '{{ site.baseurl ' }}}}/">{{ '{{ site.name ' }}}}</a></h1>
          <p class="site-description">{{ '{{ site.description ' }}}}</p>
        </div>

        <nav>
          <a href="{{ '{{ site.baseurl ' }}}}/">Blog</a>
          <a href="{{ '{{ site.baseurl ' }}}}/about">About</a>
        </nav>
      </header>
    </div>
  </div>

  <div id="main" role="main" class="container">
    {{ '{{ content '}}}}
  </div>
  <div class="push"></div>
</div>
```

#### Widen the Layout

```scss
/* styles.scss */

.nav-container {
  margin: 0 auto;
  padding: 0 15%;
  width: 100%;
}

.container {
  margin: 0 auto;
  max-width: 760px;
  padding: 0 10px;
  width: 100%;
}
```

```html
<!-- _layouts/default.html -->

<div class="wrapper">
  <div class="wrapper-masthead">
    <div class="nav-container">
      <header class="masthead clearfix">
```

#### Code-Highlighting Changes

```scss
/* _sass/_highlights.scss */

.highlighter-rouge {
    color: #586e75;
    background-color: #efefef;
}

code {
  font-family:'Bitstream Vera Sans Mono','Courier', monospace;
  font-size: 14px;
}
```

## Committing and Deploying

#### Create a Local Repository

Once we are satisfied with our layout, we are ready to commit and deploy our blog! First, we need to initialize a local repository.

```bash
[coffeegist: blog]$ git init
[coffeegist: blog]$ git add .
[coffeegist: blog]$ git commit -m "* Initial Commit"
```

#### Create a Remote Repository

Now that our code is being tracked in a local repository, we need to create a remote repository on <https://github.com> that we can push to. You can obviously do this through the web interface, but we will use the [Github API](https://developer.github.com/v3/) to create our new repository :)

One thing worth mentioning is that your repository name must match the format of _username_.github.io **_exactly_**. If it doesn't, Github won't properly host your site.

Example: My username is `coffeegist`, so my repository would be `coffeegist.github.io`.

```bash
[coffeegist: blog]$ curl -i -u coffeegist https://api.github.com/user/repos -d '{"name":"coffeegist.github.io"}'
```

#### Push to Deploy

Our final step is to set our remote origin, and push our finished site to Github.

```bash
# Make sure to replace my username with yours below
[coffeegist: blog]$ git remote add origin git@github.com:coffeegist/coffeegist.github.io.git

[coffeegist: blog]$ git push -u origin master
```

Tada! Your new blog can now be viewed at <https://USERNAME.github.io/>.

## Next Steps

We've covered a lot of ground. We started with absolutely nothing, created a Jekyll-based blog, and set up Github Pages to host our new content! This is a great time to start writing and sharing your knowledge with the world!

In a follow-up post, we will cover setting up a custom domain, and enabling HTTPS for your blog. Until then, Happy Hacking!
