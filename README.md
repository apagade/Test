# Using Git to implement a new feature/change without affecting the main branch 

    
## Why do we need to do this?


When we develop a new feature or do a bug fix, our codebase changes can cause conflict with the changes done by someone else in the same file or same part of the codebase and break something which will impact other developer’s work who are working on the same part of the codebase.
To avoid these Git Branches allow users to collaborate on the same code base at the same time without constantly getting in each other's way, they allow developers to work on features independent of the other contributions being made to the codebase and allows them to add their features to the main codebase when they're ready.


Let’s see how we can implement a new feature/change without affecting the main branch referred to as a '**master**' in git.


Lets Assume that we already have git installed and configured. Also, we have a remote repository in our Gitlab account. We can create a branch using sequence of below commands:

```
$ git clone <url of your remote repository>
```
  
 This command copies (or clone) the remote repository to the local machine

 Example:

  ```
  $ git clone https://gitlab.com/apagade/Test.git
  Cloning into 'Test'...
  remote: Enumerating objects: 31, done.
  remote: Total 31 (delta 0), reused 0 (delta 0), pack-reused 31
  Unpacking objects: 100% (31/31), done.
  ```
  
When it finishes, the new directory **Test** is created on the local machine which contains all the project’s files, branches, and history which is cloned from master.
Example:
```
$ls
Test
$ cd Test
```
To create a new feature/change branch we can use git checkout command as below:
```
$ git checkout -b new-feature-branch
Switched to a new branch 'new-feature-branch'
```
 
What this command does is it creates a new branch called 'new-feature-branch' and the '-b' argument automatically checks out your new branch.
This branch is a snapshot of the master at that current date and time. Any changes to master will not be reflected in this branch and any changes to the new branch will not be reflected in the master.

You can verify which branch you're on by using below command:

```
$ git branch
```
Output of above command will show an asterisk '*' beside the currently checked out branch, as below:
 ```
 master
* new-feature-branch
```
Now that we have created new feature branch we can make changes to the code locally. 
After the changes are done we can commit changes to the new branch.

The git status command displays the state of the working directory and the staging area. It helps us track which files have been created/modified/changed/deleted/. 

Example:
```
$ git status
On branch new-feature-branch
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
 
        modified:   README.md
 
no changes added to commit (use "git add" and/or "git commit -a")
```

Next step is to add the changes to the staging area. The 'git add' command adds the changes in the working directory to the staging area. It tells Git that we want to include updates to a particular file in the next commit. However, git add doesn't really affect the repository in any significant way—changes are not actually recorded until we run 'git commit' command.

```
$ git add <<path and name of file>>
```

Example:
```
$ git add README.md 
$ git status
On branch new-feature-branch
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
 
        modified:   README.md
```
Running `git status` again will show us a list of changes to be committed.

$ git status
On branch new-feature-branch
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
 
        modified:   README.md
 
## How to commit code changes:

We can use 'git commit - a' to add all modified files to the staging area and commit straightaway. To add all created/changed/deleted files execute below command:
  
```
$ git commit -a
```
 
This will open up a editor to add a commit message which is a short description about the code changes done. 
 
We can simply skip above step by providing the commit message as argument in commit command as below:
  
```
 $ git commit -m "Feature: Added awesome feature"
```
We can use 'git log' command to get more information about the commit history. It lists the recent commit first.
  
```
  $ git log
```
Once the changes are committed locally, the next step is to merge your feature branch into the master. 
To do so first we need to checkout master again and then execute 'git merge' command:

```
$ git checkout master
$ git merge new-feature-branch -m "Feature: New imaplemented feature"
```
Example:

```
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.

$ git merge new-feature-branch -m "Feature: New imaplemented feature"
Updating a7add27..f860def
Fast-forward (no commit created; -m option ignored)
 README.md | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)
 ```
 
Now our master branch is updated with the the latest feature created. 
The last step is to push the changes to the remote repository and create a **pull/merge** request in github.
 
Merge requests are used to contribute code to repositories where someone else needs to approve our change. You wouldn't have to open a merge request to your own repository as it can be assumed that your contributions are already at the standard that you're willing to accept and that you don't need anyone else to approve them. However when you're contributing to a repository owned by somebody else they may want to review your change before it is incorporated into the codebase. This could be to make sure that it is of an appropriate standard for that repository, that the code works, and that it will be a positive contribution to the codebase.
 
