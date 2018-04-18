[< Back](README.md)

# April 16th
## Covering:
- [Authentication Using Devise](#authentication-using-devise)
- [Creating An Instagram Clone](#creating-an-instagram-clone)

---

## Authentication using Devise
Website requires:
- login/logout/sign up
- users table
- email
- password (secure - hashed/salted)
- When dealing with passwords use [Devise](https://github.com/plataformatec/devise) gem: 
    - Database Authenticatable: hashes and stores a password in the database to validate the authenticity of a user while signing in. The authentication can be done both through POST requests or HTTP Basic Authentication.
    - Omniauthable: adds [OmniAuth](https://github.com/omniauth/omniauth) support.
    - Confirmable: sends emails with confirmation instructions and verifies whether an account is already confirmed during sign in.
    - Recoverable: resets the user password and sends reset instructions.
    - Registerable: handles signing up users through a registration process, also allowing them to edit and destroy their account.
    - Rememberable: manages generating and clearing a token for remembering the user from a saved cookie.
    - Trackable: tracks sign in count, timestamps and IP address.
    - Timeoutable: expires sessions that have not been active in a specified period of time.
    - Validatable: provides validations of email and password. It's optional and can be customized, so you're able to define your own validations.
    - Lockable: locks an account after a specified number of failed sign-in attempts. Can unlock via email or after a specified time period.
- [Devise wiki how-to](http://github.com/plataformatec/devise/wiki/How-Tos) 
- Keep profile details (name, address, contact etc) in a separate table to the user account table
- Usernames aren't necessary, just make user log in with email address (unless social media, then offer username)

---

## Creating an Instagram Clone
Steps wrapped in () are for installing Bootstrap in the process.
- `rails new instarails`
- `gem 'devise'` to gemfile and `gem 'rspec-rails', '~> 3.7'` to :development and :test group
- (add `gem 'bootstrap', '~> 4.1.0'` and `gem 'jquery-rails'` to Gemfile if adding Bootstrap)
- `bundle install`
- (`@import "bootstrap";` in `application.css` and rename to `.scss`, remove all `*= require` from file)
- (add `//= require jquery3`, `//= require popper` and `//= require bootstrap-sprockets` to `application.js` file)
- `rails g rspec:install` installs rspec
- `rails g devise:install` installs devise
- In `config/environments/development.rb` add `config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }`
- Make sure the root is defined
- add `<p class="notice"><%= notice %></p>` and `<p class="alert"><%= alert %></p>` to `app/views/layouts/application.html.erb`
- `rails g devise User` creates the User model, migration file and routes using devise
- edit devise migration file to include modules required if necessary
- `rails db:migrate`
- tests for User model located in spec > models > user_spec.rb
- `rails g controller Home index` to create index
- `before_action :authenticate_user!, except: [:index]` in applicaton_controller.rb to ensure that user needs to log in to be able to access any section of the website
- Navbar with login/signup and logout buttons in application view
- `rails g scaffold Media image_data:text description:text user:references`
- `rails db:migrate` to migrate new table
- Remove `user_id` from permitted params in `media_controller.rb`
- Remove `user_id` field from `_form.html.erb` in views > media
- `image_tag` helper will convert url to img
- partials folder inside views folder, files begin with `_filename.html.erb`, loaded in to page with `<%= render 'partials/filename' %>`
