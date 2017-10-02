Role Name
=========

Install openldap server

Requirements
------------

FreeBSD 10|11, debian 8

Role Variables (default)
------------------------

  * `openldap_schmas` ([core, cosine, inetorgperson, nis ])
    LDAP schemas to include in config
    if a file exists in files/openldap/{{name}}.schema, it will be copied
  * `openldap_slave_rid` (0)
    value for slave 'rid' suitable for host_vars (must be unique)
    while rest of config can stay unique in playbook or group_vars
  * `openldap_db_engine` (mdb - hdb for OpenBSD)
  * `openldap_db_maxsize` (1073741824)
  * `openldap_bases` ([])
    List of ldap base dict's
    One base can have: (slapd.conf syntax)
    * `database` (`openldap_db_engine`)
    * `maxsize` (`openldap_db_maxsize`))
      estimated size of database in RAM
    * `rootdn` ()
    * `suffix` ()
    * `directory` ( `openldap_datadir`, system-dependant)
      **you NEED to define it if more than one db**
    * `includes` ([])
      List of files to include, relative path to:
      * in playbook: playbook/files/openldap/ for source
      * destination: `openldap_confdir`/
    * `slave`: dict ({})
      Makes this server a slave (syncrepl protocol)
      * `rid` (`openldap_slave_rid`) *
      * `provider` () *
      * `searchbase` (suffix)
      * `binddn` () *
      * `credentials` () *
      * `bindmethod` (simple)
      * `scope` (sub)
      * `schemachecking` (on)
      * `type` (refreshAndPersist)
      * `retry` ("60 10 120 +")
      * `interval` (00:00:00:15)
      * `tls_cacert` (ldap_tls_cacert)
      * `updateref` ()
    * `indexes` (["objectClass","pres,eq"]) (+ ["entryUUID,entryCSN","eq"] if slave)
      list of couples attributeName(,s…), typeofsearch(es)
      "objectClass" and "entryUUID,entryCSN" (if slave) will always be appended
      can be generated with: 
```
grep ^index openldap/templates/slapd.conf.j2|sed 's/index *//; s/\([^ ]*\) *\([^ ]*\) *$/  - [ "\1", "\2" ]/'
```

### TLS
  * `ldap_tls_cacert` () path to ca certificate file
    if absolute (starts with /), path of an existing ca certificate file (shared with ldap_client role)

Files are relative to openldap/`inventory_hostname` for source
go to `openldap_confdir`/ssl/ at destination
  * `openldap_tls_cert` ()
    if defined, name of server cert
  * `openldap_tls_key` ()
    if defined, name of server key
  * `openldap_tls_cacert` (`ldap_tls_cacert`)
    path to a file to copy to `openldap_confdir`/ca.crt, override `ldap_tls_cacert`
 
Dependencies
------------


Example Playbook
----------------

```
- hosts: servers
  roles:
    - criecm.openldap
  vars:
    openldap_schemas:
      - core
      - cosine
      - nis
      - inetorgperson
      - rfc2739
    openldap_bases:
      rootdn: cn=admin
      suffix: dc=at,dc=home
      includes: [ slapd.access ]
      indexes:
        - [ "uid,uidNumber,gidNumber,memberUID", "pres,eq" ]
      slave:
        rid: 675
        provider: ldaps://master.ldap.univ.fr:636
        binddn: cn=bind,dc=dn
        credentials: bindpw
        bindmethod: simple
        
```

License
-------

BSD

