<!-- toc -->

# Basic Shell Scripting

## Creating a shell script

To create script file just touch it and make it executable. Example:

```shell
$ cd /tmp
$ touch hello
$ chmod u+x hello
$ ls -al hello
-rwxr--r-- 1 bioboost bioboost 0 Sep 30 11:45 hello
```

Open the file with nano and add this `#!/usr/bin/env bash` as a first line:

```shell
nano hello
```

This is what is called the shebang. Under Unix-like operating systems, when a script with a shebang is run as a program, the program loader parses the rest of the script's initial line as an interpreter directive; the specified interpreter program is run instead, passing to it as an argument the path that was initially used when attempting to run the script. In this example the script is run by the bash Linux shell.


> #### Note::The older shebang
>
> In the older days the shebang for a bash script was `#!/bin/bash`. However as some
> Linux systems dare to put the bash shell executable in a different place, these
> scripts would fail. By using the newer `#!/usr/bin/env bash` this problem is solved.

> #### Alert::sh is not Bash
>
> Please do not be fooled by scripts or examples on the Internet that use `/bin/sh` as the interpreter. sh is not bash! Bash itself is a "sh-compatible" shell (meaning that it can run most 'sh' scripts and carries much of the same syntax) however, the opposite is not true; some features of Bash will break or cause unexpected behavior in sh. Also, please refrain from giving scripts a .sh extension. It serves no purpose, and it's completely misleading (since it's going to be a bash script, not an sh script).

## Outputting text to the console

By using `echo` you can output text to the terminal:

```bash
#!/usr/bin/env bash

echo "Hello World"
```

## Running the script

The script can be run using the following command:

```shell
$ cd /tmp
$ ./hello
Hello World
```

## Variables

Variables are areas of memory that can be used to store information and are referred to by a name.

To create a variable, put a line in your script that contains the name of the variable followed immediately by an equal sign ("="). No spaces are allowed. After the equal sign, assign the information you wish to store. Note that no spaces are allowed on either side of the equal sign.

To use a variable place a dollar sign ("$") in front of the name where you want to use it.

```bash
#!/usr/bin/env bash

hello="Hello World"

echo $hello
```

As the name variable suggests, the content of a variable is subject to change. This means that it is expected that during the execution of your script, a variable may have its content modified by something you do.

On the other hand, there may be values that, once set, should never be changed. These are called constants. I bring this up because it is a common idea in programming. Most programming languages have special facilities to support values that are not allowed to change. Bash also has these facilities but, to be honest, I never see it used. Instead, if a value is intended to be a constant, it is simply given an uppercase name. Environment variables are usually considered constants since they are rarely changed. Like constants, environment variables are given uppercase names by convention.

So the previous script should actually be written as:

```bash
#!/usr/bin/env bash

HELLO="Hello World"

echo $HELLO
```

## Command substitution

You can substitute the output of a command into your script by using a dollar sign
followed by parentheses. Let's for example say we want to combine the output of a `cat` command with the output of a `date` command to
make some sort of log entry.

```bash
#!/usr/bin/env bash

mem=$(cat /proc/meminfo | grep 'MemAvailable')
logdate=$(date)

echo "[$logdate] $mem"
```
The characters "$( )" tell the shell, "substitute the results of the enclosed command."

Or even shorter:

```bash
#!/usr/bin/env bash

echo "[$(date)] $(cat /proc/meminfo | grep 'MemAvailable')"
```

## Quoting

Quoting is used to accomplish two goals:

* To control (i.e., limit) substitutions and
* To perform grouping of words.

### Single and double quotes

The shell recognizes both single and double quote characters. The following are equivalent:

```bash
var="this is some text"
var='this is some text'
```

However, there is an important difference between single and double quotes. Single quotes limit substitution. As we saw in the previous lesson, you can place variables in double quoted text and the shell still performs substitution. We can see this with the echo command:

```shell
$ echo "My host name is $HOSTNAME."
My host name is linuxbox.
```

If we change to single quotes, the behavior changes:

```shell
$ echo 'My host name is $HOSTNAME.'
My host name is $HOSTNAME.
```

Double quotes do not suppress the substitution of words that begin with "$"
but they do suppress the expansion of wildcard characters.

For example, try the following:

```shell
$ echo *
embedded_course hello hello.txt hsperfdata_mdm icedteaplugin-mdm-H8q7Cj mintUpdate pulse-PKdhtXMmr18n ssh-5OZhO7zNIJ8U systemd-private-f412405725f34e05a9ab0c320a71be53-colord.service-HGUSHh systemd-private-f412405725f34e05a9ab0c320a71be53-rtkit-daemon.service-bCX7DF
```

```shell
$ echo "*"
*
```

### Escaping a character

There is another quoting character you will encounter. It is the backslash.
The backslash tells the shell to "ignore the next character."
This is typically called escaping a character. Here is an example:

```shell
$ echo "My host name is \$HOSTNAME."
My host name is $HOSTNAME.
```

By using the backslash, the shell ignored the `$` symbol. Since the shell ignored it, it did not perform the substitution on $HOSTNAME.

Here is a more useful example:

