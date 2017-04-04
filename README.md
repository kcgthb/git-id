# git-id

## Synopsis
git-id manages Git identities.

### Features
* **SSH keys management**: each identity is associated with a SSH key for remote 
  pulls/pushes. Each identity can have its own SSH key, that will be used 
in-place (no overwriting of existing keys, no key swapping, no `.ssh/config` 
file to modify),
* **Token management**: for HTTPS remotes, tokens can be used instead of SSH keys.
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
git-id aims to solve a particular problem with a specific Git use-case, but may 
come handy in other situations too. 

I sometimes have to share a user account with other people, and we all contribute to the
same file tree. We're all authenticated under the same account, but need to
retain original authorship for our contributions, and cloning our own copy of
the tree is not practical.
So I've been looking for a way to switch identities before committing and
pushing Git modifications. I've found some tools and wrappers, but none of them
fulfilled those requirements. I needed that tool to:
* manage SSH keys (for pushes to GitHub, each user must have a separate key),
* not modify the filesystem (for file-based configuration such as `$HOME/.gitconfig`), 
since several users may need to work on the tree simultaneously, under the same identity (same `$HOME`),
* not launch daemons or create temporary files that could survive a shell session,

So here comes `git-id`. It's a Bash script that can be sourced in a user 
environment, and that provides commands to manage Git identities. It will define 
and set environment variables so one can choose the name under which commits 
will be logged, as well as the SSH key that will be used for pushes/pulls.


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

Actions: add        add a new identity, or update an existing one
         delete     remove an existing identity
         list       list existing identities
         show       show identity info
         current    display current identity
         use        use the selected identity
         reset      unset the environment, reset to the default id
         help       this message

Usage: git-id add    <username> <full name> <email> <s:sshkey>|<t:token>
       git-id delete <username>
       git-id list
       git-id show   <username>
       git-id current
       git-id use    <username>
       git-id reset
       git-id help

Notes:
   1. a path to a SSH private key *or* an access token can be provided, but not
      both, they're mutually exclusive.
   2. The type of credential should be indicated with the "s:" (for SSH key) or
      "t:" (for token) prefix.
   3. when invoked without any parameters, git-id acts as a command to use in
      GIT_SSH and/or GIT_ASKPASS

```

### Examples
```
$ echo $GIT_AUTHOR_NAME

$ source /home/kilian/git-id
$ cd project.git
$ git pull
Permission denied (publickey).
fatal: The remote end hung up unexpectedly

$ git-id add john "John Doe" john@example.com s:/home/jdoe/.ssh/id_rsa
success: identity john added.

$ git-id current
error: identity not set

$ git-id use john
$ git-id current
john

$ git-id show john
[john]
   name: John Doe
  email: john@example.com
ssh key: /home/jdoe/.ssh/id_rsa

$ git pull
Already up-to-date.
```
