# Finished downloading and enqueueing headers and operations

Once the bootstrapping has downloaded and prevalidated all relevant block headers and operationsand enqueued them in their respective pipes, they move to the `apply` phase to form the blocks

### Validating l -> Apply l

```
MajorToApply == \E bn \in GOOD_BOOTSTRAPPING :
    \E l \in Levels :
        /\ phase[bn] \in Phase_major
        /\ phase[bn][2] = 1..l
        \* TODO prevalidated_headers
        /\ { _l \in Levels : \E hd \in fetched_headers(bn) : hd.level = _l } = 1..l
        \* TODO prevalidated_ops
        \* /\ all headers represented
        /\ phase' = [ phase EXCEPT ![bn] = apply_phase(1..l) ]
        /\ log_transition(bn, apply_phase(1..l))
        /\ UNCHANGED non_phase_vars
```
