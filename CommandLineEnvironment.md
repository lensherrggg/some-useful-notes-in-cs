# Command-line Environment

## Job Control

```bash
sleep 20
```

The process will sleep for 20s. 

If we type ctrl + C the process will be stopped immediately.  When we type ctrl + C, the system sends out a `SIGINT` signal that will interrupt the program. There is also a signal called `SIGHUP`. If you are running a program and leave your terminal like logout ssh, the signal will be sent which will delete all processes. There are also other signals, use `man signal` for more details. 

Here is an example that shows how to alternate the behavior when receiving a signal. 

```python
import signal, time

def handler(signum, time):
print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

When running, this program will print out consecutive integers. When you type ctrl + C, the program won't stop. Instead it will print out "I got a SIGINT, but I am not stopping" and continues to print out numbers. If you type ctrl + \\, which sends out a SIGQUIT signal, you could quit the program. 


This is helpful when you want to do some intermediate behavior like save current state to some file when someone stopped the program through ctrl + C. 

ctrl + Z will send SIGSTOP signal and suspend the frontground process. The process will be changed to background. 

Run the following commandds

```bash
>> sleep 1000 
# type ctrl + Z
[1] + 63285 suspended  sleep 1000
>> nohup sleep 2000 &
[2] 65367
appending output to nohup.out
>> jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000
```

The first process is suspended and the second process is running background and when it finishes it will write the result to `nohup.out` file. Even if you leave the terminal it will still be running. 

If we run this

```bash
>> bg %1
[1]  - 63286 continued sleep 1000
>> jobs
[1]  - running  sleep 1000
[2]  + running  nohup sleep 2000
```

The kill command allows you to send out any unix signals

```bash
>> kill -STOP %1
[1] + 63286 suspended (signal) sleep 1000
>> jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000
>> kill -HUP %1
>> jobs
[2] + running nohup sleep 2000
>> kill -HUP %2
>> jobs
[2] + running nohup sleep 2000 # nohup ignores SIGHUP
>> kill -KILL %2
[2] + killed nohup sleep 2000
```

`bg` command continues the program suspended in the backgroum. Another command `fg` could recovery a command to foreground and redirect its output to STDOUT. 

## Terminal Multiplexers

There are sometime that you want to put your editor on the left stead and program on the right side or monitoring system resources. Instead of opening multiple terminal windows, we could take a more convenient way, which is called terminal multiplexer. There is a useful tool called `tmux`. 

See online resources for more details. 

## Dot Files

`alias` command is very useful. You could map a short string to a longer one. 

```bash
>> alias ll="ls -lah"
>> ll # this executes "ls -lah" command
```

You could even give a command a default flag

```bash
>> alias mv="mv -i" # this gives mv a default -i flag
>> alias mv
mv='mv -i'
```

Many programs are configured using plain-text files known as dotfiles (because the file names begin with a ., e.g. ~/.vimrc, so that they are hidden in the directory listing ls by default).

```bash
>> vim ~/.bashrc
# append alias sl=ls
>> bash
bash-5.0 $ sl
# the result is the same as ls
```

We could do as the follows to change the prompt

```bash
>> PS1="> "
> ls
# result
```

You could put it into .bashrc file. 

You could also put a directory in `~` directory named `dotfiles` and put some dotfiles inside. Use `ln -s` command to create symlink for your dotfiles. Through this you could put your dotfiles on a github repository and do some version control but you don't need to create a `.git/` folder inside `~` directory. 

## Remote Machine

The main command for working on remote machines is `ssh`. 

```bash
>> ssh username@xxx.xxx.xxx.xxx # username@IP address
>> ssh username@url # connect to a server that have a dns name
>> logout
Connection to xxx is closed. 
```

You could also just execute remote commands. 

```bash
>> ssh username@URL ls -la
# After you type in your password, result of "ls -la" command will be printed but you are still in your local terminal. 
```

Key-based authentication exploits public-key cryptography to prove to the server that the client owns the secret private key without revealing the key. This way you do not need to reenter your password every time. Nevertheless, the private key (often ~/.ssh/id_rsa and more recently ~/.ssh/id_ed25519) is effectively your password, so treat it like so.

To generate a pair you can run ssh-keygen.

```bash
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

You should choose a passphrase, to avoid someone who gets hold of your private key to access authorized servers. Use ssh-agent or gpg-agent so you do not have to type your passphrase every time.

If you have ever configured pushing to GitHub using SSH keys, then you have probably done the steps outlined here and have a valid key pair already. To check if you have a passphrase and validate it you can run 

```bash
ssh-keygen -y -f /path/to/key.
```

ssh will look into .ssh/authorized_keys to determine which clients it should let in. To copy a public key over you can use:

```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

A simpler solution can be achieved with ssh-copy-id where available:

```bash
ssh-copy-id -i .ssh/id_ed25519.pub foobar@remote
```

We could use `scp` command to copy local file to remote machine. 

```bash
scp notes.md username@url:foobar.md
```

We could also use `rsync` command which could continue from breakpoint if you break into sudden offline. 

Using ~/.ssh/config could save you from typing `ssh` every time, which is a better way compared with `alias`. 

```
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```

After copied the above `config` file, we could just run `ssh vm` to connect the host called vm. 

