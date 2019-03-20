# shally

> Send dotfiles with you over SSH

I was using [sshrc](https://github.com/Russell91/sshrc) so far for carrying my dotfiles with me over SSH. It was really useful until I got the following remark from the person managing the server I was connecting to:

> Does anyone know what this super long and weird process is? It feels like an attacker that has succeeded to run a custom program.

![ps-sshrc](https://raw.githubusercontent.com/lobre/shally/master/ps-sshrc.png)

Effectivery, the footprint of sshrc on the processes is quite heavy so I wanted to find a more discreet way of bringing my dotfiles with me. On top of this, there is a restriction on the dotfiles folder size with sshrc.

> If the folder contents are > 64kB, the server may block your sshrc attempts.

That is why I came up with this new shally script that would answer these problematics.

It rely on the ControlMaster feature of the OpenSSH client to reuse a same connection for the following tasks:

- Initiate a master connection and store the socket file.
- Send dotfiles in the `/tmp/` folder using `scp` and the same connection as before.
- Call the actual `ssh` process using again the same connection.
- Connect a last time before exit to delete all traces.
- Close the master connection.

Using the same connection transparently allow to speed up the process so it feels like only the actual `ssh` call is made. Then the speed with depend on the heavyness of your dotfiles but it should allow to transmit bigger than 64kB.

In the `ps` table, the command is much more discreet.
