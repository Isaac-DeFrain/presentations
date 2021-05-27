# Searching for a major branch

[Back](../phase_diagram_vertical.dot.svg)

The bootstrapping node is in the process of locating a major branch. This is their sole focus in this phase.

## Messages

- `Get_current_branch`/`Current_branch`
- `Get_block_header`/`Block_header`

### Current branch

- Bootstrapping node requests a `Current_branch`

```
SendGetCurrentBranch == \E bn \in GOOD_BOOTSTRAPPING :
    \E n \in connections[bn] :
        /\ All_bootstrapping_initialized
        /\ phase[bn] \in Phase_search
        /\ has_requested_branch_from(bn, n)
        /\ Send_n(n, get_current_branch_msg(bn))
        /\ sent_get_branch' = [ sent_get_branch EXCEPT ![bn] = @ \cup {n} ]
        /\ UNCHANGED <<b_messages, blacklist, node_vars, b_non_branch_vars, recv_branch>>
```

- Peer responds with a `Current_head`

```
HandleGetCurrentBranch == \E n \in GOOD_NODES :
    \E msg \in get_current_branch_msgs(n) :
        LET bn  == msg.from
            hist == SAMPLES[n, bn]
            curr_branch_msg == current_branch_msg(n, [ current_head |-> CURRENT_HEAD[n], history |-> hist ])
        IN
        /\ Drop_n(n, msg)
        /\ Send_b(bn, curr_branch_msg)
        /\ recv_get_branch' = [ recv_get_branch EXCEPT ![n] = @ \cup {bn} ]
        /\ sent_branch'     = [ sent_branch     EXCEPT ![n][bn] = @ \cup ToSet(hist) ]
        /\ UNCHANGED <<blacklist, bootstrapping_vars, n_non_branch_vars>>
```

- Bootstrapping node handles a `Current_branch`

```
HandleCurrentBranch == \E bn \in GOOD_BOOTSTRAPPING :
    \E msg \in current_branch_msgs(bn) :
        LET n == msg.from
            hist == msg.locator.history
            curr_hd == msg.locator.current_head
        IN
        /\ phase[bn] \in Phase_search \cup Phase_major
        /\ has_requested_branch_from(bn, n)
        /\ Drop_b(bn, msg)
        /\ fittest_head' = [ fittest_head EXCEPT ![bn][n] = IF curr_hd.fitness > @.fitness THEN curr_hd ELSE @ ]
        /\ recv_header'  = [ recv_header  EXCEPT ![bn][n] = @ \cup {curr_hd} ]
        /\ recv_branch'  = [ recv_branch  EXCEPT ![bn][n] = @ \cup ToSet(hist) ]
        /\ UNCHANGED <<n_messages, blacklist, node_vars, connections, current_head, pipe_vars, b_sent_vars, recv_header, recv_operation>>
```

### Block headers

- Bootstrapping node requests a `Block_header`

```
SendGetBlockHeaders == \E bn \in GOOD_BOOTSTRAPPING :
    \E n \in connections[bn], bhs \in NESet_hd(fetched_hashes(bn)) :
        /\ phase[bn] \in Phase_search \cup Phase_major
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
        IN
        /\ phase[bn] \in Phase_search \cup Phase_major
        /\ hash(hd) \in sent_get_headers[bn][n]
        /\ hd \notin fetched_headers(bn)
        /\ Drop_b(bn, msg)
        /\ recv_header' = [ recv_header EXCEPT ![bn][n] = @ \cup {hd} ]
        /\ UNCHANGED <<n_messages, blacklist, node_vars, b_non_recv_vars, recv_branch, recv_operation>>
```

The bootstrapping node does not request operations and should not receive any operations in this phase

[Back](../phase_diagram_vertical.dot.svg)

[End](../final.html)
