## Trying to add a remote github theme

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
