# Applying blocks

[Back](../phase_diagram_vertical.dot.svg)

The bootstrapping node has downloaded and prevalidated all necessary headers and operations. Now they will enqueue the data and form blocks.

## Messages

Bootstrapping nodes in this phase do not send any requests or handle any incoming responses

- Enqueuing headers

```
EnqueueHeader == \E bn \in GOOD_BOOTSTRAPPING :
    LET pvhds == prevalidated_hds[bn] IN
    /\ pvhds /= {}
    /\ phase[bn] \in Phase_major
    /\ LET hd == Max_level_set(pvhds) IN
        /\ prevalidated_hds' = [ prevalidated_hds EXCEPT ![bn] = @ \ {hd} ]
        /\ header_pipe'      = [ header_pipe      EXCEPT ![bn] = Cons(hd, @) ]
        /\ UNCHANGED <<messages, blacklist, node_vars, b_non_hd_q_vars>>
```

- Enqueuing operations

```
EnqueueOperations == \E bn \in GOOD_BOOTSTRAPPING :
    LET pvops == prevalidated_ops[bn] IN
    /\ pvops /= {}
    /\ phase[bn] \in Phase_major
    /\ LET max  == Max_set({ op[1] : op \in pvops })
           pair == Pick({ op \in pvops : op[1] = max })
           ops  == pair[2]
       IN
        /\ prevalidated_ops' = [ prevalidated_ops EXCEPT ![bn] = @ \ {pair} ]
        /\ operation_pipe'   = [ operation_pipe   EXCEPT ![bn] = Cons(ops, @) ]
        /\ UNCHANGED <<messages, blacklist, node_vars, b_non_op_q_vars>>
```

- Forming blocks

```
ApplyBlock == \E bn \in GOOD_BOOTSTRAPPING :
    LET hds == header_pipe[bn]
        ops == operation_pipe[bn]
    IN
    /\ phase[bn] \in Phase_apply
    /\ hds /= <<>>
    /\ ops /= <<>>
    /\ LET hd == Head(hds)
           op == Head(ops)
           b  == block(hd, ops)
        IN
        /\ header_pipe'       = [ header_pipe      EXCEPT ![bn] = Tail(@) ]
        /\ operation_pipe'    = [ operation_pipe   EXCEPT ![bn] = Tail(@) ]
        /\ validated_blocks'  = [ validated_blocks EXCEPT ![bn] = @ \cup {b} ]
        /\ UNCHANGED <<messages, blacklist, node_vars, b_non_pipe_vars>>
```

[Back](../phase_diagram_vertical.dot.svg)

[End](../final.html)
