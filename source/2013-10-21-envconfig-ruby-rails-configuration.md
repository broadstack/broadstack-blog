---
title: "Envconfig: Environmentally friendly Ruby configuration."
date: 2013-10-21
---

*envconfig reads your ENV and builds a config for your backing services / add-ons.*

```ruby
Envconfig.load("OPENREDIS_URL" => "redis://u:p@example.org:1234/").redis[:host]
# => "example.org"
```

Add-ons at [Broadstack][bs] and [Heroku][ha] let you easily add and remove
[backing services][12bs] and change providers at a [deployment, not codebase,
level][12codebase].  But in reality your app needs to know where to look in the
[ENV for configuration][12config], which usually means pushing new code to swap
a backing service.

[Envconfig][ecgithub] changes that for Ruby / Rails apps.

Envconfig knows about many popular add-ons across [Broadstack][bs] and [Heroku
Add-ons][ha], and it's easy to add more.  It finds them in your environment,
exposing them as a consistent configuration interface.  This lets you easily
switch between providers, or between development and production, without having
  configuration conditionals in your code.

It plays nicely with [dotenv][dotenv] in development, and it takes care of
menial work like parsing out config URIs into their components, splitting comma
separated lists into arrays, etc.


## An example

You've used Broadstack or Heroku to provision [Postmark][bspostmark] for mail
delivery, and [MongoHQ][bsmongohq] for persistence, so your ENV contains
something like this:

```sh
POSTMARK_SMTP_SERVER="smtp.example.org"
POSTMARK_API_KEY="bcca0a78abbaed6533f3c8017b804bda"
MONGOHQ_URL="mongodb://user:pass@mongo.example.com:2468/stack618"
```

Restart Rails, and you'll see `Envconfig: Using Postmark for SMTP` in the logs.
This is `envconfig-rails` telling you ActionMailer has been configured:

```ruby
Rails.application.config.action_mailer.smtp_settings  # => {
  :port => "25",
  :authentication => :plain,
  :address => "smtp.example.org",
  :user_name => "bcca0a78abbaed6533f3c8017b804bda",
  :password => "bcca0a78abbaed6533f3c8017b804bda",
}
```

Of course, you can also get at the configuration directly:

```ruby
# Loads from ENV by default.
config = Envconfig.load

# Individual keys:
config.smtp[:address] # => "smtp.example.org"
config.mongodb[:host] # => "mongo.example.com"

# Service configuration:
config.mongodb.to_h # => {
  url: "mongodb://user:pass@mongo.example.com:2468/stack618",
  database: "stack618",
  username: "user",
  password: "pass",
  host: "mongo.example.com",
  port: 2468
}
```


## Services and Providers

The following services and the providers are currently supported.
(Check the [latest README][readme] for more news as it happens!)

* Database: generic `DATABASE_URL`.
* memcached: MemCachier.
* MongoDB: MongoHQ, MongoLab.
* Redis: openredis, RedisCloud, RedisGreen, Redis To Go.
* SMTP: generic, Mailgun, Mandrill, Postmark, Sendgrid.


## Try it out

It's not just for Broadstack or Heroku. [The Twelve-Factor App][12factor] is wise
words by Heroku Co-Founder Adam Wiggins, worth reading no matter where you deploy
your app.

> The twelve-factor app stores config in environment variables (often shortened
> to env vars or env). Env vars are easy to change between deploys without
> changing any code; unlike config files, there is little chance of them being
> checked into the code repo accidentally; and unlike custom config files, or
> other config mechanisms such as Java System Properties, they are a language-
> and OS-agnostic standard.  — http://12factor.net/config

* Clone, bundle or fork [envconfig on GitHub][ecgithub]
* gem install [envconfig on RubyGems][ecrubygems]
* gem install [envconfig-rails on RubyGems][ecrubygemsrails]


[bs]: https://broadstack.com/
[ha]: https://addons.heroku.com/
[12factor]: http://12factor.net/
[12bs]: http://12factor.net/backing-services
[12config]: http://12factor.net/config
[12codebase]: http://12factor.net/codebase
[readme]: https://github.com/broadstack/envconfig#readme
[bspostmark]: https://broadstack.com/addons/postmark
[bsmongohq]: https://broadstack.com/addons/mongohq
[ecgithub]: https://github.com/broadstack/envconfig
[ecrubygems]: http://rubygems.org/gems/envconfig
[ecrubygemsrails]: http://rubygems.org/gems/envconfig-rails
[dotenv]: https://github.com/bkeepers/dotenv
