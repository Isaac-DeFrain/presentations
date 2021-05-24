# New major branch

In the course of prevalidating and enqueuing headers and operations, the bootstrapping node has learned about a branch with higher fitness. They now only request headers and operations on this new branch.

### Validating l -> Validating k, l <= k

```
MajorToMajor == \E bn \in GOOD_BOOTSTRAPPING :
    \E new \in fetched_headers(bn), l \in Levels :
        LET k == new.level IN
        /\ new.fitness > current_head[bn].fitness
        /\ phase[bn] \in Phase_major
        /\ phase[bn][2] = 1..l
        /\ current_head' = [ current_head EXCEPT ![bn] = new ]
        /\ phase'        = [ phase        EXCEPT ![bn] = major_phase(1..k) ]
        /\ log_transition(bn, major_phase(1..k))
        /\ UNCHANGED non_phase_vars
```