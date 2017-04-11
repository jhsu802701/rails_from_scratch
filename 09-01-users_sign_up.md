# Unit 9
## Chapter 1: Users Sign Up

In this chapter, you will provide the general public the ability to sign up for a user account.

### New Branch
Enter the command "git checkout -b 09-01-users_sign_up".

### Integration Test
* Enter the command "rails generate integration_test users_signup".
* Edit the resulting test/integration/users_signup_test.rb file and replace the content with the following:
```
require 'test_helper'

class UsersSignupTest < ActionDispatch::IntegrationTest
  test 'Home page provides access to user signup page' do
    visit root_path
    assert page.has_link?('Sign up now!', href: new_user_registration_path)
  end

  test 'User signup page has expected content' do
    visit root_path
    click_on 'Sign up now!'
    assert page.has_css?('title', text: full_title('New User'), visible: false)
    assert page.has_css?('h1', text: 'New User')
    assert page.has_text?('password management program')
    assert page.has_text?('create much better passwords')
    assert page.has_link?('KeePassX', href: 'http://www.keepassx.org')
  end

  test 'Proper response for invalid sign-up' do
    assert_no_difference 'User.count' do
      sign_up_user('jhiggins', 'Higgins', 'Jonathan',
                   'jhiggins@example.com', 'Zeus and Apollo',
                   'Higgy Baby')
    end
    assert page.has_text?("Password confirmation doesn't match Password")
  end

  test 'Proper response for valid sign-up' do
    assert_difference 'User.count', 1 do
      sign_up_user('jhiggins', 'Higgins', 'Jonathan',
                   'jhiggins@example.com', 'Zeus and Apollo',
                   'Zeus and Apollo')
    end

    # Check for the message about the account activation link
    assert page.has_text?('A message with a confirmation link')
    assert page.has_text?('has been sent to your email address.')
    assert page.has_text?('Please follow the link to activate your account.')
  end
end
```
* Enter the command "sh build_fast.sh".  All 4 of your new integration tests will fail.  Two tests fail because the sign_up_user method is undefined, and two tests fail because the "Sign up now!" button is missing.
* Enter the command "alias test1='(command for running failed tests minus TESTOPTS portion)'".
* Enter the command "test1" to run just these tests again.
* In your browser, go to the URL http://localhost:3000/users/sign_up .  (Replace "localhost" and/or "3000" if your setup calls for that.)  You'll see the standard Devise sign up form.  The debug window will show that the controller in use is "devise/registrations", and the action in use is "new".

### Test Helper
* Add the following lines to the end of the test/test_helper.rb file:
```
# rubocop:disable Metrics/ParameterLists
def sign_up_user(name_u, name_l, name_f, e, p1, p2)
  visit root_path
  click_on 'Sign up now!'
  fill_in('Last name', with: name_l)
  fill_in('First name', with: name_f)
  fill_in('Username', with: name_u)
  fill_in('Email', with: e)
  fill_in('Password', with: p1) # not yet changed
  fill_in('Password confirmation', with: p2)
  click_button('Sign up')
end
# rubocop:enable Metrics/ParameterLists
```
* Enter the command "test1".  Now all 4 tests fail because the "Sign up now!" button is missing from the home page.

### Home Page
* In this section, you will provide the "Sign up now!" button that provides access to the user sign up page from the home page.
* Replace the contents of the app/views/static_pages/home.html.erb with the following:
```
<% provide(:title, '') %>

<div class="center jumbotron">
  <h1>Home</h1>
  Welcome to Generic App Template!
  <br><br>
  <%= link_to "Sign up now!", new_user_registration_path, class: "btn btn-lg btn-primary" %>
  <br><br>
  <%= link_to image_tag("rails.png", alt: "Rails logo"),
              'http://rubyonrails.org/' %>
</div>
```
* Enter the command "test1".  The first integration test will pass, but the other three will fail because of missing content on the user sign-up page.
* In your web browser, go to the home page in your local app.  Click on the "Sign up now!" button.  You'll find yourself back in the default user sign up form, with the "devise/registrations" controller and "new" action once again being utilized.

