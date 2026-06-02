# BloodHound Custom Cypher Queries

```cypher
// Users with DCSync rights
MATCH (n)-GetChanges|GetChangesAll->(d:Domain) RETURN n

// Paths from compromised user to DA
MATCH (n {name:"USER@DOMAIN"}), (m:Group {name:"DOMAIN ADMINS@DOMAIN"})
CALL shortestPath(n,m) YIELD path RETURN path

// Computers with unconstrained delegation
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c

// Users who can reset any password
MATCH (n)-:ForceChangePassword->(m:User) RETURN n.name, m.name

// GPOs writable by non-admins
MATCH (n)-:GenericAll|GenericWrite->(g:GPO) RETURN n,g
```