```shell
$ echo "My host name is \"$HOSTNAME\"."
My host name is "linuxbox".
```

As you can see, using the `\"` sequence allows us to embed double quotes into our text.

## Flow control

### Making decision

When writing scripts there will always come a time when it is necessary to define a different path based on a certain condition.

Some examples:

* Only execute some commands if a file exists
* Make a log entry if something fails
* Check user input data for validity
* ...

To add such decisions to our scripts we will require constructs that allow us to test for conditions. The if statement is one of those constructs. It's syntax is the following:

```bash
if condition; then
  # Do something
elif condition; then
  # Do another thing
else
  # Do something else
fi
```

The condition is a bit different from the conditions we know from programming languages such as C++ or PHP.

#### Exit status

A first option for the condition can be the exit status of a command. Each command or
program that is executed will end with a status code between 0 and 255 (can be negative on windows).
By convention, the value 0 means that everything went fine, while an exit status different from 0 means
that something went wrong.

You can actually output the return values of Linux system commands such as ls and all by
echoing `$?` on the command line. Try the following example:

```shell
$ ls /usr/bin
$ echo $?
0
```

The status of 0 indicates all went well.

Now try:

```shell
$ ls /does_not_exist
ls: cannot access 'does_not_exist': No such file or directory
$ echo $?
2
```

The status is not zero, indicating that the `ls` command did not terminate properly or something went wrong.

Putting it all together you can build an if statement based on the output of a command.
Let's see an example where we try to create a file in a directory we don't have access too.
Next we try to create a file in the `/tmp` dir where everyone has access:

```bash
#!/usr/bin/env bash

if touch /usr/bin/hello; then
    echo "Created hello in bin"
else
    echo "Failed to create hello in bin."
fi

if touch /tmp/hello; then
    echo "Created hello in tmp"
else
    echo "Failed to create hello in tmp."
fi
```

This construct is less used but can sometimes be helpful.

#### Testing

The test condition is most often used for the if statement. While there are two
syntactically different form, both work exactly the same.

```bash
#!/usr/bin/env bash

if test expression; then
    # Do something
else
    # Do another thing
fi
```
The second form, which is used more often is shown below:

```bash
#!/usr/bin/env bash

if [ expression ]; then
    # Do something
else
    # Do another thing
fi
```

Bash features a lot of built-in checks and comparisons, coming in quite handy in many situations.

String comparisons:

| Expression | Description |
| ---------- | ----------- |
| str1 = str2 | Returns true if the strings are equal |
| str1 != str2 | Returns true if the strings are NOT equal |
| -z str | Returns true if the string is empty |
| -n str | Returns true if the string is NOT empty |

Always make sure to add double quotes around the string and string variable.

Numeric comparisons

| Expression | Description |
| ---------- | ----------- |
| expr1 -eq expr2 | Returns true if the expressions are equal |
| expr1 -ne expr2 | Returns true if the expressions are not equal |
| expr1 -gt expr2 | Returns true if expr1 is greater than expr2 |
| expr1 -ge expr2 | Returns true if expr1 is greater than or equal to expr2 |
| expr1 -lt expr2 | Returns true if expr1 is less than expr2 |
| expr1 -le expr2 | Returns true if expr1 is less than or equal to expr2 |
| ! expr1 | Negates the result of the expression |

Some file conditionals:

| Expression | Description |
| ---------- | ----------- |
| -d <directory> | Returns true if the directory exists |
| -f <file> | Returns true if the file exists |
| -r <file> | Returns true if the file is readable by the user running the script |
| -w <file> | Returns true if the file is writeable by the user running the script |
| -x <file> | Returns true if the file is executable by the user running the script |


Let's rewrite the small script mentioned in the previous section that checks the status
of the touch command:

```bash
#!/usr/bin/env bash

touch /usr/bin/hello;

if [ $? -eq 0 ]; then
    echo "Created hello in bin"
else
    echo "Failed to create hello in bin."
fi

touch /tmp/hello;

if [ $? -eq 0 ]; then
    echo "Created hello in tmp"
else
    echo "Failed to create hello in tmp."
fi
```

If something fails in your script or it should be prematurely ended then you can
always make use of the exit command. By providing a numeric argument (0 to 255) you can
set the exit code of your script.

Example:

```bash
#!/usr/bin/env bash

userid=$(id -u)

if [ "$userid" = "0" ]; then
    echo "Running script as root"
else
    echo "You must be superuser to run this script"
    exit 1
fi
```

## Assignments

> #### Assignment::Cloning course repositories
>
> Create a script that clones the course repositories from Github. Make sure to check if repository already exists and if it does only do a pull. The repos are https://github.com/BioBoost/embedded_systems_course and https://github.com/BioBoost/embedded_systems_tools_and_scripts

<!-- How to break here? -->

> #### Assignment::Updating a project on the pi
>
> Create a script which copies a directory, including all its files, to the raspberry pi. Do this using the secure shell copy (scp).

<!-- How to break here? -->

> #### Assignment::SSH setup
>
> Create a script which setups the SSH keys on the pi and also creates keys for the current user if none exist yet. Hint: you can use ssh to execute commands on a remote host.
