# Finished downloading and enqueueing headers and operations

Once the bootstrapping has downloaded and prevalidated all relevant block headers and operations, they move to the `apply` phase to enqueue them in their respective pipes and form the blocks

### Validating l -> Apply l

```
MajorToApply == \E bn \in GOOD_BOOTSTRAPPING :
    \E l \in Levels :
        /\ phase[bn] \in Phase_major
        /\ phase[bn][2] = 1..l
        /\ Card(prevalidated_hds[bn]) = l
        /\ Card(prevalidated_ops[bn]) = l
        /\ log_transition(bn, apply_phase(1..l))
        /\ phase' = [ phase EXCEPT ![bn] = apply_phase(1..l) ]
        /\ UNCHANGED <<non_phase_vars, current_head>>
```

[Back](../phase_diagram_vertical.dot.svg)

[End](../final.html)
