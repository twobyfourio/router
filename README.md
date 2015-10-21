# twobyfour :: router

twobyfour.io is composed of several smaller parts. Currently, we're looking at: 

* app: the Rails app
* errors: The Middleman static error pages
* help: via HelpScout
* site : the Middleman marketing site / home page

On Production and Staging, we glue these apps (I'm not going to call them services or microservices, because eh, not really, but somebody probably will for me) together with CloudFront. 

For local development, its not so easy. We want a cohesive experience as much like production as possible, but each app will run on a separate port locally. So nginx comes into play, and it's actually not too painful.

## Install nginx 

Router assumes you're running [nginx](https://www.nginx.com/) locally. If you don't have nginx already, [homebrew](http://brew.sh/) has you covered. If you don't use OS X, use your package manager of choice. :) 

`brew install nginx`

You don't have to queue up nginx on system boot, as Router will start the nginx process for you if you don't already have it running.

## Install Foreman (for your Procfile)

What's a Procfile? It's a list of processes a project needs to run. [Learn more here](https://devcenter.heroku.com/articles/procfile), then [install Foreman via the Heroku Toolbelt](https://toolbelt.heroku.com/).

## Install POW (for https://twobyfour.dev support)

[Here's the manual](http://pow.cx/manual.html), I should probably write more here when I have time.  tl;dr: `echo 2949 > ~/.pow/twobyfour`

This only really affects the `http://` URLs, so don't sweat this too much if you're not so inclined. All `https://` URLs are handled via `stud` (see `bootup-powssl`). Protip: your `~/.stud/config` backend port config should match the port your HTTPS server is listening to in your `nginx.conf`:

``` ~/.stud/config:
backend = "[127.0.01]:2940"
...
nginx.conf: 
  # HTTPS server
  server {
    listen 2940; # 2x4... hehe
```

Many of our OAuth callbacks will redirect back to SSL URLs for local development, and @nthj is a bit of a stickler for matching production environment as able, so hence the slightly lengthier setup process. 

## Other dependencies

The individual apps may have their own dependencies, like Ruby and PostgreSQL, that your local machine needs. Please refer to their respective READMEs as dependencies may change over time. 

## Configuring nginx

You've got a couple options for configuring nginx for the Router. If you aren't using nginx with any other projects (I'm not - NJ), you can do a hard link from your Router project directory to your nginx configuration directory. This way, nginx picks up your configuration straight from the Router project, and if we ever need to tweak the configuration or add another app, a simple `git pull` will get us all going. For example:

```
cd ~/Sites/twobyfour/router/
ln /opt/boxen/homebrew/etc/nginx/nginx.conf nginx.conf
```

Please check `brew info nginx` to verify your nginx configuration directory.

If you have other nginx projects, please splice the `nginx.conf` file together with your existing configuration. This is left as an exercise to the developer. :) 

## Configuring secret tokens

You'll want to set up any secret tokens each of the apps depends on in a `.env` file.  You'll probably have to talk to @nthj about access to some of these systems. And `cp .env.example .env` to kick things off.

## Booting up the Router

```
cd ~/Sites/twobyfour/router/
foreman start
```

## OK, great. What is nginx actually doing for me?

So `foreman start` is going to (hopefully) boot up each of your apps onto their respective ports. nginx will give you one nice, clean URL to access each separate app on.  Read through the source, it's not too long.

* The marketing site will boot on to port `2941`, nginx will send all URLs by default to the marketing site.
* Anything in the `(assets|settings|store|users)` directory will be sent over to the Rails app.
* `(article|category)` proxies directly to the help scout site—you can't change anything here locally, nor should you need to.
* Errors will head over to port `2943`.
```

## How do I know if it works?

Head over to `https://twobyfour.dev` in your browser and click around. If all the links more or less work, you're good to go. :)
