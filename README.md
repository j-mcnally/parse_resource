ParseResource 
=============

ParseResource makes it easy to interact with Parse.com's REST API. It adheres to the ActiveRecord pattern. ParseResource is fully ActiveModel compliant, meaning you can use validations and Rails forms.

Ruby/Rails developers should feel right at home.

If you're used to `Post.create(:title => "Hello, world", :author => "Octocat")`, then this is for you.

Features
---------------
*   ActiveRecord-like API, almost no learning curve
*   Validations
*   Rails forms and scaffolds **just work**

Use cases
-------------
*   Build a custom admin dashboard for your Parse.com data
*   Use the same database for your web and native apps
*   Pre-collect data for use in iOS and Android apps


Installation
------------

Include in your `Gemfile`:

```ruby
gem "parse_resource", "~> 1.7.1"
```

Or just gem install:

```ruby
gem install parse_resource
```

Create an account at Parse.com. Then create an application and copy the `app_id` and `master_key` into a file called `parse_resource.yml`. If you're using a Rails app, place this file in the `config` folder.

```yml
development:
  app_id: 1234567890
  master_key: abcdefgh

test:
  app_id: 1234567890
  master_key: abcdefgh

production:
  app_id: 1234567890
  master_key: abcdefgh
```

You can create separate Parse databases if you want. If not, include the same info for each environment.

In a non-Rails app, include this somewhere (preferable in an initializer):


```ruby
ParseResource::Base.load!("your_app_id", "your_master_key")
```


Usage
-----

Create a model:

```ruby
class Post < ParseResource::Base
  fields :title, :author, :body

  validates_presence_of :title
end
```

If you are using version `1.5.11` or earlier, subclass to just `ParseResource`--or just update to the most recent version.

Creating, updating, and deleting:

```ruby
p = Post.new

# validations
p.valid? #=> false 
p.errors #=> #<ActiveModel::Errors:0xab71998 ... @messages={:title=>["can't be blank"]}> 
p.title = "Introducing ParseResource" #=> "Introducing ParseResource" 
p.valid? #=> true 

# setting more attributes, then saving
p.author = "Alan deLevie" 
p.body = "Ipso Lorem"
p.save #=> #<Post:0xab74864 ... > 

# checking the id generated by Parse's servers
p.id #=> "QARfXUILgY" 
p.updated_at #=> nil 
p.created_at #=> "2011-09-19T01:32:04.973Z" # does anybody want this to be a DateTime object? Let me know.

# updating
p.title = "[Update] Introducing ParseResource"
p.save
p.updated_at #=> "2011-09-19T01:32:37.930Z" # more magic from Parse's servers

# destroying an object
p.destroy #=> nil 
p.title #=> nil
```

Finding:

```ruby
posts = Post.where(:author => "Arrington")
# the query is lazy loaded
# nothing gets sent to the Parse server until you run #all, #count, or any Array method on the query 
# (e.g. #first, #each, or #map)

posts.each do |post|
  "#{post.title}, by #{post.author}"
end

posts.map {|p| p.title} #=> ["Unpaid blogger", "Uncrunched"]

id = "DjiH4Qffke"
p = Post.find(id) #simple find by id

# you can chain method calls, just like in ActiveRecord
Post.where(:param1 => "foo").where(:param2 => "bar").all

# destroy all objects
Post.destroy_all

# limit the query
posts = Post.limit(5).where(:foo => "bar")
posts.length #=> 5

# get a count
Post.where(:bar => "foo").count #=> 1337
```

Users

```ruby
# app/models/user.rb
class User < ParseUser
end

# create a user
user = User.new(:username => "adelevie")
user.password = "asecretpassword"
user.save
# after saving, the password is automatically hashed by Parse's server
# user.password will return the unhashed password when the original object is in memory
# from a new session, User.where(:username => "adelevie").first.password will return nil

# check if a user is logged in
User.authenticate("adelevie", "foooo") #=> false
User.authenticate("adelevie", "asecretpassword") #=> #<User...>


# A simple controller to authenticate users
class SessionsController < ApplicationController
  def new
  end
  
  def create
    user = User.authenticate(params[:username], params[:password])
    if user
      session[:user_id] = user.id
      redirect_to root_url, :notice => "logged in !"
    else
      flash.now.alert = "Invalid username or password"
      render "new"
    end
  end
  
  def destroy
    session[:user_id] = nil
    redirect_to root_url, :notice => "Logged out!"
  end

end

```

If you want to use parse_resource to back a simple authentication system for a Rails app, follow this [tutorial](http://asciicasts.com/episodes/250-authentication-from-scratch), and make some simple modifications.


Associations

```ruby
class Post < ParseResource::Base
  belongs_to :author
  fields :title, :body
end

class Author < ParseResource::Base
  has_many :posts
  field :name
end

author = Author.create(:name => "RL Stine")
post1 = Post.create(:title => "Goosebumps 1")
post2 = Post.create(:title => "Goosebumps 2")

# assign from parent class
author.posts << post1
author.posts << post2 

# or assign from child class
post3 = Post.create(:title => "Goosebumps 3")
post3.author = author
post3.save

# relational queries
posts = Post.include_object(:author).all
posts.each do |post|
	puts post.title 
	# because you used Post#include_object, calling post.title won't execute a new query
	# this is similar to ActiveRecord's eager loading
end
```

Documentation
-------------
[Here](http://rubydoc.info/gems/parse_resource/)

To-do
--------------
*   User authentication
*   Better documentation
*   ~~Associations~~
*   Callbacks
*   Push notifications
*   Better type-casting
*   HTTP request error handling

User authentication is my top priority feature. Several people have specifically requested it, and Parse just began exposing [User objects in the REST API](https://www.parse.com/docs/rest#users).

Let me know of any other features you want.


Contributing to ParseResource
-----------------------------

*   Check out the latest master to make sure the feature hasn't been implemented or the bug hasn't been fixed yet
*   Check out the issue tracker to make sure someone already hasn't requested it and/or contributed it
*   Fork the project
*   Start a feature/bugfix branch
*   Commit and push until you are happy with your contribution
*   Make sure to add tests for it. This is important so I don't break it in a future version unintentionally.
*   Create `parse_resource.yml` in the root of the gem folder. Using the same format as `parse_resource.yml` in the instructions (except only creating a `test` environment, add your own API keys.
*   Please try not to mess with the Rakefile, version, or history. If you want to have your own version, or is otherwise necessary, that is fine, but please isolate to its own commit so I can cherry-pick around it.

Copyright
---------

Copyright (c) 2011 Alan deLevie. See LICENSE.txt for
further details.

