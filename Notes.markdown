GG Active Directory
Active directory is very scalable.

## Active Directory Components
AD is an alternative to a workgroup environment.

Workgroup each machine has its own SAM file, no computer implicitly trusts
another.

Workgroups use multiple accounts for multiple machines and grows exponentially
and nothing is implicitly coordinated between the network.

Account management between machines becomes cumbersome. 

Active Directory provides scalabitility. 

AD uses a domain controller and a database to store the accounts centrally 
which is trusted among all the user machines. 

Authentication is centralized by active directory.

Configuration for policies is not centralized and work becomes difficult with
WorkGroup circumstances. Enables machines to be outside organizational policy
due to ignorance or negligence.

AD can handle millions of entities.

AD can be used throughout the entire IT infrastructure and multiple physical
locations.

### Terminology 

Domain Controller: Server that hosts the AD role.

NTDS.dit: AD databse C:\Windows\ntds is the default location.

SYSVOL: Things not stored in database that all domain controllers need.

NTDS.DIT has separate components and more can be created.

1. Schema: Stores the schema, blueprints for the objects
2. Configuration: Information about the infrastructure itself, topology of 
enterprise.
3. Domain: Where domain objects are stored.

Domain: Logical structure over the network topology. An administrative 
boundary and all objects get stored in relation to the domain.

Sites: Physical structures and boundaries in the network topology. Enable
replication and service localization.

Forest: The entire enterprise, i.e all domains, sites other network structures.
Acts as a security boundary. Enterprise adminsitrators have access to all
domains. Shares a common administration and schema.

Root domain in a forest shares its name with the forest and holds the 
enterrpise administrators group.

Contiguous namespaces: domains grown from the root domain. 

Forests can have multiple trees with different names and different namespaces.

Mergers often result in multitree forests.

Organizational Units: OU Organizational structures to organize objects and can
be used to apply group policy and administrative control.

Schema is the formality that results from relational databases. 

Schema modifications are forest-wide.

Global catalog: index for AD.

## AD Fundamentals

Functional levels refer to features available in new version of AD.

Domain and forest functional levels are important.

Domain functional level is based on the OS version on the domain controller.

Each system allows for multiple domain levels available and the domain 
functional level is dependent on the lowest domain functional level.

Using a lower functional level prevents the use of new features. Old domain 
functional levels are sometimes useful in future proofing against
organizational changes that may introduce older domain controllers. Need to
think preventively about future changes and whether the features introduced
by new functional levels are important enough. 

Forest functional levels refer to the functional level of all domains in the
forest and falls to the lowest version of the domain functional levels in the 
forest.

Domain/forest functional level are manually changed not enforced automatically
 by the domain controller/domains in their respective spheres.

Raising functional levels is not easily reversible.

Trusts are tools that allow users from different domains to interact.

Trust directions are 1/2 way. Access is given in the opposite direction
in which the trust is granted.

Trusts require explicit acknowledgement of their capabilities.

Trusts are transitive and then are artificially implicit via chains of trust.

Transitive trusts are created between parent and child domains, implicitly.

Tree roots have transitive trusts with the forest root.

All domains in a forest trust all other domains in the forest.

External trusts are used for NT domains or when domains in two separate 
forests need to trust one another but not the entire forests. 

External trusts are non transitive.

Sid filtering is a protection enabled by default, for external trusts. 

Trusts are susceptible to replay attacks.

Sid: Domain + Relative id.

Sid history is the explicit past of a user who has migrated domains. Useful 
to ensure users don't lose access to old resources. 

Sid filtering ignores sid history.

Realm trusts: Kerberos authentication enables domain controllers to access
alternative systems like Linux that use the same authentication mechanism and
are not transitive.

Forest trusts: 1/2 way, implicitly results in trusts between all domains in 
both forests. However forest trusts are not transitive between several forests.

Forest-wide trusts enable access to all resources in the other forest. Can be
too generous.

Selective authentication forest-trusts allow for selective filtering of
resource access. Allows for fileserver access directly.

Shortcut trusts: Shortcut trusts prevent domain controllers from unecessarily
forwarding trusts and allows a domain controller to quickly hop the path.

Trust path: result of transitive trusts. Need to be resolved at each DC in the 
trust path.

Any of the default trusts and their transitivity can be modified.

## Installing AD

Prerequisites are important for AD.

Win Server main prereq.

Other prereqs:
1. NTFS volume (enables file/folder permissions)
2. TCPIP settings are configured properly, changing later is a problem.
   DC should have a static IP address.
3. Setting the DNS server, required for AD.
4. AD Prep (used to be required for old AD). AD prep is used to prepare the
   AD for a new environment. 2012 R2 automatically runs all commands assuming
    you have enterprise admin credentials.
5. RO require a writable DC on the domain and 2008 running on it. FFL needs to 
    be 2003.
6. Have the correct credentials for the domain/enterprise (admin).
7. Windows updates (logical).

ADPrep: Forestprep, domainprep grouppolicyprep Read only domain controller prep

Server manager: add role + configure AD.
Requires choosing the AD role then adding features required for active 
directories.

Core DC would not require GUI features.
Choose Active Directory Directory Services is the role to choose.

