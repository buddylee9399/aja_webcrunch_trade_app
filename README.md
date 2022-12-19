# web crunch - trade app
- https://web-crunch.com/posts/lets-build-with-ruby-on-rails-trade-app-in-app-messaging
- GEMS

```
group :development do
  # Use console on exceptions pages [https://github.com/rails/web-console]
  gem "web-console"
  gem 'better_errors'
  gem 'binding_of_caller', '~> 1.0'
  # Add speed badges [https://github.com/MiniProfiler/rack-mini-profiler]
  # gem "rack-mini-profiler"

  # Speed up commands on slow machines / big apps [https://github.com/rails/spring]
  # gem "spring"
end

# gem 'sidekiq'
gem 'devise'
gem 'bulma-rails'
gem 'simple_form'
gem 'kramdown'
```

- rails g simple_form:install
- rails g devise:install
- rails g devise:views
- rails g devise User name
- rails db:migrate
- rails 7, update app controller
```
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

    def configure_permitted_parameters
      devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
      devise_parameter_sanitizer.permit(:account_update, keys: [:name])
    end
end
```
- rails g scaffold trade title description:text user:references
- rails active_storage:install
- rails db:migrate
- rails g scaffold conversation sender_id:integer recipient_id:integer
- rails g scaffold message body:text conversation:references user:references
- rails db:migrate
- update routes

```
Rails.application.routes.draw do
  resources :trades

  resources :conversations do
    resources :messages
  end

  devise_for :users
  root to: 'trades#index'

end
```

- updated the app.scss and other scss
- updated the controllers
- added helpers

```
module ApplicationHelper

   def gravatar_for(user, options = { size: 200})
    gravatar_id = Digest::MD5::hexdigest(user.email.downcase)
    size = options[:size]
    gravatar_url = "https://secure.gravatar.com/avatar/#{gravatar_id}?s=#{size}"
    image_tag(gravatar_url, alt: user.name, class: "border-radius-50")
  end

  def markdown_to_html(text)
    # Kramdown::Document.new(text, input: "GFM").to_html
    Kramdown::Document.new(text).to_html
  end

  def trade_author(trade)
    user_signed_in? && current_user.id == trade.user_id
  end

end
```
- updated conversation.rb

```
class Conversation < ApplicationRecord
  belongs_to :sender, foreign_key: :sender_id, class_name: "User"
  belongs_to :recipient, foreign_key: :recipient_id, class_name: "User"

  has_many :messages

  validates_uniqueness_of :sender_id, scope: :recipient_id

  # This scope validation takes the sender_id and recipient_id for the conversation and checks whether a conversation exists between the two ids because we only want two users to have one conversation.

  scope :between, -> (sender_id, recipient_id) do
    where("(conversations.sender_id = ? AND conversations.recipient_id = ?) OR (conversations.sender_id = ? AND conversations.recipient_id = ?)", sender_id, recipient_id, recipient_id, sender_id)
  end
end
```

- update message.rb

```
  belongs_to :conversation
  belongs_to :user
  validates_presence_of :body, :conversation_id, :user_id

  def message_time
    created_at.strftime("%m/%d/%y at %l:%M %p")
  end
end

```

- update trade.rb 
```
  belongs_to :user
  has_many_attached :images, dependent: :destroy
end

```

- update user.rb

```
  has_many :trades
end

```

- updated the views
- added font awesome via cdn
- in app messaging, not action cable
- used kramdown for markdown but i like the other one better
- he used javascript direct upload, i didnt do it, ill look for it in another tutorial
- used scopes
- used validation of uniquenss

## THE END