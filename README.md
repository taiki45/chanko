# Chanko [![Build Status](https://travis-ci.org/cookpad/chanko.png)](https://travis-ci.org/cookpad/chanko) [![Code Climate](https://codeclimate.com/github/cookpad/chanko.png)](https://codeclimate.com/github/cookpad/chanko) [![Coverage Status](https://coveralls.io/repos/cookpad/chanko/badge.png?branch=master)](https://coveralls.io/r/cookpad/chanko)

Chanko provides a simple framework for rapidly and safely prototyping new
features in your production Rails app, and exposing these prototypes to
specified segments of your user base.

With Chanko, you can release many concurrent features and independently manage
which users see them. If there are errors with any chanko, it will be
automatically removed, without impacting your site.


## Requirements
* Ruby >= 1.8.7
* Rails >= 3.0.10


## Usage
Add to your Gemfile.

```ruby
gem "chanko"
```

## Files
Chanko provides a generator to create a template of unit.

```
$ rails generate chanko:install example_unit
```

## Invoke
You can invoke the logics defined in your units via `invoke` and `unit` methods.
In controller class context, `unit_action` utility is also provided.
The block passed to `invoke` is a fallback executed if any problem occurs in invoking.

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  unit_action :example_unit, :show

  def index
    invoke(:example_unit, :index) do
      @users = User.all
    end
  end
end
```

```
-# app/views/examples/index.html.slim
= unit.helper_method
= invoke(:example_unit, :render_example)
```

```
-# app/units/example_unit/views/_example.html.slim
= foo
```

## Unit

### module
You can define your MVC code here.

```ruby
# app/units/example_unit/example_unit.rb
module ExampleUnit
  include Chanko::Unit
  ...
end
```

### active_if
This block is used to decide if this unit is active or not.
`context` is the receiver object of `invoke`.
`options` is passed via `invoke(:foo, :bar, :active_if_options => { ... })`.
By default, this is set as `active_if { true }`.

```ruby
active_if do |context, options|
  true
end
```

### raise_error
By default, any error raised in production env is ignored.
`raise_error` is used to force an unit to raise up errors occured in invoking.
You can force all units to raise up errors by `Config.raise_error = true`.

```ruby
raise_error
```

### function
In controller or view context, you can call functions defined by `function`
via `invoke(:example_unit, :function_name)`.

```ruby
scope(:controller) do
  function(:show) do
    @user = User.find(params[:id])
  end

  function(:index) do
    @users = User.active
  end
end
```

### render
The view path app/units/example_unit/views is added into view_paths in invoking.
So you can render app/units/example_unit/views/_example.html.slim in invoking.

```ruby
scope(:view) do
  function(:render_example) do
    render "/example", :foo => hello("world")
  end
end
```

### models
In models block, you can expand model features by `expand` method.
The expanded methods are available via unit proxy like `User.unit.active`,
and `User.find(params[:id]).unit.active?`, and so on.

```ruby
models do
  expand(:User) do
    scope :active, lambda { where(:deleted_at => nil) }

    def active?
      deleted_at.nil?
    end
  end
end
```

### shared
You can call methods defined by `shared` in invoking.

```ruby
shared(:hello) do |world|
  "Hello, #{world}"
end
```

### helpers
You can call helpers in view via unit proxy like `unit.helper_method`.

```ruby
helpers do
  def helper_method
    "helper method"
  end
end
```


## Example
https://github.com/cookpad/chanko/tree/master/spec/dummy  
Chanko provides an example rails application in spec/dummy directory.

```
$ git clone git@github.com:cookpad/chanko.git
$ cd chanko/spec/dummy
$ rails s
$ open http://localhost:3000
```
