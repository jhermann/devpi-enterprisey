# "devpi" Debian Packaging

![Apache 2.0 licensed](http://img.shields.io/badge/license-Apache_2.0-red.svg)

| `devpi-server` | `devpi-client` | `devpi-common` | `devpi-web` | `devpi-findlinks` |
|:---:|:---:|:---:|:---:|:---:|
| ![devpi-server](http://img.shields.io/pypi/v/devpi-server.svg) | ![devpi-client](http://img.shields.io/pypi/v/devpi-client.svg) | ![devpi-common](http://img.shields.io/pypi/v/devpi-common.svg) | ![devpi-web](http://img.shields.io/pypi/v/devpi-web.svg) | ![devpi-findlinks](http://img.shields.io/pypi/v/devpi-findlinks.svg) |
| ![deb](http://img.shields.io/badge/deb-v2.0.6-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v2.0.2-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v2.0.2-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v2.1.0-d80854.svg) | ![deb](http://img.shields.io/badge/deb-v1.0.0-d80854.svg) |


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
sudo dpkg -i devpi_2*.deb
apt-cache show devpi
/usr/bin/devpi --version # ensure it basically works
```

The version of `devpi` and other core components used is determined in `debian/rules`.

☛ **Note that for a successful build, you need [this branch](https://github.com/jhermann/dh-virtualenv/tree/trigger-on-python-update) of `dh-virtualenv` right now.**


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
sudo -u devpi bash -c "cd /tmp && /usr/sbin/devpi-server --gen-config"
cp /tmp/gen-config/supervisor-devpi.conf /etc/supervisor/conf.d/devpi-server.conf
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
