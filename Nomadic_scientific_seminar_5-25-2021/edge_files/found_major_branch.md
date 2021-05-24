# Found major branch

The bootstrapping node has discovered a major branch and now switches their focus to prevalidating headers and operations for this branch (`major` phase)

## Search -> Major

```
SearchToMajor == \E bn \in GOOD_BOOTSTRAPPING :
    \E l \in Levels :
        /\ phase[bn] \in Phase_search
        /\ l = highest_major_level(bn)
        /\ phase' = [ phase EXCEPT ![bn] = major_phase(1..l) ]
        /\ log_transition(bn, major_phase(1..l))
        /\ UNCHANGED non_phase_vars
```
