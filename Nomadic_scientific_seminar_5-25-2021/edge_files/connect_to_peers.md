# Connect to peers

In the initial state, bootstrapping nodes do not have peers and thus cannot make any requests. Once they obtain sufficiently many peers, they move to the `search` phase and start requesting current branches from their peers.

## Init -> Search

```
InitConnections == \E bn \in Bootstrapping_nodes :
  \E ps \in ConnectionSets :
    /\ phase[bn] \in Phase_init
    /\ connections[bn] = {}
    /\ phase' = [ phase EXCEPT ![bn] = search_phase ]
    /\ connections' = [ connections EXCEPT ![bn] = ps ]
    /\ UNCHANGED <<blacklist, messages, b_non_conn_vars, node_vars>>
```

[Back](../phase_diagram_vertical.dot.svg)

[End](../final.html)
