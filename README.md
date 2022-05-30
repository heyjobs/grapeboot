# README

This is a minimal Rails app to demonstrate an uninitialized constant error when using `sidekiqswarm` (from [sidekiq-ent](https://sidekiq.org/products/enterprise.html)) and [Bootsnap](https://github.com/Shopify/bootsnap) with [Grape](https://github.com/ruby-grape/grape).

The error happens when there is a class that inherits from `Grape::API` (here being `Thing`) and a 
nested class (here `Thing::NestedThing`). When the nested class is loaded an `uninitialized constant` error
is raised when app is booted with `sidekiqswarm` and Bootsnap is enabled (which is a Rails default).

## Setup

`bundle install`

## Reproduce error

To reproduce the error, simple run `./bin/sidekiqswarm`

This is how the error look like:

```
/home/guigs/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/grape-1.6.2/lib/grape/api.rb:87:in `const_get': uninitialized constant #<Class:0x00007f5b1c6cb2d0>::NestedThing (NameError)
Did you mean?  Thing::NestedThing
        from /home/guigs/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/grape-1.6.2/lib/grape/api.rb:87:in `const_missing'
        from /home/guigs/grapeboot/config/routes.rb:8:in `block in <main>'
        from /home/guigs/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/actionpack-7.0.3/lib/action_dispatch/routing/route_set.rb:428:in `instance_exec'
        from /home/guigs/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/actionpack-7.0.3/lib/action_dispatch/routing/route_set.rb:428:in `eval_block'
        from /home/guigs/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/actionpack-7.0.3/lib/action_dispatch/routing/route_set.rb:410:in `draw'
        from /home/guigs/grapeboot/config/routes.rb:1:in `<main>'
        from /home/guigs/.rbenv/versions/3.0.3/lib/ruby/gems/3.0.0/gems/bootsnap-1.11.1/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:39:in `load'

```

The error seems to happen regardless of environment (development or production).

## When the error does not happen

When app is started with any of those, the error does not happen:

* `rails server`
* `DISABLE_BOOTSNAP_LOAD_PATH_CACHE=1 ./bin/sidekiqswarm`
* `SIDEKIQ_PRELOAD= ./bin/sidekiqswarm`

It also does not happen if `require 'bootsnap/setup'` is removed from `config/boot.rb`.

## Possible fix

The error does not happen if `bin/sidekiqswarm` script is modified to include `require 'bootsnap/setup'` after `require 'bundler/setup'`.

But then it requires explicit set of `BOOTSNAP_CACHE_DIR` env variable. For example:

`BOOTSNAP_CACHE_DIR=tmp/cache ./bin/sidekiqswarm`

A proper fix should detect if bootsnap is installed and not disabled before requiring it and somehow provide or use default value for `BOOTSNAP_CACHE_DIR`.
