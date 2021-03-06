[![Gem](https://img.shields.io/gem/v/graphql-pundit.svg)](https://rubygems.org/gems/graphql-pundit)
[![Build Status](https://travis-ci.org/ontohub/graphql-pundit.svg?branch=master)](https://travis-ci.org/ontohub/graphql-pundit)
[![Coverage Status](https://codecov.io/gh/ontohub/graphql-pundit/branch/master/graph/badge.svg)](https://codecov.io/gh/ontohub/graphql-pundit)
[![Code Climate](https://codeclimate.com/github/ontohub/graphql-pundit/badges/gpa.svg)](https://codeclimate.com/github/ontohub/graphql-pundit)
[![Dependency Status](https://gemnasium.com/badges/github.com/ontohub/graphql-pundit.svg)](https://gemnasium.com/github.com/ontohub/graphql-pundit)
[![GitHub issues](https://img.shields.io/github/issues/ontohub/graphql-pundit.svg?maxAge=2592000)](https://waffle.io/ontohub/ontohub-backend?source=ontohub%2Fgraphql-pundit)

# GraphQL::Pundit



## Installation

Add this line to your application's Gemfile:

```ruby
gem 'graphql-pundit'
```

And then execute:

```bash
$ bundle
```

## Usage

### Add the authorization middleware

Add the following to your GraphQL schema:

```ruby
MySchema = GraphQL::Schema.define do
  ...
  instrument(:field, GraphQL::Pundit::Instrumenter.new)
  ...
end
```

By default, `ctx[:current_user]` will be used as the user to authorize. To change that behavior, pass a symbol to `GraphQL::Pundit::Instrumenter`. 

```ruby
GraphQL::Pundit::Instrumenter.new(:me) # will use ctx[:me]
```

### Authorize fields

For each field you want to authorize via Pundit, add the following code to the field definition:

```ruby
field :email do
  authorize # will use UserPolicy#email?
  resolve ...
end
```

By default, this will use the Policy for the parent object (the first argument passed to the resolve proc), checking for `:email?` for the current user. Sometimes, the field name will differ from the policy method name, in which case you can specify it explicitly:

```ruby
field :email do
  authorize :read_email # will use UserPolicy#read_email?
  resolve ...
end
```

Now, in some cases you'll want to use a different policy, or in case of mutations, the passed object might be `nil`:

```ruby
field :createUser
  authorize! :create, User # or User.new; will use UserPolicy#create?
  resolve ...
end
```

This will use the `:create?` method of the `UserPolicy`. You can also pass in objects instead of a class, if you wish to authorize the user for the specific object.

You might have also noticed the use of `authorize!` instead of `authorize` in this example. The difference between the two is this:

- `authorize` will set the field to `nil` if authorization fails
- `authorize!` will set the field to `nil` and add an error to the response if authorization fails

You would normally want to use `authorize` for fields in queries, that only e.g. the owner of something can see, while `authorize!` would be usually used in mutations, where you want to communicate to the client that the operation failed because the user is unauthorized.

If you still need more control over how policies are called, you can pass a lambda to `authorize`:

```ruby
field :email
  authorize ->(obj, args, ctx) { UserPolicy.new(obj, ctx[:me]).private_data?(:email) }
  resolve ...
end
```

If the lambda returns a falsy value or raises a `Pundit::UnauthorizedError` the field will resolve to `nil`, if it returns a truthy value, control will be passed to the resolve function. Of course, this can be used with `authorize!` as well.

### Scopes

Pundit scopes are supported by using `scope` in the field definition

```ruby
field :posts
  scope
  resolve ...
end
```

By default, this will use the Scope definied in the `PostPolicy`. If you do not want to define a scope inside of the policy, you can also pass a lambda to `scope`. The return value will be passed to `resolve` as first argument.

```ruby
field :posts
  scope ->(_root, _args, ctx) { Post.where(owner: ctx[:current_user]) }
  resolve ->(posts, args, ctx) { ... }
end
```

In case you only want to specify the Policy class containing the Scope explicitly, you can pass the Policy class explicitly:

```ruby
field :posts
  scope PostablePolicy
  resolve ...
end
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/ontohub/graphql-pundit.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

