# Automated WSL Setup
I like to use Windows 10 for gaming and other purposes. I like using Linux/BSDs
in the cloud, however they don't interoperate very well. Until such time comes
when I can buy a MacBook Pro (the only UNIX that works on Desktop) I have
decided to use WSL v1 on my Windows host for all of my Unix related workload.

WSL v1 allows me to use all of my CPU cores, works well with VS Code and does
90% of all I need from it. It is also less clunky compared to a full blown VM or
Docker on Windows (which, by the way, also uses VM).

As an added bonus, if I automate things properly. I can break things in the
userland as much as I want, and simply reset the app from Windows settings and I
am back with  a clean installation. This is an overview of what is being
automated and how. I am using the Debian 10 app from Microsoft Store and the
script is stored at this [Git repo](https://github.com/sranvir/wsl-setup).

## Fetching SSH keys
As stated previously, I use Unix in the cloud. To work with it I need
openssh-client: `apt install openssh-client` . The next step is to bring in my
SSH keys. My SSH keys are store in my Windows home folder in following
subdirectory `C:\User\Ranvir\.ssh`. It also contains a `config` file for SSH
aliases, etc.

The first order of business is to fetch the contents of this directory to my WSL
environment. Since, the Windows username is not the same for everyone, I am
taking the first argument to the below script as your Windows username:

```
sh
WIN_USERNAME=$1

# Get SSH keys from Windows environment
# Need to make the below block more readable, basically it sets permissions for ssh-keys. 
umask 077
if [ ! -d ~/.ssh ]
then
    cp -r /mnt/c/Users/$WIN_USERNAME/.ssh ~/
    chmod 600 ~/.ssh/config
    chmod 400 ~/.ssh/id*
fi
```

I set the umask to 077 so that any directory/file created on the WSL end, by
this script, will have permissions. It is same as permission 700 using `chmod`,
i.e, only the user running the script will have read, write and execute
permission for all the files being created. Everyone else, except `root`, and
other won't have any permission whatsoever.

OpenSSH warns you if the permissions are too liberal. Next I set sane
permissions on config and id_* keys and that's that.

## Autolaunching SSH agent
If you start a process in WSL it will continue to run even when you close the
terminal. Go ahead and run the command `sleep 100` in your WSL terminal, list
the process using `ps`, note the PID and close the terminal. If you open the
terminal again and list the processes, you will find `sleep 100` still running
with the same PID.

However, if you create a variable in your bash environment, e.g VAR=1, that
won't persist the terminal being closed and then being started again.

```sh
$ echo $VAR
1
## Close and reopen your terminal
$ echo $VAR

$
```

So processes persist, but environment variables don't.

So, in theory, if you just manual start `ssh-agent` and then and your keys using
`ssh-add` the process will not die, however, it also requires a couple of
environment variables, i.e, `SSH_AGENT_PID` and `SSH_AGENT_AUTH_SOCK` and these
need to persist as long as you are logged into your Windows host. So we store
these variables in `~/.ssh/agent.env` file and load it using `.profile`.

The below script autolaunches SSH agent if it detects that the agent is not
running, or running without a key and it also handles all the logic with
`agent.env`file. Of course, this is taken from [GitHub help
page](https://help.github.com/en/github/authenticating-to-github/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows),
where it was being used for the Git Bash utility on Windows, but it works quite
well on WSL as well.

```sh
## SSH agent
sshenv=~/.ssh/agent.env

agent_load_sshenv () { test -f "$sshenv" && . "$sshenv" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$sshenv")
    . "$sshenv" >| /dev/null ; }

agent_load_sshenv

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2= agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add
fi

unset sshenv
```
I get a few error codes now and again, when exiting a session, but I don't know
if its because of this script or the Windows Terminal which is still in preview
at the time of this writing.

## Splitting everything into files
When it comes to automation, organization is the name of the game. I don't have
much going on with my repo, still I have different bits of data split into
files. The SSH agent script mentioned above is its own file, so are the various
aliases.

Even though they all are going to be added to the same `~/.profile` file, it is
important to segregate them into discreet chunks. The main function, the
`wsl-setup.sh` script then puts them all together. The script itself is pretty
self explanatory so I won't go into details here.

But you can find `.gitconfig`, `.vimrc`, etc in the repository. Which are simply
copied from the repo into my home directory.

## Installing needed apps
I use Ansible for everyday use, so the script installs ansible, curl, git and a
few other packages that I might need.

## That's it
This simple repo, with way fewer lines that this long post, sets up everything I
need to be productive in a Unix-like userland.
