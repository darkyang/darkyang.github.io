#github FAQ FOR ME

## **git normal commands**

### init commands

1. git clone ex-- git clone git://github.com/someone/some_project.git   some_project which will clone project from remote repositories to local path - some_project.


2. git init and git remote create version control files .git for local project or files, u can also publish it to remote repositories by command -- git remote add origin git://github.com/someone/another_project.git and use origin to update or commit.
 
## normal commands

### commands for remote repositories

*  checkout -- $git clone [url]
*  view     -- $git remote -v
*  add      -- $git remote add [name] [url]
*  delete   -- $git remote rm [name]
*  change   -- $git remote set-url --push[name][newURL]
*  pull     -- $git pull [remoteName] [localBranchName]
*  push     -- $git push [remoteName] [localBranchName]
*  commit branch --  $git push origin test:master (test-local branch name)


### commands for branch

* view local branch -- $git branch
* view remote branch -- $git branch -r
* create branch -- $git branch [name]
* checkout to new branch -- $git checkout [name]
* create and checkout to new branch -- $git checkout -b[name]
* delete branch --  $git branch -d [name]
* merge branch -- $git merge [name]
* create remote branch -- $git push origin [name]
* delete remote branch -- $git push origin:heads/[name] or git push origin:[name]
* create empty branch -- $git symbolic-ref HEAD refs/heads/[name] $rm .git/index $git clean -fdx


### ignore commands

create file with suffix ".gitignore" and write names of files or folders, ex:

target

bin

*.db


## details

















