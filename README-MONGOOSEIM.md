Overview
--------

These instructions are part of the set of instructions for setting up
ZCS Chat feature. These instructions were tested with MongooseIM 1.5.1 on Ubuntu 12.04.
You can find a video walk-through of these instructions here:
<https://www.youtube.com/watch?v=C2jxWZNgyX8>

Download and install MongooseIM
-------------------------------

You can install ejabberd either on the same machine as ZCS or on a
separate machine using your default package manager such as APT.

### Install Perl packages

The following Perl packages are required by MongooseIM/ZCS
authentication script:

-   IO::Socket:SSL
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

### Set up external authentication for MongooseIM

1.  copy all files from ZimbraIMExtension/src/perl to
    {mongooseim-install-folder}/bin/zimbra
          mkdr -p /usr/lib/mongooseim/bin/zimbra/
          cp src/perl/* /usr/lib/mongooseim/bin/zimbra/

2.  Edit MongooseIM config file
          vi /usr/lib/mongooseim/etc/ejabberd.cfg 

3.  Change this line
        {auth_method, internal }

    to this:

        {auth_method, external }

4.  Find the line that looks like this (about 15 lines down):
        %%{extauth_program, "/path/to/external/authentication/script"}.

    and change it to this:

        {extauth_program, "/usr/lib/mongooseim/bin/zimbra/ejabberd_zimbraauth.pl"}

    use the path to your MongooseIM folder instead of
    /usr/lib/mongooseim if your installation of MongooseIM resides in a
    different path. Make sure to remove the dot "." at the end of the
    line, otherwise MongooseIM will complain about syntax error.

5.  Configure domain name in MongooseIM. Find a section that looks like
    this:
        %%%.   ================
        %%%'   SERVED HOSTNAMES

        %%
        %% hosts: Domains served by ejabberd.
        %% You can define one or several, for example:
        %% {hosts, ["example.net", "example.com", "example.org"]}.
        %%
        {hosts, ["localhost"] }.

    replace <i>localhost</i> with the name of your Zimbra domain.

</li>
<li>
save ejabebrd.cfg

</li>
<li>
Make sure ejabberd\_zimbraauth.pl is executable:

     chmod +x e/usr/lib/mongooseim/bin/zimbra/ejabberd_zimbraauth.pl

</li>
</ol>
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

### Start MongooseIM

    mongooseimctl start

Multiple domains
----------------

Edit MongooseIM configuration file (ejabberd.cfg). Find this section:

    %%%.   ================
    %%%'   SERVED HOSTNAMES

    %%
    %% hosts: Domains served by ejabberd.
    %% You can define one or several, for example:
    %% {hosts, ["example.net", "example.com", "example.org"]}.
    %%
    {hosts, ["localhost"] }.

Enter the list of domains you want to serve from this MongooseIM server
instead of “localhost”. E.g., to serve ubuntu2.local and ubuntu3.local:

    {hosts, ["ubuntu2.local", "ubuntu3.local"] }.

Any time you edit this section, you will need to restart MongooseIM to
load the changes to the configuration file.