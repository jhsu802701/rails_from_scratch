# Unit 3
## Chapter 1: Minitest Reporters
In this chapter, you will set up Minitest Reporters.  It provides features like red/green displays and a list of the most time-consuming tests within your test suite.

### New Branch
Enter the command "git checkout -b 03-01-minitest_reporters".

### Gemfile
*  Add the following lines to the end of the Gemfile:
```
# BEGIN: minitest-reporters
# Shows red/green displays and a list of the most time-consuming tests within your test suite
group :testing do
  gem 'minitest-reporters', '1.1.11'
end
# END: minitest-reporters
```
* Enter the command "bundle install".
* Enter the following commands:
```
sh git_check.sh
git add .
git commit -m "Installed minitest-reporters"
```
### Configuration
*  Insert the code for configuring Minitest Reporters between the line "require 'rails/test_help'" and the line "class ActiveSupport::TestCase".  Your code should look like this:
```
...
require 'rails/test_help'

# BEGIN: use minitest-reporters
# AwesomeReporter configuration from
# http://chriskottom.com/blog/2014/06/dress-up-your-minitest-output/
require 'minitest/reporters'
Minitest::Reporters.use!

module Minitest
  module Reporters
    class AwesomeReporter < DefaultReporter
      GREEN = '1;32'
      RED = '1;31'

      def color_up(string, color)
        color? ? "\e\[#{ color }m#{ string }#{ ANSI::Code::ENDCODE }" : string
      end

      def red(string)
        color_up(string, RED)
      end

      def green(string)
        color_up(string, GREEN)
      end
    end
  end
end

reporter_options = { color: true, slow_count: 5 }
Minitest::Reporters.use! [Minitest::Reporters::AwesomeReporter.new(reporter_options),
                          Minitest::Reporters::HtmlReporter.new]
# END: use minitest-reporters

class ActiveSupport::TestCase
...
```
* Add the following code to the end of .rubocop.yml:
```
Style/MutableConstant:
  Exclude:
    - test/test_helper.rb
    
Style/SpaceInsideStringInterpolation:
  Exclude:
    - test/test_helper.rb
```
* Add the following line to the end of .gitignore:
```
test/html_reports/
```
* Enter the following commands:
```
sh git_check.sh
git add .
git commit -m "Updated test/test_helper.rb for minitest-reporters"
```

### First Test
* Enter the command "rails generate integration_test test1".  This creates the file test/integration/test1_test.rb.
* Give test/integration/test1_test.rb the following content:
```
require 'test_helper'

class Test1Test < ActionDispatch::IntegrationTest
  test 'home page has expected title and heading' do
    get '/'
    assert_select 'title', 'Home'
    assert_select 'h1', 'Home'
  end

  test 'home page has expected content' do
    get '/'
    assert_match 'Welcome to public/index.html!', response.body
  end
end
```
* Enter the command "rails test".  You'll see that the test for the title and heading fail.
* Replace the contents of the public/index.html file with the following:
```
<html>
    <head>
        <title>Home</title>
    </head>
    <body>
	<h1>Home</h1>	
    Welcome to public/index.html!
    </body>
</html>
```
* Enter the command "rails test".  Now all of the tests pass, and the display output contains green instead of red.
* Enter the following commands:
```
sh git_check.sh
git add .
git commit -m "Successfully completes first test"
```

### Wrapping Up
* Enter the command "sh git_check.sh". There should be no new files or changes left to add.
* Enter the command "git push origin 03-01-minitest_reporters".
* Go to the GitHub repository and click on the "Compare and pull request" button for this branch.
* Accept this pull request to merge it with the master branch, but do NOT delete this branch.
* Enter the following commands:
```
git checkout master
git pull
sh heroku.sh
```