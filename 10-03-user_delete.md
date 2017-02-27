# Unit 10
## Chapter 3: User Deletion

In this chapter, you will give the power to delete users to admins and ONLY admins.  (Remember that users can delete themselves through the registration edit channel.)  

### New Branch
Enter the command "git checkout -b 10-03-user_delete".

### User Controller
* In the file test/controllers/users_controller_test.rb, add the following lines to the DELETE section:
```
  test 'should not allow visitor to delete user' do
    get :destroy, params: { id: @u7 }
    assert_redirected_to root_path
  end

  # NOTE: User can delete self through edit registration form.
  test 'should not allow user to delete self' do
    sign_in @u7, scope: :user
    get :destroy, params: { id: @u7 }
    assert_redirected_to root_path
  end

  test 'should not allow user to delete another user' do
    sign_in @u1, scope: :user
    get :destroy, params: { id: @u7 }
    assert_redirected_to root_path
  end

  test 'should allow super admin to delete user' do
    sign_in @a1, scope: :admin
    get :destroy, params: { id: @u7 }
    assert :success
    assert_redirected_to users_path
  end

  test 'should allow regular admin to delete user' do
    sign_in @a4, scope: :admin
    get :destroy, params: { id: @u7 }
    assert :success
    assert_redirected_to users_path
  end
```
* Enter the command "sh testc.sh".
* 

### Wrapping Up
* Enter the command "git push origin 10-03-user_delete".
* Go to the GitHub repository and click on the "Compare and pull request" button for this branch.
* Accept this pull request to merge it with the master branch, but do NOT delete this branch.
* Enter the following commands:
```
git checkout master
git pull
sh heroku.sh
```