### Routing
* Edit the config/routes.rb file.  Replace the line "devise_for :users" with the following lines:
```
  # BEGIN: user section
  devise_for :users,
             controllers: { registrations: 'users/registrations' }
  # END: user section
```
* In your web browser, attempts to refresh or visit the URL http://localhost:3000/users/sign_up give you a routing error with the message "uninitialized constant Users".  This is because the user registration controller specified in the routing does not exist.


### Devise Controllers and Views
* Create user authentication controllers. Enter the command "rails generate devise:controllers users". This creates user authentication controllers.
* In your web browser, visit the URL http://localhost:3000/users/sign_up .  The controller is now "users/registrations", but  the default user sign up form still appears.
* If you look at the app/views directory, you'll see that there is no user registration form available.  What you see in the browser is the default page provided by Devise.
* Create user authentication pages. Enter the command "rails generate devise:views users". This creates user authentication pages.
* Now there is a new user registration form at app/views/users/registrations/new.html.erb.  There are also various other forms in the app/views/users directory that can be customized.

### User Signup Page
* Edit the user signup page.  Replace the contents of app/views/users/registrations/new.html.erb with the following:
```
<% provide(:title, "New User") %>

<h1>New User</h1>
Using the same password for all of your accounts is risky.
Limiting yourself to passwords that you can easily remember is risky.
You should use a password management program like <a href='http://www.keepassx.org'>KeePassX</a>
to create much better passwords AND store them in encrypted form.

<%= form_for(resource, html: { multipart: true }, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= devise_error_messages! %>

  <div class="field">
    <%= f.label :username %> (for logging in) <br />
    <%= f.text_field :username, autofocus: true %>
  </div>

  <div class="field">
    <%= f.label :last_name %><br />
    <%= f.text_field :last_name %>
  </div>

  <div class="field">
    <%= f.label :first_name %><br />
    <%= f.text_field :first_name %>
  </div>

  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email %>
  </div>

  <div class="field">
    <%= f.label :password %><br />
    <%= f.password_field :password, autocomplete: "off" %>
  </div>

  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation, autocomplete: "off" %>
  </div>

  <div class="actions">
    <%= f.submit "Sign up" %>
  </div>
<% end %>

<%= render "users/shared/links" %>
```
* Enter the command "test1".  Two of the tests fail, but one test fails because filling out the sign-up form and submitting it does not cause the new user to be saved to the database.
* Go to the user sign up page.  The expected content appears, but the appearance leaves something to be desired.  (Additions to the app/assets/stylesheets/custom.scss stylesheet will take care of this later.)  If you fill out the user sign up form, you'll get error messages telling you that your email address, last name, and first name cannot be blank.  (Modifications to the controller will take care of this later.)

### Stylesheet (Forms)
* Add the following code to the end of the file app/assets/stylesheets/custom.scss:
```
/* forms */

input, textarea, select, .uneditable-input {
  border: 1px solid #bbb;
  width: 100%;
  margin-bottom: 15px;
  @include box_sizing;
}

input {
  height: auto !important;
}

#error_explanation {
  color: red;
  ul {
    color: red;
    margin: 0 0 30px 0;
  }
}

.field_with_errors {
  @extend .has-error;
  .form-control {
    color: $state-danger-text;
  }
}

.checkbox {
  margin-top: -10px;
  margin-bottom: 10px;
  span {
    margin-left: 20px;
    font-weight: normal;
  }
}

#session_remember_me {
  width: auto;
  margin-left: 0;
}
```
* The checkbox and session_remember_me specifications in the stylesheet will be used for logging in as a user or admin.  Logging in will be covered later.
* In your browser, go to the user sign up page.  The form looks better but still does not provide the desired functionality.

