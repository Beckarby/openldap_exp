# Basic OpenLDAP Server in Docker

A minimal OpenLDAP container setup using `osixia/openldap`.

## Quick start
1. Start the server:
	 ```bash
	 docker compose up -d
	 ```
2. Verify its running:
	 ```bash
	 docker ps
	 ```
3. Import users.ldif data:
	 ```bash
	 ldapadd -x -H ldap://localhost -D "cn=admin,dc=atlas,dc=com" -w seguridad_segura -f users.ldif
	 ```

## Test Commands

### Check authentication
```bash
ldapwhoami -x -H ldap://localhost -D "cn=admin,dc=atlas,dc=com" -w seguridad_segura
```

### List all users
```bash
ldapsearch -x -H ldap://localhost -b "ou=users,dc=atlas,dc=com" -D "cn=admin,dc=atlas,dc=com" -w seguridad_segura "(objectClass=inetOrgPerson)" cn mail
```

### Test user authentication
```bash
ldapwhoami -x -H ldap://localhost -D "uid=andres.garcia,ou=users,dc=atlas,dc=com" -w marico
```

## Demo

### 1. Access Control - User can't add entries
```bash
# Try to add a user AS a regular user (will fail with "Insufficient access")
ldapadd -x -H ldap://localhost -D "uid=andres.garcia,ou=users,dc=atlas,dc=com" -w marico <<EOF
dn: uid=hacker,ou=users,dc=atlas,dc=com
objectClass: inetOrgPerson
cn: Hacker
sn: Bad
uid: hacker
EOF
```

### 2. User can't delete others
```bash
# Try to delete a group as regular user (will fail)
ldapdelete -x -H ldap://localhost -D "uid=andres.garcia,ou=users,dc=atlas,dc=com" -w marico "cn=admins,ou=groups,dc=atlas,dc=com"
```

### 3. Failed authentication
```bash
# Wrong password returns "Invalid credentials"
ldapwhoami -x -H ldap://localhost -D "uid=andres.garcia,ou=users,dc=atlas,dc=com" -w wrongpassword
```

### 4. Admin CAN add users
```bash
# Admin adds a new user successfully
ldapadd -x -H ldap://localhost -D "cn=admin,dc=atlas,dc=com" -w seguridad_segura <<EOF
dn: uid=maria.lopez,ou=users,dc=atlas,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Maria Lopez
sn: Lopez
uid: maria.lopez
uidNumber: 10002
gidNumber: 10002
homeDirectory: /home/maria.lopez
mail: maria.lopez@atlas.com
userPassword: secret123
EOF
```

### 5. Password change (user changes own password)
```bash
ldappasswd -x -H ldap://localhost -D "uid=andres.garcia,ou=users,dc=atlas,dc=com" -w marico -s newpassword
```
