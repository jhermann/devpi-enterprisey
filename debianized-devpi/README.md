# "devpi" Debian Packaging

## What is this?

The Debian packaging metadata in
[debianized-devpi/debian](https://github.com/jhermann/devpi-enterprisey/tree/master/debianized-devpi/debian)
puts the `devpi` Python package and its dependencies into a DEB package,
using [dh-virtualenv](https://github.com/spotify/dh-virtualenv).
The resulting package is thus easily installed to and removed from a machine, but contains no configuration,
and is not a ‘normal’ Debian `python-*` package. If you want that, look elsewhere.

The created virtualenv also contains any plugins configured in `debian/rules` — right now, that is `devpi-findlinks` by default.
And at the time of this writing, a development snapshot of `requests` is installed, which fixes a problem with HTTP proxy
servers in the contained `urllib3` vendor package.


## How to build and install the package?

You need of course a machine with the build dependencies installed, specifically
[dh-virtualenv](https://github.com/spotify/dh-virtualenv) in addition to the normal Debian packaging tools.
Then the following commands will install a *release* version of `devpi` into `/usr/share/python/devpi/`, and place symlinks
for `devpi` (in `bin`) and `devpi-server` (in `sbin`) into the machine's PATH.

```sh
git clone https://github.com/jhermann/devpi-enterprisey.git
cd devpi-enterprisey/
( cd debianized-devpi/ && dpkg-buildpackage -uc -us -b )
sudo dpkg -i devpi_2*.deb
apt-cache show devpi
devpi --version # ensure it basically works
```

The version of `devpi` used is determined by the top-most entry and its upstream version in the `changelog`.


## How to configure a simple "devpi" instance?

Once the package is installed, use these commands as `root` to configure and start your `devpi` server:

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
documentation, starting with “devpi install: installing a package”.
