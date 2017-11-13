---
layout: default
title: "Comparison of serialization in .Net"
date: 2017-11-06
tags: tech tips notes
---

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `bundle exec jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

```ruby
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
```

```csharp
static void Main(string[] args)
{
    var tr = "true";
    bool flag = Convert.ToBoolean(tr);
    System.Console.ReadLine();
}
```