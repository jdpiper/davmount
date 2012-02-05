davmount
========

A wrapper for mount.davfs that uses GPG to encrypt credentials and lets users
mount WebDAV shares under their home directory.

Dependencies
------------

* [GnuPG] (http://gnupg.org/) and either gpg-agent or a keychain like
  [Seahorse] (http://projects.gnome.org/seahorse)
* [Sudo] (http://gratisoft.us/sudo/)

Installation
------------

Copy davmount somewhere on the path:

    sudo cp davmount /usr/local/bin
    sudo chmod 755 /usr/local/bin/davmount

Edit sudoers so users can use davmount:

    # Let root connect to the user's gpg-agent
    Defaults	env_keep+=GPG_AGENT_INFO

    # Let members of the davfs2 group run davmount
    %davfs2	ALL=(ALL):/usr/local/bin/davmount
    
    # Or without requiring authentication:
    # %davfs2	ALL=(All)NOPASSWD:/usr/local/bin/davmount

Configuration
-------------

Create a directory for davmount secrets:

    mkdir -p ~/.davmount/secrets

Create a secrets file for the server you want to connect to containing the
username and password separated by a space.

    unset HISTFILE
    gpg --default-recipient-self -o ~/.davmount/secrets/dav.example.com -e <<EOF
    "alice" "swordfish"
    EOF

Invocation
----------

To mount dav.example.com at /home/alice/dav.example.com

    sudo davmount https://dav.example.com/ /home/alice/dav.example.com

To unmount dav.example.com

    sudo davmount -u https://dav.example.com/

Or

    sudo davmount -u /home/alice/dav.example.com
