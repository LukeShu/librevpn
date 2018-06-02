# LibreVPN

LibreVPN is a free (libre and gratis) community-run Virtual Private
Network that uses the Tinc protocol / `tincd` program.

The "LibreVPN" software package (the `lvpn` program) is a set of tools
to make it easier to be a member of the LibreVPN network.

## LibreVPN - The Network

When you join LibreVPN, it is the 192.168.9.X/24 IPv4 network and the
2001:1291:200:83ab:X:X:X:X/64 IPv6 network.

Most Virtual Private Networks that you can join have one or a few
designated *servers*, and everyone else is a *client*.  On LibreVPN,
there is no single server; every *node* with a stable public IP
address or domain name is a server.  (Well, *every* node is a server,
but you won't be able to connect to them if they don't have a public
address!)

To join LibreVPN, you need to connect to one or more nodes that are
already in LibreVPN.  Connecting with a node must be mutual---you must
connect with them, and they must add you.  This means that to join
LibreVPN you must arrange with an existing LibreVPN member to connect;
you must engage with the LibreVPN community.

> There used to be an emailing list to make that easier, but the
> domain name it was on lapsed.  For now, if you don't know anyone
> already on LibreVPN that you can ask, send a pull-request to
> <https://github.com/LibreVPN/librevpn> with the your host file,
> which contains the information they need to accept a connection from
> you (we'll cover what information that is in a bit).

### A brief introduction to Tinc

The general notion of Tinc is that you have one instance of the
`tincd` daemon running for each Tinc network that you are on, the
configuration files for each network are in
`/etc/tinc/${NETWORK_NAME}/`.

On systems using systemd, once you have configured the network, you
may join/enable a network by running

	sudo systemctl start tinc@${NETWORK_NAME}.service

> Note: Unlike most services, `tinc@.service` is
> `WantedBy=tinc.service`, instead of being
> `WantedBy=multi-user.target`.  This means that to have tinc start at
> boot, you'll need to `systemctl enable` both
> `tinc@${NETWORK_NAME}.service` to enable *that network in
> particular*, and `tinc.service` to enable tinc *running at all*.
>
> 	sudo systemctl enable tinc.service tinc@${NETWORK_NAME}.service
>
> The Tinc developers probably should have named `tinc.service`
> `tinc.target` instead.  Oh well.

> Note: Currently, `lvpn install` actually installs a custom
> `tincd@lvpn.service` instead of using the normal
> `tinc@lvpn.service`.  IDK.

### LibreVPN and Tinc

By convention, most LibreVPN members have LibreVPN named as `lvpn` to
Tinc (this convention is semi-enforced by the `lvpn` tool).  So, how
do they set up `/etc/tinc/lvpn/`?  Well, mostly they let the `lvpn`
tool take care of it for them, but first let's cover some high-level
bits of what you'll see there.  Don't worry, we won't get too much in
to the technical details of Tinc configuration (that's what the Tinc
manual is for), we'll cover just enough that you can have an idea of
what the `lvpn` tool is doing.

The tinc daemon is configured primaril in `/etc/tinc/lvpn/tinc.conf`;
this includes "private" information; such as "this node's name"
(`Name=`), and "which nodes should I initiate connections to?"
(`ConnectTo=`).  Every node that is mentioned in `tinc.conf` (both in
`Name=` and `ConnectTo=`) must have a "host file" at
`/etc/tinc/lvpn/hosts/${node_name}`, which has the "public"
information for that node..  Additionally, a node that has others
connect directly to it needs the host file for each connecting node.

The host file for a node contains a public key; this is way a host can
be verified to really be the one described in the file.

A node that has other nodes connect directly to it needs a public
address of some sort (how else would the connection be initiated?).
This can be a stable IP address, or a domain name (potentially a
dynamic-DNS solution).  That public address ends up going in the
node's host file (`Address=`).

## LibreVPN - The Software

The `lvpn` program exists to help you manage the configuration for one
or more nodes so that they can join LibreVPN.  Normally, you'll just
be managing one node, but it supports keeping track of multiple
separate configurations by name.  This is useful for:

 - Generating the configuration for an Android device that can't run
   the `lvpn` program.
 - Generating the configuration for an embedded device, like an
   OpenWRT router.
 - Centrally managing the configurations for several hosts that you
   own.

The general flow is that you

 1. Run `lvpn init [NODE_NAME]` to create the configuration in a
    working directory.
 2. Send the host file that `lvpn init` generated to an existing
    member who will add you.
 3. Run `lvpn connectto YOUR_NODE THEIR_NODE` and other `lvpn COMMAND`
    commands to modify the base configuration.to configure which nodes
 4. Run `lvpn install NODE_NAME` to copy the configuration from the
    working directory to `/etc/tinc/lvpn/`.

## Putting it all together / quick-start

You can run the LibreVPN software either from a system installation,
or directly from the git repository.  If runing from the git
repository, simply run `./lvpn` instead of `lvpn`.

 1. Run `lvpn init YOUR_NODE_NAME`

    	$ lvpn init YOUR_NODE_NAME
    	> Creating /home/USERNAME/.config/lvpn/nodos/YOUR_NODE_NAME...
    	> Generating /home/USERNAME/.config/lvpn/nodos/YOUR_NODE_NAME/hosts/YOUR_NODE_NAME...
    	> ALERTA: El nodo no tiene una dirección pública, se configura como cliente
    	> Utilizando puerto al azar 24814
    	> Determining IPv4 address for the node...
    	> Generating IPv6...
    	> Adding subnets...
    	> This node is a switch
    	> Generating /home/USERNAME/.config/lvpn/nodos/YOUR_NODE_NAME/tinc.conf...
    	> Adding hosts...
    	> Copying remaining files...
    	> Generating keys...
    	Generating 4096 bits keys:
    	.....++ p
    	.................++ q
    	Done.
    	> Node has been succesfully created
    	> Important: 
    	> * Send the host file for your node to vpn@hackcoop.com.ar (/home/USERNAME/.config/lvpn/nodos/YOUR_NODE_NAME/hosts/YOUR_NODE_NAME), or
    	> * Share your node on the local network with `lvpn announce`
    	> * Add /home/USERNAME/.config/lvpn/nodos/YOUR_NODE_NAME/hosts/YOUR_NODE_NAME to the git repository and push it.

 2. Send the file
    `/home/USERNAME/.config/lvpn/nodos/YOUR_NODE_NAME/hosts/YOUR_NODE_NAME`
    to the existing members that you will be connecting to.  This will
    be `./nodos/YOUR_NODE_NAME/hosts/YOUR_NODE_NAME` if you are
    running directly from the git repository.
	
 3. Run `lvpn connectto YOUR_NODE THEIR_NODE` for each node that you
    will be connecting to.

    	$ lvpn connectto YOUR_NODE THEIR_NODE
    	> Copying host file for THEIR_NODE
    	> Connect to THEIR_NODE

 4. Run `lvpn install YOUR_NODE` to install the created configuration.
 
    	$ lvpn install YOUR_NODE
    	> Installing on system...
    	[sudo] password for USER: 
    	> Checking permissions...
    	> Tip: nss-mdns is needed to resolve .local addresses

 5. Run `sudo systemctl start tincd@lvpn.service` (the `lvpn install`
    command will enable the service (automatic start on boot), but not
    start it right away).
