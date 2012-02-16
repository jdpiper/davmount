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

Check that ~/.gnupg/gpg.conf specifies the "use-agent" option. If it's missing
or commented out add it. You'll probably need to log out and back in to get the
X session manager to load gpg-agent.

Copy davmount somewhere on the path:

    sudo cp davmount /usr/local/bin
    sudo chmod 755 /usr/local/bin/davmount

Edit sudoers so users can use davmount:

    # Let root connect to the user's gpg-agent
    Defaults    env_keep+=GPG_AGENT_INFO

    # Allow members of the davfs2 group to use davmount as root
    %davfs2     ALL=(root):/usr/local/bin/davmount

    # Or if you're lazy like me
    # %davfs2     ALL=(root)NOPASSWD:/usr/local/bin/davmount

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

sitemount
=========

A script that uses davmount to mount and unmount multiple WebDAV shares for a
site.

Configuration
-------------

Create a folder for the sites that contains a file named `servers` and a
subfolder for each site that contains a file named `mountpoints`. If you
operate Acme Hosting and host sites for AAA Graphics and A1 Accounting your
folders might look like this:

    mkdir -p ~/Sites/AcmeHosting/AAA-Graphics
    mkdir -p ~/Sites/AcmeHosting/A1-Accounting
    touch ~/Sites/AcmeHosting/servers
    touch ~/Sites/AcmeHosting/AAA-Graphics/mountpoints
    touch ~/Sites/AcmeHosting/A1-Accounting/mountpoints

The `servers` file contains a list of WebDAV urls for each deployment context.
If you have Release and Draft contexts hosted on example.com, your `servers`
file might look like this:

    SERVERS=(
        [Release]=https://example.com/webdav
        [Draft]=https://draft.example.com/webdav
    )

The `mountpoints` file contains a list of the folders to mount. The key is the
name of the mountpoint, and the value is the WebDAV URL, which is relative to
the URL from the `servers` file. If the files for AAA Graphics are stored at
https://example.com/webdav/aaa-graphics, your `mountpoints` might look like
this:

    MOUNTPOINTS=(
        [scripts]=aaa-graphics/scripts
        [images]=aaa-graphics/images
    )

Invocation
----------

To mount all the shares for a site:

    sitemount ~/Sites/AcmeHosting/AAA-Graphics/Release

To unmount all the shares:

    sitemount -u ~/Sites/AcmeHosting/AAA-Graphics/Release

You can mount several sites at once. To mount both the Release and Draft
contexts for AAA Graphics:

    sitemount ~/Sites/AcmeHosting/AAA-Graphics/{Release,Draft}

To unmount everything and remount AAA Graphics:

    sitemount -u ~/Sites/AcmeHosting/*/* \
              -m ~/Sites/AcmeHosting/AAA-Graphics/Release
