devpi-enterprisey
=================

Augmentation of the awesome [devpi](http://doc.devpi.net/latest/) project
with ‘enterprisey’ requirements, configuration, and deployment.


## Debian Packaging

The Debian packaging metadata in
[debianized-devpi/debian](https://github.com/jhermann/devpi-enterprisey/tree/master/debianized-devpi/debian)
puts the `devpi` Python package and its dependencies into a DEB package,
using [dh-virtualenv](https://github.com/spotify/dh-virtualenv).
The resulting package is thus easily installed and removed from a machine, but contains no configuration,
and is not a ‘normal’ Debian `python-*` package. If you want that, look elsewhere.

The created virtualenv also contains any plugins configured in `debina/rules` — right now, that is `devpi-findlinks` by default.
And at the time of this writing, a development snapshot of `requests` is installed, which fixes a problem with HTTP proxy
servers in the contained `urllib3` vendor package.

**How to build and install the package?**

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
```

The version of `devpi` used is determined by the top-most entry and its upstream version in the `changelog`.


## Related Tickets

* [Integrate LDAP-based authentication/authorization](https://bitbucket.org/hpk42/devpi/issue/138/integrate-ldap-based-authentication)
* [Prevent "pre" uploading / pushing into an index](https://bitbucket.org/hpk42/devpi/issue/137/prevent-pre-uploading-pushing-into-an)
* [Enable inheriting from external index (other than pypi.python.org)](https://bitbucket.org/hpk42/devpi/issue/12/enable-inheriting-from-external-index)


## References

* [hpk42/devpi](https://bitbucket.org/hpk42/devpi)
* [devpi/devpi-findlinks](https://github.com/devpi/devpi-findlinks)
* [dev snapshot of ‘requests’](https://github.com/jhermann/requests) (`proxy-fixed-for-devpi` branch, some time after v3.2.0)
