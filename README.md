# shally (ssh-ally)

> Send dotfiles with you over SSH

I was using [sshrc](https://github.com/Russell91/sshrc) so far for carrying my dotfiles with me over SSH. It was really useful until I got the following remark from the person managing the server I was connecting to:

> Does anyone know what this super long and weird process is? It feels like an attacker that has succeeded to run a custom program.

![ps-sshrc](https://raw.githubusercontent.com/lobre/shally/master/ps-sshrc.png)

Effectivery, the footprint of sshrc on the processes is quite heavy so I wanted to find a more discreet way of bringing my dotfiles with me. On top of this, there is a restriction on the dotfiles folder size with sshrc.

> If the folder contents are > 64kB, the server may block your sshrc attempts.

That is why I came up with this new shally script that would answer these problematics.

It relies on the ControlMaster feature of the OpenSSH client to reuse a same connection for the following tasks:

- Initiate a master connection and store the socket file.
- Send dotfiles in the `/tmp/` folder using `scp` or `rsync` (if available) using the same connection as before.
- Call the actual `ssh` process using again the same connection.
- Connect a last time before exit to delete all traces.
- Close the master connection.

Using the same connection allows to speed up the process so it feels only the actual `ssh` call is made. Then the speed with depend on the heavyness of your dotfiles but it should allow to transmit bigger than 64kB.

The biggest the files you transfer, the longest you will have to wait to get the ssh prompt as shally needs to send your files remotely before.

In the `ps` table, the command is much more discreet.

![ps-shally](https://raw.githubusercontent.com/lobre/shally/master/ps-shally.png)

## Installation

Just download the shally script, make it executable and put it in your path.

If you want shally as default instead of ssh, alias it in your `$HOME/.bashrc`.

    # Don't alias if currently in remote ssh session
    # This allows to use this same bashrc sent remotely by shally
    if command -v shally >/dev/null && [ -z "$SSHHOME" ]; then
        alias ssh="shally"
    fi

## Configuration

shally uploads the two following paths on the remote machine:
- `$HOME/.sshrc` under `$SSHHOME/sshrc`
- `$HOME/.sshrc.d/` under `$SSHHOME/sshrc.d/`

Then it sources the `$HOME/.sshrc`

So for instance, if you want to have your bashrc sourced on the remote, copy it under `$HOME/.sshrc.d/bashrc` locally and then use this your `$HOME/.sshrc` file.

```
$ cat $HOME/.sshrc
source $SSHHOME/sshrc.d/bashrc;
```

This works pretty much the same way as sshrc. Don't hesitate to check [sshrc](https://github.com/Russell91/sshrc)'s README for more information.
