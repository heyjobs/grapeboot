# README

This is a minimal Rails app to demonstrate an uninitialized constant error when using `sidekiqswarm` (from [sidekiq-ent](https://sidekiq.org/products/enterprise.html)) and [Bootsnap](https://github.com/Shopify/bootsnap) with [Grape](https://github.com/ruby-grape/grape).

The error happens when there is a class that inherits from `Grape::API` (here being `MyApi::Thing`) and a 
nested class (here `MyApi::Thing::NestedThing`). When the nested class is loaded an `uninitialized constant` error
is raised when app is booted with `sidekiqswarm` and Bootsnap is enabled (which is a Rails default).

## Setup

`bundle install`

Since this uses `sidekiq-pro` you might need to sidekiq enterprise key setup in `SIDEKIQ_ENT_KEY` environment variable.

## Reproduce error

To reproduce the error, simple run `sidekiqswarm` or `SIDEKIQ_PRELOAD_APP=1 sidekiqswarm` (the later fails a bit
faster and exits sidekiqswarm, which can be more convenient to test/debug)

This is how the error look like:

```
[swarm] uninitialized constant #<Class:0x00007fbbfc58e340>::NestedThing
Did you mean?  MyApi::Thing::NestedThing
[swarm] /home/guigs/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/grape-1.6.2/lib/grape/api.rb:87:in `const_get'
/home/guigs/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/grape-1.6.2/lib/grape/api.rb:87:in `const_missing'
```

The error seems to happen regardless of environment (development or production).

## When the error does not happen

When app is started with any of those, the error does not happen:

* `rails server`
* `sidekiq`
* `DISABLE_BOOTSNAP_LOAD_PATH_CACHE=1 sidekiqswarm`

It also does not happen if `require 'bootsnap/setup'` is removed from `config/boot.rb`.

## Possible fix

The error does not happen if `bin/sidekiqswarm` script is modified to include `require 'bootsnap/setup'` after `require 'bundler/setup'`.

But then it requires explicit set of `BOOTSNAP_CACHE_DIR` env variable. For example:

`BOOTSNAP_CACHE_DIR=tmp/cache sidekiqswarm`

A proper fix should detect if bootsnap is installed and not disabled before requiring it and somehow provide or use default value for `BOOTSNAP_CACHE_DIR`.
