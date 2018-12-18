---
layout: post
title: "Setting up a GitHub pages site locally"
excerpt: "Locally running GitHub pages - Ruby, Bundler, Jekyll & more"
date: 2017-11-08
categories: notes
modified: 2017-11-08T22:11:53+05:30
hidden: true
published: false
---

Setting up Jekyll (Github page) site locally:
=============================================

https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/

1. Install Ruby from - https://rubyinstaller.org/downloads/ (v2.4.3-1 x64) - takes quite a bit of time on a cli
	Which in turn installs msys64 (for c-based gems)
  
2. Check in Git Bash with
	`$ ruby --version`

3. Install bundler (http://bundler.io/)
	`$ gem install bundler`
	Successfully installed bundler-1.16.1
	
4. Add Gemfile to site root, with following content
	source 'https://rubygems.org'
	gem 'github-pages', group: :jekyll_plugins
	
5. Install Jekyll and other dependencies
	* Go to project root in Git Bash (well it installs stuffs globally it seems)
	* `$ bundle install`
	* Installs Jekyll, all stupid themes, http pipeline and what not!
	* Only new file in site root is Gemfile.lock
	
6. Run the site locally
	Go to project root in Git Bash
	`$ bundle exec jekyll serve`

Error: 
    
	Dependency Error: Yikes! It looks like you don't have jekyll-remote-theme or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'Could not open library 'libcurl': The specified module could not be found. . Could not open library 'libcurl.dll': The specified module could not be found. . Could not open library 'libcurl.so.4': The specified module could not be found. . Could not open library 'libcurl.so.4.dll': The specified module could not be found. ' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/!
    
	jekyll 3.6.2 | Error:  jekyll-remote-theme

I though it was an 'jekyll-remote-theme' gem issue, so i tried to install it as

	https://github.com/benbalter/jekyll-remote-theme
	Add - gem 'jekyll-remote-theme' to your Gemfile
	Add this to _config.yml
		plugins:
		  - jekyll-remote-theme
	$ bundle install >> worked fine
	$ bundle exec jekyll serve >> same error 
	
	Tried installing the gem again
	$ gem install jekyll-remote-theme
	$ bundle install >> worked fine
	$ bundle exec jekyll serve >> same error 
	
Seems like the issue is with dependency >> libcurl.dll

Solution: http://talk.jekyllrb.com/t/error-with-github-pages-locally/1239/5
* Download the 7zip from https://curl.haxx.se/gknw.net/7.40.0/dist-w64/curl-7.40.0-devel-mingw64.7z
* extract all files
* copy `libcurl.dll` from `/bin` to `C:\Ruby24-x64\bin`
	
New error: 

	$ bundle exec jekyll serve
	Configuration file: C:/Arghya/Repos/chakrabar.github.io/_config.yml
				Source: C:/Arghya/Repos/chakrabar.github.io
			Destination: C:/Arghya/Repos/chakrabar.github.io/_site
		Incremental build: disabled. Enable with --incremental
			Generating...
		Liquid Exception: No repo name found. Specify using PAGES_REPO_NWO environment variables, 'repository' in your configuration, or set up an 'origin' git remote pointing to your github.com repository. in /_layouts/default.html
					ERROR: YOUR SITE COULD NOT BE BUILT:
						------------------------------------
						No repo name found. Specify using PAGES_REPO_NWO environment variables, 'repository' in your configuration, or set up an 'origin' git remote pointing to your github.com repository.
							
No repor name issue

Solution: https://github.com/jekyll/jekyll/issues/4705
* Add "repository: chakrabar/chakrabar.github.io" to _config.yml

```
$ bundle exec jekyll serve

Configuration file: C:/Arghya/Repos/chakrabar.github.io/_config.yml
			Source: C:/Arghya/Repos/chakrabar.github.io
		Destination: C:/Arghya/Repos/chakrabar.github.io/_site
	Incremental build: disabled. Enable with --incremental
		Generating...
	GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
					done in 3.611 seconds.
	Please add the following to your Gemfile to avoid polling for changes:
	gem 'wdm', '>= 0.1.0' if Gem.win_platform?
	Auto-regeneration: enabled for 'C:/Arghya/Repos/chakrabar.github.io'
	Server address: http://127.0.0.1:4000
	Server running... press ctrl-c to stop.
```
	
## YOOO HOOOOOO!!! Voila...!!! Site running at http://localhost:4000

It created a publish folder at /root/_sit