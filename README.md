# git-id

## Synopsis
git-id manages Git identities.

### Features
* **SSH keys management**: each identity is associated with a SSH key for remote 
  pulls/pushes. Each identity can have its own SSH key, that will be used 
in-place (no overwriting of existing keys, no key swapping, no `.ssh/config` 
file to modify),
* **identity sets**: the list of defined identities is kept inside the Git 
  repository, so different sets of identities could be defined for different 
projects,
* **per-session identity**: identity is set per-session, nothing is written in 
  Git config files, so different users can work on the same repository at the 
same time, with different identities,
* **lightweight and environment-friendly**: doesn't use any external daemon or 
  configuration files that could persist beyond a shell session. Self-contained, 
portable script.


### Rationale
git-id aims to solve a particular problem with a specific Git use case, but may 
come handy in other situations too. 

I sometimes have to share a user account with other people, and we all contribute to the
same file tree. We're all authenticated under the same account, but need to
retain original authorship for our contributions, and cloning our own copy of
the tree is not practical.
So I've been looking for a way to switch identities before committing and
pushing Git modifications. I've found some tools and wrappers, but none of them
fulfilled those requirements. I need that tool to:
* manage SSH keys (for pushes to GitHub, each user must have a separate key),
* not modify the filesystem (for file-based configuration such as `$HOME/.gitconfig`), 
since several users may need to work on the tree simultaneously, under the same identity (same `$HOME`),
* not launch daemons or create temporary files that could survive a shell session,

So here comes `git-id`. It's a Bash script that can be sourced in a user 
environment, and that provides command to manage Git identities. It will define 
and set environment variables so one can choose the name under which commits 
will be logged, and the SSH key that will be used for pushes/pulls.


## Installation
Just copy the script somewhere and source it. This could be done in your profile
or bashrc files. I have this in `/etc/profile.d/git-id` on my machine:

```
# provides git-id
# http://github.com/kcgthb/git-id
source /share/scripts/git-id
```

## Usage
```
git-id: manage Git user identities
Usage: git-id add    <username> <full name> <email> <sshkey>
       git-id delete <username>
       git-id list
       git-id show   <username>
       git-id current
       git-id use    <username>
       git-id reset
       git-id help
```

### Examples
```
$ echo $GIT_AUTHOR_NAME

$ source /home/kilian/git-id
$ cd project.git
$ git pull
Permission denied (publickey).
fatal: The remote end hung up unexpectedly

$ git-id add john "John Doe" john@example.com /home/jdoe/.ssh/id_rsa
success: identity john added.
$ git-id use john
$ git-id show john
[john]
   name: John Doe
  email: john@example.com
ssh key: /home/jdoe/.ssh/id_rsa
$ git-id current
john
$ git pull
Already up-to-date.
```
