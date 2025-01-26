---
title: 'Removing Trailing Slashes from a Rails App on Heroku'
tags: [rails, ruby, rack-redirect]
categories: blog

comments: false
---

I need to redirect those terrible trailing URL slashes on a large project with thousands of articles. Turns out that trailing slash `==` duplicate content.

The first suggestion is usually to [use Apache's mod_rewrite](http://stackoverflow.com/questions/627006/how-to-remove-a-urls-trailing-slash-in-a-rails-app-in-a-seo-view), however I wanted a server-agnostic way to handle these redirects. Not every project these days is sitting on top of Apache. And in my particular case, I need this to work on Heroku's very tightly controlled configuration stack.

The solution which worked for me is just including and configuring the [rack-rewrite](https://github.com/jtrupiano/rack-rewrite) gem.

#### 1. Add `rack-rewrite` to your Gemfile

```ruby
gem 'rack-rewrite'
```

Then rerun `bundle install` from your shell to make it available to your project.

#### 2. Configure

**Rails 4**

In `config/application.rb`:

```ruby
require "rack/rewrite"

...

# Rewrite trailing slashes

# goes inside your main Application class

config.middleware.insert_before(Rack::Runtime, Rack::Rewrite) do
r301 %r{^/(.\*)/$}, '/$1'
end
```
