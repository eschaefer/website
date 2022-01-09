---
title: "Avoid a Headache: Precompiling Zurb (Foundation) on Heroku"
tags: [programming, rails]
categories: blog
published: true
comments: true
---

It always happens. My pristine development environment is running my app just fine. Then I deploy it to Heroku and whomp, trouble abounds. Let's leave alone the fact that I'm using the zurb-foundation gem, and instead focus on the problem which has to be afflicting you as well if you're reading this.

I got this message in my Heroku log while deploying my app:

```
    ....
    Preparing app for Rails asset pipeline
    Running: rake assets:precompile
    rake aborted!
    File to import not found or unreadable: foundation/common/ratios.
```

Well, look in your assets/stylesheets folder, the zurb-foundation gem generated a file called <code>foundation_and_overrides.scss</code>. The first thing it does is <code>@import "foundation/common/ratios";</code>. So, what? Well, if you're lazy like me, you're loading the zurb-foundation gem without a specified version number:

```ruby
group :assets do
gem 'sass-rails', '~> 3.2.3'
gem 'coffee-rails', '~> 3.2.1'
gem 'compass-rails'
gem 'zurb-foundation'
end
```

At the time of writing this, the zurb-foundation gem is at 4.1.2. Which means that 'foundation/common/ratios' is no longer included in the gem. The quick and dirty fix would be to lock the zurb-foundation gem down to 3.2.5 ([thank you Stackoverflow](http://stackoverflow.com/questions/15123667/syntax-error-file-to-import-not-found-or-unreadable-foundation-common-ratios)). So now I declare the gem 'zurb-foundation' in the assets group like so:

```ruby
gem 'zurb-foundation', '= 3.2.5'
```

Save your gemfile, now run <code>bundle update</code>, commit your changes to git, and re-deploy to Heroku. Should be smooth sailing and your assets should compile just fine.
