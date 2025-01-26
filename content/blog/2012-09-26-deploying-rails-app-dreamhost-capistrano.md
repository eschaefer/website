---
title: 'Deploying a Rails Application to Dreamhost with Capistrano'
tags: [Rails, deployment, Git, tutorial]
categories: blog

comments: false
---

I don't think there's a quick way to figure out your first "real" Rails deployment process. Sure there's Heroku (and now cloudControl, based here in Berlin) with magic one- or two-liner deployments. But they get expensive very quickly if you're running more than a dev server. Your file system also remains fixed after deployment, so you will end up storing your user assets in yet another cloud service. So for now I am relying on a cheap VPS from Dreamhost. A little more complex but also more flexible, and I'm learning a lot.

#### Some background before getting started:

- [Railscast: Deploying to a VPS](http://railscasts.com/episodes/335-deploying-to-a-vps)
- [Capistrano on Dreamhost (kinda dated, but useful)](http://wiki.dreamhost.com/Capistrano)
- [Syncing a git repository](http://wiki.dreamhost.com/Git#Setup_One:_For_the_Impatient) directly between your machine and Dreamhost. No GitHub!

I spent a good while figuring out a recipe to deploy a Rails app using Git with Capistrano on Dreamhost's particular hosting setup. I felt compelled now to share some of the snags I hit, so hopefully I can save someone else out there from wasting a few hours.

#### A Capistrano recipe for Dreamhost

I'm assuming you set up Capistrano in your app. My config/deploy.rb ended up looking like this. Read through just to get a sense of how this works with Dreamhost. All-caps should be replaced with your own values:

```ruby
set :user, "YOURUSERNAME" # Your dreamhost account's username
set :domain, 'YOURSERVER.dreamhost.com' # Dreamhost servername where your account is located
set :project, 'YOURPROJECT' # Your application as its called in the git repository
set :application, 'YOURAPPFOLDER' # Your app's location (domain or sub-domain name as setup in panel)
set :applicationdir, "/home/#{user}/#{application}" # The standard Dreamhost setup

set :repository, "ssh://YOURUSERNAME@SERVER.dreamhost.com/home/YOURUSERNAME/repos/YOURPROJECT.git"

ssh_options[:forward_agent] = true
default_run_options[:pty] = true

set :scm, :git
set :branch, 'master' # Or whatever branch you're on.
set :git_shallow_clone, 1
set :scm_verbose, true

# Or: `accurev`, `bzr`, `cvs`, `darcs`, `git`, `mercurial`, `perforce`, `subversion` or `none`

role :web, domain # Your HTTP server, Apache/etc
role :app, domain # This may be the same as your `Web` server
role :db, domain, :primary => true # This is where Rails migrations will run

# deploy config

set :deploy_to, applicationdir
set :deploy_via, :remote_cache
set :chmod755, "app config db lib public vendor script script/_ public/disp_"

set :use_sudo, false

set :normalize_asset_timestamps, false

namespace :deploy do
namespace :assets do
task :precompile, :roles => :web, :except => { :no_release => true } do
from = source.next_revision(current_revision)
if capture("cd #{latest_release} && #{source.local.log(from)} vendor/assets/ app/assets/ | wc -l").to_i > 0
run %Q{cd #{latest_release} && #{rake} RAILS_ENV=#{rails_env} #{asset_env} assets:precompile}
else
logger.info "Skipping asset pre-compilation because there were no asset changes"
end
end
end
end

# if you want to clean up old releases on each deploy uncomment this:

after "deploy:restart", "deploy:cleanup"

# if you're still using the script/reaper helper you will need

# these http://github.com/rails/irs_process_scripts

# If you are using Passenger mod_rails uncomment this:

namespace :deploy do
task :start do ; end
task :stop do ; end
task :restart, :roles => :app, :except => { :no_release => true } do
run "#{try_sudo} touch #{File.join(current_path,'tmp','restart.txt')}"
end
end
```

If your git repo is in place, and you just want to deploy your damn shit from your local terminal, run cap deploy:setup just once before running cap deploy. Capistrano will be verbose about anything going wrong, so it won't be a mystery to track down what the problem is, if you have one. Something probably will go wrong the first time.

More like than not, your asset pipeline won't precompile properly through the Capistrano deployment. If you figured out the secret to that, let me know. So for now I SSH into my <code>/home/username/app/current</code> folder and run <code>bundle exec rake:assets --precompile</code>.

#### Set up your shared gems folder

You might also find your VPS environment needs customization to use gems shared across different Rails apps, and StackOverflow will be your friend there.

If you're sharing gems, the most important thing you'll need that's specific to Dreamhost is this bit of code in the top of your <code>config/config.ru</code> file:

```ruby
ENV['GEM_HOME'] = '/home/YOURUSERNAME/YOURSHAREDGEMFOLDER'
require 'rubygems'
Gem.clear_paths
```

Passenger will also be vocal about what you're missing if you try to access your newly deployed app from a web browser.

```

```
