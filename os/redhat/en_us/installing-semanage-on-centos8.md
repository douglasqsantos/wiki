# Installing Semanage on CentOS 8

[Semanage](https://linux.die.net/man/8/semanage) (SELinux Policy Management Tool) is used to configure certain parts of SELinux policy without requiring modification to or recompilation from policy sources.

This how-to will help you to install the necessary packages for getting **semanage** command.

Let’s see which package provides us the **semanage** command using the **YUM** command.

```bash
yum whatprovides semanage
```

OR

```bash
yum provides *bin/semanage
```

We will get something like

```bash
Last metadata expiration check: 0:34:40 ago on Sun 05 Jul 2020 01:56:07 PM -03.
policycoreutils-python-utils-2.9-9.el8.noarch : SELinux policy core python utilities
Repo        : @System
Matched from:
Other       : *bin/semanage

policycoreutils-python-utils-2.9-9.el8.noarch : SELinux policy core python utilities
Repo        : BaseOS
Matched from:
Other       : *bin/semanage
```

*The above output tells you that **policycoreutils-python-utils** package provides you the semanage command.*

Now, install the **policycoreutils-python-utils** package using the YUM command.

```bash
yum install -y policycoreutils-python-utils
```

Once the installation is complete, run the semanage to see whether it is available or not.

```bash
semanage -h
usage: semanage [-h]
                {import,export,login,user,port,ibpkey,ibendport,interface,module,node,fcontext,boolean,permissive,dontaudit}
                ...

semanage is used to configure certain elements of SELinux policy with-out
requiring modification to or recompilation from policy source.

positional arguments:
  {import,export,login,user,port,ibpkey,ibendport,interface,module,node,fcontext,boolean,permissive,dontaudit}
    import              Import local customizations
    export              Output local customizations
    login               Manage login mappings between linux users and SELinux
                        confined users
    user                Manage SELinux confined users (Roles and levels for an
                        SELinux user)
    port                Manage network port type definitions
    ibpkey              Manage infiniband ibpkey type definitions
    ibendport           Manage infiniband end port type definitions
    interface           Manage network interface type definitions
    module              Manage SELinux policy modules
    node                Manage network node type definitions
    fcontext            Manage file context mapping definitions
    boolean             Manage booleans to selectively enable functionality
    permissive          Manage process type enforcement mode
    dontaudit           Disable/Enable dontaudit rules in policy

optional arguments:
  -h, --help            show this help message and exit
```
