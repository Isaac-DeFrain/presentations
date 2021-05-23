# Connect to peers

This is when the bootstrapping nodes in this spec get their connections.
## InitialConnections

```
InitConnections == \E bn \in Bootstrapping_nodes :
  \E ps \in ConnectionSets :
    /\ phase[bn] \in Phase_init
    /\ connections[bn] = {}
    /\ phase' = [ phase EXCEPT ![bn] = search_phase ]
    /\ connections' = [ connections EXCEPT ![bn] = ps ]
    /\ UNCHANGED <<blacklist, messages, b_non_conn_vars, node_vars>>
```
