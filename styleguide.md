**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [COPASS STYLEGUIDE](#copass-styleguide)
	- [Git](#git)
		- [Gtihub issues management](#gtihub-issues-management)
	- [Ruby Best Practice](#ruby-best-practice)
	- [Rails Best Practice](#rails-best-practice)
		- [1. routes.rb : use collection and member:](#1-routesrb--use-collection-and-member)
		- [2. Active Record : prefer `.find_by_attr` to `.where.first`, and `first_or_create` instead of `find_or_create`](#2-active-record--prefer-find_by_attr-to-wherefirst-and-first_or_create-instead-of-find_or_create)
		- [3. Test if value is nil? : .blank?](#3-test-if-value-is-nil--blank)
		- [4. MVC Balance](#4-mvc-balance)
		- [6. User et abuse `# TODO`, `# FIXME`, `# OPTIMIZE`](#6-user-et-abuse-#-todo-#-fixme-#-optimize)
		- [7. Test loggued out](#7-test-loggued-out)
		- [8. Add Contraints and default value on ActiveRecord Migrations](#8-add-contraints-and-default-value-on-activerecord-migrations)
		- [9. Prefer `_path` instead of `_url`](#9-prefer-_path-instead-of-_url)
		- [10. Use Events](#10-use-events)
		- [11. Use model.method and model.method!](#11-use-modelmethod-and-modelmethod!)
	- [Font-End](#font-end)
		- [1. We use Bootstrap, so use it!](#1-we-use-bootstrap-so-use-it!)
		- [2. Use rails helpers](#2-use-rails-helpers)
		- [3. Use CSS `text-transform: uppercase` property](#3-use-css-text-transform-uppercase-property)
		- [4. Use the data attributes](#4-use-the-data-attributes)
		- [5. Carefull with the Coffeescript](#5-carefull-with-the-coffeescript)
		- [6. Use jQuery `on` method instead of `bind`, `delegate`, `live`](#6-use-jquery-on-method-instead-of-bind-delegate-live)
		- [7. Avoid useless parenthesis in coffeescript](#7-avoid-useless-parenthesis-in-coffeescript)

## COPASS STYLEGUIDE

Here are all the guidelines that you should consider when developping on Copass.

### Git

You should read the awesome [Github flow](http://scottchacon.com/2011/08/31/github-flow.html) which nicely explains the idea of how to use Git in Copass.

#### Gtihub issues management

We use Github to track issues.

When an issue is created, it should be labeled. Here are the labels with their category:

  1. Category: Action => What is to be done

    - A: Incident - To watch/reproduce
    - A: Question
    - A: To design
    - A: To discuss
    - A: Ready
    - A: Wontfix

  2. Category: Priority => When it should be done

    - P0: URGENT
    - P1: Next Push To Prod
    - P2: Nice to have
    - P∞: Ideas for the future

  3. Category: Subject => What it is about

    - S: Payment System

  4. Category: Type => What kind of issue it is

    - T: Bug
    - T: Fixing Friday
    - T: Major Enhancement
    - T: Malfunction
    - T: Minor Enhancement
    - T: Technical Enhancement

The idea is that normally every issue should be assigned to only 1 label per category (without being too strict of course). Then we can easily filter labels by selecting eg. 1 action + 1 type + 1 priority etc.

Then, when someone starts developing it, it is recommended to create a branch forked from master with the name `<issue_number>/<type>/<slug>`. Eg. 192/feat/better-invoices or 204/bug/upload-issue

It doesn't have to be uploaded to Github, unless someone else will need your branch to develop further on it.

Finally, when the issue is ready to be merged to master, your last commit message should (if it applies) contain `Fixes #204` or `Closes #192` to reference the issue, and automatically close it on Github.

### Ruby Best Practice

Concerning Ruby, we will follow the [Github styleguide](https://github.com/styleguide/ruby) which is pretty clear on how to write Ruby.

### Rails Best Practice

#### 1. routes.rb : use collection and member:

This code generates the commented routes:

```ruby
  resources :groups do
    collection do
      get :list # list_groups GET /groups/list => groups#list
    end
    member do
      put :credit # credit_group POST /groups/:id/credit => groups#credit
    end
  end
```

It is important to use these as much as possible : they are the best practices.
In general, avoid using string match in the router, but symbol instead. If you have more than one parameter to send, consider using nested resources.

```ruby
  resources :copassers, :only => [:index, :show, :new, :create] do
    member do
      # Bad
      get '/cospace_interactions/:cospace_slug' => 'copassers#get_cospace_interactions'
      # Alright but very verbose
      get :cospace_interactions, :action => :get_cospace_interactions
      # Good but name too long
      get :get_cospace_interactions
      # Good (rename the action in the controller)
      get :cospace_interactions 
      # And the cospaces_slug goes in params : /copassers/:id/cospace_interactions?cospace_slug=:slug
      get :friends
    end
    collection do
      get :list
      post :invite
    end
  end
```

#### 2. Active Record : prefer `.find_by_attr` to `.where.first`, and `first_or_create` instead of `find_or_create`

`Cospace.where(:slug=>’mutinerie’)` returns an array of results. Even when you are sure that the query will return only one result, you'll need to write `mutinerie = Cospace.where(:slug=>’mutinerie’).first`

In that case, it is better to use `Cospace.find_by_slug(‘mutinerie’)` which does exactly the same. Works with any attribute of the model, and it can combine serveral attributes:
`Copasser.find_by_fname_and_lname(‘Augustin’, ‘Riedinger’)`

There is an exception, when using `find_or_create` which is now depreciated.

Instead, you should do
```ruby
GroupMember.where(:member_id => 4, :group_id => 7).first_or_create
# and
GroupMember.where(:member_id => 4, :group_id => 7).first_or_initialize
# and you can set addition values to the new model:
GroupMember.where(:member_id => 4, :group_id => 7).first_or_initialize({:level => 1})
```


#### 3. Test if value is nil? : .blank?

You should use `obj.blank?` if you want to check if an object is `nil`. Indeed:
```ruby
array = []
array.nil? # => false
array.blank? # => true
```

#### 4. MVC Balance

How to structure your MVC always leads to infinite debates. Here are the main ideas:

- **Controllers** should be as short as possible. Only retrieves information, call model methods that need to be executed and report how it went. The logic *should* be immediately readable
- **Models** can be much (much) longer, containing all the methods of our object. See it as *what capabilities do I want to offer my object?*. Keep the idea of **blackboxing**, as much as you can.

  Eg. `cospace.make_admin(copasser)` could be tested by 
  
```ruby
cospace.is_admin?(copasser) == false
cospace.make_admin(copasser)
cospace.is_admin?(copasser) == true
```

- **Views** must only read data, and only data that are in @variables (with an exception for current_copasser). The only logic it contains is `if else end` or `list.each do end` and its derivates. There **must** not be any variable assignment in the view, it should be done in the controller instead. There is an exception for partials, in the matter of **blackboxing**, where it can be convenient to assign variables for better clarity.

  Eg.
  
```ruby
# In view: Bad
- copasser = @cospace.admins.first #should be @copasser in controller instead
= link_to copasser.name, copasser_path(copasser)

# In partial: Ok for blackboxing

- copasser = notification.receipt.sender
= link_to copasser.name, copasser_path(copasser)

```

Model methods should usually be written in two ways:

```ruby
class Cospace < ActiveRecord::Base
  # If it is for accessing information, it will return the informations.
  def full_address
    return self.geo_address.to_s
  end

  # If it is to process an action, it should return an hash with :status => true/false and :message to explain what happened

  def make_admin(copasser, by)
    if self.admins.include? by
      if !self.admins.include? copasser
        self.admins << copasser
        if self.save
          Event.trigger('promoted_cospace_admin', :copasser => copasser, :cospace => self, :by => by)
          return {
            :status => true, # Action performed
            :message => "#{copasser.name} was made admin of #{self.name}"
          }
        else
          return {
            :status => false, # Action not performed
            :message => "An error occured, please try again"
          }
        end
      else
        return {
          :status => false, # Action not performed
          :message => "#{copasser.name} is already admin of #{self.name}"
        }
      end
    else
      return {
        :status => false, # Action not performed
        :message => "You are not admin of #{self.name}"
      }
    end
  end
end
```


#### 6. User et abuse `# TODO`, `# FIXME`, `# OPTIMIZE`

There is nothing wrong about doing something later, especially when it is about optimizing the code, but writing it down helps finding them back with the `rake notes` (cf. [doc](http://guides.rubyonrails.org/command_line.html#notes))

Eg.
```ruby
# app/models/cospace.rb:
  * [  4] [TODO] remove alias

# app/models/cospace.rb

  # TODO: remove alias
  alias_attribute :avatar, :logo
```

#### 7. Test loggued out

Usually while testing, we are in logged in the application. It is important to test the behaviour of your pages when loggued out, to avoid security breaches, or nil values that would crash the app.

#### 8. Add Contraints and default value on ActiveRecord Migrations
Once `rails generate bla_bla_bla` used, a migration file will be created in `/db/migrate`. You should add constraints (`null => true/false)` and default value (`:default => 0`)
Eg. `change_column :cospace, :plan_type_id, :integer, :null => false, :default => 0 `

Note that in the case of a boolean attribute, it often makes sense to add a `:null => false, :default => false` instead of leaving the possibility of having it nil.

#### 9. Prefer `_path` instead of `_url`

```ruby
edit_copasser_path(copasser) # => /copassers/augustin-riedinger/edit
edit_copasser_url(copasser) # => http://localhost:3000/copassers/augustin-riedinger/edit in development
edit_copasser_url(copasser) # => http://dev.copass.org/copassers/augustin-riedinger/edit in staging
```

It is not necessary to use the `_url` suffix among the app.

**Carefull: whenever the content will be sent by mail, the `_url` prefix is *mandatory*!**
This is the case in the whole `/app/views/mailer` folder, but also in the `/app/views/notifications`. In general, be careful with the `*.md` files. 

#### 10. Use Events

For any code that is not related to the logic of the app, but are more falldowns (eg. mails, notifications), we designed an event system (somehow related to the javascript logic... somehow).

To trigger the event, anywhere in the app:

```ruby
hash = {:cospace => cospace, :by => copasser} # For example
Event.trigger('cospace_created', hash) # where hash are all the useful values to the event
```

It will either call the `Event.on_cospace_created` method if it exists, or have a default behaviour which is to create a notification called `cospace_created`, send a mail on this notification, and inform pusher that a notification was created.

#### 11. Use model.method and model.method!

As readable in this [article](http://dablog.rubypal.com/2007/8/15/bang-methods-or-danger-will-rubyist), it is a good practice to write two methods in one each time.

First, write `model.method!`, and don't apply any controll, then `write model.method`, which will apply all the necessary controls, and **call** the `model.method!` so that the same logic is used.

Ex:

```ruby
class Cospace

  def make_admin!(copasser, by=nil)
    self.admins << copasser
    self.save
    Event.trigger('promoted_cospace_admin', :copasser => copasser, :cospace => self, :by => by)
    return {
      :status => true,
      :message => 'Successfully made admin'
    }
  end

  def make_admin(copasser, by)
    if self.is_admin?(by)
      if self.is_admin?(copasser)
        return self.make_admin!(copasser, by)
      else
        return {
          :status => false,
          :message => "#{copasser.name} is already admin of #{self.name}"
        }
      end
    else
      return {
        :status => false,
        :message => "#{by.name} is not admin of #{self.name}"
      }
    end
  end

end

```

This allows us to use the *forcing/dangerous* method in the console, or in the [backend](http://localhost:3000/admin).

Another cool thing we could implement later, is when calling `/cospaces/mutinerie/make_admin?copasser_slug=augustin-riedinger` the `cospace.make_admin` method is called, but, when calling `/cospaces/mutinerie/make_admin?copasser_slug=augustin-riedinger&enforce=true`, `cospace.make_admin!` method is called.

### Font-End 

#### 1. We use Bootstrap, so use it!



#### 2. Use rails helpers

```haml

### link_to and _path helpers

# Bad
%a.btn.btn-primary{:href => '/login'}
  %span.glyphicon.glyphicon-user
  Login

# Still Bad
%a.btn.btn-primary{:href => login_path}
  %span.glyphicon.glyphicon-user
  Login

# Still Bad
= link_to '/login', :class => 'btn btn-primary' do
  %span.glyphicon.glyphicon-user
  Login

# Good
= link_to login_path, :class => 'btn btn-primary' do
  %span.glyphicon.glyphicon-user
  Login
  
### Image Helper
  
# Bad
%img{:src => @copasser.avatar.url(:sq_50_50)}

# Good
= image_tag @copasser.avatar(:sq_50_50)

### More to come

```


#### 3. Use CSS `text-transform: uppercase` property

If the design wants it in upcase, it doesn't mean it should be uppercased in the text, but in the CSS. Indeed, uppercasing the text changes its meaning more than its apprearance CSS). To help you respect this rule, you have several tools to use:

- a `.upcase` class that uppercase the text
- `.btn-primary` and `.btn-secondary` are always uppercased
- In general, you should think at which level is the uppercase applying, and in most cases, it is worth it adding the property to a css class

#### 4. Use the data attributes

Data attribute are very handy. It is always a good idea to use them to describe an element, and have persistent data generated from the server. The haml syntax is the following:

```haml

= link_to 'Action', action_path, :data => {:attribute => 'value'}
# Note that :attribute should not have any - or _
# To have more - in the name two options

= link_to 'Action', action_path, :data => {:attribute => { :subattribute => 'value'}} # cleanest but too long
= link_to 'Action', action_path, :data => {:'attribute-subattribute' => 'value'} # less clean but shorter

```

#### 5. Carefull with the Coffeescript

Coffeescript has arguably some advantages, but also many drawbacks, and things to be aware of.

- You need to add parenthesis for methods:

```ruby
cospace.get_price # works in Ruby
```

```coffeescript
# wrong
cospace.get_price # doesn't execute in coffescript. But doesn't return any error either.

# right
cospace.get_price()
```

#### 6. Use jQuery `on` method instead of `bind`, `delegate`, `live`

See this article if you want to dive into it: [An Introduction To DOM Events](http://coding.smashingmagazine.com/2013/11/12/an-introduction-to-dom-events)

In short, `on` can and should be used instead of the others because there are equivalents:

```javascript

$('selector').bind('eventname', callback);
// ===
$('selector').on('eventname', callback);

$('selector').delegate('subselector','eventname', callback);
// ===
$('selector').on('eventname', 'subselector', callback);

$('selector').live('eventname', callback);
// ===
$(document).on('selector','eventname', callback);

``` 

`bind` associates the event to the `selector`.
`delegate` associates the event to the `selector`, and when triggered, looks for the `subselectors` to call `callback`
`live` is the equivalent of `delegate` at document root level ==> slow 

#### 7. Avoid useless parenthesis in coffeescript

Closing parenthesis can add some complexity and be source of error.

```coffeescript
$('.admin-panel').on('click', '.edit-transaction', (e) ->
  target = $($(this).data('target'))
  target.load($(this).data('remote'), (e) ->
    target.modal('show')
  )
)

# Better

$('.admin-panel').on 'click', '.edit-transaction', (e) ->
  target = $($(this).data('target'))
  target.load $(this).data('remote'), (e) ->
    target.modal('show')


```