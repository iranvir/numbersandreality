# Managing Multiple GitHub Accounts from your Shell
2020 is the [Year of Structure](https://youtu.be/NVGuFdX5guE) for me. This means
organizing, planning and filing things in a way that frees up my mind to be
actually productive. It doesn't have to be perfect, but it needs to be portable,
and distraction free. After I completed my [WSL 1
setup](https://github.com/sranvir/wsl_setup.git) with a proper persistent SSH
agent, I immediately ran into another limitation. GitHub accounts...

One of the ways I segregate my life is by having different accounts for
different purposes. So, naturally, I ended up with 2 GitHub accounts one for my
personal projects, and another one for an existing long-term client. When I
looked up "how to manage multiple GitHub accounts" I found this [excellent
article](https://www.freecodecamp.org/news/manage-multiple-github-accounts-the-ssh-way-2dadc30ccaca/).
And if you are interested in a *how-to* please go ahead and follow along. This
is write-up is not a how-to.

## Asking the Right Questions

When confronted with an issue, the knee jerk reaction is to copy paste commands
from the Internet. I was tempted to do the same.

However, such issues also offer a precious opportunity to reflect on what you
know, to apply your knowledge to solve a new situation, and to learn more about
the uderlying system. Instead of asking how the to run multiple GitHub accounts,
let's ask:

> How does Git from shell work at all?

How can we not use GitHub username and password and still be able to push
commits to a remote repository?

## Right Parameters

Let's start with only one GitHub account and one associated key, already added
to it. Now, let's ask the following:

### 1. How GitHub knows it is you?

It uses SSH keys  to authenticate it's you. Every time you push a commit, you
"SSH into" a GitHub server somewhere, as the user `git` accessing the server by
the domain name `github.com`. Your key, is your identity, you CANNOT add the
public counterpart of your key to two different GitHub accounts. You see this
whenever you clone a repository over SSH:

```bash
git clone git@github.com/user/repo.git
```

Locally, git(1) uses `~/.gitconfig` file to specify the author, and their email,
everytime you make a commit. So if you are commiting on a work repo, and if your
.gitconfig file has your personal username and email, git will happily use your
personal name and email in the commit message. And it will be pushed to remote
repo as well. We will fix this issue in a moment.

### 2. How your local git CLI knows which repo to push the changes to?

This is simple, go to any repo, and you can use the command `git remote -v` to
see what remote repository a local clone points to. This detail is also stored
in the .git folder inside the root of any git repo, you can dig in it, to see
more details.

## Solving the problem at hand

We now know the pieces of the puzzle, git, gitconfig and most importantly SSH.
Can we use what we know about these,  to solve our problem? Let's break it down.

### GitHub and SSH keys

We know ssh(1) is involved, can we use our existing knowledge about ssh(1) to
achieve this?

If you are using `ssh(1)`, chances are that you are familiar with it's
configuration. The usual suspects for this config are `~/.ssh/` and
`/etc/ssh/ssh_config`. I would recommend picking the former, but you know your
workflow better than me.

We often tweak this file to avoid repeated typing. For a given remote endpoint,
we specify Host parameter to often shorten a lengthy domain name of an IP
address, and similarly we use User parameter to specify username to ssh as.
Let's use the config for our purpose here!

With SSH keys for both your accounts added to your ssh-agent(1), you can tweak
the `~/.ssh/config` file to use a different key for a different "Host"s:

```bash
Host personal.github.com
    Hostname github.com
    User git
    HostKeyFile ~/.ssh/personal.key
Host work.github.com
    Hostname github.com
    User git
    HostKeyFile ~/.ssh/work.key
```

> The above little snippet tells SSH, "If the user specifies
> `personal.github.com`, use personal.key, and username  `git`. And don't be
> fooled, SSH,`personal.github.com` doesn't exist. We are just connecting to
> github.com, but it is named weirdly here. Ditto for work.github.com "

You can test this by running the command

```bash
$ ssh -T git@work.github.com
Hi (your_work_username)! You have successfully authenticated, but GitHub does not provide shell access.
$ ssh -T git@personal.github.com
Hi (your_personal_username)! You have successfully authenticated, but GitHub does not provide shell access.
```

Moving forward, we can use the above aliases to github, when cloning a
repository. Instead of using:

```sh
git clone git@github.com:user/repo.git
```

Use the following

```sh
git clone git@personal.github.com/user/repo.git
git clone git@work.github.com/user/repo.git
```

You won't have to do anything special when pushing changes, the `.git` folder in
your repo will use this aliased Hostname and everything will be work as normal.

The Host need not even be work.github.com, if you want it can be any reasonable
string like `github.com-work`, etc, as is used by other blog posts.

### Git and Gitconfig

The `~/.gitconfig` can be arbitrary, and don't play a role in authentication.
With any existing gitconfig you can push your commits. But the commit message
will have that arbitrary username and email mentioned as its author. Good
enough? Not for me!

The .gitconfig file in your home folder is a global config for that user. While
it can contain other variables, the following commands set the user.name and
user.email, variables that concern us:

```sh
git config --global user.name "username"
git config --global user.email "user@example.com"
```

You can override it for a specific repo by going inside that repo, and running
`git config user.email "name@work.com"` (without the `--global` flag) and a
corresponding user.name command.

I don't do that. Too much typing and book keeping is involved with local
configs.

Instead, I wrote a simple script that allows me to toggle between the two
possible `~/.gitconfig` states. Make the script an executable, and add it to
your PATH. Now, whenever you call this script, it switches between the two
accounts.

```sh
#!/bin/bash
touch ~/.gitconfig
if [ $(git config --get user.name) == "personal_username" ]; then
    git config --global user.name "work_username"
    git config --global user.email "work@example.com"

else
    git config --global user.name "personal_username"
    git config --global user.email "personal@example.com"
fi

exit 0
```

Obviously, the above script is not a clean solution. But it works for me. You
will easily be able to find better solutions for your use cases. Try solving it
for three accounts.

I also added a little bit of code at the end of my `~/.bashrc`, now my prompt
tells me what GitHub user I have toggled into. Again, this is specific to my
personal taste, and you might find better solutions for yourself.

```sh
# Custom Prompt
parse_git_user() {
    git config --get user.name
}
export PS1="\[\e[36;40m\]\$(parse_git_user)\[\e[m\]:\[\e[35;40m\]\w\[\e[m\] \\$ "
```

Finally, my prompt looks something like this:

```bash
ranvir95:~ $ toggle
sranvir:~ $
```

And now I can switch between procrastination and paid-procrastination with ease!

## Conclusion

No setup is perfect, and it will never be, as you build more, and try to do more, you will run into issues with the systems. It doesn't matter if that's on your personal laptop with a fancy GUI, or on a remote system with nothing but a pathetic prompt.

Try and reason about the obstacles from first princples and you will build efficient and elegant solutions.
