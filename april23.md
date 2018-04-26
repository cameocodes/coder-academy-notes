[< Back](README.md)

# April 23rd
## Covering:
- [Using a Mailing Service](#using-a-mailing-service)
- [Common Mailing Servers](#common-mailing-servers)
- [Creating an Instagram Clone with Mailer](#creating-an-instagram-clone-with-mailer)
- [References](#references)
---
### SIGN UP FOR MAILGUN.COM BEFORE CONTINUING
---
## Using a Mailing Service
Reasons you might want to include a mailing service in your web app:
- Mailing list / subscription
- Error messages to devs
- Confirmation emails to users
- Notification emails to users
---
## Common Mailing Servers
- [Mailgun](https://www.mailgun.com/)
- [Sendgrid](https://sendgrid.com/)
---
## Navigating Mailgun
- Sign in > Domains > Sandbox domain
- Click on `Authorised Recipients` > `Invite New Recipient`
- In sandbox mode Mailgun will only send emails to Authorized Recipients
---
## Creating an Instagram Clone with Mailer
- Use the Mailgun gem (SendGrid also have a gem)
- `rails new instarails_mailer`
- `gem 'mailgun-ruby', '~> 1.1', '>= 1.1.9'` and `gem 'devise'` in Gemfile
- `gem 'rspec-rails', '~> 3.7'` and `gem 'dotenv-rails', '~> 2.3'` in the `:development, :test` group
- `bundle`
- `rails g devise:install`
- `rails g rspec:install`
- `rails g devise User`
- `rails db:migrate`
- Create `.env.development` file then add `.env` and `.env.*` files to `.gitignore`
- Add `MAILGUN_API_KEY=your_api_key` to `.env.development` file with secret API key from Mailgun account
- Add `MAILGUN_DOMAIN=your_sandbox_domain` to `.env.development` file with the sandbox domain from Mailgun website
- In `config > environments > development` add:
```ruby
config.action_mailer.delivery_method = :mailgun
  config.action_mailer.mailgun_settings = {
    api_key: ENV.fetch('MAILGUN_API_KEY'),
    domain: ENV.fetch('MAILGUN_DOMAIN'),
  }
```
- [Mailgun setup page](https://app.mailgun.com/app/account/setup) has a curl code snippet to test sandbox from terminal
- `rails s` to make sure everything works so far
- `rails g mailer ContactMailer` to generate mailer files
- Above command generated `mailer.html.erb` AND `mailer.text.erb` - the first is for HTML-based emails, the second is for text-only emails
- Change details in `application_mailer` in `mailers` folder to:
```ruby
default from: ENV.fetch('MAILGUN_EMAIL')
layout 'mailer'
```
- Add your email address to `.env.development` as `MAILGUN_EMAIL` (make sure it is an Authorized email through Mailgun)
- `rails g controllers Pages home contact`
- Define the root path
- In `contact.html.erb` add:
```ruby
<%= form_for :contact do |f| %>
    <p>
        <%= f.label :name %><br/>
        <%= f.text_field :name %>
    </p>

    <p>
        <%= f.label :message %><br/>
        <%= f.text_area :message %>
    </p>

    <%= f.submit %>
<% end %>
```
- Add `post '/contact', to: 'pages#contact_email'` to routes file
- In `pages_controller`: 
```ruby
def contact_email
    user = current_user
    message = email_params[:message]
    name = email_params[:name]
    user_info = {user: user, message: message, name: name}
    ContactMailer.send_contact_email(user_info).deliver_now
    render :contact
  end

  def email_params
    params.require(:contact).permit(:name, :message)
  end
```
- `Deliver_now` vs `deliver_later` : deliver now will deliver now, deliver later will set a job in memory to deliver later. If the rails server shuts down the memory is deleted and the job is never completed. Online servers will save deliver later jobs.

- In `contact-mailer.rb` add:
```ruby
def send_contact_email(user_info)
        recipient = ENV.fetch('MAILGUN_EMAIL')
        @user = user_info[:user]
        @name = user_info[:name]
        @message = user_info[:message]
        date = Time.now.strftime("%B %d, %Y, %A")
        subject = "New user message #{date}"
        mail(to: recipient, subject: subject)
end
```
- Create `send_contact_email.html.erb` and `send_contact_email.text.erb` email templates in `contact_mailer` folder and add:
```
You have received a new message

From: 
<%= current_user.email %>

Name: 
<%= @name %>

Message:
<%= @message %>
```
- `send_contact_email.html.erb` version of the above form can include HTML tags - `<h1>, <p>, <b>` etc.
- At the top of `pages_controller` before actions `before_action :authenitcate_user!, except: [:home]`, will authenticate user login for every page except index.



---

## References:
- [Mailgun Gem Docs](https://github.com/mailgun/mailgun-ruby)
- [Action Mailer Basics](http://guides.rubyonrails.org/action_mailer_basics.html)