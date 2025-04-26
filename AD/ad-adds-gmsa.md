# Group Managed Service Accounts


## Detection
 * Bloodhound Data
 ```bash
 cat <Users.json> | jq '.data[].Properties | select(.samaccountname) | "\(.pwdlastset):\(.samaccountname)"' -r|sort -n
 ```
 Example:
 ```bash
 loot cat 20241130134537_users.json | jq '.data[].Properties | select(.samaccountname) | "\(.pwdlastset):\(.samaccountname)"' -r|sort -n
 1717583255:krbtgt
 1717584108:gMSA01$
 1717594268:G.Viola
 1717594268:L.Bianchi
 1717594268:M.Rossi
 1717594268:R.Verdi
 1717621693:C.Neri
 1717681527:svc_ark
 1717681527:svc_ldap
 1717757654:C.Neri_adm
 1717846494:Administrator
 1730896036:P.Rosa
 1731507413:Guest
 1732621230:L.Bianchi_adm
 1733002326:svc_sql
 ```

