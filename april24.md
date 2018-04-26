[< Back](README.md)

# April 24th
## Covering:
- [File Attachment Gems](#file-attachment-gems)
- [Shrine](#shrine)
- [Adding Likes and Dislikes](#adding-likes-and-dislikes)
- [References](#references)

---
## File Attachment Gems
- Shrine
- CarrierWave
- Paperclip

---
- Amazon S3 (Simple Storage Service) 
- Heroku is designed to run apps, not to act as a storage solution
- Using Shrine to connect with S3 allows you to run apps on Heroku and deal with storage through S3 
- Long story short just use Shrine, don't bother with CarrierWave or Paperclip

---
## Shrine
- `rails new instashrine`
- add `devise`, `rspec-rails`, `shrine`, `pundit`, `dotenv-rails` to gemfile
- `bundle`
- `rails g rspec:install`
- `rails g devise:install`
- `rails g pundit:install`
- `rails g devise User`
- `rails g scaffold Photo image_data:text user:references description:text`
- Create file named `shrine.rb` in initializers and add:
```ruby
require "shrine"
require "shrine/storage/file_system"

Shrine.storages = {
  cache: Shrine::Storage::FileSystem.new("public", prefix: "uploads/cache"), # temporary
  store: Shrine::Storage::FileSystem.new("public", prefix: "uploads/store"), # permanent
}

Shrine.plugin :activerecord
Shrine.plugin :cached_attachment_data # for forms
```
- Create `uploaders` folder in `app` with `image_uploader.rb` file inside:
```ruby 
class ImageUploader < Shrine
    # plugins and uploading logic
  end
```
- In `photo.rb` model add `include ImageUploader::Attachment.new(:image) # adds an "image" virtual attribute`
- Add `@photo.user = current_user` inside the `create` method in `photo_controller.rb`
- Add `before_action :authenticate_user!, except: [:index, :show]` at top of photo controller
- Add `has_many :photos` in user model
- Remove `user_id` from form partial and from `pages_controller.rb`
- Add the below inside the form:
```ruby
<div class="field">
    <%= form.label :image %>
    <%= form.hidden_field :image, value: @photo.cached_image_data %>
    <%= form.file_field :image, id: :photo_image_data %>
  </div>
```
- Migrate database
- `rails g controller Pages index`
```ruby
def photo_params
      params.require(:photo).permit(:image, :description)
    end
```
- `brew install imagemagick` in terminal
- Add `gem "image_processing", "~> 1.0"` in Gemfile
- Copy to `image_uploader.rb`
```ruby
require "image_processing/mini_magick"

class ImageUploader < Shrine
  plugin :processing
  plugin :versions   # enable Shrine to handle a hash of files
  plugin :delete_raw # delete processed files after uploading

  process(:store) do |io, context|
    original = io.download
    pipeline = ImageProcessing::MiniMagick.source(original)

    size_80 = pipeline.resize_to_limit!(80, 80)
    size_500 = pipeline.resize_to_limit!(500, 500)

    original.close! #gets rid of image in memory

    { original: original, thumb: size_80, medium: size_300 }
  end
end
```
---
## Adding Likes and Dislikes

- `rails g migration CreateJoinTableLikes user photo` 
- Add to migration file:
```ruby
class CreateJoinTableLikes < ActiveRecord::Migration[5.1]
  def change
    create_join_table :users, :photos, table_name: :likes do |t|
      # t.index [:user_id, :photo_id]
      t.index [:photo_id, :user_id]
    end
  end
end
```
- Migrate database
- Put in `photo.rb` model:
```ruby
has_many_and_belongs_to_many :likers, class_name: "User", join_table :likes

def liked_by?(user)
  likers.exists?(user.id)
end

def toggle_liked_by(user)
  if liked_by?(user)
    likers.destroy(user)
  else
    likers << user 
  end
```
- Add to routes:
```ruby
resources :photos do
    member do
      get 'likes'
    end
  end
```
- In `photos_controller.rb`:
```ruby
before_action :set_photo, only: [:show, :edit, :update, :destroy, :like]
# at top

# GET photos/1/like
  def like 
    @photo.toggle_liked_by(current_user)
  end

```

---
## References
- [Shrine Docs on Github](https://github.com/shrinerb/shrine)
- [Shrine Website](http://shrinerb.com/)