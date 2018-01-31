# LibreVPN

> http://librevpn.org.ar

LibreVPN is a free virtual network.

These are the VPN's configuration and administration scripts that
perform many of the common actions on tinc.

For help, run `lvpn -h`.  Each command has its -h too.

## Create a node

    lvpn init -h

## Where is the configuration of my node?

In the `nodes/` directory, this allows you to maintain the
configuration of several nodes in a single machine, exchange them, or
copy them to other machines.

For every configuration change you have to use `lvpn install` or
`lvpn push` (depending on whether it is a local or remote node).

## My node name is full of underscores and/or is ugly

If you did not give your node a name, `lvpn` will use the name of your
machine.  If when you installed your distro you let the installer
choose on its own, you're going to have a node name full of
underscores in instead of the characters that tinc does not accept
(for example, "-" becomes "\_") and also the model of your computer.

If you want to change this, it is best to change the hostname of your
computer, following the steps indicated by your distro.  Otherwise,
put your desired name in the command `lvpn init a_cute_name`.

## How do I connect with another node?

Send your node file to the list vpn@hackcoop.com.ar or give it to
someone who is already in the network.

Then, connect to that node with:

    lvpn connectto your_node the_other_node
    lvpn install your_node

## Can I use it on Android?

Yes!  Install Tinc for Android from [F-Droid](https://f-droid.org) and
create a node with `lvpn init -A`.

## Requirements

tinc (1 or 1.1), avahi-daemon, bash.

The scripts inform about other commands that may be needed.

## Installation in the system

Building the man pages also requires the pandoc program.

    sudo make install PREFIX=/ usr


## Developers

See [CONVENTIONS](doc/en/CONVENTIONS.markdown).

## Wiki

http://wiki.hackcoop.com.ar/Category:LibreVPN
