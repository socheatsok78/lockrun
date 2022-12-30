Mirror of http://unixwiz.net/tools/lockrun.html

# lockrun

lockrun - Run cron job with overrun protection

When doing network monitoring, it's common to run a cron job every five minutes (the standard interval) to roam around the network gathering data. Smaller installations may have no trouble running within this limit, but in larger networks or those where devices are often unreachable, running past the five-minute mark could happen frequently.

The effect of running over depends on the nature of the monitoring application: it could be of no consequence or it could be catastrophic. What's in common is that running two jobs at once (the old one which ran over, plus the new one) slows down the new one, increasing the risk that it will run long as well.

This is commonly a cascading failure which can take many polling sessions to right itself, which may include lost data in the interim.

Our response has been to create this tool, lockrun, which serves as a protective wrapper. Before launching the given command, it insures that another instance of the same command is not already running.

## Building from source

```sh
$ gcc lockrun.c -o lockrun
$ sudo cp lockrun /usr/local/bin/
```

Now we'll find lockrun in the usual place: /usr/local/bin/.

We'll note that though portable, this program is nevertheless designed only to run on UNIX or Linux systems: it certainly won't build and run properly on a Windows computer.

Furthermore, file locking has always been one of the more problemtic areas of portability, there being several mechanisms in place. lockrun uses the flock() system call, and this of course requires low-level OS support.

We've tested this in FreeBSD and Linux, but other operating systems might trip over compilation issues. We welcome portability reports (for good or bad).

We've also received a report that this works on Mac's OSX.

## Example Usage

Once lockrun has been built and installed, it's time to put it to work. This is virtually always used in a crontab entry, and the command line should include the name of the lockfile to use as well as the command to run.

This entry in a crontab file runs the Cacti poller script every five minutes, protected by a lockfile:

```
# crontab entry
*/5 * * * * /usr/local/bin/lockrun --lockfile=/var/run/cacti.lockrun -- /usr/local/bin/cron-cacti-poller
```

## Command Line Options

lockrun supports GNU-style command-line options, and this includes using -- to mark their end:

```sh
$ lockrun options -- command
```

The actual command after `--` can have any arguments it likes, and they are entirely uninterpreted by lockrun.

We'll note that command-line redirection (~>/dev/null~, etc.) is not supported by this or the command which follows — it's handled by the calling shell. This is the case whether it's run from cron or not.

- `--lockfile=F` Specify the name of a file which is used for locking. This filename is created if necessary (with mode 0666), and no I/O of any kind is done. This file is never removed.
- `--maxtime=N` The script being controlled ought to run for no more than <N> seconds, and if it's beyond that time, we should report it to the standard error stream (which probably gets routed to the user via cron's email).
- `--wait` When a pre-existing lock is found, this program normally exits with error, but adding the --wait parameter causes it to loop, waiting for the prior lock to be released.
- `--verbose` Show a bit more runtime debugging.
- `--quiet` If the previous run is locked, just exit quietly (and with success) rather than exit with failure after an error message. This may be useful when using lockrun in an environment where overlap is not uncommon.
- `--` Mark the end of the options - the actual command to run follows.

## History

- 2012/09/05 — added documentation about why there's no kill option; added --retries option (thanks to Dov Murik); added setsid() for session support
- 2010/11/29 — added --quiet parameter
- 2009/06/25 — added lockf() support for Solaris 10 (thanks to Michal Bella)
- 2006/06/03 — initial release

---
First published: 2006/06/03 ([Blogged](https://blog.unixwiz.net/2006/06/new_tool_lockru.html))

