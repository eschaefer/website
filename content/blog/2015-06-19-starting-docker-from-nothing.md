---
title: "Setting Up Docker From Nothing"
tags: [docker, deployment]
categories: blog
published: true
comments: false
---

![Docker](/images/posts/docker.png)

# Docker... Starting from Nothing

On OS X, you will need Three applications... VirtualBox, Boot2Docker and Compose.

## Boot2Docker

> Because the Docker daemon uses Linux-specific kernel features, you can't run Docker natively in OS X. Instead, you must install the Boot2Docker application.

#### 1. Download/install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

#### 2. Install Boot2docker and Compose to run Docker on OS X.

```bash
$ brew install boot2docker docker-compose
```

Follow the Homebrew instructions to manage launching on startup, etc.

#### 3. Create a new Boot2Docker VM. You only need to do this once, yay!

```bash
$ boot2docker init
$ boot2docker start
```

#### 4. Display the environment variables for the Docker client.

```bash
$ boot2docker shellinit
Writing /Users/mary/.boot2docker/certs/boot2docker-vm/ca.pem
Writing /Users/mary/.boot2docker/certs/boot2docker-vm/cert.pem
Writing /Users/mary/.boot2docker/certs/boot2docker-vm/key.pem
    export DOCKER_HOST=tcp://192.168.59.103:2376
    export DOCKER_CERT_PATH=/Users/mary/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
```

#### 5. Set the environment variables in your shell:

```bash
$ eval "$(boot2docker shellinit)"
```

You’ll need to run this for every terminal session that invokes the `docker` or `docker-compose` command.

Auto initiate this by adding the following line to the bottom of either your `.zshrc` or `.bashrc` file:

```bash
$ eval `boot2docker shellinit 2>/dev/null`
```

#### 6. Run the hello-world container to verify your setup.

```bash
$ docker run hello-world
```

### Example Dockerfile for a Rails project

In the root directory of your Rails project, create a file called `Dockerfile`. Example contents for our Wildeisen project would be:

```docker
FROM ruby:2.2.2

RUN apt-get update -qq && apt-get install -y build-essential

# for postgres
RUN apt-get install -y libpq-dev

# for nokogiri
RUN apt-get install -y libxml2-dev libxslt1-dev

# for a JS runtime
# patch the version of Node since apt-get would grab an olllld version
RUN curl -sL https://deb.nodesource.com/setup_0.12 | bash -
RUN apt-get install -y nodejs

# for npm task runners
# grab newer version, not from apt-get
RUN curl -L https://www.npmjs.com/install.sh | sh

ENV APP_HOME /project
RUN mkdir $APP_HOME

WORKDIR /tmp
COPY Gemfile Gemfile
COPY Gemfile.lock Gemfile.lock
COPY package.json package.json
RUN bundle install
RUN npm install

ADD . $APP_HOME
WORKDIR $APP_HOME

# for running grunt task, which builds SVG sprites
RUN npm install -g grunt-cli

ADD . $APP_HOME
```

To build this Docker image, from Rails root run:

```bash
$ docker build .
```

## Compose

Compose lets us configure our application from one `yml` file. Pretty sweet.

In the root directory of our Rails project, create a file called `docker-compose.yml`

Here's a really basic example that works for my project:

```ruby
db:
  image: postgres

redis:
  image: redis

web:
  build: .
  volumes:
   - .:/project
  command: bin/rails server --port 3000 --binding 0.0.0.0
  links:
   - db
   - redis
  ports:
   - "3000:3000"

```

Then build the project with the above config:

```bash
$ docker-compose build
```

Now we can run it with our Docker container!

```bash
$ docker-compose up
```

You can find out which IP address to access the Rails server from by opening a new shell window and typing:

```bash
$ boot2docker ip
192.168.59.103
```

In my case, I can access 192.168.59.103, with the port we set in our `yml` file, 3000.

So voila, I can now see my Rails project in my browser at `http://192.168.59.103:3000`

Running `npm` watch/build tasks is easy too.

```bash
$ docker-compose run web npm run watch
```

```bash
$ docker-compose run web npm run build
```

or whatever task runners you have defined in your `package.json` file.
