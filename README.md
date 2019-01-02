## Welcome to My Materials

## Professional Assessment

After completing a business and software engineering degree with SNHU (Southern New Hampshire University) there were a myriad of aspects I learned. I focused on applicable knowledge rather than theoretical. I want to make an impact where I go in the future. This is the reason I learned how to build RESTful API with python script on a Mongo database. I learned how to make android applications. I learned C++ to understand the foundation of coding.

My favorite aspect of coding was frontend work. I highly enjoyed Ruby on Rails with JavaScript. The best application I worked on was my Google Maps API. I created an ePortfolio showcasing the skill I developed that were unique form the project. I was able to work with underscore JS to improve the JavaScript algorithms in the project. I used Devise to make an OAuth call to OKTA for single sign on. I also saved user search data for data analysis by making AJAX call to the Rails controller. 

The knowledge I took from my degrees has improved a variety of skills. I worked in team environments to review each other’s code and help write model tests. I did numerous presentations over marketing value towards different products and how to properly grab the right market. I worked on data analysis and reports using different algorithms to pull the right information. I set up databases for android, Rails, and other systems with Postgres, Mongo, and SQLite. Hopefully the code below will give you a taste of some of the skills I developed throughout my time at SNHU. 


### Google Maps Project


This project has several unique aspects to it. Each one has a little code preview below. You can check out a code review of project without the changes with the link above. It's a ruby on rails project design by me. The entire code isn't on the project to protect design.
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

### Saving data to database for data analysis 
```markdown

# routes.rb
  post '/saveUserSearch', to: 'maps#save_search'

  get '/userSearches', to: 'maps#show_searches'

  get '/deleteSearch/:id', to: 'maps#destroy_search'
  
# Add Table with Migration
  create_table "users_searches", force: :cascade do |t|
    t.string "address"
    t.string "state"
    t.string "choice_of_search"
    t.string "range"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end
  
# Grab data and make AJAX call with it from your js file
saveSearch = function savingUserSearchInfoEveryTime(address, state, choice, range) {
  var userSearchData = {
    address: address,
    state: state,
    choiceOfSearch: choice,
    range: range
  }

  $.ajax({
    type: "POST",
    url: "/saveUserSearch",
    data: userSearchData,
    success: function (data) {
      console.log("saved the search information");
    },
    error: function (data) {
      console.log("Did not save the search");
    }
  });
}

# Create save , show, and delete ability in controller, my instance is the maps controller
  def save_search
    puts "Nice, the AJAX call made it to this function. Now you can feasibly save"
    search = UsersSearch.new
    search.address = params[:address]
    search.state = params[:state]
    search.choice_of_search = params[:choice_of_search] || "Adjuster"
    search.range = params[:range]
    if search.save
      puts "Successfully Saving"
    else
      puts "There was an error with that search result saving"
    end
  end
  
  def show_searches
    @searchData = UsersSearch.all
  end

  def destroy_search
    @users_search = UsersSearch.find(params[:id])
    @users_search.destroy
    redirect_to '/userSearches'
    flash[:alert] = "Successfully Deleted"
  end

# Add the table to a view and you're all done
  `<table id='mySearchTable' style="width:100%;">
    <thead>
      <tr>
        <th>Address</th>
        <th>Range</th>
        <th>Type</th>
        <th>Range</th>
        <th>Modify?</th>
      </tr>
    </thead>

    <tbody>
      <% @searchData.each do |data| %>
      <tr>
        <td><%= data.address %></td>
        <td><%= data.state %></td>
        <td><%= data.choice_of_search %></td>
        <td><%= data.range %></td>
        <td><%= link_to 'Delete', controller: "maps", action: 'destroy_search', id: data%></td>
        <%end%>
      </tr>
    </tbody>
  </table>`
      
 # Finally add the route to your navigation dropdowns so user can access the data
`<li><a href="/userSearches">Search Data</a></li>`

```

### Improving javascript speed and readability using underscore
```markdown

Javascript is great, but every now and then using underscore can give an extra kick
Here is the initial Javascript method that is grabbing the state from a string

# 422 Cedar Glen Dr #3, Fort Wayne, IN 46825, USA => I need IN
# Alabama
`filterOutState = function takesUserAddressandSetsState() {
  console.time("filterOutState");
  maps.userState = 'empty';
  stateFromUser = maps.addressInput.replace(/,/g, '');
  stateFromUser = stateFromUser.split(' ');

  stateFromUser.forEach((word) => {
    StatesArray.forEach((state) => {
      if (state === word) {
        maps.userState = word;
      }
    });
  });
  if(maps.userState == 'empty'){
    filterOutStateHelper();
  }
  console.timeEnd("filterOutState");
};`

filterOutState: 0.1181640625ms

# By adding backbone and stringing methods together the speed and readability was increased drastically 
`filterOutState = function takesUserAddressandSetsState() {
  stateFromUser = maps.addressInput.replace(/,/g, '').split(' ');
  maps.userState = _.intersection(stateFromUser, StatesArray);
  if(_.isEmpty(maps.userState)) {
    filterOutStateHelper();
  }
};`

filterOutState: 0.07373046875ms

# The helper method ran if there was no state found can also be improved
# Initial method
`filterOutStateHelper = function findsStateIfUserStateIsEmpty() {
  stateFromUser = maps.addressInput.split(',');
  stateFromUser.forEach((word) => {
    if (StatesHash[word] != undefined) {
      maps.userState = StatesHash[word];
    }
  });
};`

# Improved method
`maps.userState = StatesHash[_.first(maps.addressInput.split(','))];`
```

### Support or Contact

You can email me at chasekaneki@gmail.com for recommendations or for assistance. 
