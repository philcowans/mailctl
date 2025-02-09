# mailctl

[![Build](https://github.com/pdobsan/mailctl/actions/workflows/build.yaml/badge.svg)](https://github.com/pdobsan/mailctl/actions/workflows/build.yaml)

`mailctl` provides IMAP/SMTP clients with the capabilities of renewal and
authorization of OAuth2 credentials.

Many IMAP/SMTP clients, like [msmpt](https://marlam.de/msmtp/),
[fdm](https://github.com/nicm/fdm),
[isync](http://isync.sourceforge.net/),
[neomutt](https://github.com/neomutt/neomutt) or
[mutt](http://www.mutt.org/) can use OAuth2 access tokens but lack the
ability to renew and/or authorize OAuth2 credentials. The purpose of
`mailctl` is to provide these missing capabilities by acting as a kind of
smart password manager. In particular, access token renewal happens
automatically in the background transparent to the user.

If an IMAP/SMTP client cannot even use an OAuth2 access token itself it
still may be "wrapped" with `fdm` and `msmtp` with the help of `mailctl`.

`mailctl` also has some functionalities to manage the orchestration of
a mailing system comprised of `msmtp`, `fdm`, and `(neo)mutt` or similar
ones.

If you are using `fdm` with no complex desktop system you might also be
interested in my [mailnotify](https://github.com/pdobsan/mailnotify) program
which generates pop-up notifications about incoming email messages.

`mailctl` has been successfully used with Google/Gmail, operation with other
Oauth2 service providers are planned.

## Usage

Before `mailctl` is fully operational you need to create the necessary
configuration files. See details in [Configuration](#configuration). Make
sure that all the directories referenced in the configuration exist.

After configuration the very first command you must run is `authorize`. That
is an interactive process involving a browser since during it you need to
login and authorize access to your email account. `mailctl` will lead you
through this process.

`mailctl -h` generates this help message:

    mailctl - Provide OAuth2 renewal and authorization capabilities.

    Usage: mailctl [--version] [-c|--config-file <config>] [--run-by-cron] [--debug]
                   COMMAND

      Mailctl provides IMAP/SMTP clients with the capabilities of renewal and
      authorization of OAuth2 credentials.

    Available options:
      -h,--help                Show this help text
      --version                Show version
      -c,--config-file <config>
                               Configuration file
      --run-by-cron            mailctl invoked by cron
      --debug                  Print HTTP traffic to stdout

    Available commands:
      password                 Get the password for email
      access                   Get the access token for email
      renew                    Renew the access token of email
      authorize                Authorize OAuth2 for service/email
      fetch                    Get fdm to fetch all or the given accounts
      cron                     Manage running by cron
      list                     List all accounts in fdm's config
      printenv                 Print the current Environment

If no configuration file is specified the default is
`$XDG_CONFIG_HOME/mailctl/config.yaml`.

More detailed help for individual commands can also be generated, for
example:

    % mailctl authorize -h

    Usage: mailctl authorize <service> <email>

      Authorize OAuth2 for service/email

    Available options:
      <service>                Service name
      <email>                  Email address
      -h,--help                Show this help text

Shell completion for `bash`, `zsh` and `fish` shells are provided.

### Authorization for Organizational/Institutional accounts

The authorization process of Organizational/Institutional accounts can
slightly be different from of individual accounts and that might cause the
authorization to fail. That is definitively the case for Google.

When you invoke `mailctl` to authorize an account like this below:

    mailctl authorize <service> <email>

the program takes you to a login page at the given `<service>` and derived
from the `<email>` address. In the case of `username@company.com` that could
be a different login page from the "normal" login page. The solution is to
invoke authorization like:

    mailctl authorize <service> <company>

where `<company>` is just a name not a complete email address. Then simply pick
the right account on Google's login page. Since now the only usage of
`<company>` here is for the name of the `.auth` file `mailctl` will create you
just need to pick something that makes sense and doesn't interferes with your
other accounts.

## Configuration

The various functionalities of `mailctl` are quite loosely coupled. Choose,
that is configure, only the ones you need. For details on your options, see
`configs/config-template.yaml`.

The configuration files for `mailctl` are in YAML. The files in the
`configs/` directory are templates for `mailctl`, `msmtp` and `fdm`. The
`configs/services-template.yaml` file contains details for `google`,
`microsoft` and `yahoo`.

You need to provide your own `client_id` and `client_secret` of your
application or of a suitable FOSS registered application.

Service providers varying how they communicate at their authorization and token
endpoints. The accepted HTTP methods and how the request's parameters delivered
must be configured in the `services.yaml` file for both endpoints. Options for
`*_http_method` are `POST` and `GET`. Options for `*_params_mode` are
`query-string`, `request-body` (JSON encoded), `request-body-form`
(x-www-form-urlencoded) or `both`. When `both` is configured `mailctl` uses the
`request-body` method since it is considered safer.

If you encounter difficulties during *authorization* or *renewal* try to use
the `--debug` switch which causes `mailctl` to mirror print HTTP traffic to
`stdout`.

The Oauth2 credentials are kept encrypted using [GNU PG](https://www.gnupg.org/).
so it is assumed that an authorized `gpg-agent` is running. Alternatives
like `gnome-keyring` might be used.

`fdm`, if used, can be run by `cron` with a `crontab` like this:

    PATH  = /YOUR_HOME_DIRECTORY/.cabal/bin:/usr/bin:/bin
    # fetch mail in every 10 minutes
    */10 * * * *  mailctl --run-by-cron fetch


## Installation

### Compiled static binaries

Each release contains a compiled static executable of `mailctl` which should
work on most Linux distributions. You can download the latest one from
[here](https://github.com/pdobsan/mailctl/releases/latest). Check the sha256
check sum of the downloaded file. Rename it to `mailctl`, made it executable
and put somewhere in your search path.

#### Archlinux

For Archlinux users there is also a package on AUR:
[mailctl-bin](https://aur.archlinux.org/packages/mailctl-bin)

### Building from sources

To build `mailctl` from source you need a Haskell development environment,
with `ghc 9.2.4` or higher. Either your platform package system can provide
this or you can use [ghcup](https://www.haskell.org/ghcup/). Once you have
the `ghc` Haskell compiler and `cabal` etc. installed, follow the steps
below:

Clone this repository and invoke `cabal`:

    git clone https://github.com/pdobsan/mailctl.git
    cd mailctl
    cabal update
    cabal install --install-method=copy

That installs `mailctl` into the `~/.cabal/bin/` directory.

## Logging

All transactions and exceptions are logged to `syslog`. If your OS using
`systemd` you can inspect the log with a command like below:

    journalctl --identifier fdm --identifier mailctl --identifier msmtp -e


## Acknowledgment 

The method of dealing with OAuth2 services was inspired by the
[mutt_oauth2.py](https://gitlab.com/muttmua/mutt/-/blob/master/contrib/mutt_oauth2.py)
Python script written by Alexander Perlis.


## License

`mailctl` is released under the 3-Clause BSD License, see the file
[LICENSE](LICENSE).

