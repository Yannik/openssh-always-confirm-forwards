# What does this patch do?
This openssh patch makes ssh-agent always ask for confirmation when a remote server that the ssh-agent has been forwarded to tried to use one of your keys.
It will however not hassle you (unless you specifically use `ssh-add -c`) every time your local machine uses the ssh-agent to connect to a remote machine.

This improves ssh-agent forwarding alot.

# What should I use this for?
If you are connecting to a remote host over a jump host, use `ProxyCommand` instead of this!
Use this patch only if you need to transfer files directly between two remote servers and use it only temporarily by manuall connecting with `ssh -A`.

# Why should I use it?
Because agent forwarding is a big security risk, as the remote server can unbeknownst use your ssh-agent to tamper with other servers you have access to:

 * [https://heipei.github.io/2015/02/26/SSH-Agent-Forwarding-considered-harmful/](https://heipei.github.io/2015/02/26/SSH-Agent-Forwarding-considered-harmful/)
 * [https://www.reddit.com/r/netsec/comments/2m2zpb/the_perils_of_using_an_sshagent/](https://www.reddit.com/r/netsec/comments/2m2zpb/the_perils_of_using_an_sshagent/)
 * [http://rabexc.org/posts/pitfalls-of-ssh-agents](http://rabexc.org/posts/pitfalls-of-ssh-agents)
 * [https://www.commandprompt.com/blogs/ildefonso_camargo/2013/11/security_considerations_while_using_ssh-agent/](https://www.commandprompt.com/blogs/ildefonso_camargo/2013/11/security_considerations_while_using_ssh-agent/)
 * [http://lackof.org/taggart/hacking/ssh/](http://lackof.org/taggart/hacking/ssh/)

This can patch should secure you against that.

# Why shouldn't I use it?
Because I am definitely no expert in C and just hacked this together based on my dubious programming skills in other languages.

So, use it at your own risk!

# How to secure using ssh-agent a bit more:
Use [ssh-ident](https://github.com/ccontavalli/ssh-ident) to separate your identities!

While researching for this project, I also found [reverse-ssh-agent](https://github.com/ewindisch/reverse-ssh-agent) which seemed interesting too, but I did not review it.

# How to build openssh with this patch applied (Fedora):
  * `dnf download --source openssh-clients`
  * Do add the patch as described [here](https://unix.stackexchange.com/questions/16904/how-to-unpack-modify-rebuild-and-install-a-srpm)
  * Install builddeps as described [here](https://stackoverflow.com/questions/13227162/automatically-install-build-dependencies-prior-to-building-an-rpm-package)
  * `export RPM_BUILD_NCPUS=12`
  * `rpmbuild -bb rpmbuild/SPECS/openssh.spec`

As `rpmbuild -ba` ([rpmbuild reference](http://www.rpm.org/max-rpm-snapshot/ch-rpm-b-command.html)) will completely clean the build, run configure again etc., you can just build it once and run `make` in `rpmbuild/BUILD` afterwards to build using modified versions of this patch.

  * Install using `rpm --install rpmbuild/RPMS/x86_64/openssh-clients-*.rpm`. You will probably want to use the `--replacepkgs` and/or `--replacefiles` options as you most likely already have `openssh-clients` installed.
  * Make sure that your package is not getting overwritten by a packge update in the fedora repo: 
    * Either use [`dnf versionlock openssh-clients`](https://dnf-plugins-extras.readthedocs.org/en/latest/versionlock.html). This will still list the package as installed.
    * or use the `exclude` option in `dnf.conf` (www.systutorials.com/1661/making-dnf-yum-not-update-certain-packages/). This will not list the selected packages with `dnf list installed` or on package updates anymore.

# How to build openssh with this patch applied (Debian/Ubuntu):
  * `apt-get source openssh-client`
  * `apt-get build-dep openssh-client`
  * `cd openssh*`
  * apply the patch manually, as the debian openssh package differs alot from the one on fedora from which this patch was created
  * `dpkg-buildpackage`

# Make sure that your custom version of ssh-agent is actually being used
On Fedora/Gnome by default the openssh `ssh-agent` is not used, but a custom ssh-agent provided by gnome-keyring. Check `$SSH_AUTH_SOCK`, if it is `/run/user/uid/keyring/ssh`, it is gnome keyring. The socket path for openssh `ssh-agent` is something like `/tmp/ssh-xxxxx/agent.pid`.

You can disable the gnome-keyring ssh component like this:

```
mv /etc/xdg/autostart/gnome-keyring-ssh.desktop /etc/xdg/autostart/gnome-keyring-ssh.desktop.disabled
```

Log out of your session and back in, and `$SSH_AUTH_SOCK` should now be empty when you open a shell.

Append to your `.zshrc`/`.bashrc`:
```
ssh-add -l &>/dev/null
if [ "$?" = 2 ]; then
  test -r ~/.ssh-agent && \
    eval "$(<~/.ssh-agent)" >/dev/null

  ssh-add -l &>/dev/null
  if [ "$?" = 2 ]; then
    (umask 066; ssh-agent > ~/.ssh-agent)
    eval "$(<~/.ssh-agent)" >/dev/null
    ssh-add
  fi
fi
```

Modified from: [http://rabexc.org/posts/pitfalls-of-ssh-agents](http://rabexc.org/posts/pitfalls-of-ssh-agents)

One thing you do unfortunately use from using this over gnome-keyring's ssh functionality is automatically adding keys when you start a ssh connection, instead, you need to add them manually using `ssh-add`.

A workaround for this is to add something like

    alias ssh='ssh-add -l > /dev/null || ssh-add && ssh'

to your respective `.zshrc`/`.bashrc`.

Here is a question on stackexchange on that topic: https://superuser.com/questions/325662/how-to-make-ssh-agent-automatically-add-the-key-on-demand

# License
GPLv2
