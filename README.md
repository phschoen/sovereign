# Sovereign
# Introduction

Sovereign is a set of [Ansible](http://ansible.com) playbooks that you can use to build and maintain your own [personal cloud](http://www.urbandictionary.com/define.php?term=clown%20computing) based entirely on open source software, so you’re in control.

If you’ve never used Ansible before, you might find these playbooks useful to learn from, since they show off a fair bit of what the tool can do.

The original author's [background and motivations](https://github.com/sovereign/sovereign/wiki/Background-and-Motivations) might be of interest.
tl;dr: frustrations with Google Apps and concerns about privacy and long-term support.

Sovereign offers useful cloud services while being reasonably secure and low-maintenance.
Use it to set up your server, SSH in every couple weeks, but mostly forget about it.

## Services Provided

What do you get if you point Sovereign at a server? All kinds of good stuff!

-   [IMAP](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol) over SSL via [Dovecot](http://dovecot.org/), complete with full text search provided by [Solr](https://lucene.apache.org/solr/).
-   [POP3](https://en.wikipedia.org/wiki/Post_Office_Protocol) over SSL, also via Dovecot
-   [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) over SSL via Postfix, including a nice set of [DNSBLs](https://en.wikipedia.org/wiki/DNSBL) to discard spam before it ever hits your filters.
-   Virtual domains for your email, backed by [PostgreSQL](http://www.postgresql.org/).
-   Spam fighting via [Rspamd](https://www.rspamd.com/).
-   Mail server verification using [DKIM](http://www.dkim.org/) and [DMARC](http://www.dmarc.org/) so the Internet knows your mailserver is legit.
-   Webmail via [Roundcube](http://www.roundcube.net/).
-   Mobile push notifications and autodiscovery via [Z-Push](http://z-push.sourceforge.net/soswp/index.php?pages_id=1&t=home).
-   Email client [automatic configuration](https://developer.mozilla.org/en-US/docs/Mozilla/Thunderbird/Autoconfiguration).
-   Jabber/[XMPP](http://xmpp.org/) instant messaging via [Prosody](http://prosody.im/).
-   [Matrix](https://matrix.org/) via [Riot.im](https://about.riot.im) and [Synapse](https://matrix.org/docs/projects/server/synapse.html).
-   The [Mastodon](https://mastodon.social/about) social network.
-   An RSS Reader via [Selfoss](http://selfoss.aditu.de/).
-   [CalDAV](https://en.wikipedia.org/wiki/CalDAV) and [CardDAV](https://en.wikipedia.org/wiki/CardDAV) to keep your calendars and contacts in sync, via [NextCloud](http://nextcloud.com/).
-   Your own VPN server via [OpenVPN](http://openvpn.net/index.php/open-source.html).
-   An IRC bouncer via [ZNC](http://wiki.znc.in/ZNC).
-   Git Repo hosting via [gitea](https://gitea.io/en-us/).
-   IoT Dashboard via [Grafana](https://grafana.com) with [InfluxDB](https://www.influxdata.com/time-series-platform/influxdb/) and [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/).
-   [Mosquitto](https://mosquitto.org) and [mqtt-admin](https://github.com/hobbyquaker/mqtt-admin) on `iot.domain/mqtt`.
-   [Monit](http://mmonit.com/monit/) to keep everything running smoothly (and alert you when it’s not).
-   Web hosting (ex: for your blog) via [Apache](https://www.apache.org/).
-   Statistics for the website using [Fathom](https://github.com/usefathom/fathom).
-   Comments for the website using [Commento](https://gitlab.com/commento/commento).
-   Firewall management via [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall).
-   Intrusion prevention via [fail2ban](http://www.fail2ban.org/) and rootkit detection via [rkhunter](http://rkhunter.sourceforge.net).
-   SSH configuration preventing root login and insecure password authentication
-   A bunch of nice-to-have tools like [mosh](http://mosh.mit.edu) and [htop](http://htop.sourceforge.net) that make life with a server a little easier.

Don’t want one or more of the above services? Comment out the relevant role in `site.yml`.
Or get more granular and comment out the associated `include:` directive in one of the playbooks.

# Usage

## What You’ll Need

1.  A VPS (or bare-metal server if you wanna ball hard). My VPS is hosted at [Linode](http://www.linode.com/?r=45405878277aa04ee1f1d21394285da6b43f963b). You’ll probably want at least 512 MB of RAM between Apache, Solr, and PostgreSQL. Mine has 1024.
2.  [64-bit Debian 9](http://www.debian.org/). (You can use whatever distro you want, but deviating from Debian will require more tweaks to the playbooks. See Ansible’s different [packaging](http://docs.ansible.com/ansible/list_of_packaging_modules.html) modules.)

You do not need to acquire an SSL certificate.  The SSL certificates you need will be obtained from [Let's Encrypt](https://letsencrypt.org/) automatically when you deploy your server.

## Installation

### On the remote server

The following steps are done on the remote server by `ssh`ing into it and running these commands.

#### 1. Install required packages

    apt-get install sudo python

#### 2. Prep the server

For goodness sake, change the root password:

    passwd

Create a user account for Ansible to do its thing through:

    useradd deploy
    passwd deploy
    mkdir /home/deploy

Authorize your ssh key if you want passwordless ssh login (optional):

    mkdir /home/deploy/.ssh
    chmod 700 /home/deploy/.ssh
    nano /home/deploy/.ssh/authorized_keys
    chmod 400 /home/deploy/.ssh/authorized_keys
    chown deploy:deploy /home/deploy -R

Or, in short:

    ssh-copy-id -i ~/.ssh/id_ecdsa deploy@hostname

Also, enable passwordless sudo for the deploy user:

    echo 'deploy ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/deploy

Your new account will be automatically set up for passwordless `sudo`.
Or you can just add your `deploy` user to the sudo group.

    adduser deploy sudo

### On your local machine

Ansible (the tool setting up your server) runs locally on your computer and sends commands to the remote server.

#### 3. Software

Download this repository somewhere on your machine, either through `Clone or Download > Download ZIP` above, `wget`, or `git` as below.
Also install the dependencies for password generation as well as ansible itself.
    
    git clone https://github.com/xythobuz/sovereign.git
    cd sovereign
    sudo pip install -r ./requirements.txt

Or, if you're on Arch, instead of using pip, install the required stuff manually:

    sudo pacman -Syu ansible python-jmespath python-passlib

#### 4. Configure your installation

Modify the settings in the `group_vars/sovereign` folder to your liking.
If you want to see how they’re used in context, just search for the corresponding string.
All of the variables in `group_vars/sovereign` must be set for sovereign to function.

Finally, replace the `host.example.net` in the file `hosts`.
If your SSH daemon listens on a non-standard port, add a colon and the port number after the IP address.
In that case you also need to add your custom port to the task `Set firewall rules for web traffic and SSH` in the file `roles/common/tasks/ufw.yml`.

#### 5. Set up DNS

If you’ve just bought a new domain name, point it at [Linode’s DNS Manager](https://library.linode.com/dns-manager) or similar.
Most VPS services (and even some domain registrars) offer a managed DNS service that you can use for this at no charge.
If you’re using an existing domain that’s already managed elsewhere, you can probably just modify a few records.

Create `A` or `CNAME` records which point to your server's IP address:

* `example.com`
* `mail.example.com`
* `www.example.com` (for Web hosting)
* `autoconfig.example.com` (for email client automatic configuration)
* `stats.example.com` (for web stats)
* `news.example.com` (for Selfoss)
* `cloud.example.com` (for NextCloud)
* `git.example.com` (for gitea)
* `status.example.com` (for monit)
* `matrix.example.com` (for riot)
* `social.example.com` (for mastodon)
* `comments.example.com` (for commento)
* `iot.example.com` (for grafana)

#### 6. Run the Ansible Playbooks

First, make sure you’ve [got Ansible installed](http://docs.ansible.com/intro_installation.html#getting-ansible).
This should already be done by running the pip requirements.txt from above.

To run the whole dang thing:

    ansible-playbook -i ./hosts --ask-sudo-pass site.yml
    
If you chose to make a passwordless sudo deploy user, you can omit the `--ask-sudo-pass` argument.

To run just one or more piece, use tags.
I try to tag all my includes for easy isolated development.
For example, to focus in on your firewall setup:

    ansible-playbook -i ./hosts --tags=ufw site.yml

You might find that it fails at one point or another.
This is probably because something needs to be done manually, usually because there’s no good way of automating it,
or because something changed in the upstream packages or you're not using Debian 9.
Fortunately, all the tasks are clearly named so you should be able to find out where it stopped.
I’ve tried to add comments where manual intervention is necessary.
In the best case scenario, no manual steps should be needed, everything is done via the sovereign config vars.

The `dependencies` tag just installs dependencies, performing no other operations.
The tasks associated with the `dependencies` tag do not rely on the user-provided settings that live in `group_vars/sovereign`.
Running the playbook with the `dependencies` tag is particularly convenient for working with Docker images.

#### 7. Finish DNS set-up

Create an `MX` record for `example.com` which assigns `mail.example.com` as the domain’s mail server.
To ensure your emails pass DKIM checks you need to add a `txt` record.
The name field will be `mail._domainkey.EXAMPLE.COM.`
The value field contains the public key used by DKIM.
The exact value needed can be found in the file `/var/lib/rspamd/dkim/EXAMPLE.COM.mail.txt`.
For DMARC you'll also need to add a `txt` record.
The name field should be `_dmarc.EXAMPLE.COM` and the value should be `v=DMARC1; p=reject`.
We will also add a `txt` record for SPF. This is now legacy, but some providers need it, so we provide an empty policy.

For my DNS provider, that zonefile looks like this:

    @               IN MX 10 mail
    @               IN TXT   "v=spf1 a:mail.example.com ?all"
    _dmarc          IN TXT   "v=DMARC1; p=reject;"
    mail._domainkey IN TXT   "v=DKIM1; k=rsa; p=INSERT_PUBLIC_KEY_HERE"

Correctly set up reverse DNS for your server and make sure to validate that it’s all working,
for example by sending an email to <a href="mailto:check-auth@verifier.port25.com">check-auth@verifier.port25.com</a>
and reviewing the report that will be emailed back to you.

#### 8. Miscellaneous Configuration

Sign in to the ZNC web interface and set things up to your liking.
It isn’t exposed through the firewall, so you must first set up an SSH tunnel:

	ssh deploy@example.com -L 6643:localhost:6643

Then proceed to http://localhost:6643 in your web browser.
The same goes for the RSpamD web interface on port 11334.
