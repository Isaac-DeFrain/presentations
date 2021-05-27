# Applied blocks

The bootstrapping node has applied all the blocks on their major branch and now goes back to requesting current heads from peers and handling messages

## Apply -> Search

```
ApplyToSearch == \E bn \in GOOD_BOOTSTRAPPING :
    /\ phase[bn] \in Phase_apply
    /\ prevalidated_hds[bn] = {}
    /\ prevalidated_ops[bn] = {}
    /\ header_pipe[bn] = <<>>
    /\ operation_pipe[bn] = <<>>
    /\ phase' = [ phase EXCEPT ![bn] = search_phase ]
    /\ log_transition(bn, search_phase)
    /\ UNCHANGED <<non_phase_vars, current_head>>
```

[Back](../phase_diagram_vertical.dot.svg)

[End](../final.html)
