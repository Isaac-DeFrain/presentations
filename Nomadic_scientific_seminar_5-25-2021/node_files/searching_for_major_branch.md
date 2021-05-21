# Searching for major branch

The bootstrapping node is in the process of locating a major branch. This is their sole focus in this phase.

## Messages

- `Get_current_branch`/`Current_branch`
- `Get_block_header`/`Block_header`

### Requests (Bootstrapping node)

#### `Get_current_branch`

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

#### `Get_block_headers`

```
SendGetBlockHeaders == \E bn \in GOOD_BOOTSTRAPPING :
    \E n \in connections[bn], bhs \in NESet_hd(fetched_hashes(bn)) :
        /\ phase[bn] \in (Phases \ Phase_init)
        /\ Send_n(n, get_block_headers_msg(bn, bhs))
        /\ sent_get_headers' = [ sent_get_headers EXCEPT ![bn][n] = @ \cup bhs ]
        /\ UNCHANGED <<b_messages, blacklist, node_vars, b_non_header_vars, recv_header>>
```

Bootstrapping node is not requesting operations at this time

### Responses (Peer)

#### `Current_branch`

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

#### `Block_header`

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

Operations are not being requested

## Phase transitions

Once a major branch is dicovered, the bootstrapping node proceeds to the `validation` phase

```

```
