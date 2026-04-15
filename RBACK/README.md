# Complete RBAC Setup in Jenkins with OpenLDAP

## Phase 1: OpenLDAP Setup with Users & Groups

### What is OpenLDAP?

- **OpenLDAP** is an open-source implementation of the **Lightweight Directory Access Protocol (LDAP)**.
- It stores and manages users, groups, and access rules in one central place.
- Think of it as an identity database that Jenkins (or other applications) can connect to for:
  - **Authentication** (verifying usernames/passwords)
  - **Authorization** (checking which resources a user can access)

---

### Step 1: Install OpenLDAP on EC2

Run these commands:

```bash
sudo apt-get update
sudo apt-get install slapd ldap-utils -y
```

| Package      | Purpose |
|--------------|---------|
| `slapd`      | OpenLDAP server daemon |
| `ldap-utils` | Command-line tools (`ldapadd`, `ldapsearch`, etc.) for managing LDAP |

---

### Step 2: Configure OpenLDAP (Interactive Setup)

Run the configuration command:

```bash
sudo dpkg-reconfigure slapd
```

Answer the prompts as follows:

| Prompt | Explanation | Your Answer |
|--------|-------------|--------------|
| Omit OpenLDAP server configuration? | Choose **No** to manually configure the server. | `No` |
| DNS domain name | Defines your LDAP domain structure. For `kartikeyasoft.com`, the LDAP base DN becomes `dc=kartikeyasoft,dc=com`. | `kartikeyasoft.com` |
| Organization name | A label for your organization. | `Kartikeyasoft` |
| Admin password | Set a strong password for the LDAP administrator (`cn=admin`). | *(your chosen password)* |
| Database backend | Select **MDB** (Memory-Mapped Database) – the recommended backend. | `MDB` |
| Remove database when slapd is purged? | Choose **No** to prevent accidental deletion of data. | `No` |
| Move old database? | If a previous configuration exists, move it aside. | `Yes` |

---

### Step 3: Base Directory Information Tree (DIT)

#### What is DIT?

- **DIT** (Directory Information Tree) is the hierarchical structure of your LDAP database.
- It works like a file system but stores identity data.
- The **base DN** here is `dc=kartikeyasoft,dc=com`.

> **Note:** The `dpkg-reconfigure slapd` step already created the root DN (`dc=kartikeyasoft,dc=com`).  
> You only need to add the **Organizational Units (OUs)** for users and groups.

Create a file named `base.ldif` with this content:

```ldif
dn: ou=users,dc=kartikeyasoft,dc=com
objectClass: organizationalUnit
ou: users

dn: ou=groups,dc=kartikeyasoft,dc=com
objectClass: organizationalUnit
ou: groups
```

| Element | Purpose |
|---------|---------|
| `dc=kartikeyasoft,dc=com` | Root of the LDAP hierarchy |
| `ou=users` | Organizational Unit to store user entries |
| `ou=groups` | Organizational Unit to store group entries |

#### Command: Add the base DIT

```bash
ldapadd -x -D "cn=admin,dc=kartikeyasoft,dc=com" -W -f base.ldif
```

| Option | Meaning |
|--------|---------|
| `-D` | Bind DN (LDAP admin account) |
| `-W` | Prompt for the admin password |
| `-f` | File to load (`base.ldif`) |

---

### Step 4: Create Users

Create a file named `users.ldif`:

```ldif
dn: uid=adminuser,ou=users,dc=kartikeyasoft,dc=com
objectClass: inetOrgPerson
uid: adminuser
sn: Admin
cn: Admin User
userPassword: adminpass

dn: uid=devuser1,ou=users,dc=kartikeyasoft,dc=com
objectClass: inetOrgPerson
uid: devuser1
sn: Developer1
cn: Dev User1
userPassword: devpass1

dn: uid=devuser2,ou=users,dc=kartikeyasoft,dc=com
objectClass: inetOrgPerson
uid: devuser2
sn: Developer2
cn: Dev User2
userPassword: devpass2

dn: uid=viewer1,ou=users,dc=kartikeyasoft,dc=com
objectClass: inetOrgPerson
uid: viewer1
sn: Viewer1
cn: Viewer User1
userPassword: viewerpass1
```

| Attribute | Purpose |
|-----------|---------|
| `uid` | Unique identifier (used for login) |
| `sn` | Surname (last name) |
| `cn` | Common name (full display name) |
| `userPassword` | Password (shown in plain text for simplicity; in production, use a hash) |

#### Command: Add users

