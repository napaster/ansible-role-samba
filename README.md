# ansible-samba

[Samba](//wiki.samba.org/) is an Open Source suite that has provided file and
print services to all manner of SMB/CIFS clients, including the numerous
versions of Microsoft Windows operating systems.

## Requirements

* Ansible 3.0.0+;
* Samba 4.8+;

## Example configuration

```yaml
---
samba:
# Enable samba services or not (if ctdb in setup - only 'ctdb' will be enabled).
- enable: 'true'
# Restart samba services or not (if ctdb in setup - only 'ctdb' will be
# restarted).
  restart: 'true'
# Install samba package or not.
  install_package: 'true'
# 'cluster' or 'standalone' - this control behavior of role.
# If Samba is clustered, then ctdb managing services ('smb', 'winbind', etc)
# and role will not enable this services. Also services will not be restarted,
# only reloaded.
# When Samba is standalone, services will be enabled if 'enabled' in 'true'.
  mode: 'standalone'
  ctdb:
  - settings:
    - logging:
# Controls the verbosity of ctdbd's logging. Level may be:
# 'ERROR', 'WARNING', 'NOTICE', 'INFO', 'DEBUG'. Default is 'NOTICE'.
      - log_level: 'NOTICE'
# Specifies where ctdbd will write its log.
# Location type:
# 'file' - ctdbd will write its log. This is usually /var/log/log.ctdb
# (default).
# 'syslog' - if method (e.g. 'syslog:nonblocking') is specified then it
# specifies an extension that causes logging to be done in a non-blocking
# fashion. This can be useful under heavy loads that might cause the syslog
# daemon to dequeue messages too slowly, which would otherwise cause CTDB to
# block when logging.
        location: 'syslog'
# Options in this section affect the CTDB cluster setup.
      cluster:
# CTDB uses a recovery lock to avoid a split brain, where a cluster becomes
# partitioned and each partition attempts to operate independently. Issues that
# can result from a split brain include file data corruption, because file
# locking metadata may not be tracked correctly. CTDB uses a cluster leader and
# follower model of cluster management. All nodes in a cluster elect one node
# to be the leader. The leader node coordinates privileged operations such as
# database recovery and IP address failover. CTDB refers to the leader node as
# the recovery master. This node takes and holds the recovery lock to assert
# its privileged role in the cluster. By default, the recovery lock is
# implemented using a file residing in shared storage (usually) on a cluster
# filesystem. To support a recovery lock the cluster filesystem must support
# lock coherence. The recovery lock can also be implemented using an arbitrary
# cluster mutex call-out by using an exclamation point ('!') as the first
# character of recovery lock. For example, a value of !/usr/bin/myhelper
# recovery would run the given helper with the specified arguments. See the
# source code relating to cluster mutexes for clues about writing call-outs.
# If a cluster becomes partitioned (for example, due to a communication
# failure) and a different recovery master is elected by the nodes in each
# partition, then only one of these recovery masters will be able to take the
# recovery lock. The recovery master in the "losing" partition will not be able
# to take the recovery lock and will be excluded from the cluster. The nodes in
# the "losing" partition will elect each node in turn as their recovery master
# so eventually all the nodes in that partition will be excluded. CTDB does
# sanity checks to ensure that the recovery lock is held as expected. CTDB can
# run without a recovery lock but this is not recommended as there will be no
# protection from split brains. Default is none.
      - recovery_lock:
# Clustered FS example:
      # - type: 'fs'
      #   path: '/clusterfs/.ctdb/reclock'
# Rados Cluster example:
        - type: 'rados'
          cluster: 'ceph'
          client: 'mds_samba'
          pool: 'fs_data'
          object: 'ctdb_mutex'
# Is a private IP address that ctdbd will bind to. This option is only required
# when automatic address detection can not be used. This can be the case when
# running multiple ctdbd daemons/nodes on the same physical host (usually for
# testing), using InfiniBand for the private network or on Linux when sysctl
# net.ipv4.ip_nonlocal_bind=1. By default CTDB selects the first address from
# the nodes list that it can bind to.
          node_address: '10.10.10.1'
# This option specifies which transport to use for ctdbd internode
# communications on the private network. 'ib' means InfiniBand. The InfiniBand
# support is not regularly tested. If it is known to be broken then it may be
# disabled so that a value of 'ib' is considered invalid. Default is 'tcp'.
          transport: 'tcp'
      database:
# Directory on local storage where CTDB keeps a local copy of volatile TDB
# databases. This directory is local for each node and should not be stored on
# the shared cluster filesystem. Mounting a tmpfs (or similar memory filesystem)
# on this directory can provide a significant performance improvement when
# there is I/O contention on the local disk. Default is
# '/var/lib/ctdb/volatile'.
      - volatile_database_directory: '/var/lib/ctdb/volatile'
# Directory on local storage where CTDB keeps a local copy of persistent TDB
# databases. This directory is local for each node and should not be stored on
# the shared cluster filesystem. Default is '/var/lib/ctdb/persistent'.
        persistent_database_directory: '/var/lib/ctdb/persistent'
# Directory on local storage where CTDB keeps a local copy of internal state
# TDB databases. This directory is local for each node and should not be stored
# on the shared cluster filesystem. Default is '/var/lib/ctdb/state'
        state_database_directory: '/var/lib/ctdb/state'
# This parameter enables TDB_MUTEX_LOCKING feature on volatile databases if the
# robust mutexes are supported. This optimizes the record locking using robust
# mutexes and is much more efficient that using posix locks. If robust mutexes
# are unreliable on the platform being used then they can be disabled.
        tdb_mutexes: ''
# This script used by CTDB's database locking code to attempt to provide
# debugging information when CTDB is unable to lock an entire database or a
# record. This script should be a bare filename relative to the CTDB
# configuration directory (/etc/ctdb/). Any directory prefix is ignored and the
# path is calculated relative to this directory. CTDB provides a lock debugging
# script and installs it as '/etc/ctdb/debug_locks.sh'. Default is None.
        lock_debug_script: ''
      event:
# This cript used by CTDB's event handling code to attempt to provide debugging
# information when an event times out. This script should be a bare filename
# relative to the CTDB configuration directory (/etc/ctdb/). Any directory
# prefix is ignored and the path is calculated relative to this directory. CTDB
# provides a script for debugging timed out event scripts and installs it as
# '/etc/ctdb/debug-hung-script.sh'. Default is None.
      - debug_script: ''
    failover:
# If set to 'true' then public IP failover is disabled. Default is 'false'.
    - disabled: ''
    legacy:
# If set to 'true' CTDB starts in the STOPPED state. To allow the node to take
# part in the cluster it must be manually continued with the the "ctdb continue"
# command. Default is 'false'.
    - ctdb_start_as_stopped: ''
# If set to 'true' CTDB starts in the DISABLED state. To allow the node to host
# public IP addresses and services, it must be manually enabled using the
# "ctdb enable" command. Default is 'false'.
      start_as_disabled: ''
# Usually CTDB runs with real-time priority. This helps it to perform
# effectively on a busy system, such as when there are thousands of Samba
# clients. If you are running CTDB on a platform that does not support
# real-time priority, you can set this to false. Default is 'true'.
      realtime_scheduling: ''
# Indicates whether a node can become the recovery master for the cluster. If
# this is set to 'false' then the node will not be able to become the recovery
# master for the cluster. This feature is primarily used for making a cluster
# span across a WAN link and use CTDB as a WAN-accelerator. Default is 'true'.
      recmaster_capability: ''
# Indicates whether a node can become a location master for records in a
# database. If this is set to 'false' then the node will not be part of the
# vnnmap. This feature is primarily used for making a cluster span across a WAN
# link and use CTDB as a WAN-accelerator. Default is 'true'.
      lmaster_capability: ''
# This option sets the debug level of event script output to LOGLEVEL. Default
# is 'ERROR'.
      script_log_level: 'DEBUG'
# Each node is configured with a unique, permanently assigned private address.
# This address is configured by the operating system. This address uniquely
# identifies a physical node in the cluster and is the address that CTDB daemons
# will use to communicate with the CTDB daemons on other nodes. This is list of
# private addresses for all nodes in the cluster. This file must be the same on
# all nodes in the cluster. Private addresses should not be used by clients to
# connect to services provided by the cluster.
  - private_address:
    - '10.10.8.1'
    - '10.10.8.2'
# Public addresses are used to provide services to clients. Public addresses
# are not configured at the operating system level and are not permanently
# associated with a particular node. Instead, they are managed by CTDB and are
# assigned to interfaces on physical nodes at runtime. The CTDB cluster will
# assign/reassign these public addresses across the available healthy nodes in
# the cluster. When one node fails, its public addresses will be taken over by
# one or more other nodes in the cluster. This ensures that services provided
# by all public addresses are always available to clients, as long as there are
# nodes available capable of hosting this address. In many cases the public
# addresses file will be the same on all nodes. However, it is possible to use
# different public address configurations on different nodes.
  - public_address:
    - address: '100.100.105.100/24'
      iface: 'eth3'
    - address: '10.10.8.252/24'
      iface: 'eth1'
  - script_options:
# Whether one or more offline interfaces should cause a monitor event to fail
# if there are other interfaces that are up. If this is 'yes' and a node has
# some interfaces that are down then ctdb status will display the node as
# "PARTIALLYONLINE". Note that whe option in 'yes' is not generally compatible
# with NAT gateway or LVS. NAT gateway relies on the interface configured by
# 'CTDB_NATGW_PUBLIC_IFACE' to be up and LVS replies on
# 'CTDB_LVS_PUBLIC_IFACE' to be up. CTDB does not check if these options are set
# in an incompatible way so care is needed to understand the interaction.
# Default is 'no'.
    - ctdb_partially_online_interfaces: 'no'
# Is an alternate network gateway to use on the NAT gateway master node. If set,
# a fallback default route is added via this network gateway. Setting this
# variable is optional - if not set that no route is created on the NAT gateway
# master node.
      ctdb_natgw_default_gateway: ''
# The private sub-network that is internally routed via the NAT gateway master
# node. This is usually the private network that is used for node addresses.
      ctdb_natgw_private_network: '100.100.100.0/24'
# The network interface on which the CTDB_NATGW_PUBLIC_IP will be configured.
      ctdb_natgw_public_iface: 'eth0'
# Indicates the IP address that is used for outgoing traffic (originating from
# 'ctdb_natgw_private_network') on the NAT gateway master node. This must not
# be a configured public IP address.
      ctdb_natgw_public_ip: '5.128.220.100/24'
# Each IPADDR/MASK identifies a network or host to which NATGW should create a
# fallback route, instead of creating a single default route. This can be used
# when there is already a default route, via an interface that can not reach
# required infrastructure, that overrides the NAT gateway default route. If
# GATEWAY is specified then the corresponding route on the NATGW master node
# will be via GATEWAY. Such routes are created even if
# 'CTDB_NATGW_DEFAULT_GATEWAY' is not specified. If GATEWAY is not specified
# for some networks then routes are only created on the NATGW master node for
# those networks if 'CTDB_NATGW_DEFAULT_GATEWAY' is specified. This should be
# used with care to avoid causing traffic to unnecessarily double-hop through
# the NAT gateway master, even when a node is hosting public IP addresses. Each
# specified network or host should probably have a corresponding automatically
# created link route or static route to avoid this.
      ctdb_natgw_static_routes:
      - route: '5.128.220.0/24'
        gateway: '5.128.220.1'
# Interface is the network interface that clients will use to connection to
# CTDB_LVS_PUBLIC_IP. This is optional for slave-only nodes.
      ctdb_lvs_public_iface: 'eth1'
# The LVS public address. No default
      ctdb_lvs_public_ip: '5.128.220.100'
# Provides CTDB's Samba winbind service management.
      ctdb_service_winbind: 'winbind'
# When monitoring Samba, check TCP ports in space-separated PORT-LIST.
# Default is to monitor ports that Samba is configured to listen on.
      ctdb_samba_check_ports: ''
# As part of monitoring, should CTDB skip the check for the existence of each
# directory configured as share in Samba. This may be desirable if there is a
# large number of shares. Default is 'no'.
      ctdb_samba_skip_share_check: 'no'
# Distribution specific service for managing nmbd. Default is distro-dependant.
      ctdb_service_nmb: 'nmb'
# Distribution specific service for managing smbd. Default is distro-dependant.
      ctdb_service_smb: 'smb'
# Command specifies the path to a callout to handle interactions with the
# configured NFS system, including startup, shutdown, monitoring. Default is
# the included nfs-linux-kernel-callout.
      ctdb_nfs_callout: ''
# Specifies the path to a directory containing files that describe how to
# monitor the responsiveness of NFS RPC services. Dir can be used to point to
# different sets of checks for different NFS servers. One way of using this is
# to have it point to, say, /etc/ctdb/nfs-checks-enabled.d and populate it
# with symbolic links to the desired check files. This avoids duplication and
# is upgrade-safe. Default is '/etc/ctdb/nfs-checks.d', which contains NFS RPC
# checks suitable for Linux kernel NFS.
      ctdb_nfs_checks_dir: '/etc/ctdb/nfs-checks.d'
# As part of monitoring, should CTDB skip the check for the existence of each
# directory exported via NFS. This may be desirable if there is a large number
# of exports. Default is 'no'.
      ctdb_nfs_skip_share_check: 'no'
# Address or hostname indicates the address that rpcinfo should connect to when
# doing rpcinfo check on IPv4 RPC service during monitoring. Optimally this
# would be "localhost". However, this can add some performance overheads.
# Default is '127.0.0.1'.
      ctdb_rpcinfo_localhost: '127.0.0.1'
# IPADDR or HOSTNAME indicates the address that rpcinfo should connect to when
# doing rpcinfo check on IPv6 RPC service during monitoring. Optimally this
# would be "localhost6" (or similar). However, this can add some performance
# overheads. Default is '::1'.
      ctdb_rpcinfo_localhost6: '::1'
# The type of filesystem used for a clustered NFS' shared state. Default is
# None.
      ctdb_nfs_state_fs_type: ''
# The directory where a clustered NFS' shared state will be located. Default is
# None.
      ctdb_nfs_state_mnt: ''
# Directory on shared storage containing scripts to start tgtd for each public
# IP address. Default is None
      ctdb_start_iscsi_scriptst: ''
# Is the maximum number of volatile TDB database backups to be kept (for each
# database) when a corrupt database is found during startup. Volatile TDBs are
# zeroed during startup so backups are needed to debug any corruption that
# occurs before a restart.
      ctdb_max_corrupt_db_backups: '10'
  smb:
# Parameters in this section apply to the server as a whole, or are defaults
# for sections that do not specifically define certain items.
  - global:
# This a full path name to a script called by smbd that should stop a shutdown
# procedure issued by the shutdown script. If the connected user possesses the
# 'SeRemoteShutdownPrivilege', right, this command will be run as root. Default
# is None.
    - abort_shutdown_script: '/sbin/shutdown -c'
# This is the full pathname to a script that will be run AS ROOT by smbd when a
# new group is requested. It will expand any %g to the group name passed. This
# script is only useful for installations using the Windows NT domain
# administration tools. The script is free to create a group with an arbitrary
# name to circumvent unix group name restrictions. In that case the script must
# print the numeric gid of the created group on stdout. Default is None.
      add_group_script: '/usr/sbin/groupadd %g'
# This is the full pathname to a script that will be run by smbd when a machine
# is added to Samba's domain and a Unix account matching the machine's name
# appended with a "$" does not already exist. This option is very similar to
# the add user script, and likewise uses the %u substitution for the account
# name. Do not use the %m substitution. Default is None.
      add_machine_script: '/usr/sbin/adduser -n -g machines -c Machine -d /var/lib/nobody -s /bin/false %u'
# Samba 3.0.23 introduced support for adding printer ports remotely using the
# Windows "Add Standard TCP/IP Port Wizard". This option defines an external
# program to be executed when smbd receives a request to add a new Port to the
# system. The script is passed two parameters: port name & device URI. The
# deviceURI is in the format of socket://<hostname>[:<portnumber>] or
# lpd://<hostname>/<queuename>.
      addport_command: '/etc/samba/scripts/addport.sh'
# With the introduction of MS-RPC based printing support for Windows NT/2000
# clients in Samba 2.2, The MS Add Printer Wizard (APW) icon is now also
# available in the "Printers..." folder displayed a share listing. The APW
# allows for printers to be add remotely to a Samba or Windows NT/2000 print
# server. For a Samba host this means that the printer must be physically added
# to the underlying printing system. The 'addprinter_command' defines a script
# to be run which will perform the necessary operations for adding the printer
# to the print system and to add the appropriate service definition to the
# 'smb.conf' file in order that it can be shared by smbd.
# The 'addprinter_command' is automatically invoked with the following
# parameter (in order):
# * printer name
# * share name
# * port name
# * driver name
# * location
# * Windows 9x driver location
# All parameters are filled in from the PRINTER_INFO_2 structure sent by the
# Windows NT/2000 client with one exception. The "Windows 9x driver location"
# parameter is included for backwards compatibility only. The remaining fields
# in the structure are generated from answers to the APW questions. Once the
# addprinter command has been executed, smbd will reparse the smb.conf to
# determine if the share defined by the APW exists. If the sharename is still
# invalid, then smbd will return an ACCESS_DENIED error to the client. The
# 'addprinter_command' program can output a single line of text, which Samba
# will set as the port the new printer is connected to. If this line isn't
# output, Samba won't reload its printer shares. Default is None.
      addprinter_command: '/usr/bin/addprinter'
# Samba 2.2.0 introduced the ability to dynamically add and delete shares via
# the Windows NT 4.0 Server Manager. The add share command is used to define an
# external program or script which will add a new service definition to
# smb.conf. In order to successfully execute the add share command, smbd
# requires that the administrator connects using a root account (i.e. uid == 0)
# or has the SeDiskOperatorPrivilege. Scripts defined in the add share command
# parameter are executed as root. When executed, smbd will automatically invoke
# the add share command with five parameters.
# * configFile - the location of the global smb.conf file.
# * shareName - the name of the new share.
# * pathName - path to an **existing** directory on disk.
# * comment - comment string to associate with the new share.
# * max connections Number of maximum simultaneous connections to this share.
# This parameter is only used to add file shares. To add printer shares, see
# the addprinter command. Default is None.
      add_share_command: '/usr/local/bin/addshare'
# This is the full pathname to a script that will be run AS ROOT by smbd under
# special circumstances described below. Normally, a Samba server requires that
# UNIX users are created for all users accessing files on this server. For
# sites that use Windows NT account databases as their primary user database
# creating these users and keeping the user list in sync with the Windows NT
# PDC is an onerous task. This option allows smbd to create the required UNIX
# users ON DEMAND when a user accesses the Samba server. When the Windows user
# attempts to access the Samba server, at login (session setup in the SMB
# protocol) time, smbd contacts the password server and attempts to
# authenticate the given user with the given password. If the authentication
# succeeds then smbd attempts to find a UNIX user in the UNIX password database
# to map the Windows user into. If this lookup fails, and add user script is set
# then smbd will call the specified script AS ROOT, expanding any '%u' argument
# to be the user name to create. If this script successfully creates the user
# then smbd will continue on as though the UNIX user already existed. In this
# way, UNIX users are dynamically created to match existing Windows NT accounts.
# Default is None.
      add_user_script: '/usr/local/samba/bin/add_user %u'
# Full path to the script that will be called when a user is added to a group
# using the Windows NT domain administration tools. It will be run by smbd
# AS ROOT. Any '%g' will be replaced with the group name and any '%u' will be
# replaced with the user name. Default is None.
      add_user_to_group_script: '/usr/sbin/adduser %u %g'
# This parameter controls the lifetime of tokens that the AFS fake-kaserver
# claims. In reality these never expire but this lifetime controls when the afs
# client will forget the token. Set this parameter to 0 to get NEVERDATE.
# Default is '604800'.
      afs_token_lifetime: '604800'
# If you are using the fake kaserver AFS feature, you might want to hand-craft
# the usernames you are creating tokens for. For example this is necessary if
# you have users from several domain in your AFS Protection Database. One
# possible scheme to code users as DOMAIN+User as it is done by winbind with
# the + as a separator. The mapped user name must contain the cell name to log
# into, so without setting this parameter there will be no token. Default is
# None.
      afs_username_map: '%u@afs.samba.org'
# The integer parameter specifies the maximum number of threads each smbd
# process will create when doing parallel asynchronous IO calls. If the number
# of outstanding calls is greater than this number the requests will not be
# refused but go onto a queue and will be scheduled in turn as outstanding
# requests complete. Default is '100'.
      aio_max_threads: '100'
# This determines how Samba will use its algorithmic mapping from uids/gid to
# the RIDs needed to construct NT Security Identifiers. Setting this option to
# a larger value could be useful to sites transitioning from WinNT and Win2k,
# as existing user and group rids would otherwise clash with system users etc.
# All UIDs and GIDs must be able to be resolved into SIDs for the correct
# operation of ACLs on the server. As such the algorithmic mapping can't be
# "turned off", but pushing it "out of the way" should resolve the issues. Users
# and groups can then be assigned "low" RIDs in arbitrary-rid supporting
# backends. Default is '1000'.
      algorithmic_rid_base: '100000'
# This option controls whether DCERPC services are allowed to be used with
# DCERPC_AUTH_LEVEL_CONNECT, which provides authentication, but no per message
# integrity nor privacy protection. This option yields precedence to the
# implementation specific restrictions. E.g. the drsuapi and backupkey
# protocols require DCERPC_AUTH_LEVEL_PRIVACY. The dnsserver protocol requires
# DCERPC_AUTH_LEVEL_INTEGRITY. Default is 'no'.
      allow_dcerpc_auth_level_connect: 'no'
# This option determines what kind of updates to the DNS are allowed. DNS
# updates can either be disallowed completely by setting it to disabled.
# Default is is 'secure only'.
      allow_dns_updates: 'secure only'
# In normal operation the option wide links which allows the server to follow
# symlinks outside of a share path is automatically disabled when unix
# extensions are enabled on a Samba server. This is done for security purposes
# to prevent UNIX clients creating symlinks to areas of the server file system
# that the administrator does not wish to export. Setting allow insecure wide
# links to true disables the link between these two parameters, removing this
# protection and allowing a site to configure the server to follow symlinks (by
# setting wide links to 'yes') even when unix extensions is turned on. It is not
# recommended to enable this option unless you fully understand the implications
# of allowing the server to follow symbolic links created by UNIX clients. For
# most normal Samba configurations this would be considered a security hole and
# setting this parameter is not recommended. This option was added at the
# request of sites who had deliberately set Samba up in this way and needed to
# continue supporting this functionality without having to patch the Samba code.
# Default is 'no'.
      allow_insecure_wide_links: 'no'
# This option controls whether the netlogon server (currently only in
# "active directory domain controller" mode), will reject clients which does
# not support 'NETLOGON_NEG_STRONG_KEYS' nor 'NETLOGON_NEG_SUPPORTS_AES'. If you
# have clients without RequireStrongKey = 1 in the registry, you may need to
# set to 'yes', until you have fixed all clients - allows weak crypto to be
# negotiated, maybe via downgrade attacks. This option yields precedence to the
# 'reject_md5_clients' option. Default is 'no'.
      allow_nt4_crypto: 'no'
# This option only takes effect when the security option is set to server,
# domain or ads. If it is set to 'no', then attempts to connect to a resource
# from a domain or workgroup other than the one which smbd is running in will
# fail, even if that domain is trusted by the remote server doing the
# authentication. This is useful if you only want your Samba server to serve
# resources to users in the domain it is a member of. As an example, suppose
# that there are two domains DOMA and DOMB. DOMB is trusted by DOMA, which
# contains the Samba server. Under normal circumstances, a user with an account
# in DOMB can then access the resources of a UNIX account with the same
# account name on the Samba server even if they do not have an account in DOMA.
# This can make implementing a security boundary difficult. Default is 'yes'.
      allow_trusted_domains: 'yes'
# If set to no (the default), smbd checks at startup if other smbd versions are
# running in the cluster and refuses to start if so. This is done to protect
# data corruption in internal data structures due to incompatible Samba versions
# running concurrently in the same cluster. Setting this parameter to 'yes'
# disables this safety check.
      allow_unsafe_cluster_upgrade: 'no'
# This option controls whether winbind will execute the gpupdate command defined
# in gpo update command on the Group Policy update interval. The Group Policy
# update interval is defined as every 90 minutes, plus a random offset between
# 0 and 30 minutes. This applies Group Policy Machine polices to the client or
# KDC and machine policies to a server. Default is 'no'.
      apply_group_policies: 'no'
# This parameter specifies whether Samba should fork the async smb echo handler.
# It can be beneficial if your file system can block syscalls for a very long
# time. In some circumstances, it prolongs the timeout that Windows uses to
# determine whether a connection is dead. This parameter is only for SMB1. For
# SMB2 and above TCP keepalives can be used instead. Default is 'no'.
      async_smb_echo_handler: 'no'
# When 'yes', this option causes Samba (acting as an Active Directory Domain
# Controller) to stream authentication events across the internal message bus.
# Scripts built using Samba's python bindings can listen to these events by
# registering as the service auth_event. This should be considered a developer
# option (it assists in the Samba testsuite) rather than a facility for external
# auditing, as message delivery is not guaranteed (a feature that the testsuite
# works around). Additionally Samba must be compiled with the jansson support
# for this option to be effective. The authentication events are also logged via
# the normal logging methods when the log level is set appropriately.
# Default is 'no'.
      auth_event_notification: 'no'
# This is a list of services that you want to be automatically added to the
# browse lists. This is most useful for homes and printers services that would
# otherwise not be visible. Note that if you just want all printers in your
# printcap file loaded then the load printers option is easier. Default is None.
      auto_services:
      - 'fred'
      - 'lp'
      - 'colorlp'
# This parameter lets you "turn off" a service. If 'no', then ALL attempts to
# connect to the service will fail. Such failures are logged. Default is 'yes'.
      available: 'yes'
# This parameters defines the directory samba will use to store the
# configuration files for bind, such as named.conf. NOTE: The bind dns
# directory needs to be on the same mount point as the private directory!
      binddns_dir: '/var/lib/samba/bind-dns'
# This global parameter allows the Samba admin to limit what interfaces on a
# machine will serve SMB requests. It affects file service smbd and name
# service nmbd in a slightly different ways. For name service it causes nmbd to
# bind to ports 137 and 138 on the interfaces listed in the interfaces
# parameter. nmbd also binds to the "all addresses" interface (0.0.0.0) on
# ports 137 and 138 for the purposes of reading broadcast messages. If this
# option is not set then nmbd will service name requests on all of these
# sockets. If bind interfaces only is set then nmbd will check the source
# address of any packets coming in on the broadcast sockets and discard any
# that don't match the broadcast addresses of the interfaces in the interfaces
# parameter list. As unicast packets are received on the other sockets it
# allows nmbd to refuse to serve names to machines that send packets that
# arrive through any interfaces not listed in the interfaces list. IP Source
# address spoofing does defeat this simple check, however, so it must not be
# used seriously as a security feature for nmbd. For file service it causes
# smbd to bind only to the interface list given in the interfaces parameter.
# This restricts the networks that smbd will serve, to packets coming in on
# those interfaces. Note that you should not use this parameter for machines
# that are serving PPP or other intermittent or non-broadcast network
# interfaces as it will not cope with non-permanent interfaces. If bind
# interfaces only is set and the network address 127.0.0.1 is not added to the
# interfaces parameter list smbpasswd(8) may not work as expected.
# Default is 'no'.
      bind_interfaces_only: 'no'
# This controls whether smbd will serve a browse list to a client doing a
# NetServerEnum call. Normally set to 'yes'. You should never need to change
# this.
      browse_list: 'yes'
# Usually, most of the TDB files are stored in the lock directory. Since Samba
# 3.4.0, it is possible to differentiate between TDB files with persistent data
# and TDB files with non-persistent data using the state directory and the
# cache directory options. This option specifies the directory for storing TDB
# files containing non-persistent data that will be kept across service
# restarts. The directory should be placed on persistent storage, but the data
# can be safely deleted by an administrator. Default is '/var/lib/samba'.
      cache_directory: '/var/lib/samba'
# This parameter specifies whether Samba should reply to a client's file change
# notify requests. You should never need to change this parameter.
# Default is 'yes'.
      change_notify: 'yes'
# Samba 2.2.0 introduced the ability to dynamically add and delete shares via
# the Windows NT 4.0 Server Manager. The change share command is used to define
# an external program or script which will modify an existing service
# definition in smb.conf. In order to successfully execute the change share
# command, smbd requires that the administrator connects using a root account
# (i.e. uid == 0) or has the SeDiskOperatorPrivilege. Scripts defined in the
# change share command parameter are executed as root. Default is None.
      change_share_command: ''
# The name of a program that can be used to check password complexity. The
# password is sent to the program's standard input. The program must return 0
# on a good password, or any other value if the password is bad. In case the
# password is considered weak (the program does not return 0) the user will be
# notified and the password change will fail. In Samba AD, this script will be
# run AS ROOT by samba without any substitutions. Default is None.
      check_password_script: '/usr/local/sbin/crackcheck'
# This option controls the port used by the CLDAP protocol.
      cldap_port: '389'
# The value of the parameter (a string) is the highest protocol level that will
# be supported for IPC$ connections as DCERPC transport. Normally this option
# should not be set as the automatic negotiation phase in the SMB protocol
# takes care of choosing the appropriate protocol. The value 'default' refers
# to the latest supported protocol.
      client_ipc_max_protocol: 'default'
# This setting controls the minimum protocol version that the will be attempted
# to use for IPC$ connections as DCERPC transport. Normally this option should
# not be set as the automatic negotiation phase in the SMB protocol takes care
# of choosing the appropriate protocol. The value default refers to the higher
# value of NT1 and the effective value of client min protocol.
      client_ipc_min_protocol: 'default'
# This controls whether the client is allowed or required to use SMB signing
# for IPC$ connections as DCERPC transport. Possible values are auto, mandatory
# and disabled. When set to 'mandatory' or 'default', SMB signing is required.
# When set to 'auto', SMB signing is offered, but not enforced and if set to
# 'disabled', SMB signing is not offered either. Connections from winbindd to
# Active Directory Domain Controllers always enforce signing.
      client_ipc_signing: 'default'
# This parameter determines whether or not smbclient and other samba client
# tools will attempt to authenticate itself to servers using the weaker LANMAN
# password hash. If disabled, only server which support NT password hashes
# (e.g. Windows NT/2000, Samba, etc... but not Windows 95/98) will be able to be
# connected from the Samba client. The LANMAN encrypted response is easily
# broken, due to its case-insensitive nature, and the choice of algorithm.
# Clients without Windows 95/98 servers are advised to disable this option (the
# default). Disabling this option will also disable the client plaintext auth
# option. Likewise, if the client ntlmv2 auth parameter is enabled, then only
# NTLMv2 logins will be attempted.
      client_lanman_auth: 'no'
# The client ldap sasl wrapping defines whether ldap traffic will be signed or
# signed and encrypted (sealed). Possible values are 'plain', 'sign' and 'seal'.
# The values 'sign' and 'seal' are only available if Samba has been compiled
# against a modern OpenLDAP version (2.3.x or higher). This option is needed in
# the case of Domain Controllers enforcing the usage of signed LDAP connections
# (e.g. Windows 2000 SP3 or higher). LDAP 'sign' and 'seal' can be controlled
# with the registry key
# "HKLM\System\CurrentControlSet\Services\NTDS\Parameters\LDAPServerIntegrity"
# on the Windows server side. Depending on the used KRB5 library it is possible
# that the message "integrity only" is not supported. In this case, 'sign' is
# just an alias for 'seal'. The default value is 'sign'. That implies
# synchronizing the time with the KDC in the case of using Kerberos.
# Default is 'sign'.
      client_ldap_sasl_wrapping: 'sign'
# The value of the parameter (a string) is the highest protocol level that will
# be supported by the client. Normally this option should not be set as the
# automatic negotiation phase in the SMB protocol takes care of choosing the
# appropriate protocol. The value 'default' refers to SMB3_11.
      client_max_protocol: 'default'
# This setting controls the minimum protocol version that the client will
# attempt to use. Normally this option should not be set as the automatic
# negotiation phase in the SMB protocol takes care of choosing the appropriate
# protocol. Default is 'CORE'.
      client_min_protocol: 'SMB3'
# This parameter determines whether or not smbclient will attempt to
# authenticate itself to servers using the NTLMv2 encrypted password response.
# If enabled, only an NTLMv2 and LMv2 response (both much more secure than
# earlier versions) will be sent. Similarly, if enabled, NTLMv1, client lanman
# auth and client plaintext auth authentication will be disabled.
# Default is 'yes'.
      client_ntlmv2_auth: 'yes'
# Specifies whether a client should send a plaintext password if the server
# does not support encrypted passwords. Default is 'no'.
      client_plaintext_auth: 'no'
# This controls whether the client is allowed or required to use SMB signing.
# Possible values are 'auto', 'mandatory' and 'disabled'.
# When set to 'auto' or 'default', SMB signing is offered, but not enforced.
# When set to 'mandatory', SMB signing is required and if set to 'disabled',
# SMB signing is not offered either.
      client_signing: 'default'
# With this parameter you can add additional addresses nmbd will register with
# a WINS server. These addresses are not necessarily present on all nodes
# simultaneously, but they will be registered with the WINS server so that
# clients can contact any of the nodes. Default is None.
      cluster_addresses:
      - '10.0.0.1'
      - '10.0.0.2'
      - '10.0.0.3'
# This parameter specifies whether Samba should contact ctdb for accessing its
# tdb files and use ctdb as a backend for its messaging backend. Set this
# parameter to 'yes' only if you have a cluster setup with ctdb running.
# Default is 'no'.
      clustering: 'no'
# This is a text field that is seen next to a share when a client does a
# queries the server, either via the network neighborhood or via net view to
# list what shares are available. If you want to set the string that is
# displayed next to the machine name then see the server string parameter.
# Default is None.
      comment: 'Hello'
# This controls the backend for storing the configuration. Possible values are
# 'file' (the default) and 'registry'. When 'registry' is encountered while
# loading smb.conf, the configuration read so far is dropped and the global
# options are read from registry instead. So this triggers a registry only
# configuration. Share definitions are not read immediately but instead registry
# shares is set to 'yes'. This option can not be set inside the registry
# configuration itself.
      config_backend: 'file'
# Setting this parameter to no prevents winbind from creating custom krb5.conf
# files. Winbind normally does this because the krb5 libraries are not
# AD-site-aware and thus would pick any domain controller out of potentially
# very many. Winbind is site-aware and makes the krb5 libraries use a local DC
# by creating its own krb5.conf files. Preventing winbind from doing this might
# become necessary if you have to add special options into your
# system-krb5.conf that winbind does not see. Default is 'yes'.
      create_krb5_conf: 'yes'
# If you set 'clustering' to 'yes', you need to tell Samba where ctdbd listens
# on its unix domain socket. The default path is '/tmp/ctdb.socket' which you
# have to explicitly set for Samba in smb.conf.
      ctdbd_socket: '/tmp/ctdb.socket'
# In a cluster environment using Samba and ctdb it is critical that locks on
# central ctdb-hosted databases like locking.tdb are not held for long. With
# the current Samba architecture it happens that Samba takes a lock and while
# holding that lock makes file system calls into the shared cluster file system.
# This option makes Samba warn if it detects that it has held locks for the
# specified number of milliseconds. If this happens, smbd will emit a debug
# level 0 message into its logs and potentially into syslog. The most likely
# reason for such a log message is that an operation of the cluster file system
# Samba exports is taking longer than expected. The messages are meant as a
# debugging aid for potential cluster problems. The default value of '0'
# disables this logging.
      ctdb_locktime_warn_threshold: '0'
# This parameter specifies a timeout in milliseconds for the connection between
# Samba and ctdb. When something in the cluster blocks, it can happen that we
# wait indefinitely long for ctdb, just adding to the blocking condition. In a
# well-running cluster this should never happen, but there are too many
# components in a cluster that might have hickups. Choosing the right balance
# for this value is very tricky, because on a busy cluster long service times
# to transfer something across the cluster might be valid. Setting it too short
# will degrade the service your cluster presents, setting it too long might
# make the cluster itself not recover from something severely broken for too
# long. Be aware that if you set this parameter, this needs to be in the file
# smb.conf, it is not really helpful to put this into a registry configuration
# (typical on a cluster), because to access the registry contact to ctdb is
# required. Setting ctdb timeout to n makes any process waiting longer than
# n milliseconds for a reply by the cluster panic. Setting it to '0' (the
# default) makes Samba block forever, which is the highly recommended default.
      ctdb_timeout: '0'
# This parameter is only applicable if printing is set to cups. If set, this
# option specifies the number of seconds that smbd will wait whilst trying to
# contact to the CUPS server. The connection will fail if it takes longer than
# this number of seconds.
      cups_connection_timeout: '30'
# This parameter is only applicable if printing is set to cups and if you use
# CUPS newer than 1.0.x.It is used to define whether or not Samba should use
# encryption when talking to the CUPS server. Possible values are 'auto', 'yes'
# and 'no'. When set to 'auto' we will try to do a TLS handshake on each CUPS
# connection setup. If that fails, we will fall back to unencrypted operation.
# Default is 'no'.
      cups_encrypt: 'no'
# This parameter is only applicable if printing is set to cups. If set, this
# option overrides the ServerName option in the CUPS client.conf. This is
# necessary if you have virtual samba servers that connect to different CUPS
# daemons. Optionally, a port can be specified by separating the server name
# and port number with a colon. If no port was specified, the default port for
# IPP (631) will be used.
      cups_server: '631'
# Specifies which DCE/RPC endpoint servers should be run.
# Default is 'epmapper, wkssvc, rpcecho, samr, netlogon, lsarpc, drsuapi,
# dssetup, unixinfo, browser, eventlog6, backupkey, dnsserver'.
      dcerpc_endpoint_servers:
      - 'epmapper'
      - 'wkssvc'
# The value of the parameter (a decimal integer) represents the number of
# minutes of inactivity before a connection is considered dead, and it is
# disconnected. The deadtime only takes effect if the number of open files is
# zero. This is useful to stop a server's resources being exhausted by a
# large number of inactive connections. Most clients have an auto-reconnect
# feature when a connection is broken so in most cases this parameter should be
# transparent to users. Using this parameter with a timeout of a few minutes is
# recommended for most systems. A deadtime of zero (the default) indicates that
# no auto-disconnection should be performed.
      deadtime: '0'
# With this boolean parameter enabled, the debug class (DBGC_CLASS) will be
# displayed in the debug header. Default is 'no'.
      debug_class: 'no'
# Sometimes the timestamps in the log messages are needed with a resolution of
# higher that seconds, this boolean parameter adds microsecond resolution to the
# timestamp message header when turned on. Note that the parameter
# 'debug_timestamp' must be on for this to have an effect. Default is 'yes'.
      debug_hires_timestamp: 'yes'
# When using only one log file for more then one forked smbd process there may
# be hard to follow which process outputs which message. This boolean parameter
# is adds the process-id to the timestamp message headers in the logfile when
# turned on. Note that the parameter 'debug_timestamp' must be on for this to
# have an effect. Default is 'no'.
      debug_pid: 'no'
# With this option enabled, the timestamp message header is prefixed to the
# debug message without the filename and function information that is included
# with the debug timestamp parameter. This gives timestamps to the messages
# without adding an additional line. Note that this parameter overrides the
# 'debug_timestamp' parameter.
      debug_prefix_timestamp: 'no'
# Samba is sometimes run as root and sometime run as the connected user, this
# boolean parameter inserts the current euid, egid, uid and gid to the timestamp
# message headers in the log file if turned on. Note that the parameter debug
# timestamp must be on for this to have an effect. Default is 'no'.
      debug_uid: 'no'
# Specifies the absolute path to the kerberos keytab file when kerberos method
# is set to "dedicated_keytab". Default is None.
      dedicated_keytab_file: '/usr/local/etc/krb5.keytab'
# This parameter specifies the name of a service which will be connected to if
# the service actually requested cannot be found. Note that the square brackets
# are NOT given in the parameter value. There is no default value for this
# parameter. If this parameter is not given, attempting to connect to a
# nonexistent service results in an error. Typically the default service would
# be a guest ok, read-only service. Also note that the apparent service name
# will be changed to equal that of the requested service, this is very useful
# as it allows you to use macros like %S to make a wildcard service. Note also
# that any "_" characters in the name of the service used in the default
# service will get mapped to a "/". This allows for interesting things.
      default_service: ''
# Windows allows specifying how a file will be shared with other processes when
# it is opened. Sharing violations occur when a file is opened by a different
# process using options that violate the share settings specified by other
# processes. This parameter causes smbd to act as a Windows server does, and
# defer returning a "sharing violation" error message for up to one second,
# allowing the client to close the file causing the violation in the meantime.
# UNIX by default does not have this behaviour. There should be no reason to
# turn off this parameter, as it is designed to enable Samba to more correctly
# emulate Windows. Default is 'yes'.
      defer_sharing_violations: 'yes'
# This is the full pathname to a script that will be run AS ROOT by smbd when a
# group is requested to be deleted. It will expand any %g to the group name
# passed. This script is only useful for installations using the Windows NT
# domain administration tools. Default is None.
      delete_group_script: ''
# With the introduction of MS-RPC based printer support for Windows NT/2000
# clients in Samba 2.2, it is now possible to delete a printer at run time by
# issuing the DeletePrinter() RPC call. For a Samba host this means that the
# printer must be physically deleted from the underlying printing system. The
# deleteprinter command defines a script to be run which will perform the
# necessary operations for removing the printer from the print system and from
# smb.conf. The deleteprinter command is automatically called with only one
# parameter: printer name. Once the deleteprinter command has been executed,
# smbd will reparse the smb.conf to check that the associated printer no longer
# exists. If the sharename is still valid, then smbd will return an
# ACCESS_DENIED error to the client. Default is None.
      delete_printer_command: '/usr/bin/removeprinter'
# Samba 2.2.0 introduced the ability to dynamically add and delete shares via
# the Windows NT 4.0 Server Manager. The delete share command is used to define
# an external program or script which will remove an existing service
# definition from smb.conf. In order to successfully execute the delete share
# command, smbd requires that the administrator connects using a root account
# (i.e. uid == 0) or has the SeDiskOperatorPrivilege. Scripts defined in the
# delete share command parameter are executed as root. When executed, smbd will
# automatically invoke the delete share command with two parameters.
# * configFile - the location of the global smb.conf file.
# * shareName - the name of the existing service.
# This parameter is only used to remove file shares. To delete printer shares,
# see the deleteprinter command. Default is None.
      delete_share_command: '/usr/local/bin/delshare'
# Full path to the script that will be called when a user is removed from a
# group using the Windows NT domain administration tools. It will be run by
# smbd AS ROOT. Any %g will be replaced with the group name and any %u will be
# replaced with the user name. Default is None.
      delete_user_from_group_script: '/usr/sbin/deluser %u %g'
# This is the full pathname to a script that will be run by smbd when managing
# users with remote RPC (NT) tools. This script is called when a remote client
# removes a user from the server, normally using 'User Manager for Domains' or
# rpcclient. This script should delete the given UNIX username. Default is None.
      delete_user_script: '/usr/local/samba/bin/del_user %u'
# Specifies which ports the server should listen on for NetBIOS datagram
# traffic.
      dgram_port: '138'
# Enabling this parameter will disable netbios support in Samba. Netbios is the
# only available form of browsing in all windows versions except for 2000 and
# XP. Clients that only support netbios won't be able to see your samba server
# when netbios support is disabled. Default is 'no'.
      disable_netbios: 'no'
# Enabling this parameter will disable Samba's support for the SPOOLSS set of
# MS-RPC's and will yield identical behavior as Samba 2.0.x. Windows NT/2000
# clients will downgrade to using Lanman style printing commands. Windows 9x/ME
# will be unaffected by the parameter. However, this will also disable the
# ability to upload printer drivers to a Samba server via the Windows NT Add
# Printer Wizard or by using the NT printer properties dialog window. It will
# also disable the capability of Windows NT/2000 clients to download print
# drivers from the Samba host upon demand. Be very careful about enabling this
# parameter. Default is 'no'.
      disable_spoolss: 'no'
# This option specifies the list of DNS servers that DNS requests will be
# forwarded to if they can not be handled by Samba itself. The DNS forwarder is
# only used if the internal DNS server in Samba is used. Default is None.
      dns_forwarder: '100.100.100.1'
# Specifies that nmbd when acting as a WINS server and finding that a NetBIOS
# name has not been registered, should treat the NetBIOS name word-for-word as
# a DNS name and do a lookup with the DNS server for that name on behalf of the
# name-querying client. Note that the maximum length for a NetBIOS name is 15
# characters, so the DNS name (or DNS alias) can likewise only be 15 characters,
# maximum. nmbd spawns a second copy of itself to do the DNS name lookup
# requests, as doing a name lookup is a blocking action. Default is 'yes'.
      dns_proxy: 'yes'
# This option sets the command that is called when there are DNS updates. It
# should update the local machines DNS names using TSIG-GSS.
      dns_update_command: '/usr/local/sbin/dnsupdate'
# When enabled (the default is disabled) unused dynamic dns records are
# periodically removed. Warning, this option should not be enabled for
# installations created with versions of samba before 4.9. Doing this will
# result in the loss of static DNS entries. This is due to a bug in previous
# versions of samba (BUG 12451) which marked dynamic DNS records as static and
# static records as dynamic. If one record for a DNS name is static (non-aging)
# then no other record for that DNS name will be scavenged.
      dns_zone_scavenging: 'no'
# If set to 'yes', the Samba server will provide the netlogon service for
# Windows 9X network logons for the workgroup it is in. This will also cause
# the Samba server to act as a domain controller for NT4 style domain services.
# Default is 'no'.
      domain_logons: 'no'
# Tell smbd to enable WAN-wide browse list collation. Setting this option
# causes nmbd to claim a special domain specific NetBIOS name that identifies
# it as a domain master browser for its given workgroup. Local master browsers
# in the same workgroup on broadcast-isolated subnets will give this nmbd their
# local browse lists, and then ask smbd for a complete copy of the browse list
# for the whole wide area network. Browser clients will then contact their
# local master browser, and will receive the domain-wide browse list, instead
# of just the list for their broadcast-isolated subnet. Note that Windows NT
# Primary Domain Controllers expect to be able to claim this workgroup specific
# special NetBIOS name that identifies them as domain master browsers for that
# workgroup by default (i.e. there is no way to prevent a Windows NT PDC from
# attempting to do this). This means that if this parameter is set and nmbd
# claims the special name for a workgroup before a Windows NT PDC is able to do
# so then cross subnet browsing will behave strangely and may fail. If
# 'domain_logons' is 'yes', then the default behavior is to enable the domain
# master parameter. If domain logons is not enabled (the default setting),
# then neither will domain master be enabled by default. When 'domain_logons' is
# 'yes' the default setting for this parameter is 'yes', with the result that
# Samba will be a PDC. If 'domain_master' is 'no', Samba will function as a BDC.
# In general, this parameter should be set to 'no' only on a BDC.
      domain_master: 'auto'
# DOS SMB clients assume the server has the same charset as they do. This
# option specifies which charset Samba should talk to DOS clients. The default
# depends on which charsets you have installed. Samba tries to use charset 850
# but falls back to ASCII in case it is not available. Run "testparm" to check
# the default on your system.
      dos_charset: ''
# When enabled, this option causes Samba (acting as an Active Directory Domain
# Controller) to stream Samba database events across the internal message bus.
# Scripts built using Samba's python bindings can listen to these events by
# registering as the service dsdb_event. This should be considered a developer
# option (it assists in the Samba testsuite) rather than a facility for
# external auditing, as message delivery is not guaranteed (a feature that the
# testsuite works around). The Samba database events are also logged via the
# normal logging methods when the log level is set appropriately.
# Default is 'no'.
      dsdb_event_notification: 'no'
# When enabled, this option causes Samba (acting as an Active Directory Domain
# Controller) to stream group membership change events across the internal
# message bus. Scripts built using Samba's python bindings can listen to these
# events by registering as the service dsdb_group_event. This should be
# considered a developer option (it assists in the Samba testsuite) rather than
# a facility for external auditing, as message delivery is not guaranteed (a
# feature that the testsuite works around). The group events are also logged
# via the normal logging methods when the log level is set appropriately.
# Default is 'no'.
      dsdb_group_change_notification: 'no'
# When enabled, this option causes Samba (acting as an Active Directory Domain
# Controller) to stream password change and reset events across the internal
# message bus. Scripts built using Samba's python bindings can listen to these
# events by registering as the service password_event. This should be
# considered a developer option (it assists in the Samba testsuite) rather than
# a facility for external auditing, as message delivery is not guaranteed (a
# feature that the testsuite works around). The password events are also logged
# via the normal logging methods when the log level is set appropriately.
# Default is 'no'.
      dsdb_password_event_notification: 'no'
# Hosts running the "Advanced Server for Unix (ASU)" product require some
# special accomodations such as creating a builtin [ADMIN$] share that only
# supports IPC connections. The has been the default behavior in smbd for many
# years. However, certain Microsoft applications such as the Print Migrator tool
# require that the remote server support an [ADMIN$] file share. Disabling this
# parameter allows for creating an [ADMIN$] file share in smb.conf.
# Default is 'no'.
      enable_asu_support: 'no'
# This parameter specifies whether core dumps should be written on internal
# exits. Normally set to 'yes'. You should never need to change this.
      enable_core_files: 'yes'
# This deprecated parameter controls whether or not smbd will honor privileges
# assigned to specific SIDs via either net rpc rights or one of the Windows
# user and group manager tools. This parameter is enabled by default. It can be
# disabled to prevent members of the Domain Admins group from being able to
# assign privileges to users or groups which can then result in certain smbd
# operations running as root that would normally run under the context of the
# connected user. An example of how privileges can be used is to assign the
# right to join clients to a Samba controlled domain without providing root
# access to the server via smbd. Please read the extended description provided
# in the Samba HOWTO documentation.
      enable_privileges: 'yes'
# Inverted synonym for 'disable_spoolss'.
      enable_spoolss: 'yes'
# This option enables a couple of enhancements to cross-subnet browse
# propagation that have been added in Samba but which are not standard in
# Microsoft implementations. The first enhancement to browse propagation
# consists of a regular wildcard query to a Samba WINS server for all Domain
# Master Browsers, followed by a browse synchronization with each of the
# returned DMBs. The second enhancement consists of a regular randomised browse
# synchronization with all currently known DMBs. You may wish to disable this
# option if you have a problem with empty workgroups not disappearing from
# browse lists. Due to the restrictions of the browse protocols, these
# enhancements can cause a empty workgroup to stay around forever which can be
# annoying. In general you should leave this option enabled as it makes
# cross-subnet browse propagation much more reliable. Default is 'yes'.
      enhanced_browsing: 'yes'
# The concept of a "port" is fairly foreign to UNIX hosts. Under Windows
# NT/2000 print servers, a port is associated with a port monitor and generally
# takes the form of a local port (i.e. LPT1:, COM1:, FILE:) or a remote port
# (i.e. LPD Port Monitor, etc...). By default, Samba has only one port
# defined - "Samba Printer Port". Under Windows NT/2000, all printers must have
# a valid port name. If you wish to have a list of ports displayed (smbd does
# not use a port name for anything) other than the default "Samba Printer Port",
# you can define enumports command to point to a program which should generate a
# list of ports, one per line, to standard output. This listing will then be
# used in response to the level 1 and 2 EnumPorts() RPC. Default is None.
      enumports_command: '/usr/bin/listports'
# This option defines a list of log names that Samba will report to the
# Microsoft EventViewer utility. The listed eventlogs will be associated with
# tdb file on disk in the $(statedir)/eventlog. The administrator must use an
# external process to parse the normal Unix logs such as /var/log/messages and
# write then entries to the eventlog tdb files. Default is None.
      eventlog_list:
      - 'Security'
      - 'Application'
      - 'Syslog'
      - 'Apache'
# When enabled, Samba's File Server Remote VSS Protocol (FSRVP) server checks
# all FSRVP initiated snapshots on startup, and removes any corresponding state
# (including share definitions) for nonexistent snapshot paths. Default is 'no'.
      fss_prune_stale: 'no'
# The File Server Remote VSS Protocol (FSRVP) server includes a message
# sequence timer to ensure cleanup on unexpected client disconnect. This
# parameter overrides the default timeout between FSRVP operations.
# FSRVP timeouts can be completely disabled via a value of 0. Default is
# 180 or 1800, depending on operation.
      fss_sequence_timeout: ''
# The get quota command should only be used whenever there is no operating
# system API available from the OS that samba can use. This option is only
# available Samba was compiled with quotas support. This parameter should
# specify the path to a script that queries the quota information for the
# specified user/group for the partition that the specified directory is on.
# Default is None.
      get_quota_command: '/usr/local/sbin/query_quota'
# This option sets the command that is called to apply GPO policies. The
# samba-gpupdate script applies System Access and Kerberos Policies to the KDC.
# System Access policies set minPwdAge, maxPwdAge, minPwdLength, and
# pwdProperties in the samdb. Kerberos Policies set kdc:service ticket lifetime,
# kdc:user ticket lifetime, and kdc:renewal lifetime in smb.conf.
      gpo_update_command: '/usr/local/sbin/gpoupdate'
# This is a username which will be used for access to services which are
# specified as guest ok (see below). Whatever privileges this user has will be
# available to any client connecting to the guest service. This user must exist
# in the password file, but does not require a valid login. The user account
# "ftp" is often a good choice for this parameter. On some systems the default
# guest account "nobody" may not be able to print. Use another account in this
# case. You should test this by trying to log in as your guest user (perhaps by
# using the su - command) and trying to print using the system print command
# such as lpr or lp. This parameter does not accept % macros, because many
# parts of the system require this value to be constant for correct operation.
      guest_account: 'nobody'
# If 'nis_homedir' is 'yes', and smbd is also acting as a Win95/98 logon server
# then this parameter specifies the NIS (or YP) map from which the server for
# the user's home directory should be extracted. At present, only the Sun
# auto.home map format is understood. A working NIS client is required on the
# system for this option to work. Default is None.
      homedir_map: ''
# If set to yes (the default), Samba will act as a Dfs server, and allow
# Dfs-aware clients to browse Dfs trees hosted on the server.
      host_msdfs: 'yes'
# Specifies whether samba should use (expensive) hostname lookups or use the ip
# addresses instead. An example place where hostname lookups are currently used
# is when checking the hosts deny and hosts allow. Default is 'no'.
      hostname_lookups: 'no'
# This parameter specifies the number of seconds that Winbind's idmap interface
# will cache positive SID/uid/gid query results. By default, Samba will cache
# these results for one week.
      idmap_cache_time: '604800'
# ID mapping in Samba is the mapping between Windows SIDs and Unix user and
# group IDs. This is performed by Winbindd with a configurable plugin interface.
# Samba's ID mapping is configured by options. The idmap configuration is hence
# divided into groups, one group for each domain to be configured, and one
# group with the asterisk instead of a proper domain name, which specifies the
# default configuration that is used to catch all domains that do not have an
# explicit idmap configuration of their own.
      idmap_config:
      - domain: 'DOMAIN'
# This specifies the name of the idmap plugin to use as the SID/uid/gid backend
# for this domain. The standard backends are 'tdb', 'tdb2', 'ldap', 'rid',
# 'hash', 'autorid', 'ad' and 'nss'. The first three of these create mappings
# of their own using internal unixid counters and store the mappings in a
# database. These are suitable for use in the default idmap configuration. The
# 'rid' and 'hash' backends use a pure algorithmic calculation to determine the
# unixid for a SID. The 'autorid' module is a mixture of the 'tdb' and 'rid'
# backend. It creates ranges for each domain encountered and then uses the 'rid'
# algorithm for each of these automatically configured domains individually. The
# 'ad' backend uses unix ids stored in Active Directory via the standard schema
# extensions. The 'nss' backend reverses the standard winbindd setup and gets
# the unix ids via names from nsswitch which can be useful in an ldap setup.
        backend: 'autorid'
# Defines the available matching uid and gid range for which the backend is
# authoritative. For allocating backends, this also defines the start and the
# end of the range for allocating new unique IDs. winbind uses this parameter
# to find the backend that is authoritative for a unix ID to SID mapping, so it
# must be set for each individually configured domain and for the default
# configuration. The configured ranges must be mutually disjoint.
        range:
        - low: '1000000'
          high: '19999999'
# This option can be used to turn the writing backends 'tdb', 'tdb2', and 'ldap'
# into read only mode. This can be useful e.g. in cases where a pre-filled
# database exists that should not be extended automatically.
        read_only: 'yes'
# This parameter specifies the number of seconds that Winbind's idmap interface
# will cache negative SID/uid/gid query results. Default is '120'.
      idmap_negative_cache_time: '120'
# Setting this parameter to no will prevent winbind to include the system
# /etc/krb5.conf file into the krb5.conf file it creates. See also
# 'create_krb5_conf'. This option only applies to Samba built with MIT Kerberos.
# Default is 'yes'.
      include_system_krb5_conf: 'yes'
# This parameter specifies a delay in milliseconds for the hosts configured for
# delayed initial samlogon with init logon delayed hosts. Default is '100'.
      init_logon_delay: '100'
# This parameter takes a list of host names, addresses or networks for which
# the initial samlogon reply should be delayed (so other DCs get preferred by
# XP workstations if there are any). The length of the delay can be specified
# with the 'init_logon_delay' parameter. Default is None.
      init_logon_delayed_hosts:
      - '150.203.5.'
      - 'myhost.mynet.de'
# This option allows you to override the default network interfaces list that
# Samba will use for browsing, name registration and other NetBIOS over TCP/IP
# (NBT) traffic. By default Samba will query the kernel for the list of all
# active interfaces and use any interfaces except 127.0.0.1 that are broadcast
# capable. The option takes a list of interface strings. Each string can be in
# any of the following forms:
# * a network interface name (such as 'eth0'). This may include shell-like
# wildcards so 'eth*' will match any interface starting with the substring "eth"
# * an IP address. In this case the netmask is determined from the list of
# interfaces obtained from the kernel
# * an IP/mask pair.
# * a broadcast/mask pair.
# By default Samba enables all active interfaces that are broadcast capable
# except the loopback adaptor (IP address 127.0.0.1). The example below
# configures three network interfaces corresponding to the 'eth0' device and
# IP addresses 192.168.2.10 and 192.168.3.10. The netmasks of the latter two
# interfaces would be set to 255.255.255.0.
      interfaces:
      - 'eth0'
      - '192.168.2.10/24'
      - '192.168.3.10/255.255.255.0'
# This parameter is only applicable if printing is set to iprint. If set, this
# option overrides the ServerName option in the CUPS client.conf. This is
# necessary if you have virtual samba servers that connect to different CUPS
# daemons. Default is empty.
      iprint_server: 'MYCUPSSERVER'
# The value of the parameter (an integer) represents the number of seconds
# between keepalive packets. If this parameter is zero, no keepalive packets
# will be sent. Keepalive packets, if sent, allow the server to tell whether a
# client is still present and responding. Keepalives should, in general, not be
# needed if the socket has the SO_KEEPALIVE attribute set on it by default.
# Basically you should only use this option if you strike difficulties. Please
# note this option only applies to SMB1 client connections, and has no effect
# on SMB2 clients. Default is '300'.
      keepalive: '300'
# This parameter determines the encryption types to use when operating as a
# Kerberos client. Possible values are 'all' (the default), 'strong', and
# 'legacy'. Samba uses a Kerberos library to obtain Kerberos tickets. This
# library is normally configured outside of Samba, using the krb5.conf file.
# This file may also include directives to configure the encryption types to be
# used. However, Samba implements Active Directory protocols and algorithms to
# locate a domain controller. In order to force the Kerberos library into using
# the correct domain controller, some Samba processes, such as winbindd and net,
# build a private krb5.conf file for use by the Kerberos library while being
# invoked from Samba. This private file controls all aspects of the Kerberos
# library operation, and this parameter controls how the encryption types are
# configured within this generated file, and therefore also controls the
# encryption types negotiable by Samba. When set to 'all', all active directory
# encryption types are allowed. When set to 'strong', only AES-based encryption
# types are offered. This can be used in hardened environments to prevent
# downgrade attacks. When set to 'legacy', only RC4-HMAC-MD5 is allowed.
# Avoiding AES this way has one a very specific use. Normally, the encryption
# type is negotiated between the peers. However, there is one scenario in which
# a Windows read-only domain controller (RODC) advertises AES encryption, but
# then proxies the request to a writeable DC which may not support AES
# encryption, leading to failure of the handshake. Setting this parameter to
# legacy would cause samba not to negotiate AES encryption. It is assumed of
# course that the weaker legacy encryption types are acceptable for the setup.
      kerberos_encryption_types: 'all'
# Controls how kerberos tickets are verified. Valid options are:
# 'secrets only' - use only the secrets.tdb for ticket verification (default).
# 'system keytab' - use only the system keytab for ticket verification.
# 'dedicated keytab' - use a dedicated keytab for ticket verification.
# 'secrets and keytab' - use the secrets.tdb first, then the system keytab.
# The major difference between "system keytab" and "dedicated keytab" is that
# the latter method relies on kerberos to find the correct keytab entry instead
# of filtering based on expected principals. When the kerberos method is in
# 'dedicated keytab' mode, dedicated keytab file must be set to specify the
# location of the keytab file.
      kerberos_method: 'default'
# This parameter specifies whether Samba should ask the kernel for change
# notifications in directories so that SMB clients can refresh whenever the
# data on the server changes. This parameter is only used when your kernel
# supports change notification to user programs using the inotify interface.
# Default is 'yes'.
      kernel_change_notify: 'yes'
# Specifies which ports the Kerberos server should listen on for password
# changes.
      kpasswd_port: '464'
# Specifies which port the KDC should listen on for Kerberos traffic.
      krb5_port: '88'
# This parameter determines whether or not smbd will attempt to authenticate
# users or permit password changes using the LANMAN password hash. If disabled,
# only clients which support NT password hashes (e.g. Windows NT/2000 clients,
# smbclient, but not Windows 95/98 or the MS DOS network client) will be able to
# connect to the Samba host. The LANMAN encrypted response is easily broken,
# due to its case-insensitive nature, and the choice of algorithm. Servers
# without Windows 95/98/ME or MS DOS clients are advised to disable this option.
# When this parameter is set to 'no' this will also result in sambaLMPassword
# in Samba's passdb being blanked after the next password change. As a result
# of that lanman clients won't be able to authenticate, even if lanman auth is
# re-enabled later on. Unlike the encrypt passwords option, this parameter
# cannot alter client behaviour, and the LANMAN response will still be sent
# over the network. See the client lanman auth to disable this for Samba's
# clients (such as smbclient). This parameter is overriden by 'ntlm_auth', so
# unless that it is also set to ntlmv1-permitted or yes, then only NTLMv2
# logins will be permited and no LM hash will be stored. All modern clients
# support NTLMv2, and but some older clients require special configuration to
# use it.
      lanman_auth: 'no'
# This parameter determines whether or not smbd supports the new 64k streaming
# read and write variant SMB requests introduced with Windows 2000. Note that
# due to Windows 2000 client redirector bugs this requires Samba to be running
# on a 64-bit capable operating system such as IRIX, Solaris or a Linux 2.4
# kernel. Can improve performance by 10% with Windows 2000 clients.
# Defaults to on.
      large_readwrite: 'yes'
# The ldap admin dn defines the Distinguished Name (DN) name used by Samba to
# contact the ldap server when retreiving user account information. Used in
# conjunction with the 'admin_dn_password' stored in the private/secrets.tdb
# file. Requires a fully specified DN. The ldap suffix is not appended to the
# ldap admin dn. Default is None.
      ldap_admin_dn: ''
# This parameter tells the LDAP library calls which timeout in seconds they
# should honor during initial connection establishments to LDAP servers. It is
# very useful in failover scenarios in particular. If one or more LDAP servers
# are not reachable at all, we do not have to wait until TCP timeouts are over.
# This parameter is different from 'ldap_timeout' which affects operations on
# LDAP servers using an existing connection and not establishing an initial
# connection. Default is '2'.
      ldap_connection_timeout: '2'
# This parameter controls the debug level of the LDAP library calls. In the
# case of OpenLDAP, it is the same bit-field as understood by the server and
# documented in the slapd.conf manpage. A typical useful value will be '1' for
# tracing function calls. The debug output from the LDAP libraries appears with
# the prefix [LDAP] in Samba's logging output. The level at which LDAP logging
# is printed is controlled by the parameter ldap debug threshold.
# Defaults is '0'.
      ldap_debug_level: '0'
# This parameter controls the Samba debug level at which the ldap library debug
# output is printed in the Samba logs.
      ldap_debug_threshold: '10'
# This parameter specifies whether a delete operation in the ldapsam deletes
# the complete entry or only the attributes specific to Samba. Default is 'no'.
      ldap_delete_dn: 'no'
# This option controls whether Samba should tell the LDAP library to use a
# certain alias dereferencing method. The default is 'auto', which means that
# the default setting of the ldap client library will be kept. Other possible
# values are 'never', 'finding', searching and 'always'.
      ldap_deref: 'auto'
# This option controls whether to follow LDAP referrals or not when searching
# for entries in the LDAP database. Possible values are 'on' to enable following
# referrals, 'off' to disable this, and 'auto' (default), to use the libldap
# default settings.
      ldap_follow_referral: 'auto'
# This parameter specifies the suffix that is used for groups when these are
# added to the LDAP directory. If this parameter is unset, the value of
# 'ldap_suffix' will be used instead. The suffix string is pre-pended to the
# ldap suffix string so use a partial DN.
      ldap_group_suffix: 'ou=Groups'
# This parameters specifies the suffix that is used when storing idmap mappings.
# If this parameter is unset, the value of ldap suffix will be used instead. The
# suffix string is pre-pended to the ldap suffix string so use a partial DN.
      ldap_idmap_suffix: 'ou=Idmap'
# It specifies where machines should be added to the ldap tree. If this
# parameter is unset, the value of ldap suffix will be used instead. The suffix
# string is pre-pended to the ldap suffix string so use a partial DN.
      ldap_machine_suffix: 'ou=Computers'
# This parameter specifies the number of entries per page. If the LDAP server
# supports paged results, clients can request subsets of search results (pages)
# instead of the entire list. This parameter specifies the size of these pages.
      ldap_page_size: '1000'
# This option is used to define whether or not Samba should sync the LDAP
# password with the NT and LM hashes for normal accounts (NOT for workstation,
# server or domain trusts) on a password change via SAMBA:
# 'yes' - try to update the LDAP, NT and LM passwords and update the pwdLastSet
# time.
# 'no' - update NT and LM passwords and update the pwdLastSet time (default).
# 'only' = Only update the LDAP password and let the LDAP server do the rest.
      ldap_passwd_sync: 'no'
# When Samba is asked to write to a read-only LDAP replica, we are redirected
# to talk to the read-write master server. This server then replicates our
# changes back to the 'local' server, however the replication might take some
# seconds, especially over slow links. Certain client activities, particularly
# domain joins, can become confused by the 'success' that does not immediately
# change the LDAP back-end's data. This option simply causes Samba to wait a
# short time, to allow the LDAP server to catch up. If you have a particularly
# high-latency network, you may wish to time the LDAP replication with a network
# sniffer, and increase this value accordingly. Be aware that no checking is
# performed that the data has actually replicated. The value is specified in
# milliseconds, the maximum value is 5000 (5 seconds).
      ldap_replication_sleep: '1000'
# Editposix is an option that leverages ldapsam:trusted to make it simpler to
# manage a domain controller eliminating the need to set up custom scripts to
# add and manage the posix users and groups. This option will instead directly
# manipulate the ldap tree to create, remove and modify user and group entries.
# This option also requires a running winbindd as it is used to allocate new
# uids/gids on user/group creation. The allocation range must be therefore
# configured. Default is 'no'.
      ldapsam_editposix: 'no'
# By default, Samba as a Domain Controller with an LDAP backend needs to use
# the Unix-style NSS subsystem to access user and group information. Due to the
# way Unix stores user information in '/etc/passwd' and '/etc/group' this
# inevitably leads to inefficiencies. One important question a user needs to
# know is the list of groups he is member of. The plain UNIX model involves a
# complete enumeration of the file '/etc/group' and its NSS counterparts in
# LDAP. UNIX has optimized functions to enumerate group membership. Sadly, other
# functions that are used to deal with user and group attributes lack such
# optimization. To make Samba scale well in large environments, 'yes' option
# assumes that the complete user and group database that is relevant to Samba is
# stored in LDAP with the standard posixAccount/posixGroup attributes. It
# further assumes that the Samba auxiliary object classes are stored together
# with the POSIX data in the same LDAP object. If these assumptions are met,
# 'ldapsam_trusted' = yes can be activated and Samba can bypass the NSS system
# to query user group memberships. Optimized LDAP queries can greatly speed up
# domain logon and administration tasks. Depending on the size of the LDAP
# database a factor of 100 or more for common queries is easily achieved.
      ldapsam_trusted: 'no'
# The ldap server require strong auth defines whether the ldap server requires
# ldap traffic to be signed or signed and encrypted (sealed). Possible values
# are 'no', 'allow_sasl_over_tls' and 'yes'. A value of 'no' allows simple and
# sasl binds over all transports. A value of allow_sasl_over_tls allows simple
# and sasl binds (without sign or seal) over TLS encrypted connections.
# Unencrypted connections only allow sasl binds with sign or seal. A value of
# yes allows only simple binds over TLS encrypted connections. Unencrypted
# connections only allow sasl binds with sign or seal.
      ldap_server_require_strong_auth: 'yes'
# This option is used to define whether or not Samba should use SSL when
# connecting to the ldap server This is NOT related to Samba's previous SSL
# support which was enabled by specifying the '--with-ssl' option to the
# configure script. LDAP connections should be secured where possible. This may
# be done setting either this parameter to start tlsor by specifying ldaps://
# in the URL argument of passdb backend. The ldap ssl can be set to one of two
# values:
# 'off' - never use SSL when querying the directory.
# 'start tls' - use the LDAPv3 StartTLS extended operation (RFC2830) for
# communicating with the directory server. Please note that this parameter does
# only affect rpc methods.
      ldap_ssl: 'start tls'
# This option is used to define whether or not Samba should use SSL when
# connecting to the ldap server using ads methods. RPC methods are not affected
# by this parameter. Please note, that this parameter won't have any effect if
# 'ldap_ssl' is set to no. Default is 'no'.
      ldap_ssl_ads: 'no'
# Specifies the base for all ldap suffixes and for storing the sambaDomain
# object.The ldap suffix will be appended to the values specified for the
# 'ldap_user_suffix', 'ldap_group_suffix', 'ldap_machine_suffix', and the
# 'ldap_idmap_suffix'. Each of these should be given only a DN relative to the
# 'ldap_suffix'. Default is None.
      ldap_suffix: 'dc=samba,dc=org'
# This parameter defines the number of seconds that Samba should use as timeout
# for LDAP operations.
      ldap_timeout: '15'
# This parameter specifies where users are added to the tree. If this parameter
# is unset, the value of ldap suffix will be used instead. The suffix string is
# pre-pended to the ldap suffix string so use a partial DN.
      ldap_user_suffix: 'ou=people'
# This parameter determines if nmbd will produce Lanman announce broadcasts
# that are needed by OS/2 clients in order for them to see the Samba server in
# their browse list. This parameter can have three values, 'yes', 'no', or
# 'auto'. The default is 'auto'. If set to 'no' Samba will never produce these
# broadcasts. If set to 'yes' Samba will produce Lanman announce broadcasts at
# a frequency set by the parameter lm interval. If set to 'auto' Samba will not
# send Lanman announce broadcasts by default but will listen for them. If it
# hears such a broadcast on the wire it will then start sending them at a
# frequency set by the parameter lm interval.
      lm_announce: 'auto'
# If Samba is set to produce Lanman announce broadcasts needed by OS/2 clients
# then this parameter defines the frequency in seconds with which they will be
# made. If this is set to zero then no Lanman announcements will be made despite
# the setting of the 'lm_announce' parameter.
      lm_interval: '60'
# A boolean variable that controls whether all printers in the printcap will be
# loaded for browsing by default. Default is 'yes'.
      load_printers: 'yes'
# This option allows nmbd to try and become a local master browser on a subnet.
# If set to 'no' then nmbd will not attempt to become a local master browser on
# a subnet and will also lose in all browsing elections. By default this value
# is set to 'yes'. Setting this value to 'yes' doesn't mean that Samba will
# become the local master browser on a subnet, just that nmbd will participate
# in elections for local master browser. Setting this value to 'no' will cause
# nmbd never to become a local master browser.
      local_master: 'yes'
# This option specifies the directory where lock files will be placed. The lock
# files are used to implement the max connections option. The files placed in
# this directory are not required across service restarts and can be safely
# placed on volatile storage (e.g. tmpfs in Linux).
      lock_directory: '/var/lib/samba/lock'
# The time in milliseconds that smbd should keep waiting to see if a failed
# lock request can be granted. You should not need to change the value of this
# parameter. Default is '200'.
      lock_spin_time: '200'
# This option allows you to override the name of the Samba log file. This
# option takes the standard substitutions, allowing you to have separate log
# files for each user or machine. Default is None.
      log_file: '/usr/local/samba/var/log.%m'
# This parameter configures logging backends. Multiple backends can be
# specified at the same time, with different log levels for each backend.
      logging:
      - backend: 'syslog'
        log_level: '1'
      - backend: 'file'
# The value of the parameter allows the debug level (logging level). It allows
# one to specify the debug level for multiple debug classes and distinct
# logfiles for debug classes.
      log_level: '1 full_audit:1@/var/log/audit.log winbind:2'
# This option can be set to a command that will be called when new nt tokens
# are created. This is only useful for development purposes. Default is None.
      log_nt_token_command: ''
# This parameter specifies the local path to which the home directory will be
# connected (see logon home) and is only used by NT Workstations. Note that
# this option is only useful if Samba is set up as a logon server.
# Default is None.
      logon_drive: 'h:'
# This parameter specifies the home directory location when a Win95/98 or NT
# Workstation logs into a Samba PDC. It allows you to do C:\>NET USE H: /HOME
# from a command prompt, for example. This option takes the standard
# substitutions, allowing you to have separate logon scripts for each user or
# machine. This parameter can be used with Win9X workstations to ensure that
# roaming profiles are stored in a subdirectory of the user's home directory.
# This is done in the following way: '\\%N\%U\profile'. This tells Samba to
# return the above string, with substitutions made when a client requests the
# info, generally in a NetUserGetInfo request. Win9X clients truncate the info
# to "\\server\share" when a user does net use '/home' but use the whole string
# when dealing with profiles. Note that in prior versions of Samba, the logon
# path was returned rather than logon home. This broke net use /home but
# allowed profiles outside the home directory. The current implementation is
# correct, and can be used for profiles if you use the above trick. This option
# is only useful if Samba is set up as a logon server.
      logon_home: '\\%N\%U'
# This parameter specifies the directory where roaming profiles (Desktop,
# NTuser.dat, etc) are stored. Contrary to previous versions of these manual
# pages, it has nothing to do with Win 9X roaming profiles. To find out how to
# handle roaming profiles for Win 9X system, see the logon home parameter. This
# option takes the standard substitutions, allowing you to have separate logon
# scripts for each user or machine. It also specifies the directory from which
# the "Application Data", desktop, start menu, network neighborhood, programs
# and other folders, and their contents, are loaded and displayed on your
# Windows NT client. The share and the path must be readable by the user for
# the preferences and directories to be loaded onto the Windows NT client. The
# share must be writeable when the user logs in for the first time, in order
# that the Windows NT client can create the NTuser.dat and other directories.
# Thereafter, the directories and any of the contents can, if required, be made
# read-only. It is not advisable that the NTuser.dat file be made read-only -
# rename it to NTuser.man to achieve the desired effect (a MANdatory profile).
# Windows clients can sometimes maintain a connection to the [homes] share,
# even though there is no user logged in. Therefore, it is vital that the logon
# path does not include a reference to the homes share (i.e. setting this
# parameter to '\\%N\homes\profile_path' will cause problems). This option
# takes the standard substitutions, allowing you to have separate logon scripts
# for each user or machine. Warning: do not quote the value. Setting this as
# '"\\%N\profile\%U"' will break profile handling. Where the tdbsam or ldapsam
# passdb backend is used, at the time the user account is created the value
# configured for this parameter is written to the passdb backend and that value
# will over-ride the parameter value present in the smb.conf file. Any error
# present in the passdb backend account record must be editted using the
# appropriate tool (pdbedit on the command-line, or any other locally provided
# system tool). Note that this option is only useful if Samba is set up as a
# domain controller. Disable the use of roaming profiles by setting the value
# of this parameter to the empty string. For example, logon path = "". Take
# note that even if the default setting in the smb.conf file is the empty
# string, any value specified in the user account settings in the passdb
# backend will over-ride the effect of setting this parameter to null.
# Disabling of all roaming profile use requires that the user account settings
# must also be blank.
      logon_path: '\\%N\%U\profile'
# This parameter specifies the batch file (.bat) or NT command file (.cmd) to
# be downloaded and run on a machine when a user successfully logs in. The file
# must contain the DOS style CR/LF line endings. Using a DOS-style editor to
# create the file is recommended. Note that it is particularly important not to
# allow write access to the [netlogon] share, or to grant users write permission
# on the batch files in a secure environment, as this would allow the batch
# files to be arbitrarily modified and security to be breached. This option
# takes the standard substitutions, allowing you to have separate logon scripts
# for each user or machine. This option is only useful if Samba is set up as a
# logon server in a classic domain controller role. If Samba is set up as an
# Active Directory domain controller, LDAP attribute scriptPath is used instead.
# Default is None.
      logon_script: 'scripts\%U.bat'
# When the network connection between a CIFS client and Samba dies, Samba has
# no option but to simply shut down the server side of the network connection.
# If this happens, there is a risk of data corruption because the Windows
# client did not complete all write operations that the Windows application
# requested. Setting this option to 'yes' makes smbd log with a level 0 message
# a list of all files that have been opened for writing when the network
# connection died. Those are the files that are potentially corrupted. It is
# meant as an aid for the administrator to give him a list of files to do
# consistency checks on. Default is 'no'.
      log_writeable_files_on_exit: 'no'
# This controls how long lpq info will be cached for to prevent the lpq command
# being called too often. A separate cache is kept for each variation of the
# lpq command used by the system, so if you use different lpq commands for
# different users then they won't share cache information. The cache files are
# stored in /tmp/lpq.xxxx where xxxx is a hash of the lpq command in use. The
# default is 30 seconds, meaning that the cached results of a previous identical
# lpq command will be used if the cached data is less than 30 seconds old. A
# large value may be advisable if your lpq command is very slow. A value of '0'
# will disable caching completely. Default is '30'.
      lpq_cache_time: '30'
# If a Samba server is a member of a Windows NT or Active Directory Domain then
# periodically a running winbindd process will try and change the
# "MACHINE ACCOUNT PASSWORD" stored in the TDB called secrets.tdb. This
# parameter specifies how often this password will be changed, in seconds. The
# default is one week (expressed in seconds), the same as a Windows NT Domain
# member server.
      machine_password_timeout: '604800'
# Controls the number of prefix characters from the original name used when
# generating the mangled names. A larger value will give a weaker hash and
# therefore more name collisions. The minimum value is '1' (the default) and
# the maximum value is '6'. 'mangle_prefix' is effective only when mangling
# method is hash2.
      mangle_prefix: '1'
# Controls the algorithm used for the generating the mangled names. Can take
# two different values, 'hash' and 'hash2' (the default). 'hash' is the
# algorithm that was used in Samba for many years. Now the default is 'hash2' is
# considered a better algorithm (generates less collisions) in the names. Many
# Win32 applications store the mangled names and so changing to algorithms must
# not be done lightly as these applications may break unless reinstalled.
      mangling_method: 'hash2'
# This parameter can take four different values, which tell smbd what to do
# with user login requests that don't match a valid UNIX user in some way.
# The four settings are:
# 'Never' - means user login requests with an invalid password are rejected.
# This is the default.
# 'Bad User' - means user logins with an invalid password are rejected, unless
# the username does not exist, in which case it is treated as a guest login and
# mapped into the guest account.
# 'Bad Password' - means user logins with an invalid password are treated as a
# guest login and mapped into the guest account. Note that this can cause
# problems as it means that any user incorrectly typing their password will be
# silently logged on as "guest" - and will not know the reason they cannot
# access files they think they should - there will have been no message given
# to them that they got their password wrong. Helpdesk services will hate you
# if you set the map to guest parameter this way.
# 'Bad Uid' - is only applicable when Samba is configured in some type of
# domain mode security (security = {domain|ads}) and means that user logins
# which are successfully authenticated but which have no valid Unix user
# account (and smbd is unable to create one) should be mapped to the defined
# guest account. This was the default behavior of Samba 2.x releases. Note that
# if a member server is running winbindd, this option should never be required
# because the nss_winbind library will export the Windows domain users and
# groups to the underlying OS via the Name Service Switch interface. Note that
# this parameter is needed to set up "Guest" share services. This is because in
# these modes the name of the resource being requested is not sent to the
# server until after the server has successfully authenticated the client so
# the server cannot make authentication decisions at the correct time
# (connection to the share) for "Guest" shares.
      map_to_guest: 'Never'
# This option allows you to put an upper limit on the apparent size of disks.
# If you set this option to '100' then all shares will appear to be not larger
# than 100 MB in size. Note that this option does not limit the amount of data
# you can put on the disk. In the above case you could still store much more
# than 100 MB on the disk, but if a client ever asks for the amount of free
# disk space or the total disk size then the result will be bounded by the
# amount specified in max disk size. This option is primarily useful to work
# around bugs in some pieces of software that can't handle very large disks,
# particularly disks over 1GB in size.
      max_disk_size: '0'
# This option (an integer in kilobytes) specifies the max size the log file
# should grow to. Samba periodically checks the size and if it is exceeded it
# will rename the file, adding a ".old" extension. A size of 0 means no limit.
# Default is 5000.
      max_log_size: '5000'
# This option controls the maximum number of outstanding simultaneous SMB
# operations that Samba tells the client it will allow. You should never need
# to set this parameter. Default is '50'.
      max_mux: '50'
# This parameter limits the maximum number of open files that one smbd file
# serving process may have open for a client at any one time. This parameter
# can be set very high (16384) as Samba uses only one bit per unopened file.
# Setting this parameter lower than 16384 will cause Samba to complain and set
# this value back to the minimum of 16384, as Windows 7 depends on this number
# of open file handles being available. The limit of the number of open files
# is usually set by the UNIX per-process file descriptor limit rather than this
# parameter so you should never need to touch this parameter.
      max_open_files: '16384'
# This parameter limits the maximum number of smbd processes concurrently
# running on a system and is intended as a stopgap to prevent degrading service
# to clients in the event that the server has insufficient resources to handle
# more than this number of connections. Remember that under normal operating
# conditions, each user will have an smbd associated with him or her to handle
# connections to all shares from a given host. For a Samba ADDC running the
# standard process model this option limits the number of processes forked to
# handle requests. Currently new processes are only forked for ldap and
# netlogon requests.
      max_smbd_processes: '0'
# This parameter limits the size in memory of any stat cache being used to
# speed up case insensitive name mappings. It represents the number of
# kilobyte (1024) units the stat cache can use. A value of zero, meaning
# unlimited, is not advisable due to increased memory usage. You should not
# need to change this parameter. Default is '512'.
      max_stat_cache_size: '512'
# This option tells nmbd what the default 'time to live' of NetBIOS names
# should be (in seconds) when nmbd is requesting a name using either a
# broadcast packet or from a WINS server. You should never need to change this
# parameter. The default is 3 days.
      max_ttl: '259200'
# This option tells smbd when acting as a WINS server what the maximum time to
# live of NetBIOS names that nmbd will grant will be (in seconds). You should
# never need to change this parameter. The default is 6 days (518400 seconds).
      max_wins_ttl: '518400'
# This option controls the maximum packet size that will be negotiated by
# Samba's smbd for the SMB1 protocol. The default is 16644, which matches the
# behavior of Windows 2000. A value below 2048 is likely to cause problems. You
# should never need to change this parameter from its default value.
      max_xmit: '16644'
# This parameter controls the name that multicast DNS support advertises as
# its' hostname. The default is to use the NETBIOS name which is typically the
# hostname in all capital letters. A setting of mdns will defer the hostname
# configuration to the MDNS library that is used.
      mdns_name: 'netbios'
# This specifies what command to run when the server receives a WinPopup style
# message. This would normally be a command that would deliver the message
# somehow. How this is to be done is up to your imagination.
      message_command: "csh -c 'xedit %s; rm %s' &"
# This option changes the behavior of smbd when processing SMBwriteX calls. Any
# incoming SMBwriteX call on a non-signed SMB/CIFS connection greater than this
# value will not be processed in the normal way but will be passed to any
# underlying kernel recvfile or splice system call (if there is no such call
# Samba will emulate in user space). This allows zero-copy writes directly from
# network socket buffers into the filesystem buffer cache, if available. It may
# improve performance but user testing is recommended. If set to zero Samba
# processes SMBwriteX calls in the normal way. To enable POSIX large write
# support (SMB/CIFS writes up to 16Mb) this option must be nonzero. The maximum
# value is 128k. Values greater than 128k will be silently set to 128k. Note
# this option will have NO EFFECT if set on a SMB signed connection. The default
# is zero, which disables this option.
      min_receivefile_size: '0'
# This option tells nmbd when acting as a WINS server what the minimum time to
# live of NetBIOS names that nmbd will grant will be (in seconds). You should
# never need to change this parameter. The default is 6 hours (21600 seconds).
      min_wins_ttl: '21600'
# This option specifies the path to the MIT kdc binary. If the KDC is not
# installed in the default location and wasn't correctly detected during build
# then you should modify this variable and point it to the correct binary.
      mit_kdc_command: '/usr/sbin/krb5kdc'
# If compiled with proper support for it, Samba will announce itself with
# multicast DNS services like for example provided by the Avahi daemon. This
# parameter allows disabling Samba to register itself. Default is 'yes'.
      multicast_dns_register: 'yes'
# Specifies the number of seconds it takes before entries in samba's hostname
# resolve cache time out. If the timeout is set to 0. the caching is disabled.
# Default is '660'.
      name_cache_timeout: '660'
# This option is used by the programs in the Samba suite to determine what
# naming services to use and in what order to resolve host names to IP
# addresses. Its main purpose to is to control how netbios name resolution is
# performed. The option takes a space separated string of name resolution
# options. They cause names to be resolved as follows:
# 'lmhosts' - lookup an IP address in the Samba lmhosts file. If the line in
# lmhosts has no name type attached to the NetBIOS name then any name type
# matches for lookup.
# 'host' - do a standard host name to IP address resolution, using the system
# /etc/hosts, NIS, or DNS lookups. This method of name resolution is operating
# system depended for instance on IRIX or Solaris this may be controlled by the
# /etc/nsswitch.conf file. Note that this method is used only if the NetBIOS
# name type being queried is the 0x20 (server) name type or 0x1c
# (domain controllers). The latter case is only useful for active directory
# domains and results in a DNS query for the SRV RR entry matching
# '_ldap._tcp.domain'.
# 'wins' - query a name with the IP address listed in the WINSSERVER parameter.
# If no WINS server has been specified this method will be ignored.
# 'bcast' - do a broadcast on each of the known local interfaces listed in the
# interfaces parameter. This is the least reliable of the name resolution
# methods as it depends on the target host being on a locally connected subnet.
# The example below will cause the local lmhosts file to be examined first,
# followed by a broadcast attempt, followed by a normal system hostname lookup.
# When Samba is functioning in ADS security mode it is advised to use following
# settings for name resolve order: 'wins bcast'. DC lookups will still be done
# via DNS, but fallbacks to netbios names will not inundate your DNS servers
# with needless querys for DOMAIN<0x1c> lookups.
      name_resolve_order:
      - 'lmhosts'
      - 'wins'
      - 'host'
      - 'bcast'
# Specifies which port the server should use for NetBIOS over IP name services
# traffic.
      nbt_port: '137'
# This directory will hold a series of named pipes to allow RPC over
# inter-process communication. This will allow Samba and other unix processes
# to interact over DCE/RPC without using TCP/IP. Additionally a sub-directory
# 'np' has restricted permissions, and allows a trusted communication channel
# between Samba processes.
      ncalrpc_dir: '/run/samba/ncalrpc'
# This is a list of NetBIOS names that nmbd will advertise as additional names
# by which the Samba server is known. This allows one machine to appear in
# browse lists under multiple names. If a machine is acting as a browse server
# or logon server none of these names will be advertised as either browse
# server or logon servers, only the primary name of the machine will be
# advertised with these capabilities. Default is None.
      netbios_aliases: ''
# This sets the NetBIOS name by which a Samba server is known. By default it is
# the same as the first component of the host's DNS name. If a machine is a
# browse server or logon server this name (or the first component of the hosts
# DNS name) will be the name that these services are advertised under.
# Note that the maximum length for a NetBIOS name is 15 characters.
# There is a bug in Samba that breaks operation of browsing and access to
# shares if the netbios name is set to the literal name PIPE. To avoid this
# problem, do not name your Samba server PIPE.
      netbios_name: ''
# This sets the NetBIOS scope that Samba will operate under. This should not be
# set unless every machine on your LAN also sets this value.
      netbios_scope: ''
# This option controls whether winbindd sends the
# NETLOGON_NEG_NEUTRALIZE_NT4_EMULATION flag in order to bypass the NT4
# emulation of a domain controller. Typically you should not need set this. It
# can be useful for upgrades from NT4 to AD domains. Default is 'no'.
      neutralize_nt4_emulation: 'no'
# Get the home share server from a NIS map. For UNIX systems that use an
# automounter, the user's home directory will often be mounted on a workstation
# on demand from a remote server. When the Samba logon server is not the actual
# home directory server, but is mounting the home directories via NFS then two
# network hops would be required to access the users home directory if the
# logon server told the client to use itself as the SMB server for home
# directories (one over SMB and one over NFS). This can be very slow. This
# option allows Samba to return the home share as being on a different server
# to the logon server and as long as a Samba daemon is running on the home
# directory server, it will be mounted on the Samba client directly from the
# directory server. When Samba is returning the home share to the client, it
# will consult the NIS map specified in homedir map and return the server listed
# there. Note that for this option to work there must be a working NIS system
# and the Samba server with this option must also be a logon server.
      nis_homedir: 'no'
# This option (default is 'yes') causes nmbd to explicitly bind to the broadcast
# address of the local subnets. This is needed to make nmbd work correctly in
# combination with the socket address option. You should not need to unset this
# option.
      nmbd_bind_explicit_broadcast: ''
# This option sets the path to the nsupdate command which is used for GSS-TSIG
# dynamic DNS updates.
      ns_update_command: '/usr/bin/nsupdate -g'
# This parameter determines whether or not smbd will attempt to authenticate
# users using the NTLM encrypted password response for this local passdb
# (SAM or account database). If disabled, both NTLM and LanMan authencication
# against the local passdb is disabled. Note that these settings apply only to
# local users, authentication will still be forwarded to and NTLM
# authentication accepted against any domain we are joined to, and any trusted
# domain, even if disabled or if NTLMv2-only is enforced here. To control NTLM
# authentiation for domain users, this must option must be configured on each
# DC. By default option is set to 'ntlmv2-only' only NTLMv2 logins will be
# permited. All modern clients support NTLMv2 by default, but some older
# clients will require special configuration to use it. The primary user of
# NTLMv1 is MSCHAPv2 for VPNs and 802.1x.
# The available settings are:
# 'ntlmv1-permitted' (alias 'yes') - allow NTLMv1 and above for all clients.
# 'ntlmv2-only' (alias 'no', the default) - Do not allow NTLMv1 to be used, but
# permit NTLMv2.
# 'mschapv2-and-ntlmv2-only' - Only allow NTLMv1 when the client promises that
# it is providing MSCHAPv2 authentication (such as the ntlm_auth tool).
# 'disabled' - do not accept NTLM (or LanMan) authentication of any level, nor
# permit NTLM password changes.
      ntlm_auth: 'ntlmv2-only'
# This boolean parameter controls whether smbd will allow Windows NT clients to
# connect to the NT SMB specific IPC$ pipes. This is a developer debugging
# option and can be left alone. Default is 'yes'.
      nt_pipe_support: 'yes'
# This setting controls the location of the socket that the NTP daemon uses to
# communicate with Samba for signing packets.
# Default is '/var/lib/samba/ntp_signd'
      ntp_signd_socket_directory: '/var/lib/samba/ntp_signd'
# This boolean parameter controls whether smbd will negotiate NT specific
# status support with Windows NT/2k/XP clients. This is a developer debugging
# option and should be left alone. If this option is set to 'no' then Samba
# offers exactly the same DOS error codes that versions prior to Samba 2.2.3
# reported. You should not need to ever disable this parameter.
# Default is 'yes'.
      nt_status_support: 'yes'
# Allow or disallow client access to accounts that have null passwords. Default
# is 'no'.
      null_passwords: 'no'
# When Samba is configured to enable PAM support (i.e. --with-pam), this
# parameter will control whether or not Samba should obey PAM's account and
# session management directives. The default behavior is to use PAM for clear
# text authentication only and to ignore any account or session management.
# Note that Samba always ignores PAM for authentication in the case of
# "encrypt_passwords: 'yes'". The reason is that PAM modules cannot support the
# challenge/response authentication mechanism needed in the presence of SMB
# password encryption.
      obey_pam_restrictions: 'no'
# Number of minutes to permit an NTLM login after a password change or reset
# using the old password. This allows the user to re-cache the new password on
# multiple clients without disrupting a network reconnection in the meantime.
# This parameter only applies when server role is set to Active Directory
# Domain Controller. Default is '60'.
      old_password_allowed_period: '60'
# This is a tuning parameter added due to bugs in both Windows 9x and WinNT. If
# Samba responds to a client too quickly when that client issues an SMB that
# can cause an oplock break request, then the network client can fail and not
# respond to the break request. This tuning parameter (which is set in
# milliseconds) is the amount of time Samba will wait before sending an oplock
# break request to such (broken) clients. DO NOT CHANGE THIS PARAMETER UNLESS
# YOU HAVE READ AND UNDERSTOOD THE SAMBA OPLOCK CODE.
      oplock_break_wait_time: '0'
# The parameter is used to define the absolute path to a file containing a
# mapping of Windows NT printer driver names to OS/2 printer driver names.
# Default is None.
      os2_driver_map: ''
# This integer value controls what level Samba advertises itself as for browse
# elections. The value of this parameter determines whether nmbd has a chance
# of becoming a local master browser for the workgroup in the local broadcast
# area. Note: by default, Samba will win a local master browsing election over
# all Microsoft operating systems except a Windows NT 4.0/2000 Domain
# Controller. This means that a misconfigured Samba host can effectively
# isolate a subnet for browsing purposes. This parameter is largely
# auto-configured in the Samba-3 release series and it is seldom necessary to
# manually override the default setting. Please refer to the chapter on Network
# Browsing in the Samba-3 HOWTO document for further information regarding the
# use of this parameter.  Note: The maximum value for this parameter is '255'.
# If you use higher values, counting will start at 0! Default is '20'.
      os_level: '20'
# It is possible to use PAM's password change control flag for Samba. If
# enabled, then PAM will be used for password changes when requested by an SMB
# client instead of the program listed in passwd program. It should be possible
# to enable this without changing your passwd chat parameter for most setups.
# Default is 'no'.
      pam_password_change: 'no'
# This is a Samba developer option that allows a system command to be called
# when either smbd or nmbd crashes. This is usually used to draw attention to
# the fact that a problem occurred. Default is None.
      panic_action: ''
# This option allows the administrator to chose which backend will be used for
# storing user and possibly group information. This allows you to swap between
# different storage mechanisms without recompile. Available are:
# 'smbpasswd' - the old plaintext passdb backend. Some Samba features will not
# work if this passdb backend is used. Takes a path to the smbpasswd file as an
# optional argument.
# 'tdbsam' - the TDB based password storage backend. Takes a path to the TDB as
# an optional argument (defaults to passdb.tdb in the private dir directory).
# 'ldapsam' - the LDAP based passdb backend. Takes an LDAP URL as an optional
# argument (defaults to ldap://localhost). LDAP connections should be secured
# where possible. This may be done using either Start-TLS (see ldap ssl) or by
# specifying ldaps:// in the URL argument. Multiple servers may also be
# specified in double-quotes. Whether multiple servers are supported or not and
# the exact syntax depends on the LDAP library you use.
# Examples:
# - tdbsam:/etc/samba/private/passdb.tdb
# - ldapsam:"ldap://ldap-1.example.com ldap://ldap-2.example.com"
      passdb_backend:
      - backend: 'ldapsam'
        location: 'ldap://ldap-1.example.com ldap://ldap-2.example.com'
# This parameter controls whether Samba substitutes %-macros in the passdb
# fields if they are explicitly set. We used to expand macros here, but this
# turned out to be a bug because the Windows client can expand a variable
# "%G_osver%" in which '%G' would have been substituted by the user's primary
# group.
      passdb_expand_explicit: 'no'
# This string controls the "chat" conversation that takes places between smbd
# and the local password changing program to change the user's password. The
# string describes a sequence of response-receive pairs that smbd uses to
# determine what to send to the passwd program and what to expect back. If the
# expected output is not received then the password is not changed. This chat
# sequence is often quite site specific, depending on what local methods are
# used for password control (such as NIS etc). Note that this parameter only is
# used if the unix password sync parameter is set to yes. This sequence is then
# called AS ROOT when the SMB password in the smbpasswd file is being changed,
# without access to the old password cleartext. This means that root must be
# able to reset the user's password without knowing the text of the previous
# password. In the presence of NIS/YP, this means that the passwd program must
# be executed on the NIS master. The string can contain the macro '%n' which is
# substituted for the new password. The old passsword ('%o') is only available
# when encrypt passwords has been disabled. The chat sequence can also
# contain the standard macros '\n', '\r', '\t' and '\s' to give line-feed,
# carriage-return, tab and space. The chat sequence string can also contain a
# '*' which matches any sequence of characters. Double quotes can be used to
# collect strings with spaces in them into a single string. If the send string
# in any part of the chat sequence is a full stop ".", then no string is sent.
# Similarly, if the expect string is a full stop then no string is expected.
# If the 'pam_password_change' parameter is set to 'yes', the chat pairs may be
# matched in any order, and success is determined by the PAM result, not any
# particular output. The '\n' macro is ignored for PAM conversions.
      passwd_chat: '*new*password* %n\n *new*password* %n\n *changed*'
# This boolean specifies if the passwd chat script parameter is run in debug
# mode. In this mode the strings passed to and received from the passwd chat
# are printed in the smbd log with a debug level of 100. This is a dangerous
# option as it will allow plaintext passwords to be seen in the smbd log. It is
# available to help Samba admins debug their passwd chat scripts when calling
# the passwd program and should be turned off after this has been done. This
# option has no effect if the 'pam_password_change' parameter is set. This
# parameter is off by default.
      passwd_chat_debug: 'no'
# This integer specifies the number of seconds smbd will wait for an initial
# answer from a passwd chat script being run. Once the initial answer is
# received the subsequent answers must be received in one tenth of this time.
# The default it two seconds.
      passwd_chat_timeout: '2'
# The name of a program that can be used to set UNIX user passwords. Any
# occurrences of '%u' will be replaced with the user name. The user name is
# checked for existence before calling the password changing program.
# Also note that many passwd programs insist in reasonable passwords, such as a
# minimum length, or the inclusion of mixed case chars and digits. This can pose
# a problem as some clients (such as Windows for Workgroups) uppercase the
# password before sending it. Note that if the 'unix_password_sync' parameter
# is set to 'yes' then this program is called AS ROOT before the SMB password
# in the smbpasswd file is changed. If this UNIX password change fails, then
# smbd will fail to change the SMB password also (this is by design).
# If the unix password sync parameter is set this parameter MUST USE ABSOLUTE
# PATHS for ALL programs called, and must be examined for security implications.
# Default is None.
      passwd_program: ''
# If samba is running as an active directory domain controller, it is possible
# to store the cleartext password of accounts in a PGP/OpenGPG encrypted form.
# You can specify one or more recipients by key id or user id. Note that 32bit
# key ids are not allowed, specify at least 64bit.
# The value is stored as 'Primary:SambaGPG' in the supplementalCredentials
# attribute. As password changes can occur on any domain controller, you should
# configure this on each of them. Note that this feature is currently
# available only on Samba domain controllers. This option is only available if
# samba was compiled with gpgme support. You may need to export the GNUPGHOME
# environment variable before starting samba. It is strongly recommended to only
# store the public key in this location. The private key is not used for
# encryption and should be only stored where decryption is required. Being able
# to restore the cleartext password helps, when they need to be imported into
# other authentication systems later (see samba-tool user getpassword) or you
# want to keep the passwords in sync with another system, e.g. an OpenLDAP
# server (see samba-tool user syncpasswords). While this option needs to be
# configured on all domain controllers, the samba-tool user syncpasswords
# command should run on a single domain controller only. Default is None.
      password_hash_gpg_key_ids: ''
# This parameter determines whether or not samba acting as an Active Directory
# Domain Controller will attempt to store additional passwords hash types for
# the user. The values are stored as "Primary:userPassword" in the
# "supplementalCredentials" attribute. The value of this option is a hash type.
# The currently supported hash types are: 'CryptSHA256', 'CryptSHA512'
# Multiple instances of a hash type may be computed and stored. The password
# hashes are calculated using the crypt() call. The number of rounds used to
# compute the hash can be specified by adding ':rounds=xxxx' to the hash type,
# i.e. 'CryptSHA512:rounds=4500' would calculate an SHA512 hash using 4500
# rounds. If not specified the Operating System defaults for crypt() are used.
# As password changes can occur on any domain controller, you should configure
# this on each of them. Note that this feature is currently available only on
# Samba domain controllers. Currently the NT Hash of the password is recorded
# when these hashes are calculated and stored. When retrieving the hashes the
# current value of the NT Hash is checked against the stored NT Hash.
# This detects password changes that have not updated the password hashes. In
# this case samba-tool user will ignore the stored hash values. Being able to
# obtain the hashed password helps, when they need to be imported into other
# authentication systems later (see samba-tool user getpassword) or you want to
# keep the passwords in sync with another system, e.g. an OpenLDAP server (see
# samba-tool user syncpasswords). Default is None.
      password_hash_user_password_schemes:
      - hash: 'CryptSHA512'
      - hash: 'CryptSHA256'
        rounds: '5000'
# By specifying the name of a domain controller with this option, it is possible
# to get Samba to do all its username/password validation using a specific
# remote server. Ideally, this option should not be used, as the default '*'
# indicates to Samba to determine the best DC to contact dynamically, just as
# all other hosts in an AD domain do. This allows the domain to be maintained
# (addition and removal of domain controllers) without modification to the
# smb.conf file. The cryptographic protection on the authenticated RPC calls
# used to verify passwords ensures that this default is safe. It is strongly
# recommended that you use the default of '*', however if in your particular
# environment you have reason to specify a particular DC list, then the list of
# machines in this option must be a list of names or IP addresses of Domain
# controllers for the Domain. If you use the default of '*', or list several
# hosts in the password server option then smbd will try each in turn till it
# finds one that responds. This is useful in case your primary server goes down.
# If the list of servers contains both names/IP's and the '*' character, the
# list is treated as a list of preferred domain controllers, but an auto lookup
# of all remaining DC's will be added to the list as well. Samba will not
# attempt to optimize this list by locating the closest DC. If parameter is a
# name, it is looked up using the parameter name resolve order and so may
# resolved by any method and order described in that parameter.
      password_server: '*'
# This parameter specifies the perfcount backend to be used when monitoring SMB
# operations. Only one perfcount module may be used, and it must implement all
# of the apis contained in the smb_perfcount_handler structure defined in smb.h.
# Default is None.
      perfcount_module: ''
# This option specifies the directory where pid files will be placed.
      pid_directory: '/var/run'
# This boolean parameter controls if nmbd is a preferred master browser for its
# workgroup. If this is set to 'yes', on startup, nmbd will force an election,
# and it will have a slight advantage in winning the election. It is
# recommended that this parameter is used in conjunction with 'domain_master'
# in 'yes', so that nmbd can guarantee becoming a domain master. Use this option
# with caution, because if there are several hosts (whether Samba servers,
# Windows 95 or NT) that are preferred master browsers on the same subnet, they
# will each periodically and continuously attempt to become the local master
# browser. This will result in unnecessary broadcast traffic and reduced
# browsing capabilities. Default is 'auto'.
      preferred_master: 'auto'
# This option specifies the number of seconds added to the delay before a
# prefork master or worker process is restarted. The restart is initially zero,
# the prefork backoff increment is added to the delay on each restart up to the
# value specified by "prefork maximum backoff". Additionally the the backoff
# for an individual service by using 'ldap = 2' to set the backoff increment to
# 2. If the backoff increment is 2 and the maximum backoff is 5. There will be
# a zero second delay for the first restart. A two second delay for the second
# restart. A four second delay for the third and any subsequent restarts.
# Default is '10'.
      prefork_backoff_increment: '10'
# This option controls the number of worker processes that are started for each
# service when prefork process model is enabled. The prefork children are only
# started for those services that support prefork. For processes that don't
# support preforking all requests are handled by a single process for that
# service. This should be set to a small multiple of the number of CPU's
# available on the server. Additionally the number of prefork children can be
# specified for an individual service by using 'ldap = 8' to set the number of
# ldap worker processes. Default is '4'.
      prefork_children: '4'
# This option controls the maximum delay before a failed pre-fork process is
# restarted. Default is '120'.
      prefork_maximum_backoff: '120'
# This is a list of paths to modules that should be loaded into smbd before a
# client connects. This improves the speed of smbd when reacting to new
# connections somewhat. Default is None.
      preload_modules: '/usr/lib/samba/passdb/mysql.so'
# This option specifies the number of seconds before the printing subsystem is
# again asked for the known printers. Setting this parameter to '0' disables
# any rescanning for new or removed printers after the initial startup.
# Default is '750'.
      printcap_cache_time: '750'
# This parameter may be used to override the compiled-in default printcap name
# used by the server (default is '/etc/printcap'). To use the CUPS printing
# interface set this to 'cups'. This should be supplemented by an additional
# setting 'printing' is 'cups' section. On System V systems that use lpstat to
# list available printers you can use printcap name = lpstat to automatically
# obtain lists of available printers. This is the default for systems that
# define SYSV at configure time in Samba (this includes most System V based
# systems). If 'printcap_name' is set to 'lpstat' on these systems then Samba
# will launch "lpstat -v" and attempt to parse the output to obtain a printer
# list. Under AIX the default is '/etc/qconfig'. Samba will assume the file is
# in AIX qconfig format if the string qconfig appears in the printcap filename.
      printcap_name: '/etc/printcap'
# This parameters defines the directory smbd will use for storing such files as
# smbpasswd and secrets.tdb.
      private_dir: '/var/lib/samba/private'
# This parameter determines whether or not smbd will allow SMB1 clients without
# extended security (without SPNEGO) to use NTLMv2 authentication. If this
# option, lanman auth and ntlm auth are all disabled, then only clients with
# SPNEGO support will be permitted. That means NTLMv2 is only supported within
# TLMSSP. Default is 'no'.
      raw_ntlmv2_auth: 'no'
# This is ignored if async smb echo handler is set, because this feature is
# incompatible with raw read SMB requests If enabled, raw reads allow reads of
# 65535 bytes in one packet. This typically provides a major performance
# benefit for some very, very old clients. However, some clients either
# negotiate the allowable block size incorrectly or are incapable of supporting
# larger block sizes, and for these clients you may need to disable raw reads.
# In general this parameter should be viewed as a system tuning tool and left
# severely alone. Default is 'yes'.
      read_raw: 'yes'
# This option specifies the kerberos realm to use. The realm is used as the
# ADS equivalent of the NT4 domain. It is usually set to the DNS name of the
# kerberos server. Default is None.
      realm: ''
# This turns on or off support for share definitions read from registry. Shares
# defined in smb.conf take precedence over shares with the same name defined in
# registry. Note that this parameter defaults to 'no', but it is set to 'yes'
# when 'config_backend' is set to 'registry'.
      registry_shares: 'no'
# This option controls whether the netlogon server (currently only in "active
# directory domain controller" mode), will reject clients which does not
# support NETLOGON_NEG_SUPPORTS_AES. You can set this to 'yes' if all domain
# members support aes. This will prevent downgrade attacks. This option takes
# precedence to the 'allow_nt4_crypto' option. Default is 'no'.
      reject_md5_clients: 'no'
# This option allows you to setup nmbd to periodically announce itself to
# arbitrary IP addresses with an arbitrary workgroup name. This is useful if
# you want your Samba server to appear in a remote workgroup for which the
# normal browse propagation rules don't work. The remote workgroup can be
# anywhere that you can send IP packets to. For example:
# - '192.168.2.255/SERVERS'
# - '192.168.4.255/STAFF'
# this would cause nmbd to announce itself to the two given IP addresses using
# the given workgroup names. If you leave out the workgroup name, then the one
# given in the workgroup parameter is used instead. The IP addresses you choose
# would normally be the broadcast addresses of the remote networks, but can
# also be the IP addresses of known browse masters if your network config is
# that stable. Default is None.
      remote_announce: ''
# This option allows you to setup nmbd to periodically request synchronization
# of browse lists with the master browser of a Samba server that is on a remote
# segment. This option will allow you to gain browse lists for multiple
# workgroups across routed networks. This is done in a manner that does not
# work with any non-Samba servers. This is useful if you want your Samba server
# and all local clients to appear in a remote workgroup for which the normal
# browse propagation rules don't work. The remote workgroup can be anywhere
# that you can send IP packets to. For example:
# - '192.168.2.255'
# - '192.168.4.255'
# This would cause nmbd to request the master browser on the specified subnets
# or addresses to synchronize their browse lists with the local server. The IP
# addresses you choose would normally be the broadcast addresses of the remote
# networks, but can also be the IP addresses of known browse masters if your
# network config is that stable. If a machine IP address is given Samba makes
# NO attempt to validate that the remote machine is available, is listening,
# nor that it is in fact the browse master on its segment. The remote browse
# sync may be used on networks where there is no WINS server, and may be used
# on disjoint networks where each network has its own WINS server.
# Default is None.
      remote_browse_sync: ''
# This is the full pathname to a script that will be run as root by smbd under
# special circumstances described below. When a user with admin authority or
# SeAddUserPrivilege rights renames a user (e.g.: from the NT4 User Manager for
# Domains), this script will be run to rename the POSIX user. Two variables,
# %uold and %unew, will be substituted with the old and new usernames,
# respectively. The script should return 0 upon successful completion, and
# nonzero otherwise. The script has all responsibility to rename all the
# necessary data that is accessible in this posix method. This can mean
# different requirements for different backends. The tdbsam and smbpasswd
# backends will take care of the contents of their respective files, so the
# script is responsible only for changing the POSIX username, and other data
# that may required for your circumstances, such as home directory. Please also
# consider whether or not you need to rename the actual home directories
# themselves. The ldapsam backend will not make any changes, because of the
# potential issues with renaming the LDAP naming attribute. In this case the
# script is responsible for changing the attribute that samba uses (uid) for
# locating users, as well as any data that needs to change for other
# applications using the same directory.
      rename_user_script: ''
# This option controls whether winbindd requires support for md5 strong key
# support for the netlogon secure channel. The following flags will be required
# NETLOGON_NEG_STRONG_KEYS, NETLOGON_NEG_ARCFOUR and
# NETLOGON_NEG_AUTHENTICATED_RPC. You can set this to 'no' if some domain
# controllers only support des. This might allows weak crypto to be negotiated,
# may via downgrade attacks. Note for active directory domain this option is
# hardcoded to 'yes'. This option yields precedence to the 'reject_md5_servers'
# option. This option takes precedence to the 'client_schannel' option.
      require_strong_key: 'yes'
# This boolean option controls whether an incoming SMB1 session setup should
# kill other connections coming from the same IP. This matches the default
# Windows 2003 behaviour. Setting this parameter to 'yes' becomes necessary
# when you have a flaky network and windows decides to reconnect while the old
# connection still has files with share modes open. These files become
# inaccessible over the new connection. The client sends a zero VC on the new
# connection, and Windows 2003 kills all other connections coming from the same
# IP. This way the locked files are accessible again. Please be aware that
# enabling this option will kill connections behind a masquerading router, and
# will not trigger for clients that only use SMB2 or SMB3. Default is 'no'.
      reset_on_zero_vc: 'no'
# The setting of this parameter determines whether user and group list
# information is returned for an anonymous connection. and mirrors the effects
# of the
# HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA\RestrictAnonymous
# registry key in Windows 2000 and Windows NT. When set to '0' (the default),
# user and group list information is returned to anyone who asks. When set to
# '1', only an authenticated user can retrieve user and group list information.
# For the value '2', supported by Windows 2000/XP and Samba, no anonymous
# connections are allowed at all. This can break third party and Microsoft
# applications which expect to be allowed to perform operations anonymously.
      restrict_anonymous: '0'
# This option specifies the path to the name server control utility. The rndc
# utility should be a part of the bind installation.
      rndc_command: '/usr/sbin/rndc'
# The server will chroot() to this directory on startup. This is not strictly
# necessary for secure operation. Even without it the server will deny access
# to files not in one of the service entries. It may also check for, and deny
# access to, soft links to other parts of the filesystem, or attempts to use
# ".." in file names to access other directories (depending on the setting of
# the wide smbconfoptions parameter). Adding a root directory entry other than
# "/" adds an extra level of security, but at a price. It absolutely ensures
# that no access is given to files not in the sub-tree specified in the root
# directory option, including some files needed for complete operation of the
# server. To maintain full operability of the server you will need to mirror
# some system files into the root directory tree. In particular you will need
# to mirror '/etc/passwd' (or a subset of it), and any binaries or
# configuration files needed for printing (if required). The set of files that
# must be mirrored is operating system dependent. Default is None.
      root_directory: ''
# Setting this option will force the RPC client and server to transfer data in
# big endian. If it is disabled, data will be transferred in little endian.
# The behaviour is independent of the endianness of the host machine.
# Default is 'no'.
      rpc_big_endian: 'no'
# This parameter tells the RPC server which port range it is allowed to use to
# create a listening socket for LSA, SAM, Netlogon and others without wellknown
# tcp ports. The first value is the lowest number of the port range and the
# second the hightest. This applies to RPC servers in all server roles.
# Default is None.
      rpc_server_dynamic_port_range: '49152-65535'
# Specifies which port the server should listen on for DCE/RPC over TCP/IP
# traffic. This controls the default port for all protocols, except for
# NETLOGON. If unset, the first available port from
# 'rpc_server_dynamic_port_range' is used, e.g. 49152. This option applies
# currently only when samba runs as an active directory domain controller.
# The default value '0' causes Samba to select the first available port from
# 'rpc_server_dynamic_port_range'.
      rpc_server_port: '0'
# This option specifies the path to the Samba KCC command. This script is used
# for replication topology replication. It should not be necessary to modify
# this option except for testing purposes or if the samba_kcc was installed in
# a non-default location.
      samba_kcc_command: '/usr/bin/kcc'
# This option affects how clients respond to Samba and is one of the most
# important settings in the smb.conf file. The default is 'user', as this is
# the most common setting, used for a standalone file server or a DC. The
# alternatives are 'ads' or 'domain', which support joining Samba to a Windows
# domain. You should use 'user' and 'map_to_guest' if you want to mainly setup
# shares without a password (guest shares). This is commonly used for a shared
# printer server.
# 'auto' - this is the default security setting in Samba, and causes Samba to
# consult the server role parameter (if set) to determine the security mode.
# 'user' - if server role is not specified, this is the default security setting
# in Samba. With user-level security a client must first log-on with a valid
# username and password (which can be mapped using the 'username_map'
# parameter). Encrypted passwords (see the encrypted passwords parameter) can
# also be used in this security mode. Parameters such as 'user' and 'guest_only'
# if set are then applied and may change the UNIX user to use on this
# connection, but only after the user has been successfully authenticated. Note
# that the name of the resource being requested is not sent to the server until
# after the server has successfully authenticated the client. This is why guest
# shares don't work in user level security without allowing the server to
# automatically map unknown users into the guest account. See the 'map_to_guest'
# parameter for details on doing this.
# 'domain' - this mode will only work correctly if "net" has been used to add
# this machine into a Windows NT Domain. It expects the 'encrypted_passwords'
# parameter to be set to 'yes'. In this mode Samba will try to validate the
# username/password by passing it to a Windows NT Primary or Backup Domain
# Controller, in exactly the same way that a Windows NT Server would do. Note
# that a valid UNIX user must still exist as well as the account on the Domain
# Controller to allow Samba to have a valid UNIX account to map file access to.
# Note that from the client's point of view is 'domain' is the same as 'user'.
# It only affects how the server deals with the authentication, it does not in
# any way affect what the client sees. Note that the name of the resource being
# requested is not sent to the server until after the server has successfully
# authenticated the client. This is why guest shares don't work in user level
# security without allowing the server to automatically map unknown users into
# the 'guest_account'. See the 'map_to_guest' parameter for details on doing
# this. See also the 'password_server' parameter and the 'encrypted_passwords'
# parameter.
# 'ads' - in this mode, Samba will act as a domain member in an ADS realm. To
# operate in this mode, the machine running Samba will need to have Kerberos
# installed and configured and Samba will need to be joined to the ADS realm
# using the "net" utility.
      security: 'auto'
# The value of the parameter is the highest protocol level that will be
# supported by the server. Normally this option should not be set as the
# automatic negotiation phase in the SMB protocol takes care of choosing the
# appropriate protocol.
      server_max_protocol: 'SMB3'
# This setting controls the minimum protocol version that the server will allow
# the client to use. Normally this option should not be set as the automatic
# negotiation phase in the SMB protocol takes care of choosing the appropriate
# protocol.
      server_min_protocol: 'SMB3'
# This boolean parameter controls whether smbd will support SMB3 multi-channel.
# Default is 'no'.
      server_multi_channel_support: 'no'
# This option determines the basic operating mode of a Samba server and is one
# of the most important settings in the smb.conf file. The default is 'auto',
# as causes Samba to operate according to the security setting, or if not
# specified as a simple file server that is not connected to any domain.
# The alternatives are 'standalone' or 'member server', which support joining
# Samba to a Windows domain, along with 'domain controller', which run Samba as
# a Windows domain controller. You should use 'standalone' and 'map_to_guest'
# if you want to mainly setup shares without a password (guest shares). This is
# commonly used for a shared printer server.
# 'auto' - this is the default server role in Samba, and causes Samba to
# consult the 'security' parameter (if set) to determine the 'server_role',
# giving compatible behaviours to previous Samba versions.
# 'standalone' - if 'security' is also not specified, this is the default
# security setting in Samba. In standalone operation, a client must first log-on
# with a valid username and password (which can be mapped using the
# 'username_map' parameter) stored on this machine. Encrypted passwords (see
# the 'encrypted_passwords' parameter) are by default used in this security
# mode. Parameters such as 'user' and 'guest_only' if set are then applied and
# may change the UNIX user to use on this connection, but only after the user
# has been successfully authenticated.
# 'member server' - this mode will only work correctly if "net" has been used
# to add this machine into a Windows Domain. It expects the
# 'encrypted_passwords' parameter to be set to 'yes'. In this mode Samba will
# try to validate the username/password by passing it to a Windows or Samba
# Domain Controller, in exactly the same way that a Windows Server would do.
# Note that a valid UNIX user must still exist as well as the account on the
# Domain Controller to allow Samba to have a valid UNIX account to map file
# access to. Winbind can provide this.
# 'classic primary domain controller' - this mode of operation runs a classic
# Samba primary domain controller, providing domain logon services to Windows
# and Samba clients of an NT4-like domain. Clients must be joined to the domain
# to create a secure, trusted path across the network. There must be only one
# PDC per NetBIOS scope (typcially a broadcast network or clients served by a
# single WINS server).
# 'classic backup domain controller' - this mode of operation runs a classic
# Samba backup domain controller, providing domain logon services to Windows
# and Samba clients of an NT4-like domain. As a BDC, this allows multiple Samba
# servers to provide redundant logon services to a single NetBIOS scope.
# 'active directly domain controller' - this mode of operation runs Samba as an
# active directory domain controller, providing domain logon services to
# Windows and Samba clients of the domain.
      server_role: 'auto'
# This option contains the services that the Samba daemon will run.
      server_services:
      - '-s3fs'
      - '+smb'
# This controls whether the client is allowed or required to use SMB1 and SMB2
# signing. Possible values are 'default', 'auto', 'mandatory' and 'disabled'.
# By default, and when smb signing is set to 'default', smb signing is required
# when server role is active directory domain controller and disabled otherwise.
# When set to 'auto', SMB1 signing is offered, but not enforced. When set to
# mandatory, SMB1 signing is required and if set to 'disabled', SMB signing is
# not offered either. For the SMB2 protocol, by design, signing cannot be
# disabled. In the case where SMB2 is negotiated, if this parameter is set to
# 'disabled', it will be treated as 'auto'. Setting it to mandatory will still
# require SMB2 clients to use signing.
      server_signing: 'default'
# This controls what string will show up in the printer comment box in print
# manager and next to the IPC connection in "net view". It can be any string
# that you wish to show to your users. It also sets what will appear in browse
# lists next to the machine name. A '%v' will be replaced with the Samba
# version number. A '%h' will be replaced with the hostname.
      server_string: 'Samba %v'
# Thanks to the Posix subsystem in NT a Windows User has a primary group in
# addition to the auxiliary groups. This script sets the primary group in the
# unix user database when an administrator sets the primary group from the
# windows user manager or when fetching a SAM with net rpc vampire. '%u' will
# be replaced with the user whose primary group is to be set. '%g' will be
# replaced with the group to set. For example: "/usr/sbin/usermod -g '%g' '%u'".
# Default is None.
      set_primary_group_script: ''
# The set quota command should only be used whenever there is no operating
# system API available from the OS that samba can use. This parameter should
# specify the path to a script that can set quota for the specified arguments.
# Default is None.
      set_quota_command: '/usr/local/sbin/set_quota'
# This option specifies the backend that will be used to access the
# configuration of file shares. Traditionally, Samba file shares have been
# configured in the smb.conf file and this is still the default. At the moment
# there are no other supported backends.
      share_backend: 'classic'
# This is needed to support some special application that makes QFSINFO calls
# to check whether we set the SPARSE_FILES bit (0x40). If this bit is not set
# that particular application refuses to work against Samba. With
# share_fake_fscaps in '64' the SPARSE_FILES file system capability flag is set.
# Use other decimal values to specify the bitmask you need to fake.
# Default is '0'.
      share_fake_fscaps: '0'
# With the introduction of MS-RPC based printing support for Windows NT/2000
# client in Samba 2.2, a "Printers..." folder will appear on Samba hosts in the
# share listing. Normally this folder will contain an icon for the MS Add
# Printer Wizard (APW). However, it is possible to disable this feature
# regardless of the level of privilege of the connected user. Under normal
# circumstances, the Windows NT/2000 client will open a handle on the printer
# server with OpenPrinterEx() asking for Administrator privileges. If the user
# does not have administrative access on the print server (i.e is not root or
# has granted the SePrintOperatorPrivilege), the OpenPrinterEx() call fails
# and the client makes another open call with a request for a lower privilege
# level. This should succeed, however the APW icon will not be displayed.
# Disabling the show add printer wizard parameter will always cause the
# OpenPrinterEx() on the server to fail. Thus the APW icon will never be
# displayed. None this does not prevent the same user from having administrative
# privilege on an individual printer. Default is 'yes'.
      show_add_printer_wizard: 'yes'
# This a full path name to a script called by smbd that should start a shutdown
# procedure. If the connected user possesses the SeRemoteShutdownPrivilege,
# right, this command will be run as root. Default is None.
      shutdown_script: '/usr/local/samba/sbin/shutdown'
# This boolean option tells smbd whether to globally negotiate SMB2 leases on
# file open requests. Leasing is an SMB2-only feature which allows clients to
# aggressively cache files locally above and beyond the caching allowed by SMB1
# oplocks. This is only available with 'oplocks' in 'yes' and 'kernel_oplocks'
# in 'no'. Note that the write cache won't be used for file handles with a smb2
# write lease. Default is 'yes'.
      smb2_leases: 'yes'
# This option controls the maximum number of outstanding simultaneous SMB2
# operations that Samba tells the client it will allow. This is similar to the
# 'max_mux' parameter for SMB1. You should never need to set this parameter.
# The default is '8192' credits, which is the same as a Windows 2008R2 SMB2
# server.
      smb2_max_credits: '8192'
# This option specifies the protocol value that smbd will return to a client,
# informing the client of the largest size that may be returned by a single
# SMB2 read call. The maximum is '8388608' bytes (8MiB), which is the same as a
# Windows Server 2012 r2. Please note that the default is 8MiB, but it's limit
# is based on the smb2 dialect (64KiB for SMB == 2.0, 8MiB for SMB >= 2.1 with
# LargeMTU). Large MTU is not supported over NBT (tcp port 139).
      smb2_max_read: '8388608'
# This option specifies the protocol value that smbd will return to a client,
# informing the client of the largest size of buffer that may be used in
# querying file meta-data via QUERY_INFO and related SMB2 calls. The maximum is
# '8388608' bytes (8MiB), which is the same as a Windows Server 2012 r2.
# Please note that the default is 8MiB, but it's limit is based on the smb2
# dialect (64KiB for SMB == 2.0, 1MiB for SMB >= 2.1 with LargeMTU). Large MTU
# is not supported over NBT (tcp port 139).
      smb2_max_trans: '8388608'
# This option specifies the protocol value that smbd will return to a client,
# informing the client of the largest size that may be sent to the server by a
# ingle SMB2 write call. The maximum is 8388608 bytes (8MiB), which is the same
# as a Windows Server 2012 r2. Please note that the default is 8MiB, but it's
# limit is based on the smb2 dialect (64KiB for SMB == 2.0, 8MiB for SMB => 2.1
# with LargeMTU). Large MTU is not supported over NBT (tcp port 139).
      smb2_max_write: '8388608'
# This parameter allows the administrator to enable profiling support.
# Possible values are 'off' (the default), 'count' and 'on'.
      smbd_profiling_level: 'off'
# This option sets the path to the encrypted smbpasswd file. By default the
# path to the smbpasswd file is compiled into Samba.
      smb_passwd_file: '/etc/samba/smbpasswd'
# Specifies which ports the server should listen on for SMB traffic.
      smb_ports: '445 139'
# This option allows you to set socket options to be used when talking with the
# client. Socket options are controls on the networking layer of the operating
# systems which allow the connection to be tuned. This option will typically be
# used to tune your Samba server for optimal performance for your local network.
# There is no way that Samba can know what the optimal parameters are for your
# net, so you must experiment and choose them yourself.
      socket_options:
      - 'IPTOS_LOWDELAY'
      - 'TCP_NODELAY'
      - 'SO_SNDBUF=8192'
# This option sets the command that for updating servicePrincipalName names
# from spn_update_list.
      spn_update_command: '/usr/local/sbin/spnupdate'
# Windows spoolss print clients only allow association of server-side drivers
# with printers when the driver architecture matches the advertised print
# server architecture. Samba's spoolss print server architecture can be changed
# using this parameter. Default is 'Windows NT x86'.
      spoolss_architecture: 'Windows x64'
# Windows might require a new os version number. This option allows to modify
# the build number.
# The complete default version number is: 5.0.2195 (Windows 2000).
# The example is 6.1.7601 (Windows 2008 R2).
      spoolss_os_major: '5'
# Windows might require a new os version number. This option allows to modify
# the build number.
# The complete default version number is: 5.0.2195 (Windows 2000).
# The example is 6.1.7601 (Windows 2008 R2).
      spoolss_os_minor: '0'
# Windows might require a new os version number. This option allows to modify
# the build number.
# The complete default version number is: 5.0.2195 (Windows 2000).
# The example is 6.1.7601 (Windows 2008 R2).
      spoolss_os_build: '2195'
# Windows might require a new os version number. This option allows to modify
# the build number. The complete default version number is:
# 6.1.7007 (Windows 7 and Windows Server 2008 R2).
      spoolss_client_os_major: '6'
# Windows might require a new os version number. This option allows to modify
# the build number. The complete default version number is:
# 6.1.7007 (Windows 7 and Windows Server 2008 R2).
      spoolss_client_os_minor: '1'
# Windows might require a new os version number. This option allows to modify
# the build number. The complete default version number is:
# 6.1.7007 (Windows 7 and Windows Server 2008 R2).
      spoolss_client_os_build: '7007'
# This parameter determines if smbd will use a cache in order to speed up case
# insensitive name mappings. You should never need to change this parameter.
# Default is 'yes'.
      stat_cache: 'yes'
# Usually, most of the TDB files are stored in the lock directory. It is
# possible to differentiate between TDB files with persistent data and TDB
# files with non-persistent data using the state directory and the cache
# directory options. This option specifies the directory where TDB files
# containing important persistent data will be stored.
# Default is '/var/lib/samba'.
      state_directory: '/var/lib/samba'
# This option defines a list of init scripts that smbd will use for starting
# and stopping Unix services via the Win32 ServiceControl API. This allows
# Windows administrators to utilize the MS Management Console plug-ins to
# manage a Unix server running Samba. The administrator must create a directory
# name svcctl in Samba's $(libdir) and create symbolic links to the init
# scripts in /etc/init.d/. The name of the links must match the names given as
# part of the svcctl list.
      svcctl_list: ''
# This parameter maps how Samba debug messages are logged onto the system
# syslog logging levels. Samba debug level zero maps onto syslog LOG_ERR, debug
# level one maps onto LOG_WARNING, debug level two maps onto LOG_NOTICE, debug
# level three maps onto LOG_INFO. All higher levels are mapped to LOG_DEBUG.
# This parameter sets the threshold for sending messages to syslog. Only
# messages with debug level less than this value will be sent to syslog. There
# still will be some logging to log.[sn]mbd even if syslog only is enabled.
# The 'logging' parameter should be used instead. When logging is set, it
# overrides the 'syslog' parameter.
      syslog: '1'
# If this parameter is set then Samba debug messages are logged into the system
# syslog only, and not to the debug log files. There still will be some logging
# to log.[sn]mbd even if 'syslog_only' is enabled. The logging parameter should
# be used instead. When 'logging' is set, it overrides the 'syslog_only'
# parameter. Default is 'no'.
      syslog_only: 'no'
# When filling out the user information for a Windows NT user, the winbindd
# daemon uses this parameter to fill in the home directory for that user. If
# the string %D is present it is substituted with the user's Windows NT domain
# name. If the string %U is present it is substituted with the user's Windows
# NT user name.
      template_homedir: '/home/%D/%U'
# When filling out the user information for a Windows NT user, the winbindd
# daemon uses this parameter to fill in the login shell for that user.
      template_shell: '/bin/false'
# This parameter determines if nmbd advertises itself as a time server to
# Windows clients. Default is 'no'.
      time_server: 'no'
# Samba debug log messages are timestamped by default. If you are running at a
# high debug level these timestamps can be distracting. This boolean parameter
# allows timestamping to be turned off.
      timestamp_logs: 'yes'
# This option can be set to a file (PEM format) containing CA certificates of
# root CAs to trust to sign certificates or intermediate CA certificates. This
# path is relative to private dir if the path does not start with a /.
      tls_cafile: 'tls/ca.pem'
# This option can be set to a file (PEM format) containing the RSA certificate.
# This path is relative to private dir if the path does not start with a /.
      tls_certfile: 'tls/cert.pem'
# This option can be set to a file containing a certificate revocation list
# (CRL). This path is relative to private dir if the path does not start with
# a /.
      tls_crlfile: ''
# This option can be set to a file with Diffie-Hellman parameters which will be
# used with DH ciphers. This path is relative to private dir if the path does
# not start with a /.
      tls_dh_params_file: ''
# If this option is set to 'yes', then Samba will use TLS when possible in
# communication.
      tls_enabled: 'yes'
# This option can be set to a file (PEM format) containing the RSA private key.
# This file must be accessible without a pass-phrase, i.e. it must not be
# encrypted. This path is relative to private dir if the path does not start
# with a /.
      tls_keyfile: 'tls/key.pem'
# This option can be set to a string describing the TLS protocols to be
# supported in the parts of Samba that use GnuTLS, specifically the AD DC.
# The default turns off SSLv3, as this protocol is no longer considered secure
# after CVE-2014-3566 (otherwise known as POODLE) impacted SSLv3 use in HTTPS
# applications.
      tls_priority: 'NORMAL:-VERS-SSL3.0'
# This controls if and how strict the client will verify the peer's certificate
# and name. Possible values are (in increasing order): 'no_check', 'ca_only',
# 'ca_and_name_if_available', 'ca_and_name' and 'as_strict_as_possible'.
# When set to 'no_check' the certificate is not verified at all, which allows
# trivial man in the middle attacks.
# When set to 'ca_only' the certificate is verified to be signed from a ca
# specified in the 'tls_ca_file' option. Setting 'tls_ca_file' to a valid file
# is required. The certificate lifetime is also verified. If the 'tls_crl_file'
# option is configured, the certificate is also verified against the ca crl.
# When set to 'ca_and_name_if_available' all checks from 'ca_only' are
# performed. In addition, the peer hostname is verified against the
# certificate's name, if it is provided by the application layer and not given
# as an ip address string.
# When set to 'ca_and_name' all checks from 'ca_and_name_if_available' are
# performed. In addition the peer hostname needs to be provided and even an ip
# address is checked against the certificate's name.
# When set to 'as_strict_as_possible' (the default) all checks from
# 'ca_and_name' are performed. In addition the 'tls_crl_file' needs to be
# configured. Future versions of Samba may implement additional checks.
      tls_verify_peer: 'as_strict_as_possible'
# Specifies whether the server and client should support unicode. If this
# option is set to 'no', the use of ASCII will be forced. Default is 'yes'.
      unicode: 'yes'
# Specifies the charset the unix machine Samba runs on uses. Samba needs to
# know this in order to be able to convert text to the charsets other SMB
# clients use. This is also the charset Samba will use when specifying
# arguments to scripts that it invokes.
      unix_charset: 'UTF-8'
# This boolean parameter controls whether Samba implements the CIFS UNIX
# extensions, as defined by HP. These extensions enable Samba to better serve
# UNIX CIFS clients by supporting features such as symbolic links, hard links,
# etc... These extensions require a similarly enabled client, and are of no
# current use to Windows clients. Note if this parameter is turned on, the wide
# links parameter will automatically be disabled. See the parameter allow
# insecure wide links if you wish to change this coupling between the two
# parameters. Default is 'yes'.
      unix_extensions: 'yes'
# This boolean parameter controls whether Samba attempts to synchronize the
# UNIX password with the SMB password when the encrypted SMB password in the
# smbpasswd file is changed. If this is set to 'yes' the program specified in
# the 'passwd_program' parameter is called AS ROOT - to allow the new UNIX
# password to be set without access to the old UNIX password (as the SMB
# password change code has no access to the old password cleartext, only the
# new). This option has no effect if samba is running as an active directory
# domain controller, in that case have a look at the 'password_hash_gpg_key_ids'
# option and the samba-tool user syncpasswords command. Default is 'no'.
      unix_password_sync: 'no'
# This global parameter determines if the tdb internals of Samba can depend on
# mmap working correctly on the running system. Samba requires a coherent
# mmap/read-write system memory cache. Currently only HPUX does not have such a
# coherent cache, and so this parameter is set to no by default on HPUX. On all
# other systems this parameter should be left alone. This parameter is provided
# to help the Samba developers track down problems with the tdb internal code.
# Default is 'yes'.
      use_mmap: 'yes'
# This option helps Samba to try and 'guess' at the real UNIX username, as many
# DOS clients send an all-uppercase username. By default Samba tries all
# lowercase, followed by the username with the first letter capitalized, and
# fails if the username is not found on the UNIX machine. If this parameter is
# set to non-zero the behavior changes. This parameter is a number that
# specifies the number of uppercase combinations to try while trying to
# determine the UNIX user name. The higher the number the more combinations will
# be tried, but the slower the discovery of usernames will be. Use this
# parameter when you have strange usernames on your UNIX machine, such as
# "AstrangeUser". This parameter is needed only on UNIX systems that have case
# sensitive usernames. Default is '0'.
      username_level: '0'
      username_map:
# This option allows you to specify a file containing a mapping of usernames
# from the clients to the server. This can be used for several purposes. The
# most common is to map usernames that users use on DOS or Windows machines to
# those that the UNIX box uses. The other is to map multiple users to a single
# username so that they can more easily share files. Please note that for user
# mode security, the username map is applied prior to validating the user
# credentials. Domain member servers (domain or ads) apply the username map
# after the user has been successfully authenticated by the domain controller
# and require fully qualified entries in the map table
# (e.g. biddle = DOMAIN\foo). The map file is parsed line by line. Each line
# should contain a single UNIX username on the left then a '=' followed by a
# list of usernames on the right. The list of usernames on the right may contain
# names of the form @group in which case they will match any UNIX username in
# that group. The special client name '*' is a wildcard and matches any name.
# Each line of the map file may be up to 1023 characters long. The file is
# processed on each line by taking the supplied username and comparing it with
# each username on the right hand side of the '=' signs. If the supplied name
# matches any of the names on the right hand side then it is replaced with the
# name on the left. Processing then continues with the next line. If any line
# begins with a '#' or a ';' then it is ignored. If any line begins with an '!'
# then the processing will stop after that line if a mapping was done by the
# line. Otherwise mapping continues with every line being processed. Using '!'
# is most useful when you have a wildcard mapping line later in the file.
# For example to map from the name admin or administrator to the UNIX name root
# you would use:
      - key: 'root'
        value:
        - 'admin'
        - 'administrator'
# Or to map anyone in the UNIX group system to the UNIX name sys you would use:
      - key: 'sys'
        value: '@system'
# You can have as many mappings as you like in a username map file. If your
# system supports the NIS NETGROUP option then the netgroup database is
# checked before the /etc/group database for matching groups. You can map
# Windows usernames that have spaces in them by using double quotes around the
# name. For example would map the windows username 'Andrew Tridgell' to the unix
# username 'tridge'.
      - key: 'tridge'
        value: '"Andrew Tridgell"'
# The following example would map mary and fred to the unix user sys, and map
# the rest to guest. Note the use of the '!' to tell Samba to stop processing
# if it gets a match on that line:
      - key: '!sys'
        value:
        - 'mary'
        - 'fred'
      - key: 'guest'
        value: '*'
# Note that the remapping is applied to all occurrences of usernames. Thus if
# you connect to "\\server\fred" and "fred" is remapped to "mary" then you will
# actually be connecting to "\\server\mary" and will need to supply a password
# suitable for mary not fred. The only exception to this is the username passed
# to a Domain Controller (if you have one). The DC will receive whatever
# username the client supplies without modification. Also note that no reverse
# mapping is done. The main effect this has is with printing. Users who have
# been mapped may have trouble deleting print jobs as PrintManager under WfWg
# will think they don't own the print job. Samba versions prior to 3.0.8 would
# only support reading the fully qualified username (e.g.: DOMAIN\user) from the
# username map when performing a kerberos login from a client. However, when
# looking up a map entry for a user authenticated by NTLM[SSP], only the login
# name would be used for matches. This resulted in inconsistent behavior
# sometimes even on the same server. The following functionality is obeyed in
# version 3.0.8 and later: when performing local authentication, the
# 'username_map' is applied to the login name before attempting to authenticate
# the connection. When relying upon a external domain controller for validating
# authentication requests, smbd will apply the username map to the fully
# qualified username (i.e.  DOMAIN\user) only after the user has been
# successfully authenticated. Default is empty.
#
# Mapping usernames with the username map or username map script features of
# Samba can be relatively expensive. During login of a user, the mapping is
# done several times. In particular, calling the 'username_map_script' can slow
# down logins if external databases have to be queried from the script being
# called. The parameter 'username_map_cache_time' controls a mapping cache. It
# specifies the number of seconds a mapping from the username map file or script
# is to be efficiently cached. The default of '0' means no caching is done.
      username_map_cache_time: '0'
# This script is a mutually exclusive alternative to the username map parameter.
# This parameter specifies and external program or script that must accept a
# single command line option (the username transmitted in the authentication
# request) and return a line on standard output (the name to which the account
# should mapped). In this way, it is possible to store username map tables in
# an LDAP or NIS directory services. Default is None.
      username_map_script: '/etc/samba/scripts/mapusers.sh'
# This parameter controls whether user defined shares are allowed to be
# accessed by non-authenticated users or not. It is the equivalent of allowing
# people who can create a share the option of setting 'guest_ok' in 'yes' in a
# share definition. Due to its security sensitive nature, the default is set to
# 'off'.
      usershare_allow_guests: 'no'
# This parameter specifies the number of user defined shares that are allowed
# to be created by users belonging to the group owning the usershare directory.
# If set to zero (the default) user defined shares are ignored.
      usershare_max_shares: '0'
# This parameter controls whether the pathname exported by a user defined
# shares must be owned by the user creating the user defined share or not. If
# set to 'yes' (the default) then smbd checks that the directory path being
# shared is owned by the user who owns the usershare file defining this share
# and refuses to create the share if not. If set to 'no' then no such check is
# performed and any directory path may be exported regardless of who owns it.
      usershare_owner_only: 'yes'
# This parameter specifies the absolute path of the directory on the filesystem
# used to store the user defined share definition files. This directory must be
# owned by root, and have no access for other, and be writable only by the
# group owner. In addition the "sticky" bit must also be set, restricting
# rename and delete to owners of a file (in the same way the /tmp directory is
# usually configured). Members of the group owner of this directory are the
# users allowed to create usershares.
      usershare_path: '/var/lib/samba/usershares'
# This parameter specifies a list of absolute pathnames the root of which are
# allowed to be exported by user defined share definitions. If the pathname to
# be exported doesn't start with one of the strings in this list, the user
# defined share will not be allowed. This allows the Samba administrator to
# restrict the directories on the system that can be exported by user defined
# shares. If there is a 'usershare_prefix_deny_list' and also a
# 'usershare_prefix_allow_list' the deny list is processed first, followed by
# the allow list, thus leading to the most restrictive interpretation.
# Default is None.
      usershare_prefix_allow_list:
      - '/home'
      - '/data'
      - '/space'
# This parameter specifies a list of absolute pathnames the root of which are
# NOT allowed to be exported by user defined share definitions. If the pathname
# exported starts with one of the strings in this list the user defined share
# will not be allowed. Any pathname not starting with one of these strings will
# be allowed to be exported as a usershare. This allows the Samba administrator
# to restrict the directories on the system that can be exported by user
# defined shares. If there is a 'usershare_prefix_deny_list' and also a
# 'usershare_prefix_allow_list' the deny list is processed first, followed by
# the allow list, thus leading to the most restrictive interpretation.
# Default is None.
      usershare_prefix_deny_list:
      - '/etc'
      - '/dev'
      - '/private'
# User defined shares only have limited possible parameters such as 'path',
# 'guest_ok,' etc. This parameter allows usershares to "cloned" from an
# existing share. If 'usershare_template_share' is set to the name of an
# existing share, then all usershares created have their defaults set from the
# parameters set on this share. The target share may be set to be invalid for
# real file sharing by setting the parameter "-valid = False" on the template
# share definition. This causes it not to be seen as a real exported share but
# to be able to be used as a template for usershares. Default is None.
      usershare_template_share: ''
# If set to yes then Samba will attempt to add utmp or utmpx records (depending
# on the UNIX system) whenever a connection is made to a Samba server. Sites may
# use this to record the user connecting to a Samba share. Due to the
# requirements of the utmp record, we are required to create a unique
# identifier for the incoming user. Enabling this option creates an n^2
# algorithm to find this number. This may impede performance on large
# installations. Default is 'no'.
      utmp: 'no'
# It specifies a directory pathname that is used to store the utmp or utmpx
# files (depending on the UNIX system) that record user connections to a Samba
# server. By default this is not set, meaning the system will use whatever utmp
# file the native system is set to use (usually '/var/run/utmp' on Linux).
      utmp_directory: '/var/run/utmp'
# Specifies which port the Samba web server should listen on. Default is '901'.
      web_port: '901'
# This parameter specifies the number of seconds the winbindd daemon will cache
# user and group information before querying a Windows NT server again. This
# does not apply to authentication requests, these are always evaluated in real
# time unless the winbind offline logon option has been enabled.
      winbind_cache_time: '300'
# This setting controls the location of the winbind daemon's socket. Except
# within automated test scripts, this should not be altered, as the client
# tools (nss_winbind etc) do not honour this parameter. Client tools must then
# be advised of the altered path with the WINBINDD_SOCKET_DIR environment
# varaible.
      winbindd_socket_directory: '/run/samba/winbindd'
# On large installations using winbindd it may be necessary to suppress the
# enumeration of groups through the setgrent(), getgrent() and endgrent() group
# of system calls. If the parameter is 'no' (the default), calls to the
# getgrent() system call will not return any data. Warning turning off group
# enumeration may cause some programs to behave oddly.
      winbind_enum_groups: 'no'
# On large installations using winbindd it may be necessary to suppress the
# enumeration of users through the setpwent(), getpwent() and endpwent() group
# of system calls. If the parameter is 'no' (the default), calls to the
# getpwent system call will not return any data. Warning turning off user
# enumeration may cause some programs to behave oddly. For example, the finger
# program relies on having access to the full user list when searching for
# matching usernames.
      winbind_enum_users: ''
# This option controls the maximum depth that winbindd will traverse when
# flattening nested group memberships of Windows domain groups. This is
# different from the winbind nested groups option which implements the Windows
# NT4 model of local group nesting. The "winbind expand groups" parameter
# specifically applies to the membership of domain groups. This option also
# affects the return of non nested group memberships of Windows domain users.
# With the new default is '0' winbind does not query group memberships at all.
# Be aware that a high value for this parameter can result in system slowdown as
# the main parent winbindd daemon must perform the group unrolling and will be
# unable to answer incoming NSS or authentication requests during this time.
# Some broken applications (including some implementations of newgrp and sg)
# calculate the group memberships of users by traversing groups, such
# applications will require this parameter in '1'. But the default makes
# winbindd more reliable as it doesn't require SAMR access to domain controllers
# of trusted domains.
      winbind_expand_groups: '0'
# Allows one to enter a list of trusted domains winbind should ignore (untrust).
# This can avoid the overhead of resources from attempting to login to DCs that
# should not be communicated with. Default is None.
      winbind_ignore_domains:
      - 'DOMAIN1'
      - 'DOMAIN2'
# This parameter specifies the maximum number of clients the winbindd daemon
# can connect with. The parameter is not a hard limit. The winbindd daemon
# configures itself to be able to accept at least that many connections, and if
# the limit is reached, an attempt is made to disconnect idle clients.
# Default is '200'.
      winbind_max_clients: '200'
# This parameter specifies the maximum number of simultaneous connections that
# the winbindd daemon should open to the domain controller of one domain.
# Setting this parameter to a value greater than '1' (the default) can improve
# scalability with many simultaneous winbind requests, some of which might be
# slow. Note that if 'winbind_offline_logon' is set to 'yes', then only one DC
# connection is allowed per domain, regardless of this setting.
      winbind_max_domain_connections: '1'
# If set to 'yes' (the default), this parameter activates the support for
# nested groups. Nested groups are also called local groups or aliases. They
# work like their counterparts in Windows: nested groups are defined locally on
# any machine (they are shared between DC's through their SAM) and can contain
# users and global groups from any trusted SAM. To be able to use nested groups,
# you need to run nss_winbind.
      winbind_nested_groups: 'yes'
# This parameter controls whether winbindd will replace whitespace in user and
# group names with an underscore (_) character. For example, whether the name
# "Space Kadet" should be replaced with the string "space_kadet". Frequently
# Unix shell scripts will have difficulty with usernames contains whitespace
# due to the default field separator in the shell. If your domain possesses
# names containing the underscore character, this option may cause problems
# unless the name aliasing feature is supported by your nss_info plugin. This
# feature also enables the name aliasing API which can be used to make domain
# user and group names to a non-qualified version. Please refer to the manpage
# for the configured idmap and nss_info plugin for the specifics on how to
# configure name aliasing for a specific configuration. Name aliasing takes
# precedence (and is mutually exclusive) over the whitespace replacement
# mechanism discussed previously. Default is 'no'.
      winbind_normalize_names: 'no'
# This parameter is designed to control how Winbind retrieves Name Service
# Information to construct a user's home directory and login shell. Currently
# the following settings are available:
# 'template' - the default, using the parameters of template shell and template
# homedir.
# 'sfu', 'sfu20', 'rfc2307' - when Samba is running in "security: 'ads'" and
# your Active Directory Domain Controller does support the Microsoft
# "Services for Unix" (SFU) LDAP schema, winbind can retrieve the login shell
# and the home directory attributes directly from your Directory Server. For
# SFU 3.0 or 3.5 simply choose 'sfu', if you use SFU 2.0 please choose
# 'sfu20'. The primary group membership is currently always calculated via the
# "primaryGroupID" LDAP attribute.
      winbind_nss_info: 'template'
# This parameter is designed to control whether Winbind should allow one to
# login with the pam_winbind module using Cached Credentials. If enabled,
# winbindd will store user credentials from successful logins encrypted in a
# local cache. Default is 'no'.
      winbind_offline_logon: 'no'
# This parameter specifies the number of seconds the winbindd daemon will wait
# between attempts to contact a Domain controller for a domain that is
# determined to be down or not contactable. Default is '30'.
      winbind_reconnect_delay: '30'
# This parameter is designed to control whether Winbind should refresh Kerberos
# Tickets retrieved using the pam_winbind module. Default is 'no'.
      winbind_refresh_tickets: 'no'
# This parameter specifies the number of seconds the winbindd daemon will wait
# before disconnecting either a client connection with no outstanding requests
# (idle) or a client connection with a request that has remained outstanding
# (hung) for longer than this number of seconds. Default is '60'.
      winbind_request_timeout: '60'
# Setting this parameter to 'yes' forces winbindd to use RPC instead of LDAP to
# retrieve information from Domain Controllers. Default is 'no'.
      winbind_rpc_only: 'no'
# This option only takes effect when the 'security' option is set to 'domain'
# or 'ads'. If it is set to 'yes' (the default), winbindd periodically tries to
# scan for new trusted domains and adds them to a global list inside of
# winbindd. The construction of that global list is not reliable and often
# incomplete in complex trust setups. In most situations the list is not needed
# any more for winbindd to operate correctly. E.g. for plain file serving via
# SMB using a simple idmap setup with 'autorid', 'tdb' or 'ad'. However some
# more complex setups require the list, e.g. if you specify idmap backends for
# specific domains. Some pam_winbind setups may also require the global list.
# If you have a setup that doesn't require the global list, you should set this
# to 'no'.
      winbind_scan_trusted_domains: 'yes'
# This option controls whether any requests from winbindd to domain
# controllers pipe will be sealed. Disabling sealing can be useful for
# debugging purposes. Default is 'yes'.
      winbind_sealed_pipes: 'yes'
# This parameter allows an admin to define the character used when listing a
# username of the form of DOMAIN \user. This parameter is only applicable when
# using the pam_winbind.so and nss_winbind.so modules for UNIX services.
# Please note that setting this parameter to '+' causes problems with group
# membership at least on glibc systems, as the character '+' is used as a
# special character for NIS in /etc/group. Default is '\'.
      winbind_separator: ''
# This parameter specifies whether the winbindd daemon should operate on users
# without domain component in their username. Users without a domain component
# are treated as is part of the winbindd server's own domain. While this does
# not benefit Windows users, it makes SSH, FTP and e-mail function in a way
# much closer to the way they would in a native unix system. This option should
# be avoided if possible. It can cause confusion about responsibilities for a
# user or group. In many situations it is not clear whether winbind or
# /etc/passwd should be seen as authoritative for a user, likewise for groups.
# Default is 'no'.
      winbind_use_default_domain: 'no'
# This specifies the address that is stored in the winsOwner attribute, of
# locally registered winsRecord-objects. The default is to use the ip-address
# of the first network interface. Default is None.
      winsdb_local_owner: ''
# This parameter disables fsync() after changes of the WINS database.
# Default is 'no'.
      winsdb_dbnosync: 'no'
# When Samba is running as a WINS server this allows you to call an external
# program for all changes to the WINS database. The primary use for this option
# is to allow the dynamic update of external name resolution databases such as
# dynamic DNS. Default is None.
      wins_hook: ''
# This is a boolean that controls if nmbd will respond to broadcast name
# queries on behalf of other hosts. You may need to set this to 'yes' for some
# older clients. Default is 'no'.
      wins_proxy: 'no'
# This specifies the IP address (or DNS name: IP address for preference) of the
# WINS server that nmbd should register with. If you have a WINS server on your
# network then you should set this to the WINS server's IP. You should point
# this at your WINS server if you have a multi-subnetted network. If you want
# to work in multiple namespaces, you can give every wins server a 'tag'. For
# each tag, only one (working) server will be queried for a name. The tag
# should be separated from the ip address by a colon. You need to set up Samba
# to point to a WINS server if you have multiple subnets and wish cross-subnet
# browsing to work correctly. See the chapter in the Samba3-HOWTO on Network
# Browsing. Default is None.
      wins_server:
      - '192.9.200.1'
      - '192.168.2.61'
# This boolean controls if the nmbd process in Samba will act as a WINS server.
# You should not set this to 'yes' unless you have a multi-subnetted network
# and you wish a particular nmbd to be your WINS server. Note that you should
# NEVER set this to 'yes' on more than one machine in your network. Default is
# 'no'.
      wins_support: 'no'
# This controls what workgroup your server will appear to be in when queried by
# clients. Note that this parameter also controls the Domain name used with the
# "security: 'domain'" setting. Default is 'WORKGROUP'.
      workgroup: 'WORKGROUP'
# If this parameter is enabled, then explicit (from the client) and implicit
# (via the scavenging) name releases are propagated to the other servers
# directly, even if there are still other addresses active, this applies to
# SPECIAL GROUP (2) and MULTIHOMED (3) entries. Also the replication conflict
# merge algorithm for SPECIAL GROUP (2) entries discards replica addresses
# where the address owner is the local server, if the address was not stored
# locally before. The merge result is propagated directly in case an address
# was discarded. A Windows servers doesn't propagate name releases of SPECIAL
# GROUP (2) and MULTIHOMED (3) entries directly, which means that Windows
# servers may return different results to name queries for SPECIAL GROUP (2)
# and MULTIHOMED (3) names. The option doesn't have much negative impact if
# Windows servers are around, but be aware that they might return unexpected
# results. Default is 'no'.
      wreplsrv_propagate_name_releases: 'no'
# This maximum interval in seconds between 2 periodically scheduled runs where
# we check for wins.ldb changes and do push notifications to our push partners.
# Also wins_config.ldb changes are checked in that interval and partner
# configuration reloads are done.
      wreplsrv_periodic_interval: '15'
# This is the interval in s between 2 scavenging runs which clean up the WINS
# database and changes the states of expired name records. Defaults to half of
# the value of 'wreplsrv_renew_interval'.
      wreplsrv_scavenging_interval: ''
# This is the time in s the server needs to be up till we'll remove tombstone
# records from our database. Defaults to 3 days.
      wreplsrv_tombstone_extra_timeout: '259200'
# This is the interval in s till released records of the WINS server become
# tombstone. Defaults to 6 days.
      wreplsrv_tombstone_interval: '518400'
# This is the interval in s till tombstone records are deleted from the WINS
# database. Defaults to 1 day.
      wreplsrv_tombstone_timeout: '86400'
# This is the interval in seconds till we verify active replica records with
# the owning WINS server. Unfortunately not implemented yet. Defaults to 24 day.
      wreplsrv_verify_interval: '2073600'
# This is ignored if async smb echo handler is set, because this feature is
# incompatible with raw write SMB requests. If enabled, raw writes allow writes
# of 65535 bytes in one packet. This typically provides a major performance
# benefit for some very, very old clients. However, some clients either
# negotiate the allowable block size incorrectly or are incapable of
# supporting larger block sizes, and for these clients you may need to
# disable raw writes. In general this parameter should be viewed as a system
# tuning tool and left severely alone. Default is 'yes'.
      write_raw: 'yes'
# It specifies a directory pathname that is used to store the wtmp or wtmpx
# files (depending on the UNIX system) that record user connections to a Samba
# server. The difference with the utmp directory is the fact that user info is
# kept after a user has logged out. By default this is not set, meaning the
# system will use whatever utmp file the native system is set to use
# (usually /var/run/wtmp on Linux).
      wtmp_directory: '/var/log/wtmp'
## SHARES ##
    shares:
    - name: 'share1'
# If this parameter is 'yes' for a service, then the share hosted by the
# service will only be visible to users who have read or write access to the
# share during share enumeration (for example net view \\sambaserver). The
# share ACLs which allow or deny the access to the share can be modified using
# for example the sharesec command or using the appropriate Windows tools. This
# has parallels to access based enumeration, the main difference being that
# only share permissions are evaluated, and security descriptors on files
# contained on the share are not used in computing enumeration access rights.
# Default is 'no'.
      access_based_share_enum: 'no'
# This boolean parameter controls the behaviour of smbd when receiving a
# protocol request of "open for execution" from a Windows client. With Samba
# 3.6 and older, the execution right in the ACL was not checked, so a client
# could execute a file even if it did not have execute rights on the file. In
# Samba 4.0, this has been fixed, so that by default, i.e. when this parameter
# is set to 'no', "open for execution" is now denied when execution permissions
# are not present. If this parameter is set to 'yes', Samba does not check
# execute permissions on "open for execution", thus re-establishing the
# behaviour of Samba 3.6. This can be useful to smoothen upgrades from older
# Samba versions to 4.0 and newer. This setting is not meant to be used as a
# permanent setting, but as a temporary relief: It is recommended to fix the
# permissions in the ACLs and reset this parameter to the default after a
# certain transition period.
      acl_allow_execute_always: 'no'
# This boolean parameter controls what smbd does on receiving a protocol
# request of "open for delete" from a Windows client. If a Windows client
# doesn't have permissions to delete a file then they expect this to be denied
# at open time. POSIX systems normally only detect restrictions on delete by
# actually attempting to delete the file or directory. As Windows clients
# can (and do) "back out" a delete request by unsetting the "delete on close"
# bit Samba cannot delete the file immediately on "open for delete" request as
# we cannot restore such a deleted file. With this parameter set to 'yes' (the
# default) then smbd checks the file system permissions directly on
# "open for delete" and denies the request without actually deleting the file
# if the file system permissions would seem to deny it. This is not perfect, as
# it's possible a user could have deleted a file without Samba being able to
# check the permissions correctly, but it is close enough to Windows semantics
# for mostly correct behaviour. Samba will correctly check POSIX ACL semantics
# in this case. If this parameter is set to 'no' Samba doesn't check
# permissions on "open for delete" and allows the open. If the user doesn't
# have permission to delete the file this will only be discovered at close time,
# which is too late for the Windows user tools to display an error message to
# the user. The symptom of this is files that appear to have been deleted
# "magically" re-appearing on a Windows explorer refresh. This is an extremely
# advanced protocol option which should not need to be changed.
      acl_check_permissions: 'yes'
# In a POSIX filesystem, only the owner of a file or directory and the
# superuser can modify the permissions and ACLs on a file. If this parameter is
# set, then Samba overrides this restriction, and also allows the primary group
# owner of a file or directory to modify the permissions and ACLs on that file.
# On a Windows server, groups may be the owner of a file or directory - thus
# allowing anyone in that group to modify the permissions on it. This allows
# the delegation of security controls on a point in the filesystem to the group
# owner of a directory and anything below it also owned by that group. This
# means there are multiple people with permissions to modify ACLs on a file or
# directory, easing manageability. #This parameter allows Samba to also permit
# delegation of the control over a point in the exported directory hierarchy in
# much the same way as Windows. This allows all members of a UNIX group to
# control the permissions on a file or directory they have group ownership on.
# This parameter is best used with the inherit owner option and also on a share
# containing directories with the UNIX setgid bit set on them, which causes new
# files and directories created within it to inherit the group ownership from
# the containing directory. Default is 'no'.
      acl_group_control: 'no'
# This boolean parameter controls whether smbd maps a POSIX ACE entry of "rwx"
# (read/write/execute), the maximum allowed POSIX permission set, into a
# Windows ACL of "FULL CONTROL". If this parameter is set to true any POSIX ACE
# entry of "rwx" will be returned in a Windows ACL as "FULL CONTROL", is this
# parameter is set to false any POSIX ACE entry of "rwx" will be returned as the
# specific Windows ACL bits representing read, write and execute.
# Default is 'yes'.
      acl_map_full_control: 'yes'
# If this parameter is set to 'yes' for a share, then the share will be an
# administrative share. The Administrative Shares are the default network
# shares created by all Windows NT-based operating systems. These are shares
# like C$, D$ or ADMIN$. The type of these shares is STYPE_DISKTREE_HIDDEN. See
# the section below on security for more information about this option.
# Default is 'no'.
      administrative_share: 'no'
# This is a list of users who will be granted administrative privileges on the
# share. This means that they will do all file operations as the super-user
# (root). You should use this option very carefully, as any user in this list
# will be able to do anything they like on the share, irrespective of file
# permissions. Default is None.
      admin_users: 'bobcat'
# This parameter controls whether special AFS features are enabled for this
# share. If enabled, it assumes that the directory exported via the path
# parameter is a local AFS import. The special AFS features include the attempt
# to hand-craft an AFS token if you enabled --with-fake-kaserver in configure.
# Default is 'no'.
      afs_share: 'no'
# If this integer parameter is set to a non-zero value, Samba will read from
# files asynchronously when the request size is bigger than this value. Note
# that it happens only for non-chained and non-chaining reads and when not
# using write cache. The only reasonable values for this parameter are '0' (no
# async I/O) and '1' (always do async I/O). Default is '1'.
      aio_read_size: '1'
# If Samba has been built with asynchronous I/O support, Samba will not wait
# until write requests are finished before returning the result to the client
# for files listed in this parameter. Instead, Samba will immediately return
# that the write request has been finished successfully, no matter if the
# operation will succeed or not. This might speed up clients without aio
# support, but is really dangerous, because data could be lost and files could
# be damaged. Default is None.
      aio_write_behind: '/*.tmp/'
# If this integer parameter is set to a non-zero value, Samba will write to
# files asynchronously when the request size is bigger than this value. Note
# that it happens only for non-chained and non-chaining reads and when not
# using write cache. The only reasonable values for this parameter are '0' (no
# async I/O) and '1' (always do async I/O). Compared to 'aio_read_size' this
# parameter has a smaller effect, most writes should end up in the file system
# cache. Writes that require space allocation might benefit most from going
# asynchronous. Default is '1'.
      aio_write_size: '1'
# This parameter allows an administrator to tune the allocation size reported
# to Windows clients. The default size of 1Mb generally results in improved
# Windows client performance. However, rounding the allocation size may cause
# difficulties for some applications, e.g. MS Visual Studio. If the MS Visual
# Studio compiler starts to crash with an internal error, set this parameter to
# zero for this share. The integer parameter specifies the roundup size in
# bytes. Default is '1048576'.
      allocation_roundup_size: '1048576'
# This parameter controls the behavior of smbd when given a request by a client
# to obtain a byte range lock on a region of an open file, and the request has
# a time limit associated with it. If this parameter is set and the lock range
# requested cannot be immediately satisfied, samba will internally queue the
# lock request, and periodically attempt to obtain the lock until the timeout
# period expires. If this parameter is set to 'no', then samba will behave as
# previous versions of Samba would and will fail the lock request immediately
# if the lock range cannot be obtained. Default is 'yes'.
      blocking_locks: 'yes'
# This parameter controls the behavior of smbd when reporting disk free sizes.
# By default, this reports a disk block size of 1024 bytes. Changing this
# parameter may have some effect on the efficiency of client writes, this is
# not yet confirmed. This parameter was added to allow advanced administrators
# to change it (usually to a higher value) and test the effect it has on client
# write performance without re-compiling the code. As this is an experimental
# option it may be removed in a future release. Changing this option does not
# change the disk free reporting size, just the block size unit reported to the
# client. Default is 1024.
      block_size: '1024'
# This controls whether this share is seen in the list of available shares in a
# net view and in the browse list. Default is 'yes'.
      browseable: 'yes'
# Controls whether filenames are case sensitive. If they aren't, Samba must do
# a filename search and match on passed names. The default setting of auto
# allows clients that support case sensitive filenames (Linux CIFSVFS and
# smbclient 3.0.5 and above currently) to tell the Samba server on a per-packet
# basis that they wish to access the file system in a case-sensitive manner (to
# support UNIX case sensitive semantics). No Windows or DOS system supports
# case-sensitive filename so setting this option to auto is that same as
# setting it to no for them. May be 'yes', 'no', 'auto'. Default is 'auto'.
      case_sensitive: 'auto'
# A Windows SMB server prevents the client from creating files in a directory
# that has the delete-on-close flag set. By default Samba doesn't perform this
# check as this check is a quite expensive operation in Samba.
      check_parent_directory_delete_on_close: 'no'
# This parameter allows you to "clone" service entries. The specified service
# is simply duplicated under the current service's name. Any parameters
# specified in the current section will override those in the section being
# copied. This feature lets you set up a 'template' service and create similar
# services easily. Note that the service being copied must occur earlier in the
# configuration file than the service doing the copying. Default is None.
      copy: 'share1'
# When a file is created, the necessary permissions are calculated according to
# the mapping from DOS modes to UNIX permissions, and the resulting UNIX mode
# is then bit-wise 'AND'ed with this parameter. This parameter may be thought
# of as a bit-wise MASK for the UNIX modes of a file. Any bit not set here will
# be removed from the modes set on a file when it is created. The default value
# of this parameter removes the group and other write and execute bits from the
# UNIX modes. Following this Samba will bit-wise 'OR' the UNIX mode created
# from this parameter with the value of the force create mode parameter which
# is set to 000 by default. This parameter does not affect directory masks.
# Default is '0744'.
      create_mask: '0744'
# This stands for client-side caching policy, and specifies how clients capable
# of offline caching will cache the files in the share. The valid values are:
# 'manual', 'documents', 'programs', 'disable'. These values correspond to
# those used on Windows servers. For example, shares containing roaming
# profiles can have offline caching disabled using 'disable'.
# Default is 'manual'.
      csc_policy: 'manual'
# This parameter is only applicable if printing is set to cups. Its value is a
# free form string of options passed directly to the cups library. You can pass
# any generic print option known to CUPS. You can also pass any printer
# specific option valid for the target queue. Multiple parameters should be
# space-delimited name/value pairs according to the PAPI text option ABNF
# specification. Collection values ("name={a=... b=... c=...}") are stored with
# the curley brackets intact. You should set this parameter to raw if your CUPS
# server error_log file contains messages such as
# "Unsupported format 'application/octet-stream'" when printing from a Windows
# client through Samba. Default is None.
      cups_options: 'raw media=a4'
# See the section on name mangling. Also note the short preserve case parameter.
      default_case: 'lower'
# This parameter is only applicable to printable services. When smbd is
# serving Printer Drivers to Windows NT/2k/XP clients, each printer on the
# Samba server has a Device Mode which defines things such as paper size and
# orientation and duplex settings. The device mode can only correctly be
# generated by the printer driver itself (which can only be executed on a
# Win32 platform). Because smbd is unable to execute the driver code to
# generate the device mode, the default behavior is to set this field to NULL.
# Most problems with serving printer drivers to Windows NT/2k/XP clients can be
# traced to a problem with the generated device mode. Certain drivers will do
# things such as crashing the client's Explorer.exe with a NULL devmode.
# However, other printer drivers can cause the client's spooler service
# (spoolsv.exe) to die if the devmode was not created by the driver itself
# (i.e. smbd generates a default devmode). This parameter should be used with
# care and tested with the printer driver in question. It is better to leave
# the device mode to NULL and let the Windows client set the correct values.
# Because drivers do not do this all the time, setting this to 'yes' will
# instruct smbd to generate a default one. For more information on Windows
# NT/2k printing and Device Modes, see the MSDN documentation.
      default_devmode: 'yes'
# This parameter allows readonly files to be deleted. This is not normal DOS
# semantics, but is allowed by UNIX. This option may be useful for running
# applications such as rcs, where UNIX file ownership prevents changing file
# permissions, and DOS semantics prevent deletion of a read only file. Default
# is 'no'.
      delete_readonly: 'no'
# This option is used when Samba is attempting to delete a directory that
# contains one or more vetoed directories. If this option is set to 'no' (the
# default) then if a vetoed directory contains any non-vetoed files or
# directories then the directory delete will fail. This is usually what you
# want. If this option is set to 'yes', then Samba will attempt to recursively
# delete any files and directories within the vetoed directory. This can be
# useful for integration with file serving systems such as NetAtalk which
# create meta-files within directories you might normally veto DOS/Windows
# users from seeing (e.g. '.AppleDouble'). Setting to 'yes' allows these
# directories to be transparently deleted when the parent directory is deleted
# (so long as the user has permissions to do so).
      delete_veto_files: 'no'
# The dfree cache time should only be used on systems where a problem occurs
# with the internal disk space calculations. This has been known to happen with
# Ultrix, but may occur with other operating systems. The symptom that was seen
# was an error of "Abort Retry Ignore" at the end of each directory listing.
# This is a new parameter introduced in Samba version 3.0.21. It specifies in
# seconds the time that smbd will cache the output of a disk free query. If set
# to zero (the default) no caching is done. This allows a heavily loaded server
# to prevent rapid spawning of dfree command scripts increasing the load. By
# default this parameter is zero, meaning no caching will be done.
      dfree_cache_time: '0'
# The dfree command setting should only be used on systems where a problem
# occurs with the internal disk space calculations. This has been known to
# happen with Ultrix, but may occur with other operating systems. The symptom
# that was seen was an error of "Abort Retry Ignore" at the end of each
# directory listing. This setting allows the replacement of the internal
# routines to calculate the total disk space and amount available with an
# external routine. The example below gives a possible script that might fulfill
# this function. In Samba version 3.0.21 this parameter has been changed to be
# a per-share parameter, and in addition the parameter dfree cache time was
# added to allow the output of this script to be cached for systems under heavy
# load. The external program will be passed a single parameter indicating a
# directory in the filesystem being queried. This will typically consist of the
# string './'. The script should return two integers in ASCII. The first should
# be the total disk space in blocks, and the second should be the number of
# available blocks. An optional third return value can give the block size in
# bytes. The default blocksize is 1024 bytes. Note: Your script should NOT be
# setuid or setgid and should be owned by (and writeable only by) root! Note
# that you may have to replace the command names with full path names on some
# systems. Also note the arguments passed into the script should be quoted
# inside the script in case they contain special characters such as spaces or
# newlines. By default internal routines for determining the disk capacity and
# remaining space will be used. Default is None.
      dfree_command: '/usr/local/samba/bin/dfree'
# This parameter is the octal modes which are used when converting DOS modes to
# UNIX modes when creating UNIX directories. When a directory is created, the
# necessary permissions are calculated according to the mapping from DOS modes
# to UNIX permissions, and the resulting UNIX mode is then bit-wise 'AND'ed
# with this parameter. This parameter may be thought of as a bit-wise MASK for
# the UNIX modes of a directory. Any bit not set here will be removed from the
# modes set on a directory when it is created. The default value of this
# parameter removes the 'group' and 'other' write bits from the UNIX mode,
# allowing only the user who owns the directory to modify it. Following this
# Samba will bit-wise 'OR' the UNIX mode created from this parameter with the
# value of the force directory mode parameter. This parameter is set to 000 by
# default (i.e. no extra mode bits are added). Default is '0755'.
      directory_mask: '0755'
# This parameter specifies the size of the directory name cache for SMB1
# connections. It is not used for SMB2. It will be needed to turn this off for
# *BSD systems. Default is '100'.
      directory_name_cache_size: '100'
# This parameter specifies whether Samba should use DMAPI to determine whether
# a file is offline or not. This would typically be used in conjunction with a
# hierarchical storage system that automatically migrates files to tape.
# Note that Samba infers the status of a file by examining the events that a
# DMAPI application has registered interest in. This heuristic is satisfactory
# for a number of hierarchical storage systems, but there may be system for
# which it will fail. In this case, Samba may erroneously report files to be
# offline. This parameter is only available if a supported DMAPI implementation
# was found at compilation time. It will only be used if DMAPI is found to
# enabled on the system at run time. Default is 'no'.
      dmapi_support: 'no'
# There are certain directories on some systems (e.g., the '/proc' tree under
# Linux) that are either not of interest to clients or are infinitely deep
# (recursive). This parameter allows you to specify a comma-delimited list of
# directories that the server should always show as empty. Note that Samba can
# be very fussy about the exact format of the "dont descend" entries. For
# example you may need './proc' instead of just '/proc'. Default is empty.
      dont_descend: '/proc,/dev'
# The default behavior in Samba is to provide UNIX-like behavior where only the
# owner of a file/directory is able to change the permissions on it. However,
# this behavior is often confusing to DOS/Windows users. Enabling this
# parameter allows a user who has write access to the file (by whatever means,
# including an ACL permission) to modify the permissions (including ACL) on it.
# Note that a user belonging to the group owning the file will not be allowed
# to change permissions if the group is only granted read access. Ownership of
# the file/directory may also be changed. Note that using the VFS modules
# 'acl_xattr' or 'acl_tdb' which store native Windows as meta-data will
# automatically turn this option on for any share for which they are loaded, as
# they require this option to emulate Windows ACLs correctly.
      dos_filemode: 'no'
# Under the DOS and Windows FAT filesystem, the finest granularity on time
# resolution is two seconds. Setting this parameter for a share causes Samba to
# round the reported time down to the nearest two second boundary when a query
# call that requires one second resolution is made to smbd. This option is
# mainly used as a compatibility option for Visual C++ when used against Samba
# shares. If oplocks are enabled on a share, Visual C++ uses two different time
# reading calls to check if a file has changed since it was last read. One of
# these calls uses a one-second granularity, the other uses a two second
# granularity. As the two second call rounds any odd second down, then if the
# file has a timestamp of an odd number of seconds then the two timestamps will
# not match and Visual C++ will keep reporting the file has changed. Setting
# this option causes the two timestamps to match, and Visual C++ is happy.
      dos_filetime_resolution: 'no'
# Under DOS and Windows, if a user can write to a file they can change the
# timestamp on it. Under POSIX semantics, only the owner of the file or root
# may change the timestamp. By default, Samba emulates the DOS semantics and
# allows one to change the timestamp on a file if the user smbd is acting on
# behalf has write permissions. Due to changes in Microsoft Office 2000 and
# beyond. Microsoft Excel will display dialog box warnings about the file being
# changed by another user if this parameter is not set to 'yes' and files are
# being shared between users. Default is 'yes'.
      dos_filetimes: 'yes'
# This boolean parameter controls whether Samba can grant SMB2 durable file
# handles on a share. Note that durable handles are only enabled if
# 'kernel_oplocks' is 'no', 'kernel_share_modes' is 'no', and 'posix_locking' is
# no, i.e. if the share is configured for CIFS/SMB2 only access, not supporting
# interoperability features with local UNIX processes or NFS operations. Also
# note that, for the time being, durability is not granted for a handle that
# has the delete on close flag set. Default is 'yes'.
      durable_handles: 'yes'
# This boolean parameter controls whether smbd will allow clients to attempt to
# access extended attributes on a share. In order to enable this parameter on a
# setup with default VFS modules:
# * Samba must have been built with extended attributes support.
# * The underlying filesystem exposed by the share must support extended
# attributes (e.g. the getfattr(1) / setfattr(1) utilities must work).
# Note that the SMB protocol allows setting attributes whose value is 64K bytes
# long, and that on NTFS, the maximum storage space for extended attributes per
# file is 64K. On most UNIX systems (Solaris and ZFS file system being the
# exception), the limits are much lower - typically 4K. Worse, the same 4K
# space is often used to store system metadata such as POSIX ACLs, or Samba's NT
# ACLs. Giving clients access to this tight space via extended attribute support
# could consume all of it by unsuspecting client applications, which would
# prevent changing system metadata due to lack of space. Default is 'yes'.
      ea_support: 'yes'
# NTFS and Windows VFAT file systems keep a create time for all files and
# directories. This is not the same as the ctime - status change time - that
# Unix keeps, so Samba by default reports the earliest of the various times
# Unix does keep. Setting this parameter for a share causes Samba to always
# report midnight 1-1-1980 as the create time for directories. This option is
# mainly used as a compatibility option for Visual C++ when used against Samba
# shares. Visual C++ generated makefiles have the object directory as a
# dependency for each object file, and a make rule to create the directory.
# Also, when NMAKE compares timestamps it uses the creation time when examining
# a directory. Thus the object directory will be created if it does not exist,
# but once it does exist it will always have an earlier timestamp than the
# object files it contains. However, Unix time semantics mean that the create
# time reported by Samba will be updated whenever a file is created or deleted
# in the directory. NMAKE finds all object files in the object directory. The
# timestamp of the last one built is then compared to the timestamp of the
# object directory. If the directory's timestamp if newer, then all object
# files will be rebuilt. Enabling this option ensures directories always predate
# their contents and an NMAKE build will proceed as expected. Default is 'no'.
      fake_directory_create_times: 'no'
# Oplocks are the way that SMB clients get permission from a server to locally
# cache file operations. If a server grants an oplock (opportunistic lock) then
# the client is free to assume that it is the only one accessing the file and it
# will aggressively cache file data. With some oplock types the client may even
# cache file open/close operations. This can give enormous performance benefits.
# When you set to 'yes', smbd will always grant oplock requests no matter how
# many clients are using the file. It is generally much better to use the real
# oplocks support rather than this parameter. If you enable this option on all
# read-only shares or shares that you know will only be accessed from one
# client at a time such as physically read-only media like CDROMs, you will see
# a big performance improvement on many operations. If you enable this option
# on shares where multiple clients may be accessing the files read-write at the
# same time you can get data corruption. Use this option carefully!
# Default is 'no'.
      fake_oplocks: 'no'
# This parameter allows the Samba administrator to stop smbd from following
# symbolic links in a particular share. Setting this parameter to no prevents
# any file or directory that is a symbolic link from being followed (the user
# will get an error). This option is very useful to stop users from adding a
# symbolic link to /etc/passwd in their home directory for instance. However it
# will slow filename lookups down slightly. This option is enabled (i.e. smbd
# will follow symbolic links) by default.
      follow_symlinks: 'yes'
# This parameter specifies a set of UNIX mode bit permissions that will always
# be set on a file created by Samba. This is done by bitwise 'OR'ing these bits
# onto the mode bits of a file that is being created. The default for this
# parameter is (in octal) 000. The modes in this parameter are bitwise 'OR'ed
# onto the file mode after the mask set in the create mask parameter is applied.
# The example below would force all newly created files to have read and
# execute permissions set for 'group' and 'other' as well as the
# read/write/execute bits set for the 'user'.
      force_create_mode: '0000'
# This parameter specifies a set of UNIX mode bit permissions that will always
# be set on a directory created by Samba. This is done by bitwise 'OR'ing these
# bits onto the mode bits of a directory that is being created. The default for
# this parameter is (in octal) 0000 which will not add any extra permission bits
# to a created directory. This operation is done after the mode mask in the
# parameter directory mask is applied. The example below would force all
# created directories to have read and execute permissions set for 'group' and
# 'other' as well as the read/write/execute bits set for the 'user'.
      force_directory_mode: '0000'
# This specifies a UNIX group name that will be assigned as the default primary
# group for all users connecting to this service. This is useful for sharing
# files by ensuring that all access to files on service will use the named
# group for their permissions checking. Thus, by assigning permissions for this
# group to the files and directories within this service the Samba administrator
# can restrict or allow sharing of these files. In Samba 2.0.5 and above this
# parameter has extended functionality in the following way. If the group name
# listed here has a '+' character prepended to it then the current user
# accessing the share only has the primary group default assigned to this group
# if they are already assigned as a member of that group. This allows an
# administrator to decide that only users who are already in a particular group
# will create files with group ownership set to that group. This gives a finer
# granularity of ownership assignment. For example, the setting
# "'force_group': '+sys'" means that only users who are already in group 'sys'
# will have their default primary group assigned to sys when accessing this
# Samba share. All other users will retain their ordinary primary group. If the
# force user parameter is also set the group specified in force group will
# override the primary group set in force user. Default is None.
      force_group: ''
# When printing from Windows NT (or later), each printer in smb.conf has two
# associated names which can be used by the client. The first is the sharename
# (or shortname) defined in smb.conf. This is the only printername available for
# use by Windows 9x clients. The second name associated with a printer can be
# seen when browsing to the "Printers" (or "Printers and Faxes") folder on the
# Samba server. This is referred to simply as the printername (not to be
# confused with the printer name option). When assigning a new driver to a
# printer on a remote Windows compatible print server such as Samba, the Windows
# client will rename the printer to match the driver name just uploaded. This
# can result in confusion for users when multiple printers are bound to the
# same driver. To prevent Samba from allowing the printer's printername to
# differ from the sharename defined in smb.conf, set option to 'yes'. Be aware
# that enabling this parameter may affect migrating printers from a Windows
# server to Samba since Windows has no way to force the sharename and
# printername to match. It is recommended that this parameter's value not be
# changed once the printer is in use by clients as this could cause a user not
# be able to delete printer connections from their local Printers folder.
      force_printername: 'no'
# If this parameter is set, a Windows NT ACL that contains an unknown SID
# (security descriptor, or representation of a user or group id) as the owner
# or group owner of the file will be silently mapped into the current UNIX uid
# or gid of the currently connected user. This is designed to allow Windows NT
# clients to copy files and folders containing ACLs that were created locally
# on the client machine and contain users local to that machine only (no domain
# users) to be copied to a Samba server (usually with XCOPY /O) and have the
# unknown userid and groupid of the file owner map to the current connected
# user. This can only be fixed correctly when winbindd allows arbitrary mapping
# from any Windows NT SID to a UNIX uid or gid. Try using this parameter when
# XCOPY /O gives an ACCESS_DENIED error. Default is 'no'.
      force_unknown_acl_user: 'no'
# This specifies a UNIX user name that will be assigned as the default user for
# all users connecting to this service. This is useful for sharing files. You
# should also use it carefully as using it incorrectly can cause security
# problems. This user name only gets used once a connection is established.
# Thus clients still need to connect as a valid user and supply a valid
# password. Once connected, all file operations will be performed as the
# "forced user", no matter what username the client connected as. This can be
# very useful. Default is None.
      force_user: ''
# This parameter allows the administrator to configure the string that
# specifies the type of filesystem a share is using that is reported by smbd
# when a client queries the filesystem type for a share. The default type is
# NTFS for compatibility with Windows NT but this can be changed to other
# strings such as Samba or FAT if required.
      fstype: 'Samba'
# If this parameter is 'yes' for a service, then no password is required to
# connect to the service. Privileges will be those of the guest account.
# Default is 'no'.
      guest_ok: 'no'
# If this parameter is 'yes' for a service, then only guest connections to the
# service are permitted. This parameter will have no effect if 'guest_ok' is not
# set for the service. Default is 'no'.
      guest_only: 'no'
# This is a boolean parameter that controls whether files starting with a dot
# appear as hidden files. Default is 'yes'.
      hide_dot_files: 'yes'
# This is a list of files or directories that are not visible but are
# accessible. The DOS 'hidden' attribute is applied to any files or directories
# that match. Each entry in the list must be separated by a '/', which allows
# spaces to be included in the entry. '*' and '?' can be used to specify
# multiple files or directories as in DOS wildcards. Each entry must be a
# Unix path, not a DOS path and must not include the Unix directory separator
# '/'. Note that the case sensitivity option is applicable in hiding files.
# Setting this parameter will affect the performance of Samba, as it will be
# orced to check all files and directories for a match as they are scanned.
# The example shown above is based on files that the Macintosh SMB client
# (DAVE) available from Thursby creates for internal use, and also still hides
# all files beginning with a dot. Default is None.
      hide_files: ''
# Setting this parameter to something but 0 hides files that have been modified
# less than N seconds ago. It can be used for ingest/process queue style
# workloads. A processing application should only see files that are definitely
# finished. As many applications do not have proper external workflow control,
# this can be a way to make sure processing does not interfere with file
# ingest. Default is '0'.
      hide_new_files_timeout: '0'
# This parameter prevents clients from seeing special files such as sockets,
# devices and fifo's in directory listings. Default is 'no'.
      hide_special_files: 'no'
# This parameter prevents clients from seeing the existence of files that
# cannot be read. Defaults to off. Please note that enabling this can slow down
# listing large directories significantly. Samba has to evaluate the ACLs of all
# directory members, which can be a lot of effort.
      hide_unreadable: 'no'
# This parameter prevents clients from seeing the existence of files that cannot
# be written to. Defaults to off. Note that unwriteable directories are shown
# as usual. Please note that enabling this can slow down listing large
# directories significantly. Samba has to evaluate the ACLs of all directory
# members, which can be a lot of effort.
      hide_unwriteable_files: 'no'
      hosts_allow:
# You can specify the hosts by name or IP number. For example, you could
# restrict access to only the hosts on a Class C subnet. The full syntax of the
# list is described in the man page hosts_access. Note that the localhost
# address 127.0.0.1 will always be allowed access unless specifically denied by
# a hosts deny option. You can also specify hosts by network/netmask pairs and
# by netgroup names if your system supports netgroups. The EXCEPT keyword can
# also be used to limit a wildcard list. Default is None (i.e., all hosts
# permitted access).
# The following examples may provide some help:
#
# Example 1: allow all IPs in 150.203.*.*; except one
      - '150.203.'
      - 'EXCEPT 150.203.6.66'
# Example 2: allow hosts that match the given network/netmask
      - '150.203.15.0/255.255.255.0'
# Example 3: allow a couple of hosts
      - 'lapland, arvidsjaur'
# Example 4: allow only hosts in NIS netgroup "foonet"
      - '@foonet'
      hosts_deny:
# The opposite of 'hosts_allow' - hosts listed here are NOT permitted access to
# services unless the specific services have their own lists to override this
# one. Where the lists conflict, the allow list takes precedence. In the event
# that it is necessary to deny all by default, use the keyword 'ALL' (or the
# netmask '0.0.0.0/0') and then explicitly specify to the 'hosts_allow' allow
# parameter those hosts that should be permitted access. Default is None.
      - '150.203.4.'
      - 'badhost.mynet.edu.au'
# This parameter can be used to ensure that if default acls exist on parent
# directories, they are always honored when creating a new file or subdirectory
# in these parent directories. The default behavior is to use the unix mode
# specified when creating the directory. Enabling this option sets the unix
# mode to 0777, thus guaranteeing that default directory acls are propagated.
# Note that using the VFS modules acl_xattr or acl_tdb which store native
# Windows as meta-data will automatically turn this option 'on' for any share
# for which they are loaded, as they require this option to emulate Windows
# ACLs correctly. Default is 'no'.
      inherit_acls: 'no'
# The ownership of new files and directories is normally governed by effective
# uid of the connected user. This option allows the Samba administrator to
# specify that the ownership for new files and directories should be controlled
# by the ownership of the parent directory. Valid options are:
# 'no' - Both the Windows (SID) owner and the UNIX (uid) owner of the file are
# governed by the identity of the user that created the file.
# 'windows and unix' - The Windows (SID) owner and the UNIX (uid) owner of new
# files and directories are set to the respective owner of the parent directory.
# 'yes' - a synonym for 'windows and unix'.
# 'unix only' - Only the UNIX owner is set to the UNIX owner of the parent
# directory.
# Common scenarios where this behavior is useful is in implementing drop-boxes,
# where users can create and edit files but not delete them and ensuring that
# newly created files in a user's roaming profile directory are actually owned
# by the user.
# The 'unix only' option effectively breaks the tie between the Windows owner
# of a file and the UNIX owner. As a logical consequence, in this mode, setting
# the the Windows owner of a file does not modify the UNIX owner. Using this
# mode should typically be combined with a backing store that can emulate the
# full NT ACL model without affecting the POSIX permissions, such as the
# acl_xattr VFS module, coupled with acl_xattr:ignore system acls = yes. This
# can be used to emulate folder quotas, when files are exposed only via SMB
# (without UNIX extensions). The UNIX owner of a directory is locally set and
# inherited by all subdirectories and files, and they all consume the same
# quota. Default is 'no'.
      inherit_owner: 'no'
# The permissions on new files and directories are normally governed by
# 'create_mask', 'directory_mask', 'force_create_mode' and
# 'force_directory_mode' but the boolean inherit permissions parameter overrides
# this. New directories inherit the mode of the parent directory, including bits
# such as setgid. New files inherit their read/write bits from the parent
# directory. Their execute bits continue to be determined by map archive, map
# hidden and map system as usual. Note that the setuid bit is never set via
# inheritance (the code explicitly prohibits this). This can be particularly
# useful on large systems with many users, perhaps several thousand, to allow a
# single [homes] share to be used flexibly by each user. Default is 'no'.
      inherit_permissions: 'no'
# This is a list of users that should not be allowed to login to this service.
# This is really a paranoid check to absolutely ensure an improper setting does
# not breach your security. A name starting with a '@' is interpreted as an NIS
# netgroup first (if your system supports NIS), and then as a UNIX group if the
# name was not found in the NIS netgroup database. A name starting with '+' is
# interpreted only by looking in the UNIX group database via the NSS getgrnam()
# interface. A name starting with '&' is interpreted only by looking in the NIS
# netgroup database (this requires NIS to be working on your system). The
# characters '+' and '&' may be used at the start of the name in either order
# so the value '+&group' means check the UNIX group database, followed by the
# NIS netgroup database, and the value '&+group' means check the NIS netgroup
# database, followed by the UNIX group database (the same as the '@' prefix).
# The current servicename is substituted for %S. This is useful in the [homes]
# section.
      invalid_users:
      - 'root'
      - 'fred'
      - 'admin'
      - '@wheel'
# For UNIXes that support kernel based oplocks (currently only Linux), this
# parameter allows the use of them to be turned on or off. However, this
# disables Level II oplocks for clients as the Linux kernel does not support
# them properly. Kernel oplocks support allows Samba oplocks to be broken
# whenever a local UNIX process or NFS operation accesses a file that smbd has
# oplocked. This allows complete data consistency between SMB/CIFS, NFS and
# local file access (and is a very cool feature). If you do not need this
# interaction, you should disable the parameter on Linux to get Level II oplocks
# and the associated performance benefit. This parameter defaults to 'no' and
# is translated to a no-op on systems that do not have the necessary kernel
# support.
      kernel_oplocks: 'no'
# This parameter controls whether SMB share modes are translated into UNIX
# flocks. Kernel share modes provide a minimal level of interoperability with
# local UNIX processes and NFS operations by preventing access with flocks
# corresponding to the SMB share modes. Generally, it is very desirable to
# leave this enabled. Note that in order to use SMB2 durable file handles on a
# share, you have to turn it to 'off'. This parameter defaults to 'yes' and is
# translated to a no-op on systems that do not have the necessary kernel flock
# support.
      kernel_share_modes: 'yes'
# This parameter controls whether Samba supports level2 (read-only) oplocks on a
# share. Level2, or read-only oplocks allow Windows NT clients that have an
# oplock on a file to downgrade from a read-write oplock to a read-only oplock
# once a second client opens the file (instead of releasing all oplocks on a
# second open, as in traditional, exclusive oplocks). This allows all openers
# of the file that support level2 oplocks to cache the file for read-ahead only
# (ie. they may not cache writes or lock requests) and increases performance for
# many accesses of files that are not commonly written. Once one of the clients
# which have a read-only oplock writes to the file all clients are notified
# (no reply is needed or waited for) and told to break their oplocks to "none"
# and delete any read-ahead caches. It is recommended that this parameter be
# turned on to speed access to shared executables. For more discussions on
# level2 oplocks see the CIFS spec. Currently, if kernel oplocks are supported
# then level2 oplocks are not granted (even if this parameter is set to 'yes').
# Note also, the oplocks parameter must be set to 'yes' on this share in order
# for this parameter to have any effect.
      level2_oplocks: 'yes'
# This controls whether or not locking will be performed by the server in
# response to lock requests from the client. If locking is 'no', all lock and
# unlock requests will appear to succeed and all lock queries will report that
# the file in question is available for locking. If locking is 'yes' (the
# default), real locking will be performed by the server. This option may be
# useful for read-only filesystems which may not need locking (such as CDROM
# drives), although setting this parameter of no is not really recommended even
# in this case. Be careful about disabling locking either globally or in a
# specific service, as lack of locking may result in data corruption. You
# should never need to set this parameter.
      locking: 'yes'
# This parameter specifies the command to be executed on the server host in
# order to stop printing or spooling a specific print job. This command should
# be a program or script which takes a printer name and job number to pause the
# print job. One way of implementing this is by using job priorities, where jobs
# having a too low priority won't be sent to the printer. If a '%p' is given
# then the printer name is put in its place. A '%j' is replaced with the job
# number (an integer). Note that it is good practice to include the absolute
# path in the lppause command as the PATH may not be available to the server.
# Currently no default value is given to this string.
      lppause_command: '/usr/bin/lpalt %p-%j -p0'
# This parameter specifies the command to be executed on the server host in
# order to obtain lpq-style printer status information. This command should be
# a program or script which takes a printer name as its only parameter and
# outputs printer status information. Some clients (notably Windows for
# Workgroups) may not correctly send the connection number for the printer they
# are requesting status information about. To get around this, the server
# reports on the first printer service connected to by the client. This only
# happens if the connection number sent is invalid. If a '%p' is given then the
# printer name is put in its place. Otherwise it is placed at the end of the
# command. Note that it is good practice to include the absolute path in the
# 'lpq_command' as the $PATH may not be available to the server. When compiled
# with the CUPS libraries, no 'lpq_command' is needed because smbd will make a
# library call to obtain the print queue listing. Default is None.
      lpq_command: '/usr/bin/lpq -P%p'
# This parameter specifies the command to be executed on the server host in
# order to restart or continue printing or spooling a specific print job.
# This command should be a program or script which takes a printer name and job
# number to resume the print job. See also the lppause command parameter.
# If a '%p' is given then the printer name is put in its place. A '%j' is
# replaced with the job number (an integer). Note that it is good practice to
# include the absolute path in the lpresume command as the PATH may not be
# available to the server. Default: currently no default value is given to this
# string.
      lpresume_command: '/usr/bin/lpalt %p-%j -p2'
# This parameter specifies the command to be executed on the server host in
# order to delete a print job. This command should be a program or script which
# takes a printer name and job number, and deletes the print job. If a '%p' is
# given then the printer name is put in its place. A '%j' is replaced with the
# job number (an integer). Note that it is good practice to include the
# absolute path in the 'lprm_command' as the PATH may not be available to the
# server.
      lprm_command: '/usr/bin/cancel %p-%j'
# This parameter specifies the name of a file which will contain output created
# by a magic script. If two clients use the same magic script in the same
# directory the output file content is undefined. Default is None.
      magic_output: 'myfile.txt'
# This parameter specifies the name of a file which, if opened, will be
# executed by the server when the file is closed. This allows a UNIX script to
# be sent to the Samba host and executed on behalf of the connected user.
# Scripts executed in this way will be deleted upon completion assuming that the
# user has the appropriate level of privilege and the file permissions allow
# the deletion. If the script generates output, output will be sent to the file
# specified by the magic output parameter (see above). Note that some shells
# are unable to interpret scripts containing CR/LF instead of CR as the
# end-of-line marker. Magic scripts must be executable as is on the host, which
# for some hosts and some shells will require filtering at the DOS end. Magic
# scripts are EXPERIMENTAL and should NOT be relied upon. Default is None.
      magic_script: 'user.csh'
# This controls whether non-DOS names under UNIX should be mapped to
# DOS-compatible names ("mangled") and made visible, or whether non-DOS names
# should simply be ignored. See the section on name mangling for details on how
# to control the mangling process. Possible option settings are
# 'yes' (default) - enables name mangling for all not DOS 8.3 conforming names.
# 'no' - disables any name mangling.
# 'illegal' - does mangling for names with illegal NTFS characters. This is the
# most sensible setting for modern clients that don't use the shortname anymore.
# If mangling is used then the mangling method is as follows:
# * The first (up to) five alphanumeric characters before the rightmost dot of
# the filename are preserved, forced to upper case, and appear as the first
# (up to) five characters of the mangled name.
# * A tilde "~" is appended to the first part of the mangled name, followed by
# a two-character unique sequence, based on the original root name (i.e., the
# original filename minus its final extension). The final extension is included
# in the hash calculation only if it contains any upper case characters or is
# longer than three characters. Note that the character to use may be specified
# using the mangling char option, if you don't like '~'.
# * Files whose UNIX name begins with a dot will be presented as DOS hidden
# files. The mangled name will be created as for other filenames, but with the
# leading dot removed and "___" as its extension regardless of actual original
# extension (that's three underscores). The two-digit hash value consists of
# upper case alphanumeric characters. This algorithm can cause name collisions
# only if files in a directory share the same first five alphanumeric
# characters. The probability of such a clash is 1/1300. The name mangling (if
# enabled) allows a file to be copied between UNIX directories from Windows/DOS
# while retaining the long UNIX filename. UNIX files can be renamed to a new
# extension from Windows/DOS and will retain the same basename. Mangled names
# do not change between sessions.
      mangled_names: 'yes'
# This controls what character is used as the magic character in name mangling.
# The default is a '~' but this may interfere with some software. Use this
# option to set it to whatever you prefer. This is effective only when
# 'mangling_method' is 'hash'.
      mangling_char: '~'
# This boolean parameter controls whether smbd will attempt to map the
# "inherit" and "protected" access control entry flags stored in Windows ACLs
# into an extended attribute called user.SAMBA_PAI. This parameter only takes
# effect if Samba is being run on a platform that supports extended attributes
# (Linux and IRIX so far) and allows the Windows 2000 ACL editor to correctly
# use inheritance with the Samba POSIX ACL mapping code. Default is 'no'.
      map_acl_inherit: 'no'
# This controls whether the DOS archive attribute should be mapped to the UNIX
# owner execute bit. The DOS archive bit is set when a file has been modified
# since its last backup. One motivation for this option is to keep Samba/your
# PC from making any file it touches from becoming executable under UNIX. This
# can be quite annoying for shared source code, documents, etc... Note that
# this parameter will be ignored if the store dos attributes parameter is set,
# as the DOS archive attribute will then be stored inside a UNIX extended
# attribute. Note that this requires the create mask parameter to be set such
# that owner execute bit is not masked out (i.e. it must include 100). See the
# parameter create mask for details. Default is 'yes'.
      map_archive: 'yes'
# This controls whether DOS style hidden files should be mapped to the UNIX
# world execute bit. Note that this parameter will be ignored if the
# 'store_dos_attributes' parameter is set, as the DOS hidden attribute will
# then be stored inside a UNIX extended attribute. Note that this requires the
# create mask to be set such that the world execute bit is not masked out
# (i.e. it must include 001). See the parameter create mask for details.
# Default is 'no'.
      map_hidden: 'no'
# This controls how the DOS read only attribute should be mapped from a UNIX
# filesystem. This parameter can take three different values, which tell smbd
# how to display the read only attribute on files, where either
# 'store_dos_attributes' is set to 'no', or no extended attribute is present.
# If 'store_dos_attributes' is set to 'yes' then this parameter is ignored.
# The three settings are:
# 'yes' - the read only DOS attribute is mapped to the inverse of the user or
# owner write bit in the unix permission mode set. If the owner write bit is
# not set, the read only attribute is reported as being set on the file. If the
# read only DOS attribute is set, Samba sets the owner, group and others write
# bits to zero. Write bits set in an ACL are ignored by Samba. If the read only
# DOS attribute is unset, Samba simply sets the write bit of the owner to one.
# 'permissions' - the read only DOS attribute is mapped to the effective
# permissions of the connecting user, as evaluated by smbd by reading the unix
# permissions and POSIX ACL (if present). If the connecting user does not have
# permission to modify the file, the read only attribute is reported as being
# set on the file.
# 'no' - the read only DOS attribute is unaffected by permissions, and can only
# be set by the store dos attributes method. This may be useful for exporting
# mounted CDs. The default.
# Note that this parameter will be ignored if the 'store_dos_attributes'
# parameter is set, as the DOS 'read-only' attribute will then be stored inside
# a UNIX extended attribute.
      map_readonly: 'no'
# This controls whether DOS style system files should be mapped to the UNIX
# group execute bit. Note that this parameter will be ignored if the
# 'store_dos_attributes' parameter is set, as the DOS system attribute will
# then be stored inside a UNIX extended attribute. Note that this requires the
# 'create_mask' to be set such that the group execute bit is not masked out
# (i.e. it must include 010). Default is 'no'.
      map_system: 'no'
# This option allows the number of simultaneous connections to a service to be
# limited. If max connections is greater than '0' then connections will be
# refused if this number of connections to the service are already open. A
# value of zero mean an unlimited number of connections may be made. Record
# lock files are used to implement this feature. The lock files will be stored
# in the directory specified by the lock directory option.
      max_connections: '10'
# This parameter limits the maximum number of jobs allowable in a Samba printer
# queue at any given moment. If this number is exceeded, smbd will remote
# "Out of Space" to the client.
      max_print_jobs: '1000'
# This parameter limits the maximum number of jobs displayed in a port monitor
# for Samba printer queue at any given moment. If this number is exceeded, the
# excess jobs will not be shown. A value of zero means there is no limit on the
# number of print jobs reported.
      max_reported_print_jobs: '0'
# This sets the minimum amount of free disk space that must be available before
# a user will be able to spool a print job. It is specified in kilobytes. The
# default is 0, which means a user can always spool a print job.
      min_print_space: '0'
# This parameter indicates that the share is a stand-in for another CIFS share
# whose location is specified by the value of the parameter. When clients
# attempt to connect to this share, they are redirected to one or multiple,
# comma separated proxied shares using the SMB-Dfs protocol. Only Dfs roots can
# act as proxy shares. Take a look at the msdfs root and host msdfs options to
# find out how to set up a Dfs root share. Default is None.
      msdfs_proxy: '\otherserver\someshare,\otherserver2\someshare'
# If set to 'yes', Samba treats the share as a Dfs root and allows clients to
# browse the distributed file system tree rooted at the share directory. Dfs
# links are specified in the share directory by symbolic links of the form
# 'msdfs:serverA\\shareA,serverB\\shareB' and so on. Default is 'no'.
      msdfs_root: 'no'
# If set to 'yes', Samba will shuffle Dfs referrals for a given Dfs link if
# multiple are available, allowing for load balancing across clients. For more
# information on setting up a Dfs tree on Samba, refer to the MSDFS chapter in
# the Samba3-HOWTO book. Default is 'yes'.
      msdfs_shuffle_referrals: 'no'
# This boolean parameter controls whether smbd will attempt to map UNIX
# permissions into Windows NT access control lists. The UNIX permissions
# considered are the traditional UNIX owner and group permissions, as well as
# POSIX ACLs set on any files or directories. Default is 'yes'.
      nt_acl_support: 'yes'
# This specifies the NTVFS handlers for this share.
# 'unixuid' - sets up user credentials based on POSIX gid/uid.
# 'cifs' - proxies a remote CIFS FS. Mainly useful for testing.
# 'nbench' - Filter module that saves data useful to the nbench benchmark suite.
# 'ipc' - Allows using SMB for inter process communication. Only used for the
# IPC$ share.
# 'posix' - Maps POSIX FS semantics to NT semantics.
# 'print' - Allows printing over SMB. This is LANMAN-style printing, not the be
# confused with the spoolss DCE/RPC interface used by later versions of Windows.
# Note that this option is only used when the NTVFS file server is in use. It
# is not used with the (default) s3fs file server.
      ntvfs_handler:
      - 'unixuid'
      - 'default'
# This boolean option tells smbd whether to issue oplocks (opportunistic locks)
# to file open requests on this share. The oplock code can dramatically
# (approx. 30% or more) improve the speed of access to files on Samba servers.
# It allows the clients to aggressively cache files locally and you may want to
# disable this option for unreliable network environments (it is turned on by
# default in Windows NT Servers). Oplocks may be selectively turned off on
# certain files with a share. On some systems oplocks are recognized by the
# underlying operating system. This allows data synchronization between all
# access to oplocked files, whether it be via Samba or NFS or a local UNIX
# process. See the kernel oplocks parameter for details.
      oplocks: 'yes'
# This parameter specifies a directory to which the user of the service is to
# be given access. In the case of printable services, this is where print data
# will spool prior to being submitted to the host for printing.
# For a printable service offering guest access, the service should be readonly
# and the path should be world-writeable and have the sticky bit set. This is
# not mandatory of course, but you probably won't get the results you expect if
# you do otherwise. Any occurrences of '%u' in the path will be replaced with
# the UNIX username that the client is using on this connection. Any
# occurrences of '%m' will be replaced by the NetBIOS name of the machine they
# are connecting from. These replacements are very useful for setting up pseudo
# home directories for users. Note that this path will be based on root dir if
# one was specified.
      path: ''
# The smbd daemon maintains an database of file locks obtained by SMB clients.
# The default behavior is to map this internal database to POSIX locks. This
# means that file locks obtained by SMB clients are consistent with those seen
# by POSIX compliant applications accessing the files via a non-SMB method (e.g.
# NFS or local file access). It is very unlikely that you need to set this
# parameter to 'no', unless you are sharing from an NFS mount, which is not a
# good idea in the first place. Default is 'yes'.
      posix_locking: 'yes'
# This option specifies a command to be run whenever the service is
# disconnected. It takes the usual substitutions. The command may be run as the
# root on some systems. Default is None.
      postexec: ''
# This boolean option controls whether a non-zero return code from preexec
# should close the service being connected to. Default is 'no'.
      preexec_close: 'no'
# This controls if new filenames are created with the case that the client
# passes, or if they are forced to be the default case. Default is 'yes'.
      preserve_case: 'yes'
# If this parameter is yes, then clients may open, write to and submit spool
# files on the directory specified for the service. Note that a printable
# service will ALWAYS allow writing to the service path (user privileges
# permitting) via the spooling of print data. The read only parameter controls
# only non-printing access to the resource. Default is 'no'.
      printable: 'no'
# After a print job has finished spooling to a service, this command will be
# used via a system() call to process the spool file. Typically the command
# specified will submit the spool file to the host's printing subsystem, but
# there is no requirement that this be the case. The server will not remove the
# spool file, so whatever command you specify should remove the spool file when
# it has been processed, otherwise you will need to manually remove old spool
# files.
      print_command: ''
# This parameter specifies the name of the printer to which print jobs spooled
# through a printable service will be sent. The default value may be lp on many
# systems. Default is None.
      printer_name: ''
# This parameters controls how printer status information is interpreted on
# your system. Defaults depends on the operating system.
      printing: ''
# This parameter specifies which user information will be passed to the
# printing system. Usually, the username is sent, but in some cases, e.g. the
# domain prefix is useful, too.
      printjob_username: '%U'
# Windows print clients can update print queue status by expecting the server
# to open a backchannel SMB connection to them. Due to client firewall settings
# this can cause considerable timeouts and will often fail, as there is no
# guarantee the client is even running an SMB server. By default, the Samba
# print server will not try to connect back to clients, and will treat
# corresponding requests as if the connection back to the client failed.
# Default is 'no'.
      print_notify_backchannel: 'no'
# This parameter specifies the command to be executed on the server host in
# order to pause the printer queue. This command should be a program or script
# which takes a printer name as its only parameter and stops the printer queue,
# such that no longer jobs are submitted to the printer. This command is not
# supported by Windows for Workgroups, but can be issued from the Printers
# window under Windows 95 and NT. If a %p is given then the printer name is put
# in its place. Otherwise it is placed at the end of the command. Note that it
# is good practice to include the absolute path in the command as the PATH may
# not be available to the server. Default is None.
      queuepause_command: ''
# This parameter specifies the command to be executed on the server host in
# order to resume the printer queue. It is the command to undo the behavior
# that is caused by 'queuepause_command'. This command should be a program or
# script which takes a printer name as its only parameter and resumes the
# printer queue, such that queued jobs are resubmitted to the printer. This
# command is not supported by Windows for Workgroups, but can be issued from
# the Printers window under Windows 95 and NT. If a '%p' is given then the
# printer name is put in its place. Otherwise it is placed at the end of the
# command. Note that it is good practice to include the absolute path in the
# command as the PATH may not be available to the server.
      queueresume_command: ''
# This is a list of users that are given read-only access to a service. If the
# connecting user is in this list then they will not be given write access, no
# matter what the 'read_only' option is set to. The list can include group
# names using the syntax described in the invalid users parameter.
# Default is None.
      read_list:
      - 'mary'
      - '@students'
# An inverted synonym is 'writeable'. If this parameter is 'yes' (the default),
# then users of a service may not create or modify files in the service's
# directory. Note that a printable service ('printable' is 'yes') will ALWAYS
# allow writing to the directory (user privileges permitting), but only via
# spooling operations.
      read_only: 'yes'
# This is the same as the 'postexec' parameter except that the command is run
# as root. This is useful for unmounting filesystems (such as CDROMs) after a
# connection is closed. Default is None.
      root_postexec: ''
# This is the same as the preexec parameter except that the command is run as
# root. This is useful for mounting filesystems (such as CDROMs) when a
# connection is opened. Default is None.
      root_preexec: ''
# This is the same as the preexec close parameter except that the command is
# run as root. Default is 'no'.
      root_preexec_close: 'no'
# This boolean parameter controls if new files which conform to 8.3 syntax,
# that is all in upper case and of suitable length, are created upper case, or
# if they are forced to be the default case. This option can be use with
# 'preserve_case' in 'yes' to permit long filenames to retain their case, while
# short names are lowered. Default is 'yes'.
      short_preserve_case: 'yes'
# This parameter control whether the fileserver will use sync or async methods
# for fetching the DOS attributes when doing a directory listing. By default
# sync methods will be used.
      smbd_async_dosmode: 'no'
# This parameter allows disabling fetching file write time from the open file
# handle database locking.tdb when a client requests file or directory metadata.
# It's a performance optimisation at the expense of protocol correctness.
# Default is 'yes'.
      smbd_getinfo_ask_sharemode: 'yes'
# This parameter controls how many async operations to fetch the DOS attributes
# the fileserver will queue when doing directory listings.
# Default is "'aio_max_threads' * 2".
      smbd_max_async_dosmode: ''
# This parameter allows disabling fetching file write time from the open file
# handle database locking.tdb. It's a performance optimisation at the expense
# of protocol correctness. Default is 'yes'.
      smbd_search_ask_sharemode: 'yes'
# This parameter controls whether a remote client is allowed or required to use
# SMB encryption. It has different effects depending on whether the connection
# uses SMB1 or SMB2 and newer:
# If the connection uses SMB1, then this option controls the use of a
# Samba-specific extension to the SMB protocol that makes use of the Unix
# extensions.
# If the connection uses SMB2 or newer, then this option controls the use of
# the SMB-level encryption that is supported in SMB version 3.0 and above and
# available in Windows 8 and newer.
# Possible values are 'off' (or 'disabled'), 'enabled' (or 'auto', or
# 'if_required'), 'desired', and 'required' (or 'mandatory'). A special value is
# 'default' which is the implicit default setting of "enabled".
      smb_encrypt: 'default'
# This parameter controls whether Samba allows Spotlight queries on a share.
# For controlling indexing of filesystems you also have to use Tracker's own
# configuration system. Spotlight has several prerequisites:
# * Samba must be configured and built with Spotlight support.
# * The mdssvc RPC service must be enabled, see below.
# * Tracker intergration must be setup and the share must be indexed by Tracker.
# Default is 'no'.
      spotlight: 'no'
# If this parameter is set Samba attempts to first read DOS attributes (SYSTEM,
# HIDDEN, ARCHIVE or READ-ONLY) from a filesystem extended attribute, before
# mapping DOS attributes to UNIX permission bits (such as occurs with
# 'map_hidden' and 'map_readonly'). When set, DOS attributes will be stored
# onto an extended attribute in the UNIX filesystem, associated with the file
# or directory. When this parameter is set it will override the parameters
# 'map_hidden', 'map_system', 'map_archive' and 'map_readonly' and they will
# behave as if they were set to off. This parameter writes the DOS attributes
# as a string into the extended attribute named "user.DOSATTRIB". This extended
# attribute is explicitly hidden from smbd clients requesting an EA list. On
# Linux the filesystem must have been mounted with the mount option user_xattr
# in order for extended attributes to work, also extended attributes must be
# compiled into the Linux kernel. In Samba 3.5.0 and above the "user.DOSATTRIB"
# extended attribute has been extended to store the create time for a file as
# well as the DOS attributes. This is done in a backwards compatible way so
# files created by Samba 3.5.0 and above can still have the DOS attribute read
# from this extended attribute by earlier versions of Samba, but they will not
# be able to read the create time stored there. Storing the create time
# separately from the normal filesystem meta-data allows Samba to faithfully
# reproduce NTFS semantics on top of a POSIX filesystem. The default is 'yes'.
      store_dos_attributes: 'yes'
# This is a boolean that controls the handling of disk space allocation in the
# server. When this is set to 'yes' the server will change from UNIX behaviour
# of not committing real disk storage blocks when a file is extended to the
# Windows behaviour of actually forcing the disk system to allocate real
# storage blocks when a file is created or extended to be a given size. In UNIX
# terminology this means that Samba will stop creating sparse files. This option
# is really designed for file systems that support fast allocation of large
# numbers of blocks such as extent-based file systems. On file systems that
# don't support extents (most notably ext3) this can make Samba slower. When
# you work with large files over >100MB on file systems without extents you may
# even run into problems with clients running into timeouts. When you have an
# extent based filesystem it's likely that we can make use of unwritten extents
# which allows Samba to allocate even large amounts of space very fast and you
# will not see any timeout problems caused by strict allocate. With strict
# allocate in use you will also get much better out of quota messages in case
# you use quotas. Another advantage of activating this setting is that it will
# help to reduce file fragmentation. To give you an idea on which filesystems
# this setting might currently be a good option for you: XFS, ext4, btrfs,
# ocfs2 on Linux and JFS2 on AIX support unwritten extents. On Filesystems that
# do not support it, preallocation is probably an expensive operation where you
# will see reduced performance and risk to let clients run into timeouts when
# creating large files. Examples are ext3, ZFS, HFS+ and most others, so be
# aware if you activate this setting on those filesystems. Default is 'no'.
      strict_allocate: 'no'
# This is an enumerated type that controls the handling of file locking in the
# server. When this is set to 'yes', the server will check every read and write
# access for file locks, and deny access if locks exist. This can be slow on
# some systems. When strict locking is set to 'auto' (the default), the server
# performs file lock checks only on non-oplocked files. As most Windows
# redirectors perform file locking checks locally on oplocked files this is a
# good trade off for improved performance. When strict locking is disabled, the
# server performs file lock checks only when the client explicitly asks for
# them. Well-behaved clients always ask for lock checks when it is important.
# So in the vast majority of cases, 'auto' or 'no' is acceptable.
      strict_locking: 'auto'
# By default a Windows SMB server prevents directory renames when there are
# open file or directory handles below it in the filesystem hierarchy.
# Historically Samba has always allowed this as POSIX filesystem semantics
# require it. This boolean parameter allows Samba to match the Windows behavior.
# Setting this to 'yes' is a very expensive change, as it forces Samba to
# travers the entire open file handle database on every directory rename
# request. In a clustered Samba system the cost is even greater than the
# non-clustered case. When set to 'no' smbd only checks the local process the
# client is attached to for open files below a directory being renamed, instead
# of checking for open files across all smbd processes. Because of the expense
# in fully searching the database, the default is 'no', and it is recommended
# to be left that way unless a specific Windows application requires it to be
# changed. If the client has requested UNIX extensions (POSIX pathnames) then
# renames are always allowed and this parameter has no effect.
      strict_rename: 'no'
# This parameter controls whether Samba honors a request from an SMB client to
# ensure any outstanding operating system buffer contents held in memory are
# safely written onto stable storage on disk. If set to 'yes', which is the
# default, then Windows applications can force the smbd server to synchronize
# unwritten data onto the disk. If set to 'no' then smbd will ignore client
# requests to synchronize unwritten data onto stable storage on disk. The flush
# request from SMB2/3 clients is handled asynchronously inside smbd, so
# leaving the parameter as the default value of 'yes' does not block the
# processing of other requests to the smbd process. Legacy Windows applications
# (such as the Windows 98 explorer shell) seemed to confuse writing buffer
# contents to the operating system with synchronously writing outstanding data
# onto stable storage on disk. Changing this parameter to no means that smbd
# will ignore the Windows applications request to synchronize unwritten data
# onto disk. Only consider changing this if smbd is serving obsolete SMB1
# Windows clients prior to Windows XP (Windows 98 and below). There should be
# no need to change this setting for normal operations.
      strict_sync: 'yes'
# This is a boolean parameter that controls whether writes will always be
# written to stable storage before the write call returns. If this is 'no' then
# the server will be guided by the client's request in each write call (clients
# can set a bit indicating that a particular write should be synchronous). If
# this is 'yes' then every write will be followed by a fsync() call to ensure
# the data is written to disk. Note that the strict sync parameter must be set
# to 'yes' in order for this parameter to have any effect. Default is 'no'.
      sync_always: 'no'
# This parameter applies only to Windows NT/2000 clients. It has no effect on
# Windows 95/98/ME clients. When serving a printer to Windows NT/2000 clients
# without first installing a valid printer driver on the Samba host, the client
# will be required to install a local printer driver. From this point on, the
# client will treat the print as a local printer and not a network printer
# connection. This is much the same behavior that will occur when
# 'disable_spoolss' is 'yes'. The differentiating factor is that under normal
# circumstances, the NT/2000 client will attempt to open the network printer
# using MS-RPC. The problem is that because the client considers the printer to
# be local, it will attempt to issue the OpenPrinterEx() call requesting access
# rights associated with the logged on user. If the user possesses local
# administrator rights but not root privilege on the Samba host (often the
# case), the OpenPrinterEx() call will fail. The result is that the client will
# now display an "Access Denied; Unable to connect" message in the printer
# queue window (even though jobs may successfully be printed).
# If this parameter is enabled for a printer, then any attempt to open the
# printer with the PRINTER_ACCESS_ADMINISTER right is mapped to
# PRINTER_ACCESS_USE instead. Thus allowing the OpenPrinterEx() call to succeed.
# This parameter MUST not be enabled on a print share which has valid print
# driver installed on the Samba server. Default is 'no'.
      use_client_driver: 'no'
# If this parameter is 'yes', and the sendfile() system call is supported by
# the underlying operating system, then some SMB read calls (mainly ReadAndX
# and ReadRaw) will use the more efficient sendfile system call for files that
# are exclusively oplocked. This may make more efficient use of the system
# CPU's and cause Samba to be faster. Samba automatically turns this off for
# clients that use protocol levels lower than NT LM 0.12 and when it detects a
# client is Windows 9x (using sendfile from Linux will cause these clients to
# fail). Default is 'no'.
      use_sendfile: 'no'
# This is a list of users that should be allowed to login to this service.
# Names starting with '@', '+' and '&' are interpreted using the same rules as
# described in the 'invalid_users' parameter. If this is empty (the default)
# then any user can login. If a username is in both this list and the
# 'invalid_users' list then access is denied for that user. The current
# servicename is substituted for '%S'.
      valid_users:
      - 'greg'
      - '@pcusers'
# This is a list of files and directories that are neither visible nor
# accessible. Each entry in the list must be separated by a '/', which allows
# spaces to be included in the entry. '*' and '?' can be used to specify
# multiple files or directories as in DOS wildcards. Each entry must be a unix
# path, not a DOS path and must not include the unix directory separator '/'.
# Note that the case sensitive option is applicable in vetoing files. One
# feature of the veto files parameter that it is important to be aware of is
# Samba's behaviour when trying to delete a directory. If a directory that is
# to be deleted contains nothing but 'veto_files' this deletion will fail
# unless you also set the 'delete_veto_files' parameter to 'yes'. Setting this
# parameter will affect the performance of Samba, as it will be forced to check
# all files and directories for a match as they are scanned.
# Examples:
# Veto any files containing the word "Security", any ending in ".tmp", and any
# directory containing the word "root": '/*Security*/*.tmp/*root*/'.
# Veto the Apple specific files that a NetAtalk server creates:
# '/.AppleDouble/.bin/.AppleDesktop/Network Trash Folder/'.
      veto_files: ''
# This parameter is only valid when the oplocks parameter is turned on for a
# share. It allows the Samba administrator to selectively turn off the granting
# of oplocks on selected files that match a wildcarded list, similar to the
# wildcarded list used in the 'veto_files' parameter. You might want to do this
# on files that you know will be heavily contended for by clients. A good
# example of this is in the NetBench SMB benchmark program, which causes heavy
# client contention for files ending in .SEM. Default is None.
      veto_oplock_files: '/.*SEM/'
# This parameter specifies the backend names which are used for Samba VFS I/O
# operations. By default, normal disk I/O operations are used but these can be
# overloaded with one or more VFS objects. Default is None.
      vfs_objects:
      - 'acl_xattr'
      - 'extd_audit'
# This allows you to override the volume label returned for a share. Useful for
# CDROMs with installation programs that insist on a particular volume label.
      volume: ''
# This parameter controls whether or not links in the UNIX file system may be
# followed by the server. Links that point to areas within the directory tree
# exported by the server are always allowed, this parameter controls access
# only to areas that are outside the directory tree being exported.
# Note: turning this parameter on when UNIX extensions are enabled will allow
# UNIX clients to create symbolic links on the share that can point to files or
# directories outside restricted path exported by the share definition. This
# can cause access to areas outside of the share. Due to this problem, this
# parameter will be automatically disabled (with a message in the log file) if
# the unix extensions option is on. See the parameter
# 'allow_insecure_wide_links' if you wish to change this coupling between the
# two parameters. Default is 'no'.
      wide_links: 'no'
# Inverted synonym for read only. Default is 'no'.
      writeable: 'no'
# If this integer parameter is set to non-zero value, Samba will create an
# in-memory cache for each oplocked file (it does not do this for non-oplocked
# files). All writes that the client does not request to be flushed directly to
# disk will be stored in this cache if possible. The cache is flushed onto disk
# when a write comes in whose offset would not fit into the cache or when the
# file is closed by the client. Reads for the file are also served from this
# cache if the data is stored within it. This cache allows Samba to batch
# client writes into a more efficient write size for RAID disks (i.e. writes
# may be tuned to be the RAID stripe size) and can improve performance on
# systems where the disk subsystem is a bottleneck but there is free memory for
# userspace programs. The integer parameter specifies the size of this cache
# (per oplocked file) in bytes. Note that the write cache won't be used for
# file handles with a smb2 write lease. Default is '0'.
      write_cache_size: '0'
# This is a list of users that are given read-write access to a service. If the
# connecting user is in this list then they will be given write access, no
# matter what the read only option is set to. The list can include group names
# using the @group syntax. Note that if a user is in both the read list and the
# write list then they will be given write access.
      write_list:
      - 'admin'
      - 'root'
      - '@staff'
# vfs_recycle - Samba VFS recycle bin.
# The vfs_recycle intercepts file deletion requests and moves the affected
# files to a temporary repository rather than deleting them immediately. This
# gives the same effect as the Recycle Bin on Windows computers. The Recycle
# Bin will not appear in Windows Explorer views of the network file system
# (share) nor on any mapped drive. Instead, a directory called .recycle will be
# automatically created when the first file is deleted and 'recycle_repository'
# is not configured. If 'recycle_repository' is configured, the name of the
# created directory depends on 'recycle_repository'. Users can recover files
# from the recycle bin. If the 'recycle_keeptree' option has been specified,
# deleted files will be found in a path identical with that from which the file
# was deleted.
#
# Path of the directory where deleted files should be moved. If this option is
# not set, the default path '.recycle' is used.
      recycle_repository: '.recycle'
# Set the octal mode the recycle repository should be created with. The recycle
# repository will be created when first file is deleted. If
# 'recycle_subdir_mode' is not set, mode also applies to subdirectories. If this
# option is not set, the default mode '0700' is used.
      recycle_directory_mode: '0700'
# Set the octal mode with which sub directories of the recycle repository
# should be created. If this option is not set, subdirectories will be created
# with the mode from 'recycle_directory_mode'.
      recycle_subdir_mode: ''
# Specifies whether the directory structure should be preserved or whether the
# files in a directory that is being deleted should be kept separately in the
# repository.
      recycle_keeptree: ''
# If this option is 'true', two files with the same name that are deleted will
# both be kept in the repository. Newer deleted versions of a file will be
# called "Copy #x of filename".
      recycle_versions: ''
# Specifies whether a file's access date should be updated when the file is
# moved to the repository.
      recycle_touch: ''
# Specifies whether a file's last modified date should be updated when the
# file is moved to the repository.
      recycle_touch_mtime: ''
# vfs_full_audit - record Samba VFS operations in the system log.
# The vfs_full_audit VFS module records selected client operations to the
# system log using syslog. vfs_full_audit is able to record the complete set of
# Samba VFS operations:
# 'chdir', 'chflags', 'chmod', 'chown', 'close', 'closedir', 'connect',
# 'copy_chunk_send', 'copy_chunk_recv', 'disconnect', 'disk_free', 'fchmod',
# 'fchown', 'fget_nt_acl', 'fgetxattr', 'flistxattr', 'fremovexattr',
# 'fset_nt_acl', 'fsetxattr', 'fstat', 'fsync', 'ftruncate', 'get_compression',
# 'get_nt_acl', 'get_quota', 'get_shadow_copy_data', 'getlock', 'getwd',
# 'getxattr', 'kernel_flock', 'link', 'linux_setlease', 'listxattr', 'lock',
# 'lseek', 'lstat', 'mkdir', 'mknod', 'open', 'opendir', 'pread', 'pwrite',
# 'read', 'readdir', 'readlink', 'realpath', 'removexattr', 'rename',
# 'rewinddir', 'rmdir', 'seekdir', 'sendfile', 'set_compression', 'set_nt_acl',
# 'set_quota', 'setxattr', 'snap_check_path', 'snap_create', 'snap_delete',
# 'stat', 'statvfs', 'symlink', 'sys_acl_delete_def_file', 'sys_acl_get_fd',
# 'sys_acl_get_file', 'sys_acl_set_fd', 'sys_acl_set_file', 'telldir', 'unlink',
# 'utime', 'write'.
# In addition to these operations, vfs_full_audit recognizes the special
# operation names 'all' and 'none', which refer to all the VFS operations and
# none of the VFS operations respectively. vfs_full_audit records operations in
# fixed format consisting of fields separated by '|' characters.
# The format is: 'PREFIX|OPERATION|RESULT|FILE'.
# The record fields are:
# PREFIX - the result of the 'full_audit_prefix' string after variable
# substitutions.
# OPERATION - the name of the VFS operation.
# RESULT - whether the operation succeeded or failed.
# FILE - the name of the file or directory the operation was performed on.
#
# Prepend audit messages with STRING. STRING is processed for standard
# substitution variables listed in smb.conf. The default prefix is '%u|%I' (this
# mean username and ipaddress).
      full_audit_prefix: '%u|%I'
# Is a list of VFS operations that should be recorded if they succeed.
# Operations are specified using the names listed above. Operations can be
# unset by prefixing the names with "!". The default is none operations.
      full_audit_success: 'open opendir'
# Is a list of VFS operations that should be recorded if they failed. Operations
# are specified using the names listed above. Operations can be unset by
# prefixing the names with "!". The default is none operations.
      full_audit_failure: 'all !open'
# Log messages to the named syslog facility.
      full_audit_facility: 'LOCAL7'
# Log messages with the named syslog priority.
      full_audit_priority: 'ALERT'
# Log messages to syslog (default) or as a debug level 1 message.
      full_audit_syslog: 'true'
# Log an sddl form of the security descriptor coming in when a client sets an
# acl. Defaults to false.
      full_audit_log_secdes: 'true'
# vfs_shadow_copy2 - Expose snapshots to Windows clients as shadow copies.
# The vfs_shadow_copy2 VFS module offers a functionality similar to Microsoft
# Shadow Copy services. When set up properly, this module allows Microsoft
# Shadow Copy clients to browse through file system snapshots as
# "shadow copies" on Samba shares. This is a second implementation of a shadow
# copy module which has the following additional features (compared to the
# original vfs_shadow_copy module):
# 1. There is no need any more to populate your share's root directory with
# symlinks to the snapshots if the file system stores the snapshots elsewhere.
# Instead, you can flexibly configure the module where to look for the file
# system snapshots. This can be very important when you have thousands of
# shares, or use [homes].
# 2. Snapshot directories need not be in one fixed central place but can be
# located anywhere in the directory tree. This mode helps to support file
# systems that offer snapshotting of particular subtrees, for example the GPFS
# independent file sets.
# 3. Vanity naming for snapshots: snapshots can be named in any format
# compatible with str[fp]time conversions.
# 4. Timestamps can be represented in localtime rather than UTC.
# 5. The inode number of the files can optionally be altered to be different
# from the original. This fixes the 'restore' button in the Windows GUI to work
# without a sharing violation when serving from file systems, like GPFS, that
# return the same device and inode number for the snapshot file and the
# original.
# 6. Shadow copy results are by default sorted before being sent to the client.
# This is beneficial for filesystems that don't read directories alphabetically
# (the default unix). Sort ordering can be configured and sorting can be turned
# off completely if the file system sorts its directory listing.
# vfs_shadow_copy2 relies on a filesystem snapshot implementation. Many common
# filesystems have native support for this.
# Filesystem snapshots must be available under specially named directories in
# order to be recognized by vfs_shadow_copy2. These snapshot directory is
# typically a direct subdirectory of the share root's mountpoint but there are
# other modes that can be configured with the parameters described in detail
# below. The snapshot at a given point in time is expected in a subdirectory of
# the snapshot directory where the snapshot's directory is expected to be a
# formatted version of the snapshot time. The default format which can be
# changed with the shadow:format option is @GMT-YYYY.MM.DD-hh.mm.ss, where:
# YYYY is the 4 digit year
# MM is the 2 digit month
# DD is the 2 digit day
# hh is the 2 digit hour
# mm is the 2 digit minute
# ss is the 2 digit second.
# The vfs_shadow_copy2 snapshot naming convention can be produced with the
# following date command: TZ=GMT date +@GMT-%Y.%m.%d-%H.%M.%S
#
# With this parameter, one can specify the mount point of the filesystem that
# contains the share path. Usually this mount point is automatically detected.
# But for some constellations, in particular tests, it can be convenient to be
# able to specify it. Default is empty.
      shadow_mountpoint: '/path/to/filesystem'
# Path to the directory where the file system of the share keeps its snapshots.
# If an absolute path is specified, it is used as-is. If a relative path is
# specified, then it is taken relative to the mount point of the filesystem of
# the share root.
# Note that 'shadow_snapdirseverywhere' depends on this parameter and needs a
# relative path. Setting an absolute path disables 'shadow_snapdirseverywhere'.
# Note that the 'shadow_crossmountpoints' option also requires a relative
# snapdir. Setting an absolute path disables 'shadow_crossmountpoints'.
# Default is '.snapshots'.
      shadow_snapdir: '/some/absolute/path'
# The basedir option allows one to specify a directory between the share's
# mount point and the share root, relative to which the file system's snapshots
# are taken. For example, if
# basedir = mountpoint/rel_basedir
# share_root = basedir/rel_share_root
# snapshot_path = mountpoint/snapdir
# or snapshot_path = snapdir if snapdir is absolute,
# then the snapshot of a file = mountpoint/rel_basedir/rel_share_root/rel_file
# at a time TIME will be found under
# snapshot_path/FS_GMT_TOKEN(TIME)/rel_share_root/rel_file, where
# FS_GMT_TOKEN(TIME) is the timestamp string belonging to TIME in the format
# required by the file system. See shadow_format.
# The default for the basedir is the mount point of the file system of the
# share root (see 'shadow_mountpoint').
# Note that the 'shadow_snapdirseverywhere' and 'shadow_crossmountpoints'
# options are incompatible with 'shadow_basedir' and disable the basedir
# setting.
      shadow_basedir: ''
# With this parameter, one can specify the path of the share's root directory
# in snapshots, relative to the snapshot's root directory. It is an alternative
# method to 'shadow_basedir', allowing greater control. For example, if within
# each snapshot the files of the share have a path/to/share/ prefix, then
# 'shadow_snapsharepath' can be set to path/to/share.
# With this parameter, it is no longer assumed that a snapshot represents an
# image of the original file system or a portion of it. For example, a system
# could perform backups of only files contained in shares, and then expose the
# backup files in a logical structure:
# share1/
# share2/
# .../
# Note that the 'shadow_snapdirseverywhere' and the 'shadow_basedir' options
# are incompatible with 'shadow_snapsharepath' and disable
# 'shadow_snapsharepath' setting. Default is empty.
      shadow_snapsharepath: 'path/to/share'
# By default, this module sorts the shadow copy data alphabetically before
# sending it to the client. With this parameter, one can specify the sort order.
# Possible known values are 'desc' (descending, the default) and 'asc'
# (ascending). If the file system lists directories alphabetically sorted, one
# can turn off sorting in this module by specifying any other value.
      shadow_sort: 'desc'
# This is an optional parameter that indicates whether the snapshot names are
# in UTC/GMT or in local time. If it is disabled then UTC/GMT is expected.
      shadow_localtime: 'no'
# Format specification for snapshot names. This is an optional parameter that
# specifies the format specification for the naming of snapshots in the file
# system. The format must be compatible with the conversion specifications
# recognized by str[fp]time. Default is '@GMT-%Y.%m.%d-%H.%M.%S'.
      shadow_format: ''
# This parameter can be used to specify that the time in format string is given
# as an unsigned long integer (%lu) rather than a time strptime() can parse.
# The result must be a unix time_t time. Default is 'no'.
      shadow_sscanf: 'no'
# If you enable 'shadow_fixinodes' then this module will modify the apparent
# inode number of files in the snapshot directories using a hash of the files
# path. This is needed for snapshot systems where the snapshots have the same
# device:inode number as the original files (such as happens with GPFS
# snapshots). If you don't set this option then the 'restore' button in the
# shadow copy UI will fail with a sharing violation. Default is 'no'.
      shadow_fixinodes: 'no'
# If you enable 'shadow_snapdirseverywhere' then this module will look out for
# snapshot directories in the current working directory and all parent
# directories, stopping at the mount point by default. But see
# 'shadow_crossmountpoints' how to change that behaviour. An example where this
# is needed are independent filesets in IBM's GPFS, but other filesystems might
# support snapshotting only particular subtrees of the filesystem as well.
# Note that 'shadow_snapdirseverywhere' depends on 'shadow_snapdir' and needs
# it to be a relative path. Setting an absolute snapdir path disables
# 'shadow_snapdirseverywhere'. Note that this option is incompatible with the
# 'shadow_basedir' option and removes the 'shadow_basedir' setting by itself.
# Default is 'no'.
      shadow_snapdirseverywhere: 'no'
# This option is effective in the case of 'shadow_snapdirseverywhere' in 'yes'.
# Setting this option makes the module not stop at the first mount point
# encountered when looking for snapdirs, but lets it search potentially all
# through the path instead. An example where this is needed are independent
# filesets in IBM's GPFS, but other filesystems might support snapshotting only
# particular subtrees of the filesystem as well. Note that
# 'shadow_crossmountpoints' depends on 'shadow_snapdir' and needs it to be a
# relative path. Setting an absolute snapdir path disables
# 'shadow_crossmountpoints'. Note that this option is incompatible with the
# 'shadow:basedir' option and removes the 'shadow:basedir' setting by itself.
# Default is 'no'.
      shadow_crossmountpoints: 'no'
# With growing number of snapshots file-systems need some mechanism to
# differentiate one set of snapshots from other, e.g. monthly, weekly, manual,
# special events, etc. Therefore these file-systems provide different ways to
# tag snapshots, e.g. provide a configurable way to name snapshots, which is
# not just based on time. With only 'shadow_format' it is very difficult to
# filter these snapshots. With this optional parameter, one can specify a
# variable prefix component for names of the snapshot directories in the
# file-system. If this parameter is set, together with the shadow_format and
# shadow_delimiter parameters it determines the possible names of snapshot
# directories in the file-system. The option only supports Basic Regular
# Expression (BRE).
      shadow_snapprefix: ''
# This optional parameter is used as a delimiter between 'shadow_snapprefix'
# and 'shadow_format'. This parameter is used only when 'shadow_snapprefix' is
# set. Default is '_GMT'.
      shadow_delimiter: ''
```
