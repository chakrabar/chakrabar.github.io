---
layout: post
title: "Adding remote theme to GitHub pages"
excerpt: "My experience with updating my GitHub pages site with a remote theme"
date: 2017-11-09
categories: notes
modified: 2017-11-09T22:11:53-04:00
hidden: true
---

## Adding a remote github theme

https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site/
https://github.com/benbalter/jekyll-remote-theme

#### Changes already done
1. Added `gem 'jekyll-remote-theme'` to `Gemfile`
2. Added this to `_config.yml`
```
    plugins:
      - jekyll-remote-theme
```
3. This too, to same file
    `remote_theme: mmistakes/so-simple-theme`


- $ bundle install >> worked fine
- $ bundle exec jekyll serve >> Didn't work >> local still runs though!

```
$ bundle exec jekyll serve
Configuration file: C:/Arghya/Repos/chakrabar.github.io/_config.yml
            Source: C:/Arghya/Repos/chakrabar.github.io
       Destination: C:/Arghya/Repos/chakrabar.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
      Remote Theme: Using theme mmistakes/so-simple-theme
jekyll 3.6.2 | Error:  Peer certificate cannot be authenticated with given CA certificates

```
Actually it worked partially
1. After adding the theme-name in _config, GitHub could update the site styles
2. It took a lot of time to publish though
3. It just updated the theme - literally, Without following steps, the theme is not setup


## Now trying https://mmistakes.github.io/so-simple-theme/theme-setup/

```
$ bundle exec jekyll build
Configuration file: C:/Arghya/Repos/chakrabarso_with_ssTheme/_config.yml
            Source: C:/Arghya/Repos/chakrabarso_with_ssTheme
       Destination: C:/Arghya/Repos/chakrabarso_with_ssTheme/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
  Liquid Exception: Invalid scheme format: {"feature"=>"so-simple-sample-image-7.jpg", "credit"=>"WeGraphics", "creditlink"=>"http in feed.xml
jekyll 3.5.2 | Error:  Invalid scheme format: {"feature"=>"so-simple-sample-image-7.jpg", "credit"=>"WeGraphics", "creditlink"=>"http
```

```
$ bundle update
$ bundle install
$ bundle exec jekyll build
```
 
 Error:

```
$ bundle exec jekyll build
Configuration file: C:/Arghya/Repos/chakrabarso_with_ssTheme/_config.yml
            Source: C:/Arghya/Repos/chakrabarso_with_ssTheme
       Destination: C:/Arghya/Repos/chakrabarso_with_ssTheme/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
  Liquid Exception: Invalid scheme format: {"feature"=>"so-simple-sample-image-7.jpg", "credit"=>"WeGraphics", "creditlink"=>"http in feed.xml
jekyll 3.6.2 | Error:  Invalid scheme format: {"feature"=>"so-simple-sample-image-7.jpg", "credit"=>"WeGraphics", "creditlink"=>"http
```

https://github.com/jekyll/jekyll-feed/issues/191
Had to add path to image gray matter
```
image:
  path: http://example.com/so-simple-sample-image-1.jpg
  feature: so-simple-sample-image-7.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
```

```
$ bundle exec jekyll build
Configuration file: C:/Arghya/Repos/chakrabarso_with_ssTheme/_config.yml
            Source: C:/Arghya/Repos/chakrabarso_with_ssTheme
       Destination: C:/Arghya/Repos/chakrabarso_with_ssTheme/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 5.182 seconds.
 Auto-regeneration: disabled. Use --watch to enable.

```

#### This is what worked actually

```
$ bundle update
$ bundle install
$ bundle exec jekyll build
$ bundle exec jekyll serve --port 9999
```

Results:

```
$ bundle exec jekyll serve --port 9999
Configuration file: C:/Arghya/Repos/chakrabarso_with_ssTheme/_config.yml
            Source: C:/Arghya/Repos/chakrabarso_with_ssTheme
       Destination: C:/Arghya/Repos/chakrabarso_with_ssTheme/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 5.45 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for 'C:/Arghya/Repos/chakrabarso_with_ssTheme'
    Server address: http://127.0.0.1:9999
  Server running... press ctrl-c to stop.
```

## Customizing the theme

#### Adding css

* Add css under `_sass/_page.scss` or other file as necessary. They'll be available afetr build.

#### Adding js

Custom `js` can be added to `assets/js/_main.js`, which gets minifed and bundled with other `js` files into `assets/js/scripts.min.js`.

Now, just adding to the `_main.js` will not update the site js. The `grunt` build needs to be run. For that, we need the following

* `npm` needs to be installed - use Windows installer from [npm](https://nodejs.org/en/)
* All npm dependencies needs to be installed
  * The dependencies are listed in `package.json`
  * To install them, run `$ npm install`
  * can be checked through `$ npm list -g`
* The [grunt](http://gruntjs.com/getting-started) cli needs to be installed
  * Run `$ npm install -g grunt-cli`

With these, we are ready to run the `grunt` tasks, which are listed in
  * File `Gruntfile.js`

If there are errors running the `grunt` task `imagemin` (see below), try fixing with Google help, or the links
  * [Github link](https://github.com/imagemin/imagemin/issues/216)
  * [SO link](https://stackoverflow.com/questions/19906510/npm-module-grunt-contrib-imagemin-not-found-is-it-installed)
  * For me, nothing worked, so I commented out imagemin tasks from `Gruntfile.js`. So my images are not optimized

Now, finally run the `grunt` tasks, which will update `assets/js/scripts.min.js`

* Run `$ grunt`

#### Then do the usual (for everyday build & run)
* `bundle exec jekyll build`
* `bundle exec jekyll serve`