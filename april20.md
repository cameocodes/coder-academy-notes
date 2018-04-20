[< Back](README.md)

# April 20th
## Covering:
- [Git Refresher](#git-refresher)
- [Creating an Instagram Clone with Pundit](#creating-an-instagram-clone-with-pundit)
- [References](#references)

---
## Git Refresher
#### GIT EARLY AND OFTEN
To add commit:
- `git add .` (or select individual files to stage)
- `git commit -m "[message here]`

To push to remote (i.e. GitHub):
- `git remote add origin [repo location]`
- `git push -u origin master [branch1] [branch1] [branch3]` to push master and branches to remote

To show branches:
- `git branch`

To swap between branches: 
- `git checkout [branch you want to go to]`

To create new branch then checkout to branch:
- `git checkout -b [branch name]`

To merge a branch with master:
- go to master branch
- `git merge devise`

---

## Creating an Instagram Clone with Pundit
#### ON MASTER BRANCH
- `rails new instarails_pundit -T` to create a new rails app without testing suite
- `gem 'rspec-rails', '~> 3.7'` Add Rspec to Gemfile for testing
- `gem 'dotenv-rails'` Add Dotenv to Gemfile for environment variables
- Create `.env.test` file in main folder and add:
```ruby
# use and verify fake email address
USER_EMAIL=name@example.com
USER_PASSWORD=password123
```
- Add to .gitignore:
```ruby
# Environment variables
.env.*
.env
```
- Run `bundle` for Rspec and required gems
- `rails g rspec:install` to install Rspec
- *Commit*
- Checkout to devise branch using `git checkout -b devise`

#### ON DEVISE BRANCH
- Add Devise to gemfile using `gem 'devise'`
- `bundle` for Devise
- `rails g devise:install` to install Devise
- `rails g devise User` to create User model and migration
- Run `rails db:migrate`
- *Commit*
- Add to `seeds.rb`:
```ruby
User.create!({
    email: ENV.fetch('USER_EMAIL'),
    password: ENV.fetch('USER_PASSWORD')
}) { |u| p u.encrypted_password }
```
>To verify with testing in `spec > models > user_spec.rb` add:
>```ruby
>it 'should have matching email' do
>    user = User.new
>    user.email = ENV.fetch('USER_EMAIL')
>    expect(user.email).to eq('name@example.com')
>  end
>```
>
> then run `bundle exec rspec` to run all tests in spec file


- Run `rails db:seed`
- In `application_controller.rb` at `app > controllers` add `before_action :authenticate_user!`
- *Commit*
- Once devise is installed and complete `git checkout master` to go back to master branch
#### ON MASTER BRANCH
- Run `git merge devise` to merge devise with master
- Push repo to remote (see [Git Refresher](#git-refresher) for command)
- `git checkout -b pages` to create a new pages branch and checkout
#### ON PAGES BRANCH
- `rails g controller Pages home contact`
- In `routes.rb` add:
```
root 'pages#home'

get '/contact', to: 'pages#contact'
```
- Local website should now be working!
- Checkout back to master branch
#### ON MASTER BRANCH
- Merge pages branch with `git merge pages`
- Make a new branch `git checkout -b images`
#### ON IMAGES BRANCH
- `rails g scaffold Image user:references image_data:text description:text` to create scaffolding required for Image model
- `rails db:migrate` to run migration
> Creating an `info.md` file and running `history >> info.md` in terminal will append terminal command history to file. > will re-write file with newest data.
- Add `validates :user, presence: true` to the `image.rb` model file
- Add `@image.user = current_user` to the Create action in `images_controller.rb` file
- Add `has_many :images` in `user.rb` model file
- Edit `_form.html.erb` to remove User ID field
- In `seeds.rb` add:
```ruby
user = User.create!({
    email: "example@example.com",
    password: "password123",
}) { |u| p u.encrypted_password }

Image.create!([
    image_data: "http://static.wixstatic.com/media/9059c663ff406ee7a8cc4b8b2048c783.jpg/v1/fill/w_784,h_523,al_c,q_90,usm_0.66_1.00_0.01/9059c663ff406ee7a8cc4b8b2048c783.webp",
    description: "A Siberian Husky",
    user: user
]) { |i| p i.user }
```
- Run `rails db:seed`
- In `show.html.erb` add `<%= image_tag @image.image_data %>` to show the actual image instead of the image data
- Add pundit to gemfile with `gem 'pundit', '~> 1.1'`
- Run `bundle`
- `rails g pundit:install` to install Pundit
- Create `image_policy.rb` file in `app > policies` folder then populate with the following policies:
```ruby
class ImagePolicy < ApplicationPolicy

  def create?
    user == record.user
  end

  def update?
    create?
  end
end
```
- In `application_controller.rb` add `include Pundit`
- In `images_controller.rb`, at top before all actions add `before_action :auth_actions, only: [:update, :create]` 
- At bottom of `images_controller.rb` in `private` add: 
```ruby
def auth_actions
    authorize @image
end
```
- To authenticate user before showing Edit link on pages, in `show.html.erb` and `index.html.erb` replace Edit links with:
```ruby
<p>
<% if policy(@image).update? %>
  <%= link_to 'Edit', edit_image_path(@image) %> |
<% end %>
</p>
```
- In the `images_controller.rb` file at the top add`authorize @images` to `index` action as shown below:
```ruby
 # GET /images
  # GET /images.json
  def index
    @images = Image.all
    authorize @images
  end
```
- Run `rails g migration AddAdminToUser admin:boolean` to generate migration file that will add a new column named admin to the user table with value boolean (true or false)
- `rails db:migrate` to run migration file

This is where we finished the day.

Pundit names policies after actions. If action is called `elephant`, the pundit policy will be called `elephant?`.

---
## References
- [rspec-rails Gem Docs](https://github.com/rspec/rspec-rails)
- [pundit Gem Docs](https://github.com/varvet/pundit)
- [Pundit Tutorial on FreeCodeCamp Medium](https://medium.freecodecamp.org/rails-authorization-with-pundit-a3d1afcb8fd2)