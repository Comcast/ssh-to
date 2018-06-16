# ssh-to

Hey software engineers! Do you manage servers?  Lots of servers?  Hate copying and pasting IP addresses?  Need a way to execute a command on each of a group of servers that you manage?


## Installation

```
brew tap comcast/opensource https://github.com/Comcast/homebrew-opensource.git
brew install ssh-to
```


## TL;DR

Turn this:
```
$ ssh hadop02.sys.comcast.net ^C
# Oops, I made a typo!
```

Into this:
```
$ ssh-to hadoop 2
# 
# SSHing to 10.100.200.2 (hadoop02.sys.comcast.net)...
# 
Last login: Wed Mar  9 17:00:23 2016 from 10.36.122.137
[dmuth200@hadoop02.sys.comcast.net ~]$ 
```

Or turn this:

```
$ for I in $(seq -w 1 20) do; ssh hadoop{$I}.sys.comcast.net "COMMAND"; done
# Reasonable, but requires a lot of typing and manual commands
```

Into this:

```
$ ssh-to hadoop --loop "COMMAND"
# 
# Looping over group 'hadoop'...
# 
# 
# SSHing to hadoop01.sys.comcast.net...
# 

# All hosts in this group will be SSHed to and have the above command executed.
```


## Requirements

You must have <a href="https://stedolan.github.io/jq/">jq</a> installed on your system so that
the list of servers can be parsed.  If jq is not found, ssh-to will complain and exit.


## Getting Started

In order to use this script, you'll need to create a servers.json file:

`./ssh-to --create`

This will create a sample `servers.json` in the current directory.  You *may* want to then move that
file to `$HOME/.ssh-to-servers.json` so that ssh-to can be run from any directory in the filesystem.

Next, to add hosts and hostgroups to the servers.json file:

`./ssh-to --edit`

This will bring up $EDITOR to edit the file.  After you are done editing, exit the editor, and the 
validity of the JSON will be checked.  If the JSON fails to validate, you'll be prompted to hit
ENTER to go back into the editor or ctrl-C to abort.

An exmample `servers.json` file <a href="servers.json.example">can be found here</a>.


## Usage

Syntax: `ssh-to [ --dump | --loop ] group [ host_number ] [ ssh_args ]`

- If no group if specified, all groups will be printed.
- `--dump` Dump the IP address of the target host. Useful in automation.
- `--loop` Loop through the target group and execute the same command on each host.


Example: 
```
$ ssh-to
# 
# Please re-run me with a group name to see a list of servers in that group.
# 
# Available group names:
# 
#       hadoop
#       kafka
#       splunk
#       zookeeper
# 
# 
# Exmample: ssh-to group_name
# 
```


If no index is specified, all servers in that group will be displayed and 
the user will be prompted to pick one in the next invocation.

Example:
```
$ ssh-to hadoop
# 
# Please re-run me with an index number of which server to SSH into.
# 
# Available hosts in group 'hadoop': 
# 
# 1)     10.100.200.1  hadoop01.sys.comcast.net
# 2)     10.100.200.2  hadoop02.sys.comcast.net
# 3)     10.100.200.3  hadoop03.sys.comcast.net
# 4)     10.100.200.4  hadoop04.sys.comcast.net
# 
# Example: /Users/dmuth200/bin/ssh-to hadoop 1
# 
```

If a group name and index are specified, SSH will be launched:
```
$ ssh-to hadoop 2
# 
# SSHing to 10.100.200.2 (hadoop01.sys.comcast.net)...
# 
Last login: Wed Mar  9 17:00:23 2016 from 10.36.122.137
[dmuth200@hadoop01.sys.comcast.net ~]$ 
```

That was easy! :-)


## Usage in Automated Environments

Want to use this script in an automated environment?  No problem, just use the optional `--dump` parameter:

```
$ ssh-to hadoop 1 --dump
10.100.200.1
```

This can be used within other commands like so:
- `ssh otherusername@$(ssh-to hadoop 1 --dump)`
- `scp file $(ssh-to hadoop 1 --dump):.`
- etc.

It can also be used in shell scripting as in the following example
```
for HOST in $(ssh-to --dump hadoop)
do
   scp file.txt ${HOST}:/path/to/destination/
done
```

### Passing Options to SSH

If your SSH key isn't on the remote server, you're using prompted for a password.
This can be a problem in automated environments.  Instead, try using the $SSH_OPTS 
environment variable. That will let you pass options into the SSH command line.

Example:

`SSH_OPTS=-oBatchMode=yes ssh-to hadoop --loop hostname`

This will print hostnames of hosts in the `hadoop` group that you can SSH into.


## Tmux Support

Yes, we have Tmux support. (how cool is that!?)

When a SSH connection is launched, the current window you are in will be renamed to the human-readable
name of the target host.  When you disconnect, the name will change back to what it was before.

This is useful to help you keep track of when SSH sessions get disconnected.


## Looping Through Many Hosts

Now let's say you have dozens (or even hundreds) of hosts, and you want to execute a command on all of them.
This can be done with the `--loop` parameter:

```
ssh-to hadoop --loop "hostname"
# 
# Looping over group 'hadoop'...
# 
# 
# SSHing to hadoop01.sys.comcast.net...
# 
# Executing command: hostname
# 
hadoop01.sys.comcast.net
# 
# SSHing to hadoop02.sys.comcast.net...
# 
# Executing command: hostname
# 
hadoop02.sys.comcast.net
# 
# SSHing to hadoop03.sys.comcast.net...
# 
# Executing command: hostname
# 
hadoop03.sys.comcast.net
# 
# SSHing to hadoop04.sys.comcast.net...
# 
# Executing command: hostname
# 
hadoop04.sys.comcast.net
```

Execution happens serially.

The `--loop` mode is quite useful for things like maintenance or upgrades.

Naturally, you can chain multiple commands with `--loop`, do inline shell script, etc.  For example:

`ssh-to hadoop --loop "puppet agent --test; service nginx start; nc -vz localhost 80"`


Or lets say you want to do rolling upgrades of your Splunk platform:

`ssh-to splunk --loop "yum install -y splunk; /opt/splunk/bin/splunk stop; /opt/splunk/bin/splunk start --answer-yes --no-prompt"`

That's literally all there is to it!  And I've literally (not figuratively) done this in production.


## Author
<a name="author"></a>

Douglas Muth <douglas_muth@cable.comcast.com>

Bugs can be filed here or emailed directly to me.




