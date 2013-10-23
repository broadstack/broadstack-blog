---
title: "Envconfig: Environmentally friendly Ruby configuration."
date: 2013-10-21
---

*If you run a Ruby app with Heroku or Broadstack add-ons, you should use Envconfig.*

[Envconfig][ecgithub] is easy [environment-based configuration][12config] for
Ruby / Rails.  It lets you add, remove and swap add-ons or other backing
services without changing your code, and without conditional configuration
files.  Just update your `ENV` (that's what add-ons on [Broadstack][bs] and
[Heroku][ha] do) and Envconfig figures out your configuration.

Envconfig knows about common vars like `DATABASE_URL` and `SMTP_HOST`, as well
as provider-specific vars like `MONGOHQ_URL` and `POSTMARK_SMTP_SERVER`. It
uses this knowledge to expose a consistent configuration interface, and to
automatically configure your app.

It plays nicely with [dotenv][dotenv] in development, and it takes care of
menial work like parsing out config URIs into their components, splitting comma
separated lists into arrays, etc.


## Example: Auto-configure Rails

Here's an example showing how easy it is to create a Rails app, and see it
auto-configured to use the [Postmark][bspostmark] mail settings in the environment.

```sh
# Create a new Rails application
rails new test_app -BSJTq
cd test_app

# dotenv-rails to read `.env` file into Ruby's ENV
# envconfig-rails to read ENV and configure Rails
cat >> Gemfile << DONE
gem "dotenv-rails"
gem "envconfig-rails"
DONE
bundle --quiet

# Postmark settings in ENV
# (This is like provisioning Postmark on Broadstack or Heroku)
cat >> .env << DONE
POSTMARK_SMTP_SERVER="example.org"
POSTMARK_API_KEY="topsecret"
DONE

# Watch envconfig-rails auto-configure ActionMailer
mkdir -p log && touch log/development.log
tail -f log/development.log &
rails runner 'p Rails.application.config.action_mailer.smtp_settings'

# Output:
# Envconfig: Using Postmark for SMTP
# {:port=>"25", :authentication=>:plain, :address=>"example.org", :user_name=>"topsecret", :password=>"topsecret"}
```


## Example: Configuration Interface

We'll put the following into Ruby's `ENV` via our `.env` file.
(This is the same as provisioning [MongoHQ][bsmongohq] and MemCachier via
Broadstack or Heroku.)

```sh
MONGOHQ_URL="mongodb://user:pass@mongo.example.com:2468/stack618"
MEMCACHIER_SERVERS="m1.example.org:11211,m2.example.org:11211"
MEMCACHIER_USERNAME="memcachieruser"
MEMCACHIER_PASSWORD="memcachierpass"
```

And then from within Rails, e.g. `rails console`:

```ruby
# Loads from ENV by default.
config = Envconfig.load

# Individual keys:
config.mongodb[:url]  # => "mongodb://user:pass@mongo.example.com:2468/stack618"
config.mongodb[:host] # => "mongo.example.com"
config.memcached[:servers]  # => "m1.example.org:11211,m2.example.org:11211"
config.memcached[:server_strings]  # => ["m1.example.org:11211", "m2.example.org:11211"]

# Entire service configuration:
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

[Comments on Hacker News](https://news.ycombinator.com/item?id=6602098)


[bs]: https://broadstack.com/
[ha]: https://addons.heroku.com/
[12factor]: http://12factor.net/
[12config]: http://12factor.net/config
[readme]: https://github.com/broadstack/envconfig#readme
[bspostmark]: https://broadstack.com/addons/postmark
[bsmongohq]: https://broadstack.com/addons/mongohq
[ecgithub]: https://github.com/broadstack/envconfig
[ecrubygems]: http://rubygems.org/gems/envconfig
[ecrubygemsrails]: http://rubygems.org/gems/envconfig-rails
[dotenv]: https://github.com/bkeepers/dotenv
