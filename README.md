Factorio-multienv-ctl
=====================

Factorio-multienv-ctl is a fork of factorio-init, which allows you to have multiple environments
of factorio servers on the same machine. It also includes a Debian package
which will set up both the factorio server binaries and factorio-multienv-ctl.

[![Build Status](https://travis-ci.org/factoriommo/factorio-multienv-ctl.svg?branch=master)](https://travis-ci.org/factoriommo/factorio-multienv-ctl)

Quickstart
----------

- Install this package (see prepared Debian package section).
- Run `factorio new-game somenameforyourgame` or upload a
  existing savegame to `/var/factorio/instances/default/saves/`
- Run `factorio start` or `systemctl start factorio@default`
- Optionally: run on system boot: `systemctl enable factorio@default`
- To use another environment:
  - run `FENV=myenvname factorio new-game somenameforyourgame`
  - run `FENV=myenvname factorio start` or `systemctl start factorio@myenvname`


Prepared Debian package
-----------------------

You can find prebuilt packages for Debian 8 at http://repo.factoriommo.org/

These have been known to work on Ubuntu 16.04 and 16.10 as well, they might also work for other apt based distros running systemd as init.

```
# Add gpg pubkey
curl http://repo.factoriommo.org/apt.gpg | apt-key add -
# or if you prefer wget instead of curl (installed by default on Ubuntu)
wget -O - http://repo.factoriommo.org/apt.gpg | apt-key add -

# Add deb repo entry
echo "deb http://repo.factoriommo.org/ jessie main" > /etc/apt/sources.list.d/factoriommo.list

# Update apt lists
apt-get update

# Install package
apt-get install factorio-multienv-ctl
```


How does it work
----------------

Factorio-multienv-ctl uses the `FENV` environment variable. This defaults to `default`.
To explain what factorio-multienv-ctl does, here is an overview of what paths are used,
and what can be customized.


- Core game files are always installed to `/opt/factorio/factorio/`.

- The Debian package will create `/var/factorio/mods/`, in which you
  can store shared modpacks. More on this later.

- Every instance will have its write-data location set to 
  `/var/factorio/instance/{name}/`, where `{name}` defaults to `default`.
  This folder contains both `saves` and `mods` folders for this instance to use.
  The `server.out` and `server.pid` files will also be put here.
  Factorio-multienv-ctl automatically overwrites `config.ini` with the correct write-data entry.

- You can set the environment by setting `FENV` before using `factorio` command,
  which is the factorio-multienv-ctl script, not the actual factorio server binary.

- For auto-startup of factorio servers, you can use the parametrized systemd service.
  This allows you to enable/start a server like this: `systemctl start factorio@{name}`,
  where `{name}` is the name of the environment. There is no default, you will need to 
  manually run `systemctl start factorio@default` to enable the default server.


Configuration folder
--------------------

All configuration that factorio-multienv-ctl recognizes should live in `/etc/factorio`.
Factorio-multienv-ctl will load and try to inherit customizations in the correct order.
This however is not done for json files, those completely overwrite the defaults,
so ensure your customized server-settings.json contains all values.

Required configfiles:

- config.ini
- server-settings.json

Optionally you may create `map-gen-settings.json` and/or `init.conf` (see below), 
and their paramatric versions.

You can create overrides for specific instances by putting them in `/etc/factorio/$FENV/`. 
For instance `FENV=asdf` to override `server-settings.json` you create
`/etc/factorio/asdf/server-settings.json`.

Note that `/etc/factorio/asdf/init.conf` inherits from `/etc/factorio/init.conf` if it exists,
and the json files don't.

If you create `/etc/factorio/asdf/config.ini`, 
it will be copied to `/var/factorio/instances/asdf/config.ini`,
and the `write_dir` will be overwritten in the latter on every start.


Available values for init.conf
------------------------------

The following values all have sane defaults, but can be customized.
In theory you can override every variable used in the main script,
but it's recommended you don't.

```
EXTRA_BINARGS="--start-server-load-latest"  # you should use `factorio load-save` with this.
RCON_PORT=""
RCON_PASSWORD=""
SAVELOG=0
SERVER_BIND=""
SERVER_PORT=34197
MOD_DIRECTORY=""  # eg: /var/factorio/mods/ArumbaMegaModPack
USERGROUP=factorio
USERNAME=factorio
```

Debugging
---------

If you find yourself wondering why stuff is not working the way you expect:

 - Check the logs, you can use `factorio log` and `factorio log --tail`, also parametric: `FENV=asdf factorio log`.
 - Enable debugging in the config and/or:
 - Try running the same commands as the factorio user (`factorio invocation` will tell you what the factorio user tries to run at start)

```bash
$ factorio invocation
#  Run this as the factorio user, example:
$ sudo -u factorio 'whatever invocation gave you'
# You should see some output in your terminal here, hopefully giving
# you a hint of what is going wrong
```

Bash autocompletion
-------------------

Is included with the Debian package, and will activate if you 
install `bash-completion` package and enable system-wide completion.


Notable changes from factorio-init
----------------------------------

The `install` and `update` commands have been removed, 
dependency on updater python script is also removed.

Also, the config and environment handling has been overhauled
to accomodate for multiple environments.


A note on dpkg's configfile handling
------------------------------------

Because the Debian package also creates /etc/server-settings.json and other default files,
`dpkg` will track those and notify when a package update tries to overwrite customized versions.

You can move the default files out of the way to prevent this message during updates:

`dpkg-divert --add --rename --divert /etc/factorio/.old-server-settings.json /etc/factorio/server-settings.json`

You can copy .old-server-settings.json or create a customized server-settings.json afterwards.


Building the Debian package
---------------------------

Prerequisites: `devscripts dpkg-dev debhelper debianutils bash-completion`

Then run `debuild` in the folder containing the `debian` folder (the same one as this readme is in.)

Afterwards, you will find your .deb files one directory up.


Credits and License
-------------------

`factorio-multienv-ctl` is based on `factorio-init`, see https://github.com/Bisa/factorio-init for details.

This code is realeased under the MIT license, see the LICENSE file.