### User Registration Controller
* Edit the file app/controllers/users/registrations_controller.rb and replace the content with the following:
```
#
class Users::RegistrationsController < Devise::RegistrationsController
  before_action :configure_sign_up_params, only: [:create]
  # before_action :configure_account_update_params, only: [:update]

  # GET /resource/sign_up
  # def new
  #   super
  # end

  # POST /resource
  # def create
  #   super
  # end

  # GET /resource/edit
  # def edit
  #   super
  # end

  # PUT /resource
  # def update
  #   super
  # end

  # DELETE /resource
  # def destroy
  #   super
  # end

  # GET /resource/cancel
  # Forces the session data which is usually expired after sign
  # in to be expired now. This is useful if the user wants to
  # cancel oauth signing in/up in the middle of the process,
  # removing all OAuth session data.
  # def cancel
  #   super
  # end

  protected

  # If you have extra params to permit, append them to the sanitizer.
  def configure_sign_up_params
    devise_parameter_sanitizer.permit(:sign_up,
                                      keys: [:username, :last_name,
                                             :first_name, :email])
  end

  # If you have extra params to permit, append them to the sanitizer.
  # def configure_account_update_params
  #   devise_parameter_sanitizer.permit(:account_update, keys: [:attribute])
  # end

  # The path used after sign up.
  # def after_sign_up_path_for(resource)
  #   super(resource)
  # end

  # The path used after sign up for inactive accounts.
  # def after_inactive_sign_up_path_for(resource)
  #   super(resource)
  # end
end
```
* The user registration controller above uses the default Devise setup EXCEPT when instructed otherwise.  Because the default Devise behavior is not desired in some cases, the exceptions must be specified in the user registration controller.
* Enter the command "test1".  Now all of the new integration tests pass, and the user sign up feature is working.

### Stylesheet (Flash)
* When you successfully sign up as a user at this point, you will see the message "A message with a confirmation link has been sent to your email address. Please follow the link to activate your account."  This message will be in black letters on white text, but it would be desirable to display it in dark green letters in a light green box.  If you view the source, you'll see that the message about the confirmation link is within a div tag of class "alert alert-notice".
* Add the following lines to the end of the file app/assets/stylesheets/custom.scss:
```
/*flash*/
.alert-error {
    background-color: #f2dede;
    border-color: #eed3d7;
    color: #b94a48;
    text-align: left;
}

.alert-alert {
    background-color: #f2dede;
    border-color: #eed3d7;
    color: #b94a48;
    text-align: left;
}

.alert-success {
    background-color: #dff0d8;
    border-color: #d6e9c6;
    color: #468847;
    text-align: left;
}

.alert-notice {
    background-color: #dff0d8;
    border-color: #d6e9c6;
    color: #468847;
    text-align: left;
}
```
* Now flashes messages will appear with special colors to alert you that an action was successful or unsuccessful.

### RuboCop Compliance
* In the .rubocop.yml file, add "app/controllers/users/*" to the list of files excluded from the Style/ClassAndModuleChildren cop.
* Add the following lines to the end of .rubocop.yml:
```
Style/CommentIndentation:
  Exclude:
    - app/controllers/users/*
```
* Enter the command "sh git_check.sh".  All tests should pass, and there should be no more RuboCop offenses.
* Enter the following commands:
```
git add .
git commit -m "Added user signup"
```

### Development Environment
* Restart the local server.  Until you do so, the email confirmation message will not show up in the MailCatcher window.
* Try out the user sign-up feature in the development environment.

### Wrapping Up
* Enter the command "git push origin 09-01-users_sign_up".
* Go to the GitHub repository and click on the "Compare and pull request" button for this branch.
* Accept this pull request to merge it with the master branch, but do NOT delete this branch.
* Enter the following commands:
```
git checkout master
git pull
sh heroku.sh
```
