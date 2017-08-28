# Authentication (Devise) / Authorization (CanCanCan)

## Adding Authentication
1. Add following to Gemfile
```ruby
gem 'cancancan'
gem 'devise'
```

2. run `bundle install`
3. run `rails g devise:install`
  > Follow the instructions that Devise gives to complete installation
4. run `rails g devise User`
5. Add relationships between models
```bash
  # example
  rails g migration AddUserToPosts user:references
  rails g migration AddPostsToUsers post:references
```
  > Don't forget to MIGRATE
6. Add dynamic links to Log-in/out
```ruby
<% if user_signed_in? %>
  Logged in as <strong><%= current_user.email %></strong>.
  <%= link_to "Logout", destroy_user_session_path, method: :delete %>
<% else %>
  <%= link_to "Sign up", new_user_registration_path %> |
  <%= link_to "Login", new_user_session_path  %>
<% end %>
```

7. Add authentication command to controllers.
```ruby
  before_action :authenticate_user!
```

## Adding fields to Devise Model
8. Create migration to add columns to User schema
```bash
  rails g migration AddUsernameToUsers username:string
```
  > MIGRATE!

9. Expose devise views if not yet available.
```bash
  rails g devise:views
``` 

10. Update forms with needed input fields. (`devise/registrations` has the Register User field)

11. Overwrite devise input sanitizer in the `application_controller`

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  before_action :configure_permitted_parameters, if: :devise_controller?
  
  protected
  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:username])
  end
end
```

### Adding Authorization
12. Create a CanCanCan ability using the command line
```bash
  rails g cancan:ability
```

13. Add logic to initialize command
```ruby
class Ability
  include CanCan::Ability

  def initialize(user)
    user ||= User.new # guest user (not logged in)
    can :read, Post

    can [:create, :update, :destroy], Post do |post|
      post.user == user
    end
  end
end
```

14. Load ability into the controller using `load_and_authorize_resource`
```
load_and_authorize_resource only: [:edit, :update, :destroy]
```

15. Add `rescue_from` method to ApplicationController
```ruby
  rescue_from CanCan::AccessDenied do |exception|
    flash[:notice] = "Access denied!"
    flash.keep(:notice)
    redirect_to root_url
  end
```

16. Remove unnecessary links by using `can?` method in erb
```ruby
 <% if can? :update, post %>
  <td><%= link_to 'Edit', edit_post_path(post) %></td>
<% end %>
      
<% if can? :destroy, post %>
  <td><%= link_to 'Destroy', post, method: :delete, data: { confirm: 'Are you sure?' } %></td>
<% end %>
```
