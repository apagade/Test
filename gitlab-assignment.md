# Question 1

## 1.1. Script to get username and their home directories
```bash
$ cat user-home-list.sh
#! /bin/bash
# Process the /etc/passwd file:
# - Use ':' as the field separator.
# - For each line, check that the first field (username) does not start with '#' and that the line is not empty.
# - If both conditions are met, print the username and home directory (fields 1 and 6) separated by ':'.
awk -F: '$1 !~ /^#/ && NF {print $1 ":" $6}' /etc/passwd
```

## 1.2. Script to be executed hourly
```bash
$ cat check-user-changes.sh
#! /bin/bash

LOG_PATH=/var/log/

# Redirect stdout and stderr to a log file for logging any errors and troubeshooting
LOG_FILE="${LOG_PATH}check-user-changes.log"
exec > "$LOG_FILE" 2>&1

USER_HOME_SCRIPT_PATH=./
USER_HOME_SCRIPT_NAME=user-home-list.sh

# Combine path and name to get the full path to the user home script
USER_HOME_SCRIPT="${USER_HOME_SCRIPT_PATH}${USER_HOME_SCRIPT_NAME}"

CURRENT_USERS_FILE="${LOG_PATH}current_users"
USER_CHANGES_FILE="${LOG_PATH}user_changes"

# Calculate the new MD5 checksum of the current users list
NEW_MD5=$($USER_HOME_SCRIPT | md5sum | awk '{ print $1 }') || { 
    echo "Error: Failed to run ${USER_HOME_SCRIPT}" >&2
    exit 1
}

# Check if the file storing the current MD5 checksum exists
if [[ ! -f "$CURRENT_USERS_FILE" ]]; then
        # If the file does not exist, create it and store the new MD5 checksum
        echo "$NEW_MD5" > $CURRENT_USERS_FILE || {
                echo "Error: Failed to create ${CURRENT_USERS_FILE}" >&2
                exit 1
        }
else
        # If the file exists, read the existing MD5 checksum from the file
        EXISTING_MD5=$(<$CURRENT_USERS_FILE)

        # Compare the new MD5 checksum with the existing one
        if [[ "$NEW_MD5" != "$EXISTING_MD5" ]]; then

                # If they are different, log the date and time of the change
                DATE_TIME=$(date "+%F %T")
                echo "$DATE_TIME" "changes occurred" >> "$USER_CHANGES_FILE" || {
                    echo "Error: Failed to write to $USER_CHANGES_FILE" >&2
                    exit 1
                }

                # Update the file with the new MD5 checksum
                echo "$NEW_MD5" > "$CURRENT_USERS_FILE" || {
                    echo "Error: Failed to update $CURRENT_USERS_FILE" >&2
                    exit 1
                }
        fi
fi
```

## 1.3. Crontab entry
```bash
 0 * * * * /home/user/scripts/check-user-changes.sh > /dev/null 2>&1
```

# Question 2

## 2.1. Write down and discuss the possible cause(s) of the slowness

1. (User) User has a slow internet connection.
2. (UI) Very high content is being sent to the client (including html, css, js, images etc).
3. (UI) Caching for static files is disabled.
4. (UI) Certain JS code is taking too long to complete and render the page completely.
5. (Webserver) Too many open/stale connections to the webserver.
6. (Webserver) Webserver connection pool is misconfigured to a very small value.
7. (Web application) Web application is slow due to too many threads or performing compute heavy operations, memory leaks, waiting for DB conn/response etc.
8. (Web application) Web application is slow since it is logging debug logs due to incorrect logging configuration.
9. (Web application) Certain API is slow due to various reasons (compute, db etc).
10. (Server) High CPU utilization due to the web application.
11. (Server) High CPU utilization due to the other processes (cron jobs, agents etc) running on the server.
12. (Server) Server running out of RAM due to the web application.
13. (Server) Server running out of RAM due to the other processes.
14. (Server) Server busy in I/O operations like heavy logging.
15. (DB) Too many open/stale DB connections.
16. (DB) DB connection pool is misconfigured to a very small value.
17. (DB) Certain DB queries taking too long to execute.
18. (DB) No index or index was accidentially dropped which was required for performance of certain queries.

## 2.2. Describe how you would begin to troubleshoot this issue?

Usually there are monitoring and alerting dashboards for such applications which should notify any SLO breach. Looking at monitoring dashboards should give quick idea if any of the server component is facing/faced any issues during the reported time frame. This should be a good starting point. Based on this we could access if this is user-side issue / one-off issue / genuin issue at our end.

If such dashboards are missing, we need to identify if this is a user-side or problem at our end. We could quickly try to reproduce the issue at our end by loading the page. If required, we could ask user to run internet speed test (using online speed test website).

If the issue is at our end, we should try to identify which application tier is causing this. We could check various logs, run health endpoints, run the web application's concerned API and measure response time etc. We can use various linux commands to check the health of the server.

From here on we could formulate hypothesis and try to prove them right or wrong.

# Question 3

## What sequence of Git commands could have resulted in this commit graph?

```
$ git checkout main
$ git commit -m "first commit"
$ git commit -m "second commit"

$ git checkout -b feature-branch
$ git commit -m "awesome feature"

$ git checkout main
$ git commit -m "third commit"
$ git merge feature-branch
$ git branch -d feature-branch

$ git commit -m "fourth commit"
```

