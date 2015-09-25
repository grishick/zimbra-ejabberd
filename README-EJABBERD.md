Overview
--------

These instructions are part of the set of instructions for setting up
ZCS Chat feature: <http://wiki.eng.zimbra.com/index.php/ZimbraIM/>\
These instructions were tested with versions 14.07, 15.04 and 15.06.

Download and install ejabberd
-----------------------------

You can install ejabberd either on the same machine as ZCS or on a
separate machine. Note: DO NOT use apt-get to install ejabberd, because
current version of ejabberd available on Ubuntu 12 is very old. Download
ejabberd installer from this site:
<https://www.process-one.net/en/ejabberd/downloads> Follow the
installation steps. Accept default values. Do not set up a cluster
unless you know how.

### Install Perl packages

The following Perl packages are required by ejabberd/ZCS authentication
script:

-   IO::Socket:SSL
-   YAML
-   XML::Parser
-   LWP::UserAgent
-   Log::Log4perl
-   Net::SSLeay
-   LWP::Protocol::https

you can install these using cpan:

    sudo apt-get update
    sudo apt-get install openssl,libssl-dev
    sudo cpan install IO::Socket
    sudo cpan install YAML
    sudo cpan install XML::Parser
    sudo cpan install LWP::UserAgent
    sudo cpan install Log::Log4perl
    sudo cpan install Net::SSLeay
    sudo cpan install LWP::Protocol::https

### Set up external authentication for ejabberd

1.  copy all files from ZimbraIMExtension/src/perl to
    {ejabberd-install-folder}/bin/zimbra
          mkdr -p ejabberd-15.04/bin/zimbra/
          cp src/perl/* ejabberd-15.04/bin/zimbra/

2.  Edit ejabberd config file
          vi ejabberd-15.04/conf/ejabebrd.yml

    Comment out this line:

          auth_method: internal

    uncomment this line

          auth_method: external

    add this line bellow “auth\_method: external"

          extauth_program: "ejabberd-15.04/bin/zimbra/ejabberd_zimbraauth.pl"

3.  save ejabebrd.yml
4.  Make sure ejabberd\_zimbraauth.pl is executable:
         chmod +x ejabberd-15.04/bin/zimbra/ejabberd_zimbraauth.pl

### Configure Ejabberd domain name

Edit Ejabberd configuration file (<b>ejabberd.yml</b>) and find the
section that looks like this:

    ###   ================
    ###   SERVED HOSTNAMES

    ##
    ## hosts: Domains served by ejabberd.
    ## You can define one or several, for example:
    ## hosts:
    ##   - "example.net"
    ##   - "example.com"
    ##   - "example.org"
    ##
    hosts:
      - "localhost"

Change “localhost” to your ZCS domain name, e.g.:

    hosts:
      - "ubuntu.local"
### Edit ejabberd\_zimbraauth.pl

You need to edit the authentication script to match your ZCS
configuration.

1.  Change 'admin' to the username of a Zimbra admin account with
    sufficient rights to call GetAccountInfoRequest on any user.
        my $ZCS_ADMIN = "admin";

2.  Change “test123” to the ZCS admin's password
        my $ZCS_ADMIN_PW = "test123";

3.  Change to host:port where Zimbra SOAP interface is running. If
    Zimbra Proxy is using HTTP or HTTPS mode with default configuration,
    a port number is not needed. In development environment without ZCS
    proxy, this value would be localhost:7070
        my $ZCS_HOST_PORT="localhost";

4.  Change to host:port where Zimbra Admin SOAP interface is running.
    Unless you have modified Zimbra Proxy and mailboxd configuration,
    Admin SOAP interface should be running on port 7071.
        my $ZCS_ADMIN_HOST_PORT="localhost:7071";

5.  If Zimbra Proxy is running in HTTPS or mixed mode, change
    <b>\$ZCS\_PROTO</b> to “https” on the following line:
        my $ZCS_PROTO="http";

### Start ejabberd

    ejabebrd-15.04/bin/ejabberdctl start

Multiple Domains
----------------

Ejabberd supports multiple domains via ejabberd.yml configuration file.
Every time you add a domain, you have to restart Ejabberd.

### Adding domains

Edit Ejabberd configuration file (<b>ejabberd.yml</b>) and find the
section that looks like this:

    ###   ================
    ###   SERVED HOSTNAMES

    ##
    ## hosts: Domains served by ejabberd.
    ## You can define one or several, for example:
    ## hosts:
    ##   - "example.net"
    ##   - "example.com"
    ##   - "example.org"
    ##
    hosts:
      - "localhost"

Enter all domain names served by the same ZCS installation, one domain
name per line. E.g.:

    hosts:
      - "ubuntu1.local"
      - "ubuntu2.local"
      - "ubuntu3.local"

Any time you edit this section, you will need to restart Ejabberd to
load the changes to the configuration file.

### Preventing cross-domain communication

May be possible with mod\_filter: <https://www.ejabberd.im/node/4992>