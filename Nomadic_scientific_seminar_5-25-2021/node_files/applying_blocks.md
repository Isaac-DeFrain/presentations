# Applying blocks

The bootstrapping node has downloaded and prevalidated all necessary headers and operations. Now they will enqueue the data and form blocks.

## Messages

Bootstrapping nodes in this phase do not send any requests or handle any incoming responses

- Enqueuing headers

```
EnqueueHeader == \E bn \in GOOD_BOOTSTRAPPING :
    LET hds == remaining_headers(bn) IN
    /\ hds /= {}
    /\ LET hd  == Min_level_set(hds) IN
        /\ header_pipe' = [ header_pipe EXCEPT ![bn] = SortSeq(Append(@, hd), min_level_cmp) ]
        /\ UNCHANGED <<messages, blacklist, node_vars, b_non_pipe_vars, operation_pipe>>
```

- Enqueuing operations

```
EnqueueOperations == \E bn \in GOOD_BOOTSTRAPPING :
    LET rem_ops == remaining_operations(bn)
        h_pipe  == header_pipe[bn]
    IN
    /\ h_pipe /= <<>>
    /\ rem_ops /= {}
    /\ LET bh  == (Pick(rem_ops)).block_hash
           ops == rem_ops_block_hash(bn, bh)
           hd  == Head(h_pipe)
       IN
        /\ bh = hash(hd)
        /\ num_fetched_ops_block_hash(bn, bh) = hd.ops_hash
        /\ operation_pipe' = [ operation_pipe EXCEPT ![bn] =
            LET block_hash_at_index(i) == Pick({ op.block_hash : op \in @[i] }) IN
            IF bh \in { block_hash_at_index(i) : i \in DOMAIN @ } THEN
                LET i == { ii \in DOMAIN @ : bh \in block_hash_at_index(ii) } IN [ @ EXCEPT ![i] = @ \cup ops ]
            ELSE Cons(ops, @) ]
        /\ UNCHANGED <<messages, blacklist, node_vars, b_non_pipe_vars, header_pipe>>
```

- Forming blocks

```
ValidateBlock == \E bn \in GOOD_BOOTSTRAPPING :
    LET hds == header_pipe[bn]
        ops == operation_pipe[bn]
    IN
    /\ phase[bn] \in Phase_apply
    /\ hds /= <<>>
    /\ ops /= <<>>
    /\ LET hd == Head(hds)
            op == Head(ops)
        IN
        /\ op.block_hash = hash(hd)
        /\ validate_block(bn, hd, ops, 0) \* TODO fix ctx hash
```
