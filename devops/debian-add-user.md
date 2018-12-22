# Adding a user on Linux
If you are responsible for administering a cluster and a large set of users, automation using Ansible (or alternatives) is the way to go. These instructions can come in handy when you are getting started or interested in knowing what happens under the hood of DevOps magic.

## Prerequisites
The following information must be available in the work order to complete the task:

- New user's full name (first and last)
- New user's public key
- New user's password hash
- [optional] New user's UID (if your org is into infrastructure management, you should consider UID management)
- List of groups the user should belong to

## User steps
### New user generates and/or provides public key and password hash.
It is not required to provide an e-mail address as a comment (-c option below), and in fact not recommended. Instead provide a meaninful "totem" that only the user can recognize to its purpose. We also recommend that the user set a short password on the key when prompted.

```shell
> ssh-keygen -t rsa -b 4096 -c "my-totem" -f ~/.ssh/totem-rsa
> mkpasswd --method=sha-512
```

NOTE: never share your private key. Double check you are providing the contents of .pub key file when prompted. You are responsible for keeping your private key file private!

## Sys Admin steps
### Create user and add the user to specified groups
Copy the key and password hash to /tmp. Get the UID and group list from the work order. In the instructions below replace the $variables with values from the work order. The convention in most unixy systems is to use the first name of the user in lowercase.

```shell
> sudo useradd -m -s /bin/bash -c $NUFullName -p /tmp/$NUPasswdHashFileName -u $NUUid $NUFirstName
> sudo usermod -aG sudo $NUFirstName
```

### Setup public key and passwordless login
```shell
> su - $NUFirstName
> mkdir ~/.ssh
> cp /tmp/$NUPublicKeyFileName ~/.ssh/authorized_keys
> chmod 700 -R ~/.ssh && chmod 600 ~/.ssh/authorized_keys
> exit
```

## User test
Get the user to verify that they can successfully scp and ssh to the system on which they got added.

```shell
> scp -i ~/.ssh/totem-rsa test-file $NUFirstName@system.example:~/
> ssh -i ~/.ssh/totem-rsa $NUFirstName@system.example.com
> ls -ltr
> # user's test-file should be listed.
```

# References
- [Crypt](http://man7.org/linux/man-pages/man3/crypt.3.html)
- [Generating a new ssh key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
- [Use mkpasswd from whois package](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=116260)
- [User add command](http://man7.org/linux/man-pages/man8/useradd.8.html)