```bash
ldapadd -x -D "cn=admin,dc=kartikeyasoft,dc=com" -W -f users.ldif
```

---

### Step 5: Create Groups

Create a file named `groups.ldif`:

```ldif
dn: cn=jenkins-admins,ou=groups,dc=kartikeyasoft,dc=com
objectClass: groupOfNames
cn: jenkins-admins
member: uid=adminuser,ou=users,dc=kartikeyasoft,dc=com

dn: cn=jenkins-devs,ou=groups,dc=kartikeyasoft,dc=com
objectClass: groupOfNames
cn: jenkins-devs
member: uid=devuser1,ou=users,dc=kartikeyasoft,dc=com
member: uid=devuser2,ou=users,dc=kartikeyasoft,dc=com

dn: cn=jenkins-viewers,ou=groups,dc=kartikeyasoft,dc=com
objectClass: groupOfNames
cn: jenkins-viewers
member: uid=viewer1,ou=users,dc=kartikeyasoft,dc=com
```

| Object Class | Purpose |
|--------------|---------|
| `groupOfNames` | Defines groups with `member` attributes pointing to user DNs |

#### Command: Add groups

```bash
ldapadd -x -D "cn=admin,dc=kartikeyasoft,dc=com" -W -f groups.ldif
```

---

### Step 6: Verify LDAP Directory

Run this search to confirm everything is added:

```bash
ldapsearch -x -b "dc=kartikeyasoft,dc=com"
```

This command queries the entire directory tree and displays all users and groups.

---

## Phase 2: Jenkins RBAC with OpenLDAP

### Step 1: Install LDAP Plugin

1. Go to: **Manage Jenkins → Manage Plugins → Available**
2. Search for: **LDAP Plugin**
3. Install it and restart Jenkins.

---

### Step 2: Configure LDAP Authentication

1. Go to: **Manage Jenkins → Configure Global Security**
2. Under **Security Realm**, select: **LDAP**

Fill in the LDAP fields as follows:

| Field | Value | Explanation |
|-------|-------|-------------|
| **Server** | `ldap://3.110.123.68:389` | LDAP server address and port (389 = plain LDAP) |
| **Root DN** | `dc=kartikeyasoft,dc=com` | Starting point in the directory tree for all searches |
| **User Search Base** | `ou=users` | Location under Root DN where user entries reside |
| **User Search Filter** | `(uid={0})` | Filters user logins. `{0}` is replaced by the username entered at login |
| **Group Search Base** | `ou=groups` | Where Jenkins should search for groups |
| **Group Membership Filter** | `(member={0})` | How Jenkins finds group memberships. `{0}` is replaced by the user's DN (e.g., `uid=devuser1,...`) |
| **Manager DN (Bind DN)** | `cn=admin,dc=kartikeyasoft,dc=com` | LDAP admin account used to query the directory |
| **Manager Password** | *(your LDAP admin password)* | Password for the bind account |

> **Test the connection** using `adminuser` / `adminpass`.

---

### Step 3: Configure Jenkins Authorization (RBAC)

#### Option A: Matrix-based Security (Recommended for LDAP)

1. Under **Authorization**, select: **Matrix-based security**
2. Add LDAP groups with the `@` prefix:

| Identity | Permissions |
|----------|--------------|
| `@jenkins-admins` | ✔️ **Administer** (all Jenkins permissions) |
| `@jenkins-devs` | ✔️ **Job Read**, ✔️ **Job Build** |
| `@jenkins-viewers` | ✔️ **Job Read** |

> The `@` prefix tells Jenkins that the name refers to an LDAP group, not a local user.

#### Option B: Role Strategy Plugin (if you need more flexibility)

1. Install the **Role Strategy Plugin**.
2. Set **Authorization** to **Role-Based Strategy**.
3. Define roles like `admin`, `developer`, `viewer`.
4. Assign LDAP groups to these roles.

---

## Files Recap

| File | Purpose |
|------|---------|
| `base.ldif` | Creates root DN, `users` OU, and `groups` OU |
| `users.ldif` | Adds multiple user entries |
| `groups.ldif` | Adds groups and links members to groups |

---

## Jenkins Configuration Summary

| Section | Details |
|---------|---------|
| **Security Realm** | LDAP (uses OpenLDAP for authentication) |
| **Authorization** | Matrix-based security (RBAC via LDAP groups) |
| **Groups** | `jenkins-admins`, `jenkins-devs`, `jenkins-viewers` |
| **Users** | `adminuser`, `devuser1`, `devuser2`, `viewer1` |

---
