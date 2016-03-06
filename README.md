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
  * `rpmbuild -ba rpmbuild/SPECS/openssh.spec`

As `rpmbuild -ba` will completely clean the build, run configure again etc., you can just build it once and run `make` in `rpmbuild/BUILD` afterwards to build using modified versions of this patch.

#License
GPLv2
