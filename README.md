# ratcraft
Minimal Bash Minecraft server launch/control/backup script.

The purpose of this script is to daemonize a Minecraft server (run it in the background) using GNU `screen` (present on most UNIX-like OSs) and allow simple control of the server.  It follows the principal of "the simplest thing which could possibly work."

Features:

* Start the server and run it in the background
* Stop the server
* Run arbitrary Minecraft server commands at the command line
* Pause Minecraft world saving and create a compressed backup of the world files

## Setup

There are some settings at the top of the script which you'll likely need to change:

```
rootdir=~/mcserver
world=$rootdir/world
backups=$rootdir/backups
jar=~/minecraft_server.1.15.2.jar
sname=ratcraftscreen
start_cmd="java -Xms512M -Xmx1024M -XX:ParallelGCThreads=1 -jar '$jar' nogui"
```

In particular, the first four are likely to need some changes. These are my current actual settings.

* `rootdir` - The server's working directory. Minecraft server will populate this with files such as `eula.txt`, `server.properties`, `whitelist.json`, and so forth.
* `world` - (Directory) I have mine under `rootdir`, but this could be anywhere. Minecraft will populate this with files such as `level.dat`, etc. when it creates the world for the first time. You could also point this to the path of an existing world.
* `backups` - (Directory) Again, I've placed this under `rootdir`, but feel free to point this to any valid path. This is where the command `ratcraft backup` will attempt to save `*.tgz` backup files.
* `jar` - This is the exact name of the Minecraft Java ARchive file (`*.jar`) you wish to run.  Could be a Bukkit or Spigot server or Vanilla Minecraft server from minecraft.net.  You choose the flavor!

## Commands

Run `ratcraft` with arguments to see terse usage instructions:

```
root@darkstar:~# ratcraft
Usage: ratcraft ( start | stop | status | backup | watch | cmd <command> )
```

Let's look at each of these sub-command options in turn:

* `ratcraft start` - Starts the specified Minecraft server .jar under a GNU `screen` session. Runs in the background.
* `ratcraft stop` - Stops the running server and/or `screen` session.
* `ratcraft status` - Prints out a neat little summary of the settings and whether or not the server appears to be running. Example below.
* `ratcraft backup` - Creates a backup of the current running world. (Halts world saving, uses GNU `tar` to create a compressed archive of the world directory, resumes world saving.)
* `ratcraft watch` - Runs GNU `tail -f` on the Minecraft server log so you can see new log messages as they are written. End with `CTRL+C`.
* `ratcraft cmd '<your command>'` - Send a command to the running Minecraft server.  See the examples below.

### status Example

As promised, here's an example of running the status sub-command. Here's my current server status:

```
root@darkstar:~# ratcraft status
Settings:
    rootdir:    /root/mcserver
    world:      /root/mcserver/world
    backups:    /root/mcserver/backups
    jar:        /root/minecraft_server.1.15.2.jar
    sname:      ratcraftscreen (id of screen session)
    start_cmd:  java -Xms512M -Xmx1024M -XX:ParallelGCThreads=1 -jar '/root/minecraft_server.1.15.2.jar' nogui
Server is running.
```

### cmd Examples

You can talk to your players with `say`:

```
root@darkstar:~# ratcraft cmd 'say Hello from the server.'
...
[23:03:48] [Server thread/INFO]: [Server] Hello from the server.
```

Or `teleport` somebody:

```
root@darkstar:~# ratcraft cmd 'teleport Ratf -983 24 340'
```

The important thing to note in these examples is that the entire command needs to be quoted. Single-quotes are ideal unless you want to use variable interpolation, in which case you already know what you're doing.

Also note that after your command is sent, the script waits a second and then displays the last three lines from the server log.  Though this should show the server's reaction to your command, it may also be confusing.    This would be a great place for script improvement. Showing only _new_ lines written after the command is sent would be ideal.
