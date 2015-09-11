---
title: "Configuration"
type: "manual"
zones:
    - "Seed"
sections:
    - "SeedSecurity"
tags:
    - "security"
    - "configuration"
    - "ldap"
    - "certificate"
    - "role"
    - "permission"
    - "example"
menu:
    SeedSecurity:
        weight: 20
---

Beyond defining the [security model](../model) of the application, the security infrastructure should be
configured.

# Realm configuration

A Realm is a component that can access specific security data such as users, roles, and permissions. The Realm translates 
this specific data into a format that is understood by the security engine. There is usually a 1-to-1 relation between
a realm and a datasource, such as an LDAP directory, a set of relational tables or configuration properties.
 
{{% callout info %}}
Most of the security datasources usually store both authentication and authorization data, so a realm can perform both 
the authentication and authorization tasks.
{{% /callout %}}

To specify the realm(s) to enable, use the following configuration property:

```ini
org.seedstack.seed.security.realms = list of realms to use
```
    
The realm name correspond to the simple name of the Java class that implements the realm. For instance to enable the
LDAP realm (which is implemented in the `org.seedstack.seed.security.ldap.internal.realms.LDAPRealm` Java class), use 
the following configuration:

```ini
org.seedstack.seed.security.realms = LdapRealm
```
    
If the property is not specified, the default realm is `ConfigurationRealm`.

{{% callout info %}}
Note that you can specify multiple realms. In that case the realms are tried in sequence and if at least one realm
successfully authenticates the subject, the overall attempt is considered successful. If none authenticate successfully, 
the attempt fails. The data from all the successful realms are merged.
{{% /callout %}}

## Configuration realm

This realm relies on the application configuration to authenticate subject and retrieve their roles. It is mainly intended
to be used for testing purposes. To declare subjects (called users in this realm), use the following configuration:
 
```ini
[org.seedstack.seed.security.users]
user1 = password, role1, role2
user2 = password, role3
```

This will define two subjects, `user1` and `user2` with their respective passwords and roles.

## LDAP realm

...

## X509 realm

This realm, which is intended to be used in a Web context, uses the certificates authorized by the Web server when an SSL 
connection is in use. It stores the certificates in the user principals as well as the UID declared in the certificate. 
It also uses the CN of the issuer of the certificates to define the basic roles of the user.

# Role/permission resolver

There is a role/permission resolver component per Realm. It resolves the Permissions assigned to a Role and provides them 
to the Realm. To attach a role/permission resolver to a Realm, use the following configuration:

```ini
org.seedstack.seed.security.<RealmName>.role-permission-resolver = ConfigurationRolePermissionResolver
```
    
Where `<RealmName>` corresponds to the name of the Realm this role/permission resolver is mapped to. The value corresponds
to the simple name of the implementing Java class. When no resolver is specified, the configuration-based role/permission
resolver is used.

## Configuration-based role/permission

This role/permission resolver uses the application configuration to do resolution. You can assign permissions to roles
with the following configuration:

```ini
[org.seedstack.seed.security.permissions]
role1 = permission1a:permission1b, permission2a:permission2b
role2 = permission3, permission4a:permission4b
role3 = permission5
role4 = permission6
```

This configuration assign permissions listed in values to their respective roles as keys. This is the default role/permission
resolver.

# Role mapping

**Optionally**, roles provided by realms can be mapped to application-specific roles. To do this, a role mapping component
should be defined in configuration:

```ini
org.seedstack.seed.security.<RealmName>.role-mapping = ConfigurationRoleMapping
```
    
Where `<RealmName>` corresponds to the name of the Realm this role mapping component is mapped to. The value corresponds
to the simple name of the implementing Java class. When no mapping is specified, the configuration-based role mapping
is used.

{{% callout info %}}
When no role mapping is specified, the roles provided by realms are directly used as application roles.
{{% /callout %}}
    
## Configuration-based role mapping

This role mapping uses the application configuration to do the mapping:

```ini
[org.seedstack.seed.security.roles]
role1 = ORG.APP.ROLE1, ORG.GLOBAL.ADMIN
role2 = ORG.APP.ROLE2
role3 = ORG.APP.{location}.ROLE3
role4 = *
```

This configuration defines the following mappings:

* Application-role `role1` is attributed to the subject when the realm provides `ORG.APP.ROLE1` **OR** `ORG.GLOBAL.ADMIN`.
* Application-role `role2` is attributed to the subject when the realm provides `ORG.APP.ROLE2`.
* Application-role `role3` is attributed to the subject when the realm provides `ORG.APP.FR.ROLE3`, where `FR` is converted 
into a security scope. As such a scoped `role3` is attributed to the subject, which is only valid in `FR` location.
* Application-role `role4` is attributed to every subject authenticated.

# Example

The following example is based on the defaults: a `ConfigurationRealm`, a `ConfigurationRolePermissionResolver` and a `ConfigurationRoleMapping`. Their declaration is optional but present here for clarity. You may want to replace each by a more suitable component, especially the `ConfigurationRealm` which uses the configuration as its users repository.

```ini
[org.seedstack.seed.security]
realms = ConfigurationRealm
ConfigurationRealm.role-mapping = ConfigurationRoleMapping
ConfigurationRealm.role-permission-resolver = ConfigurationRolePermissionResolver

[org.seedstack.seed.security.users]
admin = password1, APP.ADMIN
user1 = password2, APP.FR.MANAGER, APP.UK.MANAGER
user2 = password3, APP.BASIC

[org.seedstack.seed.security.roles]
admin = APP.ADMIN
manager = APP.ADMIN, APP.{location}.MANAGER
normal = APP.ADMIN, APP.BASIC
guest = *

[org.seedstack.seed.security.permissions]
admin = users:clear, cache:invalidate
manager = users:delete, users:create
normal = users:list
```

Note that:

* Application-roles (`admin`, `manager`, `normal` and `guest`) are attributed to a subject if it has **at least one** of the corresponding realm roles (`APP.ADMIN`, `APP.FR.MANAGER` , `APP.UK.MANAGER`, `APP.BASIC`). For instance, having the `APP.ADMIN` realm role is enough to have the `manager` application-role.
* Subject `user1` will only have the `users:delete` and `users:create` permissions on `FR` and `UK` locations.
* Subject `admin` will have the `users:delete` and `users:create` permissions everywhere (no location restriction).
* The `guest` application-role will be attributed to every identified subject.
