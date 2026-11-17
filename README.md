# hopper

Change which Claude account a machine's running sessions use, without restarting them.

A session keeps its conversation when the machine's sign-in changes under it. So each account's sign-in is kept
in its own folder while it is not in use, and a switch MOVES sign-ins: the one in use goes back into its folder,
the one wanted goes into `~/.claude`. A sign-in is never in two places (two copies would each renew it and sign
the other out). No content of a sign-in file is ever printed or read by this tool; it only moves the file.

```
hopper adopt <name>   say which account is in use now (once, at setup)
hopper add <name>     make the folder of another account, and print its one-time browser sign-in command
hopper list           the accounts here: in use / ready / needs its sign-in
hopper to <name>      switch; checks the account really changed, puts everything back if not
hopper auto           switch to the first account your pick command names that is usable here
hopper timer on|off   run « auto » every minute by itself (a systemd user timer)
hopper prefer <name>|none     try this account first, for as long as the pick command still lists it
```

`auto` knows nothing about usage: it runs the one command line in `~/.config/hopper/pick`, which prints
account names, best first, and takes the first one usable on this machine (the one in use, or one whose
sign-in is here). Put any rule you like behind that command, and run `auto` from a timer if you want the
switch to happen by itself.

Several machines on the same account: ONE machine decides and announces — it has a second line of
configuration, `~/.config/hopper/announce`, a command that receives the chosen name as its last
argument (run only when the name changes) — and the others' pick command reads what was announced. Each
machine still switches itself, and only to an account that was signed in on it.

## A machine that follows another

```
hopper adopt <name-in-use>            # which account this machine is on now
hopper add <other>                   # for each other account: then the browser sign-in it prints, once
echo '<command that prints the announced name>' > ~/.config/hopper/pick
hopper timer on
```

The names must be the same on every machine (they are yours: `work`, `home`, …).

After a switch, the one command line in `~/.config/hopper/switched` is run with the old and the new name
(for instance to tell running sessions to take up what a spent account interrupted).

## Install

```
git clone https://github.com/skhattane/hopper.git ~/Projects/hopper
ln -sfn ~/Projects/hopper/hopper ~/.local/bin/hopper     # or any folder on your PATH
```

`hopper timer on` records the `PATH` of the shell it is run from: run it from a shell where `claude` is found.

## Linux and macOS

On Linux a sign-in is a file (`~/.claude/.credentials.json` in use, `<folder>/.credentials.json` kept). On macOS it is
an item of the login keychain: « Claude Code-credentials » in use, and a sign-in made with `CLAUDE_CONFIG_DIR=<folder>`
gets its own item, « Claude Code-credentials-<first 8 hex of sha256(folder)> ». `hopper` moves the one or the other;
the commands are the same. On a Mac the first switch may show the system's keychain prompt once (« Always Allow »),
and `hopper timer on` installs a launchd agent (`~/Library/LaunchAgents/io.hopper.auto.plist`, log in
`~/Library/Logs/hopper.log`) instead of a systemd timer.
