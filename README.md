Overview
--------

EasyPermissions is a permissions plugin that is easy to use, resilient, and just makes sense.
It does not use YML, instead, it uses a simple syntax to define group permissions, groups of
users and permissions for users. It is highly configurable, and provides methods to change
permissions from in game and via file editing. The general syntax should be easy for developers
to pick up, and defining permissions and groups follows standard Object Oriented Design. All
permissions files in the plugins/EasyPermissions/perms are automatically read in and processed.
Only one control group is allowed per file, though as many files as necessary may be defined.
All group permissions must go in the group directory, and user permissions must go in the users
directory. By default, all permissions are revoked for all users. Various global configuration options
are defined and documented in settings.ini.

Group/User Settings
-------------------

To define a group:

groups/default.gp

```
group default1 {
	# Line comment, though comments that users add to this file will be wiped
	# when the file is processed. The compiler will add some comments automatically
	use for default; # This marks this group to apply to all users that aren't otherwise defined in a group

	grant permission1.*;
	grant permission2.*;
	grant permission3.subPermission1;
	revoke permission4.*;
}
```

groups/default.gp

```
group default2 {
	use world world2; // Makes this group only apply to users in the specified world

	grant permission3.subPermission2;
}
```

groups/moderator.gp

```
group moderator extends default1 {
	grant permission5.*;
	revoke permission2.subPermission;

	adduser User1;
}
```

groups/admin.gp

```
# Inherits all permissions from default2 and moderator groups (and default1 indirectly through moderator group)
group admin extends default2, moderator {
	grant *;
	adduser User2;
}
```

There is only one file for all users that have individual permissions.
To grant individual permissions to users, add a section to users/users.up

users/users.up

```
user User3 {
	grant permission5.*;
	revoke permission6;
}

# Note that this user extends User4. Each group or user functions like an object. Both
# Users and groups may extend each other, and it essentially uses the extended object
# as a base object, overridding as needed.
user User4 extends User4 {
	grant permission5.*;
}
```

When determining if a user has a particular permission, the following algorithm is
followed to determine what applies to the user, with latter values overriding former
ones if there is a conflict:

* Groups with use for default; directive
* Groups that inherit from the group the user is in
* Groups that the user is in.
 * If a user is in two groups with conflicting values, if either group revokes a permission, that
takes priority. This does not apply to group inheritance, derived objects always override their
super objects.
* Specific user permissions that apply to the user.

Users may be listed in multiple groups which will affect their permission inheritance,
though only groups that they are specifically defined in are considered to be that users
"groups" listing.

The * is a wildcard permission, and is overridden by more specific permissions. For
instance, if a user is granted permission1.*, but permission1.subPermission1 is revoked,
then the user would have permission for permission1.subPermission2 (as well as
permission1.subPermission3 and on), but not permission.subPermission1.

Note about UUIDs
----------------

Due to the nature of usernames not necessarily referring to a unique user, usernames
may be used, but they will be replaced with the player's UUID as soon as the file
is processed, with a comment beside the UUID with the player's current name, as of
file processing time. UUIDs may always be used in place of the players name.


In game commands
----------------

All functions that are available in a file (including settings) may be changed via in game commands, which
are described below. Values wrapped in <this> denote input that you must provide, 'quoted' values are literal
strings you must type, and <<'values'|'like'|'this'>> denote that you must provide one of the specified
literals. [Square brackets] denote optional portions.

* /ep setting <setting> <value> - Changes a global configuration setting.
* /ep addgroup <group name> ['extends' <other objects>] - Adds a new, empty group object, optionally extending existing objects.
* /ep adduser <username or uuid> ['extends' <other objects>] - Adds a new, empty user object, optionally extending existing objects.
* /ep delgroup <group name> - Deletes the specified group.
* /ep deluser <user name or uuid> - Deletes the specified user.
* /ep alter <<'group'|'user'>> <group or user name> <directive>[';'] - Alters an existing group or user configuration. Directive values are
the same as the directives listed in the file, for instance "/ep alter group admin adduser User1".
* /ep reload - Reprocesses permission settings from the file system.

Directives
----------

All directives are user above, but here is a complete listing of the directives, and their purpose.

group and user directives:

* grant <permission>; - Gives a permission to the containing object. It is not an error to grant a permission that
may have already
* revoke <permission>; - Revokes a permission that may or may not have previously been granted to the object.

group object directives:

* use for default; - Directs the plugin to use this group for users that aren't specifically listed in any groups. This
directive may be applied to no groups, in which case users not listed in any group will have no permissions and will
be in no groups, or it may be applied to more than one group, in which case default users will be in multiple groups.
* use world <world>; - Only applies this group to users in the specified world. By default, permissions apply to all
users, regardless of what world they are in.
* adduser <username or uuid>; - Adds the specified user to this group.

Global settings
---------------

Global settings affect the behavior of the plugin, and are listed below.

* op-overrides - If false, users with op will not get all permissions by default
* enable-commands - If false, none of the /ep commands will do anything, and changes can only be made via file system
* confirm-delete - If false, no confirmation will be requested when deleting a group or user via command
* auto-reload - If true, you do not need to run /ep reload when you make a permissions change to a file.
Changes to the file will automatically be detected and reloaded immediately upon file save, if the server is
currently running.