# Question 4

## Using Git to implement a new feature/change without affecting the main branch 

### 1. Introduction

One of the key advantages of Git is that it makes collaboration easier allowing multiple people to work in parallel without blocking each other. In this article we will see how we can implement a new feature without affecting the main branch.

### 2. Why do we need branching?

When we develop a new feature or fix a bug, it is very likely that some other developer is also working on the same file or same part of the codebase. It would be extremely difficult to coordinate with each other if we have to continuously ensure that we are not breaking each other's code. Imagine when dozens or even hundreds of developers are working on the same codebase.

To avoid such situations, Git provides branching feature. It allows users to collaborate on the same code base at the same time without constantly getting in each other's way, it allows developers to work on features independent of the other contributions being made to the codebase and allows them to add their features to the main codebase when they're ready.

### 3. How to use branches?

Let's assume that we already have `git` installed and configured. Also, we have a remote repository in our Gitlab account. 

Let's create a branch using following commands:
```
$ git clone <url of your remote repository>
```
 
This command copies (or clone) the remote repository to the local machine:
```
$ git clone https://gitlab.com/apagade/Test.git
Cloning into 'Test'...
remote: Enumerating objects: 31, done.
remote: Total 31 (delta 0), reused 0 (delta 0), pack-reused 31
Unpacking objects: 100% (31/31), done.
```

After this command, the new directory `Test` is created on the local machine which contains all the files, branches, and history which is cloned from the master:
```
$ ls
Test

$ cd Test
```

To create a new feature/change branch we can use the `git checkout` command:
```
$ git checkout -b new-feature-branch
Switched to a new branch 'new-feature-branch'
```

This command creates a new branch called `new-feature-branch` and the `-b` argument automatically checks out the new branch.

This branch is a snapshot of the `master` at that current date and time. Any changes to `master` will not be reflected in this branch and vis-a-versa.

We can verify which branch we're on by using the `git branch` command. The output will show an asterisk `*` beside the currently checked out branch:
 ```
$ git branch
  master
* new-feature-branch
```

Now that we have created a new feature branch we can make our changes to the code locally. After the changes are done we can commit changes to this branch.

The `git status` command displays the state of the working directory and the staging area. It helps us track which files have been created/modified/changed/deleted. For example:
```
$ git status
On branch new-feature-branch
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
 
        modified:   README.md
 
no changes added to commit (use "git add" and/or "git commit -a")
```

Now let's add the changes to the staging area. The `git add` command adds the changes in the working directory to the staging area. It tells Git that we want to include updates to a particular file in the next commit.
```
$ git add README.md 
$ git status
On branch new-feature-branch
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
 
        modified:   README.md
```

Running `git status` again will show us a list of changes to be committed.
```
$ git status
On branch new-feature-branch
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
 
        modified:   README.md
```

Note that `git add` doesn't really affect the repository in any significant way. The changes are not actually recorded until we commit our changes. To do this, we use the `git commit -a` command. It will add all modified files to the staging area and commit straightaway:  
```
$ git commit -a
```

This will open up an editor to add a commit message. It is good practice to write a short description about the code changes done.

A simpler and common alternative is to provide the commit message as argument in ` git commit` command itself:  
```
 $ git commit -m "Feature: Added awesome feature"
```

We can use the 'git log' command to get information about the commit history. It lists the recent commit first.

Once the changes are committed locally, the next step is to merge our feature branch into the master. To do this we first need to checkout the `master` and then execute `git merge` command:
```
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.

$ git merge new-feature-branch -m "Feature: New feature"
Updating a7add27..f860def
Fast-forward (no commit created; -m option ignored)
 README.md | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)
 ```

Now our `master` branch is updated with the latest feature created. 

Note how we carried our work independently on the `new-feature-branch` until the code was ready to be merged. Only at the point when we are ready with the new feature, we need to merge the changes into the `master` branch. During the merge we may have to resolve the merge conflict due to the changes made by other developers that were merged into the `master` branch before us. However this is a minor tradeoff in comparison to the advantage we get. There are several best practices to ease the process of conflict resolution which we do not cover here.

The last step is to push the changes to the remote repository and create a **pull/merge** request in gitlab.
 
Merge requests are used to contribute code to repositories where someone else needs to approve our change. When we're contributing to a repository owned by somebody else they may want to review our change before it is incorporated into the codebase. This could be to make sure that it is of an appropriate standard for that repository, that the code works, and that it will be a positive contribution to the codebase.

### 4. Conclusion

Git's branching mechanism empowers teams to collaborate efficiently by isolating changes into separate branches. This approach minimizes conflicts and enables developers to work independently on features or fixes. When changes are ready, merging them into the main master branch becomes a controlled process, ensuring code quality and facilitating seamless integration of new features. This structured workflow not only enhances productivity but also promotes better code management and team coordination in software development projects.

# Question 5

## What is a technical book/blog you read recently that you enjoyed?

I frequently refer to https://www.baeldung.com/linux/ blog for learning various linux commands and concepts. The posts are simple, easy to understand and contains the working commands which could be easily adopted and used for our purpose.
