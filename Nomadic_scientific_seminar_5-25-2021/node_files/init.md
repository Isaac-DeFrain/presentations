# Init

The initial state

```
Init ==
    \* messages
    /\ b_messages        = [ n \in GOOD_BOOTSTRAPPING  |-> {} ]
    /\ n_messages        = [ n \in GOOD_NODES |-> {} ]
    
    \* blacklist
    /\ b_blacklist       = [ n \in GOOD_BOOTSTRAPPING  |-> {} ]
    /\ n_blacklist       = [ n \in GOOD_NODES |-> {} ]
    
    \* bootstrapping
    /\ phase             = [ n \in GOOD_BOOTSTRAPPING  |-> <<"init", {}>> ]
    /\ connections       = [ n \in Bootstrapping_nodes |-> {} ]
    /\ current_head      = [ n \in GOOD_BOOTSTRAPPING  |-> genesis ]
    /\ fittest_head      = [ n \in GOOD_BOOTSTRAPPING  |-> gen_header ]
    /\ header_pipe       = [ n \in GOOD_BOOTSTRAPPING  |-> <<>> ]
    /\ operation_pipe    = [ n \in GOOD_BOOTSTRAPPING  |-> <<>> ]
    /\ validated_blocks  = [ n \in GOOD_BOOTSTRAPPING  |-> [ l \in Levels |-> {} ] ]
    /\ sent_get_branch   = [ n \in GOOD_BOOTSTRAPPING  |-> {} ]
    /\ sent_get_headers  = [ n \in GOOD_BOOTSTRAPPING  |-> [ m \in Nodes |-> {} ] ]
    /\ sent_get_ops      = [ n \in GOOD_BOOTSTRAPPING  |-> [ m \in Nodes |-> {} ] ]
    /\ recv_branch       = [ n \in GOOD_BOOTSTRAPPING  |-> [ m \in Nodes |-> {} ] ]
    /\ recv_header       = [ n \in GOOD_BOOTSTRAPPING  |-> [ m \in Nodes |-> {} ] ]
    /\ recv_operation    = [ n \in GOOD_BOOTSTRAPPING  |-> [ m \in Nodes |-> {} ] ]
    /\ phase_trace       = [ n \in GOOD_BOOTSTRAPPING  |-> <<>> ]

    \* node
    /\ serving_headers   = [ n \in GOOD_NODES |-> [ m \in Bootstrapping_nodes |-> {} ] ]
    /\ serving_ops       = [ p \in GOOD_NODES |-> [ m \in Bootstrapping_nodes |-> [ h \in BlockHashes |-> {} ] ] ]
    /\ recv_get_branch   = [ n \in GOOD_NODES |-> {} ]
    /\ recv_get_headers  = [ n \in GOOD_NODES |-> [ m \in Bootstrapping_nodes |-> {} ] ]
    /\ recv_get_ops      = [ n \in GOOD_NODES |-> [ m \in Bootstrapping_nodes |-> {} ] ]
    /\ sent_branch       = [ n \in GOOD_NODES |-> [ m \in Bootstrapping_nodes |-> {} ] ]
    /\ sent_headers      = [ n \in GOOD_NODES |-> [ m \in Bootstrapping_nodes |-> {} ] ]
    /\ sent_ops          = [ n \in GOOD_NODES |-> [ m \in Bootstrapping_nodes |-> {} ] ]
```
