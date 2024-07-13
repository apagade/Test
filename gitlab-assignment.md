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

### 1. System running out of resources

If the system is running out of CPU, memory, disk, or network resources, it will cause the DB, web server, and web application to run slow. Some useful commands to quickly check the system's health:
ps
top
htop
vmstat 1
sar
iotop
iostat -x 1
nstat
ip

### 2. Slow DB Queries

The DB could be slow in processing the queries. This could be either due to an overloaded DB, inefficient/slow queries, or a lack of optimizations like missing indexes (which could have been deleted due to recent deployment), etc.

Some useful commands (assuming PostgreSQL Database):

SELECT * FROM pg_stat_activity; -- Views currently running queries

SELECT * FROM pg_indexes WHERE tablename = 'your_table'; -- Lists all indexes in the database

SELECT * FROM pg_stat_user_tables; -- Shows statistics about table activity

EXPLAIN ANALYZE SELECT * FROM your_table WHERE your_conditions; -- Analyzes a query's execution plan

SHOW max_connections; -- The maximum number of concurrent connections to the database

### 3. Slow Web Server

The web server serving the requests could be slow. This could be due to insufficient resources like CPU/memory, too many open/stale connections to the web server, and/or misconfiguration of heap size, connection pool, etc.

Some useful commands (assuming Tomcat server):
ps aux | grep tomcat
top -p <tomcat_pid> // Monitor tomcat resource usage

jstack -l <tomcat_pid> > thread_dump.txt // enerate a thread dump to analyze what the Tomcat threads are doing

jmap -dump:live,format=b,file=heap_dump.hprof <tomcat_pid> // Create a heap dump to analyze memory usage and potential memory leaks.

netstat -anp | grep <tomcat_pid> // Monitor open network connections and their states.

vi /path/to/tomcat/conf/server.xml // Tomcat configurations

### 4. Slow Application

Recent code changes or deployment issues could cause performance degradation. Look for change logs, any changes to configurations, etc. A quick look at the application logs for an increase in errors or stack traces could be helpful.

Tools like VisualVM (jvisualvm) or jconsole could be helpful to understand JVM-based application performance.

### 5. User Issues

Finally, there could be issues on the user's end as well. For example, the user has a slow internet connection or network congestion, or a slow/overloaded machine/browser. A quick chat with the user and asking them to perform an internet speed test or system restart could help quickly identify and unblock the user.

## 2.2. Describe how you would begin to troubleshoot this issue?

Ideally, there are monitoring and alerting dashboards for such applications that should notify of any SLO breach proactively, even before the user reports the issue. Reviewing monitoring dashboards should quickly indicate if any server components are experiencing or have experienced issues during the reported time frame, which is a good starting point.

If such alerts and monitoring are not in place, we should first attempt to reproduce the issue by following the user's steps. We need to identify which REST endpoint is handling the user's request and analyze its response time. Logging into the machine, we can utilize the commands mentioned in the above section to troubleshoot and pinpoint the root cause.

Subsequently, we can formulate hypotheses and proceed to test them to verify their validity.

## 2.3. System Design and Architectural Tradeoffs

In the current setup, there is only one machine serving user requests. Additionally, the application, web server, and database are all hosted on the same machine. This architecture might suffice for trivial, non-critical, low-usage services, particularly for internal users. However, it is not recommended for serious use cases due to the lack of high availability or the ability to scale horizontally for load balancing. Such a system represents a single point of failure. Moreover, it would require downtime every time a production deployment is performed.

Typically, we would host the web server and database on separate machines. Additionally, placing the web server behind a load balancer ensures that all user requests are routed through it. This setup allows for multiple instances of the web server to handle the load, thereby improving availability even during deployments. Moreover, configuring the database machine with higher resources independent of the web server optimizes resource utilization.

# Question 3

## What sequence of Git commands could have resulted in this commit graph?

```
# Initialize an empty Git repository with 'main' as the default branch
git init --initial-branch=main

# Create the first commit
echo "first" > file.txt
git add file.txt
git commit -m "first commit"

# Create the second commit
echo "second" >> file.txt
git add file.txt
git commit -m "second commit"

# Create and switch to a new branch feature-branch
git checkout -b feature-branch

# Create a commit on feature-branch
echo "awesome feature" >> feature.txt
git add feature.txt
git commit -m "awesome feature"

# Switch back to main branch
git checkout main

# Create the third commit
echo "third" >> file.txt
git add file.txt
git commit -m "third commit"

# Merge feature-branch into main
git merge feature-branch

# Delete feature-branch
git branch -d feature-branch

# Create the fourth commit on main branch
echo "fourth" >> file.txt
git add file.txt
git commit -m "fourth commit"
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

* I recently read [Learning SQL, 3rd Edition](https://www.oreilly.com/library/view/learning-sql-3rd/9781492057604/) by Alan Beaulieu to refresh my SQL knowledge and also gain some new insights.
* I frequently visit the famous [Brendan Gregg's website](https://www.brendangregg.com/index.html) and always find something new to learn.
