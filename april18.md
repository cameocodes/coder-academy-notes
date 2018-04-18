# April 18th
Covering
- [Geocoding](#geocoding)
- [User Profiles](#recreating-instagram-with-profiles)

### Geocoding
- Putting a Google Maps frame inside a website is done by using iframes (inline frames), which runs the Google Maps website inside the frame
- iframes can be a security risk
- `gem geocoder` for geocoding solution
- `gem country_select`


### Recreating Instagram with Profiles
Steps wrapped in () are for installing Bootstrap during the process.
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
- Add `<p class="notice"><%= notice %></p>` and `<p class="alert"><%= alert %></p>` to `app/views/layouts/application.html.erb`
- `rails g devise User` creates the User model, migration file and routes using devise
- Edit devise migration file to include modules required if necessary
- `rails g model Profile first_name last_name latitude:decimal longitude:decimal user:references street_address city postcode state country_code` to create profile model
- `rails db:migrate`
- `has_one :profile` in apps > models > user.rb to create relationship between user > profile
- Add to routes:
```ruby
  get '/profile', to: 'profiles#show'
  get '/profile/edit', to: 'profile#edit'
  post '/profile', to: 'profiles#create'
  patch '/profile', to: 'profiles#update'
```
- `rails routes | grep profile` will filter all routes to only show ones that include profile
- create `profiles_controller`:
```ruby
class ProfilesController < ApplicationController

    def show
        redirect_to :root unless user_signed_in?
        @profile = current_user.profile
    end

    def edit
        @profile = Profile.find_or_initialize_by(set_profile)
        @profile.user = current_user
    end

    def update
        @profile = current_user.profile

        if @profile.update(profile_params)
            flash[:notice] = "Profile updated"
            redirect_to profile_path
        else
            flash[:alert] = "Could not save profile"
            redirect_to root_path
        end
    end

    def create
        @profile = Profile.new(profile_params)
        @profile.user = current_user

        if @profile.save 
            flash[:notice] = "Profile updated"
            redirect_to root_path
        else
            flash[:alert] = "Could not save profile"
            redirect_to :back
        end
    end


    private
    def profile_params
        params.require(:profile).permit(:first_name, :last_name, :street_address, :city, :postcode, :state, :country_code)
    end

    def set_profile
        {user: current_user}
    end
end
```
- `redirect_to :root unless user_signed_in?` in the `show` action of `profiles_controller.rb` will prevent user from seeing profile unless logged in
- Add to `profile.rb` model in app > models to make fields a requirement for save:
```ruby
validates(
    :street_address, 
    :city, 
    :postcode, 
    :state,
    :first_name,
    :last_name,
    :country_code,
    presence: true)
```
- Add the following to `profile.rb` model in app > model (below validates()) to allow `geocoder` gem to figure out the long/lat of the address
```ruby
geocoded_by :full_address
after_validation :geocode
```
- Creating the below in `profile.rb` model in app > model will allow you to call `profile.full_address` instead of needing to call each individual part of address
```ruby
def full_address
  # "120 Spencer Street, Melbourne, Victoria, 3000, AU"
  "#{street_address}, #{city}, #{state}, #{postcode}, #{country_code}"
end
```
- Creating the below in `profile.rb` model in app > model will allow you to call `profile.full_name` instead of using `profile.first_name` and `profile_last_name`
```ruby
def full_name
  "#{first_name} #{last_name}"
end
``` 
- add the following to `application_helper.rb` in app > helpers to create `google_map_image_tag` helper 
```ruby
def google_map_image_tag(profile)
  image_tag "http://maps.googleapis.com/maps/api/staticmap?center=#{profile.latitude},#{profile.longitude}&zoom=14&size=400x400&sensor=false"
end
```


###  Params can be passed through:
- Forms
- URL params (?question=what%20are%20params&filter=20)
- Routes