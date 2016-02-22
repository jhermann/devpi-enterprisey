# "devpi" Debian Packaging

![Apache 2.0 licensed](http://img.shields.io/badge/license-Apache_2.0-red.svg)

| `devpi-server` | `devpi-client` | `devpi-common` | `devpi-web` | `devpi-findlinks` | `requests` |
|:---:|:---:|:---:|:---:|:---:|:---:|
| [![devpi-server](http://img.shields.io/pypi/v/devpi-server.svg)](https://pypi.python.org/pypi/devpi-server/) | [![devpi-client](http://img.shields.io/pypi/v/devpi-client.svg)](https://pypi.python.org/pypi/devpi-client/) | [![devpi-common](http://img.shields.io/pypi/v/devpi-common.svg)](https://pypi.python.org/pypi/devpi-common/) | [![devpi-web](http://img.shields.io/pypi/v/devpi-web.svg)](https://pypi.python.org/pypi/devpi-web/) | [![devpi-findlinks](http://img.shields.io/pypi/v/devpi-findlinks.svg)](https://pypi.python.org/pypi/devpi-findlinks/) | [![requests](http://img.shields.io/pypi/v/requests.svg)](https://pypi.python.org/pypi/requests/) |
| ![deb](http://img.shields.io/badge/deb-v3.0.1-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v2.5.0-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v2.0.8-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v3.0.0-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v1.0.1-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v2.9.1-d80854.svg) |


## What is this?

The Debian packaging metadata in
[debianized-devpi/debian](https://github.com/jhermann/devpi-enterprisey/tree/master/debianized-devpi/debian)
puts the `devpi` Python package and its dependencies into a DEB package,
using [dh-virtualenv](https://github.com/spotify/dh-virtualenv).
The resulting package is thus easily installed to and removed from a machine, but contains no configuration,
and is not a ‘normal’ Debian `python-*` package. If you want that, look elsewhere.

The created virtualenv also contains any plugins configured in `debian/rules`
— right now, that is `devpi-web` and `devpi-findlinks` by default.


## How to build and install the package?

You need of course a machine with the build dependencies installed, specifically
[dh-virtualenv](https://github.com/spotify/dh-virtualenv) in addition to the normal Debian packaging tools.
You can get it from [this PPA](https://launchpad.net/~dh-virtualenv/+archive/ubuntu/stable) or [Debian packages](https://packages.debian.org/source/experimental/dh-virtualenv).

Then the following commands will install a *release* version of `devpi` into `/usr/share/python/devpi/`, and place symlinks
for `devpi` (in `bin`) and `devpi-server` (in `sbin`) into the machine's PATH.

```sh
git clone https://github.com/jhermann/devpi-enterprisey.git
cd devpi-enterprisey/
( cd debianized-devpi/ && dpkg-buildpackage -uc -us -b )
sudo dpkg -i devpi_3*.deb
apt-cache show devpi
/usr/bin/devpi --version # ensure it basically works
```

The version of `devpi` and other core components used is determined in `debian/rules`.


## How to configure a simple "devpi" instance?

To get a running `devpi-server` instance, you need to install the package and then add the necessary configuration.
This can be done automatically using the [devpi-puppet](https://github.com/jhermann/devpi-puppet) module (see there for details), which also gives you instant theming and NginX proxying to an external port.
Otherwise, use the following instructions to do so
– but be aware that these are only appropriate for workstation installations of a single developer.

So once the package is installed as shown in the previous section,
use these commands as `root` to configure and start your `devpi` server:

```sh
apt-get install supervisor
addgroup devpi
adduser devpi --ingroup devpi --home /var/lib/devpi --system --disabled-password
sudo -l -u devpi bash -c "cd /tmp && /usr/sbin/devpi-server --gen-config"
cp /tmp/gen-config/supervisor-devpi.conf /etc/supervisor/conf.d/devpi-server.conf
echo >>/etc/supervisor/conf.d/devpi-server.conf "directory = /var/lib/devpi"
supervisorctl update
supervisorctl tail -f devpi-server
```

Then, in a 2nd non-root shell:

```sh
devpi use "http://localhost:3141/"
devpi login root --password=
  devpi user -m root password=…
devpi user -c local # … and enter password
devpi login local # … and enter password
devpi index -c dev
devpi use dev --set-cfg # be aware this changes 'index_url' of several configs in your $HOME
```

Finally, you can open the [web interface](http://localhost:3141/) and browse your shiny new local repositories.
For further details, consult devpi's
[Release Process Quickstart](http://doc.devpi.net/latest/quickstart-releaseprocess.html)
documentation, starting with *“devpi install: installing a package.”*


## How to migrate to a new version?

Consult the [release announcements](https://groups.google.com/forum/#!searchin/devpi-dev/releases|sort:date)
whether you actually need to migrate your data or not.
It is however safest to always do so.
You'll find the
[basic migration procedure](http://doc.devpi.net/latest/quickstart-server.html#versioning-exporting-and-importing-server-state)
in the official docs.

:exclamation: | Also consult the [devpi administration manual](http://doc.devpi.net/3.0/adminman/)!
----: | :----

The following is a condensed sequence of commands
you need when using a [devpi-puppet](https://github.com/jhermann/devpi-puppet) setup,
call them in a `root` shell. You should build the new Debian package before that and
have it ready for upgrading, e.g. uploaded to Artifactory.

```sh
now="$(date +'%Y-%m-%d-%H%M')"

# Export your data
supervisorctl stop devpi-server
devpi-server --export "$HOME/devpi-export-$now"
mv /var/lib/devpi/data /var/lib/devpi/_data-backup-$now

# Update (use 'dpkg -i' if you have no local Debian repository)
apt-get update
apt-get install devpi

# Restore data and start new version
devpi-server --serverdir /var/lib/devpi/data --import "$HOME/devpi-export-$now"
chown -R devpi.devpi /var/lib/devpi/data

# Start server
supervisorctl start devpi-server; supervisorctl tail -f devpi-server
```

In a setup with a master and replicas, follow this procedure:

* First stop the master and its Nginx server.
* Unless you have an automatic fail-over from master to replica, you also have to switch over to the replica manually.
* Upgrade the master without the final start.
* Stop all the replicas, and start both the master web and devpi server (and switch back to it). This minimizes the downtime window, and you can always go back to running the replica if anything goes wrong.
* Once the new master holds up, finish the upgrade procedure for the replicas.
