[< Back](README.md)

# April 24th
## Covering:
- [Stripe](#stripe)
- [Deploying to Heroku](#deploying-to-heroku)
- [References](#references)

---
### Stripe

- Going to use PostgreSQL for the database this time
- Download [Postgres.app](http://postgresapp.com/) for GUI interface
- `gem install pg` in terminal will install postgres
- ((`gem install pg -- --with-pg-config=/Applications/Postgres.app/Contents/Versions/latest/bin/pg_config` in terminal will install pg gem if above doesn't work))
- `rails new sneakers_pg_stripe --database=postgresql`
- Add Devise to Gemfile and Rspec to `:development, :test` group in Gemfile
- Bundle, then `rails g rspec:install && rails g devise:install`
- Change development database name to `sneakers_development` and test database name to `sneakers_test` in `database.yml` (config folder)
- Change production database name to `sneakers` and change username to your name
- `rails db:create` -- postgres is different to sqlite and actually requires you to create the database first
- `rails g scaffold Sneaker name description:text price:decimal`
- `rails db:migrate`
- Run `rails s` to test database connection
- When referencing other tables, make sure the other table is already created before referencing otherwise postgres will error out
- Fill seed file with shoes with name, description, price then `Sneaker.create!(sneakers) { |s| p s.name }`
- `rails db:seed`
- `rails g devise User`
- `gem 'stripe'` in Gemfile and `gem 'dotenv-rails', '~> 2.4'` to `:development, :test` group in Gemfile
- Bundle
- Create `.env.development` file and add `.env and .env*` to `.gitignore` file
- Add to `.env.development` file: (include your own keys from Stripe website)
```ruby
PUBLISHABLE_KEY=key
SECRET_KEY=key
```
- `rails g controller Charges`
- Follow [Stripe guide](https://stripe.com/docs/checkout/rails) to set up form, views and controller
- Add `charges_controller.rb` details to `sneaker_controller.rb`
- `before_action :set_sneaker, only: [:show, :edit, :update, :destroy, :charge]` at top of `sneaker_controller.rb` to allow charge method access to `set_sneaker`
- Add to routes:
```ruby
resources :sneakers do
    member do
      post 'charge'
    end
  end 
```
- `rails g migration AddStripeIDToUser stripe_id`
- Migrate
- `before_action :authenticate_user!, only: [:charge]` in top of `sneakers_controller.rb`
- Put the form into a partial and render it inside `show.html.erb`
- Add `<%= stylesheet_link_tag "https://checkout.stripe.com/v3/checkout/button.css"%>` in application layout
- Add inside `show.html.erb`:
```ruby
<% if user_signed_in? %>
<%= render 'partials/charge_form', sneaker: @sneaker, user: current_user %>
<% else %>
<%= link_to new_user_session_path, class: 'stripe-button-el' do%>
  <span style="display: block; min-height: 30px;">Pay with Card</span>
<% end %>
<% end %>
```
- Look in to using Devise to redirect back after signing in
- Add `data-amount="<%= user.email %>"` in the charge form partial
- In `sneakers_controller.rb` add:
```ruby
  def charge 
    
    if current_user.stripe_id.nil?
    customer = Stripe::Customer.create(
      email: params[:stripeEmail],
      source: params[:stripeToken]
    )
    current_user.stripe_id = customer.id
    current_user.save!
    
    end

      charge = Stripe::Charge.create(
        customer: current_user.stripe_id,
        amount: @sneaker.price,
        description: @sneaker.description,
        currency: 'aud'
      )

      current_user.charges << Charge.new(charge_id: charge.id)

      flash[:notice] = "Payment Made!"
      redirect_back fallback_location sneakers_path
      

    rescue Stripe::CardError => e
      flash[:error] = e.message
      redirect_to new_charge_path
    end   
```
---
### Deploying to Heroku
- Navigate to project folder
- `heroku login` with your Heroku account details
- Ensure `gem pg` is in your Gemfile (run `bundle install` if you have to add it)
- `git init`
- `git add .`
- `git commit -m "First commit"`
- `git status` to ensure all files are up to date
- `heroku create` will create a remote app on Heroku
- `git push heroku master` to push project to Heroku
- Check for any errors!
- `heroku run rails db:migrate` to create database on Heroku
- `heroku run` will allow you to run terminal commands on the Heroku app
- `heroku run rails db:seed` if you need to seed the database

---
### References
- [Stripe Gem Github](https://github.com/stripe/stripe-ruby)
- [PostreSQL website](http://postgresapp.com/)
- [Stripe Guide](https://stripe.com/docs/checkout/rails)
- [Stripe Mock Button](https://mattarkin.com/pretty-custom-integration-stripe-button/)
- [Webhooks on Stripe](https://stripe.com/docs/webhooks)
- [Heroku Config Vars](https://devcenter.heroku.com/articles/config-vars)