## Usage

sshrc works just like ssh, but it also sources the `~/.sshrc` on your local computer after logging in remotely.

    $ echo "echo welcome" > ~/.sshrc
    $ sshrc me@myserver
    welcome

    $ echo "alias ..='cd ..'" > ~/.sshrc
    $ sshrc me@myserver
    $ type ..
    .. is aliased to `cd ..'

You can use this to set environment variables, define functions, and run post-login commands. It's that simple, and it won't impact other users on the server - even if they use sshrc too. This makes sshrc very useful if you share a server with multiple users and can't edit the server's `~/.bashrc` without affecting them, or if you have several servers that you don't want to configure independently.

## Installation

### Ubuntu (12.04 and 14.04 only)

    $ sudo add-apt-repository ppa:russell-s-stewart/ppa
    $ sudo apt-get update
    $ sudo apt-get install sshrc

### Archlinux

Install the [sshrc-git][] AUR package.

### Everything else

    $ wget https://raw.githubusercontent.com/Russell91/sshrc/master/sshrc && 
    chmod +x sshrc && 
    sudo mv sshrc /usr/local/bin #or anywhere else on your PATH

## Advanced configuration

Your most important configuration files (e.g. `~/.vimrc`, `~/.inputrc`) might not be bash scripts. Put them in `~/.sshrc.d` and sshrc will copy them to a (guaranteed) unique folder in the server's `/tmp` directory after login. You can find them at `$SSHHOME/.sshrc.d`. You can usually tell programs to load their configuration from the `$SSHHOME/.sshrc.d` directory by setting the right environment variables. Putting too much data in `~/.sshrc.d` will slow down your login times. If the folder contents are > 64kB, the server may block your sshrc attempts.

### Vim

    $ mkdir -p ~/.sshrc.d
    $ echo ':imap <special> jk <Esc>' > ~/.sshrc.d/.vimrc
    $ cat << 'EOF' > ~/.sshrc
    export VIMINIT="let \$MYVIMRC='$SSHHOME/.sshrc.d/.vimrc' | source \$MYVIMRC"
    EOF
    $ sshrc me@myserver
    $ vim # jk -> normal mode will work


### Tmux

If you use tmux frequently, you can make sshrc work there as well. The following seems complicated, but hopefully it should just work.

    $ cat << 'EOF' > ~/.sshrc
    alias foo='echo I work with tmux, too'
    
    tmuxrc() {
        local TMUXDIR=/tmp/russelltmuxserver
        if ! [ -d $TMUXDIR ]; then
            rm -rf $TMUXDIR
            mkdir -p $TMUXDIR
        fi
        rm -rf $TMUXDIR/.sshrc.d
        cp -r $SSHHOME/.sshrc $SSHHOME/bashsshrc $SSHHOME/sshrc $SSHHOME/.sshrc.d $TMUXDIR
        SSHHOME=$TMUXDIR SHELL=$TMUXDIR/bashsshrc /usr/bin/tmux -S $TMUXDIR/tmuxserver $@
    }
    export SHELL=`which bash`
    EOF
    $ sshrc me@myserver
    $ tmuxrc
    $ foo
    I work with tmux, too

The -S option will start a separate tmux server. You can still safely access the vanilla tmux server with `tmux`. Tmux servers can persist for longer than your ssh session, so the above `tmuxrc` function copies your configs to the more permenant `/tmp/russelltmuxserver`, which won't be deleted when you close your ssh session. Starting tmux with the `SHELL` environment variable set to `bashsshrc` will take care of loading your configs with each new terminal. Setting `SHELL` back to `/bin/bash` when you're done is important to prevent quirks due to tmux sessions having a non-default `SHELL` variable.

### Specializing .sshrc to individual servers

You may have different configurations for different servers. I recommend the following structure for your `~/.sshrc` control flow:

    if [ $(hostname | grep server1 | wc -l) == 1 ]; then
        echo 'server1'
    fi
    if [ $(hostname | grep server2 | wc -l) == 1 ]; then
        echo 'server2'
    fi

### Tips

* I don't recommend trying to throw your entire `.vim` folder into `~/.sshrc.d`. It will more than likely be too big.

* For larger configurations, consider copying files to an obscure folder on the server and using `~/.sshrc` to automatically source those configurations on login..

[sshrc-git]: https://aur.archlinux.org/packages/sshrc-git