Once the role is configured select promote this server to a DC to launch the 
configuration wizard.

DCpromo is the former AD configuration wizard, its deprecated.

Notifications is the alternate way to launch the configuration wizard.

1. Must choose the level of modification of domain: parent/forest/add existing.
2. Then pick DFL/FFL and at least 1 global catalog service.
3. Configure the Directory Services Restore Mode (Safe mode for the DC), needs
strong password.
4. Needs to configure the DNS for getting authorization.
5. Verify the netbios name (only 15 chars will truncate).
6. Pick the paths for database, log and sysvol.

7. Confirm settings.
8. Check warnings/errors. AD will cause the machine to reboot (make sure its
not in use).

View script can be used to grab a script from the configuration wizard for 
future DC configurations.


In Server Manager, there is a AS DS stab on the side for the installed role.

There is a summary log file in the dashboard.

Records missing may be do to TCPIP issues. Netstart/netstop can make DNS
be forced to reregister those records. 

Improperly configured DNS settings can cause issues adding a machine to a 
domain.

Changing the domain of a computer requires a reboot.

2 Domain controllers are important for fault tolerance, load balancing etc.

Server manager can be used to remotely configure AD.

The server will be needed to added to the server manager. 

Adding a server to the domain makes it very easily to manage it remotely 
before configuring it.

Remote configuration requires logging in as enterprise admin.

Fault tolerance for installing DNS on all DCs is a good policy.

DSRM passwords are local, and can be different if needed.

Install from media uses NDFSUtil to create a IFM backup that can be pushed to
extractable physical media.

Install from media requires the path. It will make a copy. Replication is not 
perfect as any changes since the backup need to be done (will be done
 automatically).

Specify source allows the user to pick a domain controller they want to 
get a copy of the database from.

Using ntdsutil:
``` powershell
> ntdsutil: activate instance ntds
Active instance set to "ntds".
> ntdsutil: ifm
> ifm: create **sysvol** **full/rodc** path
```

Installing the role Core

``` powershell
> add-windowsFeature -Name ad-domain-services
```

Use a script to configure
``` powershell
Import-Module ADDSDeployment
Install-ADDSDomainController `
-Credential (Get-Credential) ` 
-CriticalReplicationOnly: $false `#only barebones for boot
-DatabasePath "" `
-DomainName "" `
-InstallDNS: $true `
-LogPath "" `
-SiteName "" `
-SYSVOLPath ""
```

Invoking the script
``` powershell
Invoke-Command -ComputerName $name $path
```

Server Core much more secure and has smaller resource footprint.

Adding a new domain script
``` powershell
Import-Module ADDSDeployment
Install-ADDSDomain`
-NoGlobalCatalog: $false `
-CreateDNSDelegation `
-Credential (Get-Credential) ` 
-DomainMode "" ` #DFL
-DomainType: "" ` #Child/parnet
-DatabasePath "" ` 
-DomainName "" ` #parital if a child
-NewDomainNetBIOSName "" ` #partial
-ParentDomainName "" ` #if applicable
-Norebootoncompletion: $false ` #bad practice unless necessary
-SiteName "" ` #Site Name
-InstallDNS: $true `
-LogPath "" `
-SiteName "" `
-SYSVOLPath ""
```

Powershell should be used and necessary for server administration on core.

Accounts for pcs on other domains still exist while creating a new DC for a
new domain.

AD domains and trusts MMC can be used to manage and view trusts.

## Read Only Domain Controllers

RODC prevents small branch offices from causing configuration errors in the
DC structure as its shared across all the domains. Physical security also 
is a primary use case.

RODC cannot have any changes made locally. RODC has no passwords in the
database.

Password caching allows the users who need the RODC rather than the entire 
domain's passwords.

Prereqs:
1. FFL 2003
2. One writable DC with 2008.

Make sure records are cleared in DNS when removing DCs from the domain. 

Removing a DC should include removing the account from the network to prevent
name collision.

AD Users and Computers -> Precreate RODC.

DNS/GC/RODC can create a lot of traffic if GC is enabled.

Delegation of RODC administration and Installation is to allow an account at
the branch office to be used to finish the RODC installation and to be 
used for recovery purposes if need be. Prevents the necessity of having to add
them as a domain admin and give to many permissions. Should specify a group 
instead of a specific user.

Server needs to be unjoined from the domain. 

During the configuration the user specified would login.

By preconfiguring the RODC it prepopulates the configuration wizard.

Important to decide which passwords are cached. If the password isn't cached, 
then the request is forwarded to another DC.

The first time a cached password user logs on they need access to the full DC
over WAN.

Password replication password policy is where caching is decided.

The Denied RODC Password Replication Group is a master group of all admins or
more users that can added who would not be cached on any RODC.

Advanced options shows passwords that are cached on the RODC and who has been
authenticated from the RODC but not cached.

Prepopulation can be used to allow authentication if the WAN is down at the 
RODC and the password would be cached locally, unless policy prohibits it. 

Using the MMC sometimes may be confusing if you attempt to connect to the RODC
and can still create users/groups. Make sure that the MMC is pointing to the 
RODC.

## AD Virtualization

