## Welcome to My Materials

The purpose of this page is to demonstrate different features I have implemented. Maybe you're here to see if I am talented enough to help you, or maybe you're here to get my help. Either way, I am happy to have you here and I hope you are able to feasibly work your way through the page. 

### Google Maps Project
https://github.com/chasedougherty4628/HelloWorld

This project has several unique aspects to it. Each one has a little code preview below. You can check out the full project with the link above. It's a ruby on rails project design by me. The entire code isn't on the project to protect design.
1. Implementation of Okta for single user sign-on 
2. Javascript implemented with Vue using the Google Maps API for the project
3. Active Record usage to store user search information for data analysis 


### OKTA
```markdown

# Gemfile

gem 'omniauth-oktaoauth', '~> 0.1.6'
gem 'devise'
gem 'figaro'
gem 'activerecord-session_store'

# Generate a application.yml using figaro gem
# Create okta developer account, grab your url from dashboard in top right, and grab your id and secret from the project

OKTA_CLIENT_ID: "0oain4bcexamplethatwillnotworkforyouACshj0h7"
OKTA_CLIENT_SECRET: "Qu0qov5lZ2KJp8W_i0examplethatwillnotworkforyousroLEjHq71I"
OKTA_ORG: "dev-435540" Comes from url
OKTA_DOMAIN: "oktapreview" Comes from url
OKTA_URL: "https://dev-435540.oktapreview.com"
OKTA_ISSUER: "https://dev-435540.oktapreview.com/oauth2/default"
OKTA_AUTH_SERVER_ID: "default" Leave this as default
OKTA_REDIRECT_URI: "http://localhost:3000/users/auth/oktaoauth/callback" use this link if you port to 3000

# Set up activerecord session store

# Run devise setup command: rails generate devise:install && rails generate devise MODEL && rake db:migrate
# Don't worry about the view files

# devise.rb
# add at the top of file
require 'omniauth-oktaoauth'

# add below config.email_regexp
  config.omniauth(:oktaoauth,
                ENV['OKTA_CLIENT_ID'],
                ENV['OKTA_CLIENT_SECRET'],
                :scope => 'openid profile email',
                :fields => ['profile', 'email'],
                :client_options => {site: ENV['OKTA_ISSUER'], authorize_url: ENV['OKTA_ISSUER'] + "/v1/authorize", token_url: ENV['OKTA_ISSUER'] + "/v1/token"},
                :redirect_uri => ENV["OKTA_REDIRECT_URI"],
                :auth_server_id => ENV['OKTA_AUTH_SERVER_ID'],
                :issuer => ENV['OKTA_ISSUER'],
                :strategy_class => OmniAuth::Strategies::Oktaoauth)
                
 # modify your schema to have new tables and values
   create_table "authorizations_tables", force: :cascade do |t|
    t.string "provider"
    t.string "uid"
    t.integer "user_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end
  
    create_table "sessions", force: :cascade do |t|
    t.string "session_id", null: false
    t.text "data"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["session_id"], name: "index_sessions_on_session_id", unique: true
    t.index ["updated_at"], name: "index_sessions_on_updated_at"
  end
  
  create_table "users", force: :cascade do |t|
t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.string "reset_password_token"
    t.datetime "reset_password_sent_at"
    t.string "password"
    t.string "password_confirmation"
   t.string "provider"
    t.string "uid"
    t.integer "sign_in_count", default: 0, null: false
    t.datetime "current_sign_in_at"
    t.datetime "last_sign_in_at"
    t.string "current_sign_in_ip"
    t.string "last_sign_in_ip"
    t.string "name"
t.index ["reset_password_token"], name: "index_users_on_reset_password_token", unique: true

# routes.rb
devise_for :users, controllers: { omniauth_callbacks: 'users/omniauth_callbacks' }
  get 'sessions/new'
  get 'sessions/create'
  get 'sessions/failure'
  get "/logout" => "sessions#destroy", as: :logout
  
# View Template of your choice where the user can click login and logout
`<% if !session[:oktastate] %>
  <li><%= link_to "Sign In Or Sign Up", user_oktaoauth_omniauth_authorize_path %></li>
<% else %>
  <li><%= link_to "logout", ENV['OKTA_URL'] + "/login/signout?fromURI=" + request.base_url + "/logout", class: "item" %></li>
<% end %>`
  
# User.rb
  devise :omniauthable, omniauth_providers: [:oktaoauth]
  has_many :authorizations

  # how we create the actual user
  `def self.from_omniauth(auth)
    user = User.find_or_create_by(email: auth["info"]["email"]) do |user|
      user.provider = auth['provider']
      user.uid = auth['uid']
      user.name = auth['info']['name']
      user.email = auth['info']['email']
    end
  end

  def oauth_permissions(state)
    state.to_s.split('scp=')[1].split("[")[1].split("]")[0].split(",").collect{|x| x.strip || x }
  end`
  
# Authorization.rb
  `belongs_to :user
  validates :provider, :uid, :presence => true

  def self.find_or_create(auth_hash)
    auth
  end`
  
# session_controller.rb
  `def new
  end

  def create
  end

  def destroy
    session[:oktastate] = nil
    redirect_to user_oktaoauth_omniauth_authorize_path
  end

  def failure
  end`
  
# application_controller.rb
  
  `protect_from_forgery with: :exception
  add_flash_types :success

  def user_is_logged_in?
    if !!session[:oktastate]
    else
      redirect_to user_oktaoauth_omniauth_authorize_path
    end
  end

  def after_sign_in_path_for(resource)
    request.env['omniauth.origin'] || root_path
  end`

# Create a users folder and create omniauth_callbacks_controller.rb

`class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def oktaoauth
    # You need to implement the method below in your model (e.g. app/models/user.rb)
    # can we create a user with the information we are getting from omniauth(okta)
    @user = User.from_omniauth(request.env["omniauth.auth"])
    print(@user)
    if @user.save
      # set a session with the okta state and then redirect to the root path
      session[:oktastate] = request.env["omniauth.auth"]
      print(@user.oauth_permissions(session[:oktastate]))
    else
      print(@user.errors.full_messages)
      redirect_to user_oktaoauth_omniauth_authorize_path
    end
    if @user.persisted?
      redirect_to "/maps"
    end
  end

  def failure
  end
end`
  

```

### Google Maps API with VUE and Javascript
```markdown
Syntax highlighted code block

# Header 1

- Bulleted
- List

Ruby Code

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

### User info stored everytime user enters in information
```markdown
Syntax highlighted code block

# Header 1

- Bulleted
- List

Ruby Code

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

### Support or Contact

You can email me at chasekaneki@gmail.com for recommendations or for assistance. 
