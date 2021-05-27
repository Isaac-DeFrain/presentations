# Validating a major branch

[Back](../phase_diagram_vertical.dot.svg)

The bootstrapping node is in the process of retrieving and prevalidating the headers and operations for their current branch

## Messages

- `Get_block_headers`/`Block_header`
- `Get_operations`/`Operation`

### Block headers

- Bootstrapping node requests a `Block_header`

```
SendGetBlockHeaders == \E bn \in GOOD_BOOTSTRAPPING :
    \E n \in connections[bn], bhs \in NESet_hd(fetched_hashes(bn)) :
        /\ phase[bn] \in Phase_search \cup Phase_major
        /\ bn \notin n_blacklist[n]
        /\ Send_n(n, get_block_headers_msg(bn, bhs))
        /\ sent_get_headers' = [ sent_get_headers EXCEPT ![bn][n] = @ \cup bhs ]
        /\ UNCHANGED <<b_messages, blacklist, node_vars, b_non_header_vars, recv_header>>
```

- Peer responds with a `Block_header`

```
HandleGetBlockHeaders == \E n \in GOOD_NODES :
    \E msg \in get_block_headers_msgs(n) :
        LET bn == msg.from
            hs == msg.hashes \cap node_hashes(n)
        IN
        /\ Drop_n(n, msg)
        /\ recv_get_headers' = [ recv_get_headers EXCEPT ![n][bn] = @ \cup hs ]
        /\ serving_headers'  = [ serving_headers  EXCEPT ![n][bn] = @ \cup hs ]
        /\ UNCHANGED <<b_messages, blacklist, bootstrapping_vars, n_non_serving_vars, serving_ops>>
```

- Bootstrapping node handles a `Block_header`

```
HandleBlockHeader == \E bn \in GOOD_BOOTSTRAPPING :
    \E msg \in block_header_msgs(bn) :
        LET n  == msg.from
            hd == msg.header
            h  == hash(hd)
        IN
        /\ phase[bn] \in Phase_search \cup Phase_major
        /\ h \in sent_get_headers[bn][n]
        /\ hd \notin fetched_headers(bn)
        /\ Drop_b(bn, msg)
        /\ recv_header' = [ recv_header EXCEPT ![bn][n] = @ \cup {<<h, hd>>} ]
        /\ UNCHANGED <<n_messages, blacklist, node_vars, b_non_recv_vars, recv_branch, recv_operation>>
```

### Operations

- Bootstrapping node requests a set of `Operation`s

```
SendGetOperations == \E bn \in GOOD_BOOTSTRAPPING :
    \E n \in connections[bn], hd \in fetched_headers(bn) :
        LET bh  == hash(hd)
            ops == operations(bh, 1..hd.ops_hash)
            req == ops \ all_recv_operations_block_hash(bn, bh)
            ohs == { <<bh, op_hash[op]>> : op \in req }
        IN
        /\ req /= {}
        /\ phase[bn] \in Phase_major
        /\ bn \notin n_blacklist[n]
        /\ Send_n(n, get_operations_msg(bn, ohs))
        /\ sent_get_ops' = [ sent_get_ops EXCEPT ![bn][n] = @ \cup ohs ]
        /\ UNCHANGED <<b_messages, blacklist, node_vars, b_non_op_vars, recv_operation>>
```

- Peer responds with an `Operation`

```
HandleGetOperations == \E n \in GOOD_NODES :
    \E msg \in get_operations_msgs(n) :
        LET bn  == msg.from
            ohs == msg.op_hashes \ node_op_hashes(n)
            bh  == Pick({ oh[1] : oh \in ohs })
        IN
        /\ bh \in node_hashes(n)
        /\ Drop_n(n, msg)
        /\ recv_get_ops' = [ recv_get_ops EXCEPT ![n][bn] = @ \cup ohs ]
        /\ serving_ops'  = [ serving_ops  EXCEPT ![n][bn][bh] = @ \cup { oh[2] : oh \in ohs } ]
        /\ UNCHANGED <<b_messages, blacklist, bootstrapping_vars, n_non_serving_vars, serving_headers>>
```

- Bootstrapping node handles an `Operation`

```
HandleOperation == \E bn \in GOOD_BOOTSTRAPPING :
    \E msg \in operation_msgs(bn) :
        LET n  == msg.from
            op == msg.operation
            bh == op.block_hash
        IN
        \E hd \in headers(bn) :
            /\ bh = hash(hd)
            /\ op \notin recv_operation[bn][n]
            /\ Drop_b(bn, msg)
            /\ recv_operation' = [ recv_operation EXCEPT ![bn][n] = @ \cup {op} ]
            /\ UNCHANGED <<n_messages, blacklist, node_vars, b_non_recv_vars, recv_branch, recv_header>>
```

Bootstrapping node is not requesting current branches, but may still receive current branch responses sent in the previous phase

[Back](../phase_diagram_vertical.dot.svg)

[End](../final.html)
