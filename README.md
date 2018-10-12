
 `$Id$`

This document describes the steps required in order to prevent _Slack_
[packaged](https://slack.com/downloads/linux)
for _Debian Linux_ from silently auto-updating itself.

It is assumed that all commands are run as `root`.

1. Create the global configuration file:
   ```bash
   cat <<EOF >/etc/default/slack
   repo_add_once="false"
   repo_reenable_on_distupgrade="false"
   EOF

   # Forbid future changes to the file:
   chattr +i /etc/default/slack
   ```
1. Add the necessary diversions (will affect package files once those are installed):
   ```bash
   mkdir -p /etc/apt/sources.list.d.disabled
   dpkg-divert --local --rename --divert /etc/apt/sources.list.d.disabled/slack.list /etc/apt/sources.list.d/slack.list

   mkdir -p /etc/cron.daily.disabled
   dpkg-divert --local --rename --divert /etc/cron.daily.disabled/slack /etc/cron.daily/slack
   ```
   There's an alternative method to prevent packages from installing new sources
   under  `/etc/apt/sources.list.d`: one needs to add a file to
   `/etc/dpkg/dpkg.cfg.d/` (named, let's say, `no-new-sources`), with the
   following contents:
   ```
   #
   # /etc/dpkg/dpkg.cfg.d/no-new-sources
   #
   # vim:ft=conf:
   #
   # Prevent packages from installing new sources under /etc/apt/sources.list.d
   #

   path-exclude=/etc/apt/sources.list.d/*
   ```
1. Install the package as usual:
   ```bash
   dpkg -i slack-desktop-3.3.3-amd64.deb
   ```
1. _Apt_ may still have _Slack_'s GPG keys present in its trusted database
   (`/etc/apt/trusted.gpg`):
   ```
   $ apt-key list
   ...
   pub   rsa4096 2014-01-13 [SCEA] [expires: 2019-01-12]
         418A 7F2F B0E1 E6E7 EABF  6FE8 C2E7 3424 D590 97AB
   uid           [ unknown] packagecloud ops (production key) <ops@packagecloud.io>
   sub   rsa4096 2014-01-13 [SEA] [expires: 2019-01-12]

   pub   rsa4096 2016-02-18 [SCEA]
         DB08 5A08 CA13 B8AC B917  E0F6 D938 EC0D 0386 51BD
   uid           [ unknown] https://packagecloud.io/slacktechnologies/slack (https://packagecloud.io/docs#gpg_signing) <support@packagecloud.io>
   sub   rsa4096 2016-02-18 [SEA]
   ...
   ```
   If this is the case, delete the keys:
   ```bash
   apt-key del D59097AB
   apt-key del 038651BD
   ```
1. That's it.
