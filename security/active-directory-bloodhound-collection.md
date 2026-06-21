# Active Directory Enumeration with BloodHound

Using BloodHound and SharpHound to map Active Directory attack paths.

## Collection

### SharpHound (from Windows)

```powershell
# Run from a domain-joined machine
# Full collection (all methods)
SharpHound.exe -c All --zipfilename bloodhound.zip

# Specific collection methods
SharpHound.exe -c Sessions,LocalGroups,ACLs
SharpHound.exe -c Sessions --domain child.domain.local

# Stealth (slower, less detection)
SharpHound.exe -c Session --stealth
```

### bloodhound-python (from Linux)

```bash
# Need: username, password, domain, DC IP
bloodhound-python -u user -p password -d domain.local -ns 10.10.10.10 -c All

# With hash
bloodhound-python -u user -hashes :aadm3b... -d domain.local -ns 10.10.10.10 -c All

# Specific DC
bloodhound-python -u user -p password -d domain.local -dc dc01.domain.local -ns 10.10.10.10 -c All
```

## Analysis

### Import to BloodHound

```bash
# Start Neo4j + BloodHound
neo4j start
# Browse to http://localhost:7474 (neo4j:neo4j)
# Import the zip file in BloodHound UI
```

### Key Queries (Cypher)

```cypher
// Find shortest path to Domain Admin
MATCH (n), (m) WHERE m.name = "DOMAIN ADMINS@DOMAIN.LOCAL"
CALL shortestPath(n, m) YIELD path RETURN path

// Find all users with DCSync rights
MATCH (n)-GetChanges|GetChangesAll->(d:Domain) RETURN n

// Find Kerberoastable users
MATCH (n:User) WHERE n.kerberoastable = true RETURN n

// Find AS-REP Roastable users
MATCH (n:User) WHERE n.dontreqpreauth = true RETURN n

// Find paths from owned user to Domain Admin
MATCH (n:User {name: "OWNED@DOMAIN.LOCAL"}), (m:Group {name: "DOMAIN ADMINS@DOMAIN.LOCAL"})
CALL shortestPath(n, m) YIELD path RETURN path

// Find users with unconstrained delegation
MATCH (n) WHERE n.unconstraineddelegation = true RETURN n

// Find computers where DOMAIN ADMINS have sessions
MATCH (g:Group {name:"DOMAIN ADMINS@DOMAIN.LOCAL"}), (c:Computer)
WHERE g <> c AND (g)-[:MemberOf|HasSession]->(c) RETURN c

// Find GPOs that can be modified by non-admins
MATCH (n)-[:GenericAll|GenericWrite|WriteDacl]->(g:GPO) RETURN n, g
```

## Common Attack Paths

### 1. Kerberoasting
```
User -> has SPN service ticket -> crack hash -> service account access
```

### 2. DCSync
```
User -> GetChanges + GetChangesAll on Domain -> sync all hashes
```

### 3. Constrained Delegation Abuse
```
User -> AllowedToDelegate to service -> S4U2Self/S4U2Proxy -> impersonate admin
```

### 4. ACL Abuse
```
User -> GenericAll on another User -> reset password -> lateral move
User -> WriteDacl on Group -> add self to Domain Admins
```

### 5. GPO Abuse
```
User -> GenericWrite on GPO -> push malicious script -> all machines apply it
```

## Quick Win Queries

```cypher
// Users with passwords > 1 year old
MATCH (n:User) WHERE n.pwdlastset < (datetime().epochseconds - 31536000) RETURN n

// Disabled but not deleted users (security gaps)
MATCH (n:User {enabled: false}) WHERE NOT n.name CONTAINS "DISABLED" RETURN n

// Computers with local admin access from non-admin users
MATCH (n:User)-[:AdminTo]->(c:Computer) RETURN n.name, c.name
```
