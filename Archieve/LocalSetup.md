Setting up Jekyll (Github page) site locally:
=============================================

https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/

1. Install Ruby from - https://rubyinstaller.org/downloads/ (v2.4.3-1 x64) - takes quite a bit of time on a cli
	Which in turn installs msys64 (for c-based gems)
  
2. Check in Git Bash with
	$ ruby --version

3. Install bundler (http://bundler.io/)
	$ gem install bundler
	Successfully installed bundler-1.16.1
	
4. Add Gemfile to site root, with following content
	source 'https://rubygems.org'
	gem 'github-pages', group: :jekyll_plugins
	
5. Install Jekyll and other dependencies
