# Bootstrapping TLA+ spec

Each bootstrapping node sends three types of messages:

- `Get_current_branch` - request a peer's current head and a sample of their history
- `Get_block_headers` - request the headers corresponding to the given list of block hashes
- `Get_operations` - request the operations with the given hashes

and receives three types of messages:

- `Current_branch` - node's respond with their current head and sample of history
- `Block_header` - node's respond with the requested header or do not respond
- `Operation` - node's respond with the requested operation or do not respond

## Assumptions/Simplifications

- all messages are for the same `chain_id`
- all headers are for the same `proto_level`
- no `timestamp`s
- only three message types:
  - `Get_current_branch`/`Current_branch`
  - `Get_block_headers`/`Block_header`
  - `Get_operations`/`Operation`
- i.e. no `Current_head` or mempool
- bootstrapping nodes have the ability to request a specific history sample, by levels
- bootstrapping nodes only communicate with established nodes
- node's are blacklisted when we timeout while communicating with them, no matter the cause
- lengths of all chains are bounded
- number of operations per block is bounded

## Constants/Parameters

- `BAD_NODES` - byzantine nodes
- `GOOD_NODES` - nodes who follow the protocol
- `BAD_BOOTSTRAPPING` - byzantine bootstrapping nodes
- `GOOD_BOOTSTRAPPING` - bootstrapping nodes who follow the protocol
- `MIN_PEERS` - minimum number of peers
- `MAX_PEERS` - maximum number of peers
- `MAX_LEVEL` - maximum level of a block
- `MAX_OPS` - maximum number of operations per block

- `CURRENT_HEAD` - each good node's current head
- `BLOCKS` - each good node's blocks
- `VALIDATOR` - Blocks -> { "known_valid", "known_invalid", "unknown" }
- `SAMPLES` - GOOD_NODES * Bootstrapping_nodes -> Seq_n(Levels)
- `HASH_BLOCK_MAP` - BlockHashes -> Headers

## Variables

### Bootstrapping

- `b_blacklist` - set of `bn`'s blacklisted peers
- `b_messages` - set of `bn`'s messages

#### Local

Each bootstrapping node `bn` has:

- `phase` - the state of `bn`, governs the actions taken by `bn`
- `connections` - set of nodes with whom `bn` may exchange messages
- `current_head` - current head (initially `genesis` header)
- `fittest_head` - the fittest header of each of `bn`'s peers
- `header_pipe` - queue of validated headers to form blocks
- `operation_pipe` - queue of validated operations to form blocks
- `validated_blocks` - set of validated blocks to add to `bn`'s chain

#### History

Each bootstrapping node `bn` has the following history variables:

- `phase_trace` - trace of each phase experienced

- `sent_get_branch` - set of nodes from whom `bn` has requested a current branch
- `sent_get_headers` - set of requested block hashes from each of `bn`'s peers
- `sent_get_ops` - set of requested operation hashes from each of `bn`'s peers
- `recv_branch` - set of samples (block hashes) received from each of `bn`'s peers
- `recv_header` - set of block headers received from each of `bn`'s peers
- `recv_operation` - set of pairs of block hashes and operations received from each of `bn`'s peers

#### Memory/Space

- `mem_size` - estimated amount of memory used by `bn`

### Node 

Each good node has:

- `n_blacklist` - set of blacklisted peers
- `n_messages` - set of messages
- `serving_headers` - headers which have been requested by bottstrapping nodes
- `serving_ops` - operations which have been requested by bootstrapping nodes

#### History

Each good node keeps a collection of each type of data sent and received:

- `sent_branch` - set of peers to whom they have sent a `Current_branch` message
- `sent_headers` - set of headers sent to each peer who has requested a `Block_header`
- `sent_ops` - set of operations sent to each peer who has requested an `Operation`
- `recv_get_branch` - set of peers who have requested the current branch
- `recv_get_headers` - set of block hashes each peer has requested the corresponding header for
- `recv_get_ops` - set of operation hashes each peer has requested the corresponding operation for

### Searching for major branch

The bootstrapping node has sufficiently many connections

- Doing:
  - high priority: requesting current branches from all peers
  - low prioity: requesting block headers of received block hashes
  - attempting to find a header that a majority of peers have in their chain

- Not doing:
  - not requesting operations
  - not applying blocks 

### Validating a major branch

The bootstrapping node has seen majority peer support for a header

The bootstrapping node only needs to download each (header, operations) pair once and cross-validate against peer hashes

- Doing:
  - requesting latest block headers within a suffix of the major branch
  - requesting corresponding operations
  - cross-validating a portion of the history
  - enqueuing headers and operations in the pipes

- Not doing:
  - not applying blocks

If a new major header (with higher fitness) is found during this phase:

- adjust `current_head`
- adjust level of validation phase
- if on the same branch, keep the same requests and add any new ones
- if not on the same branch, delete requests down to common branch and add new ones

### Applying blocks

The bootstrapping node has received all requested headers and operations for the target suffix

- Doing:
  - continue enqueuing headers and operations into the pipes
  - applying blocks

- Not doing:
  - not requesting branches
  - not requesting headers
  - not requesting operations

If a new major header (with higher fitness) is found during this phase:

- adjust `current_head`
- return to validation phase
- if on the same branch, keep everything in the pipes and add any new headers/operations
- if not on the same branch, delete headers and operations in pipes down to the common branch

## Properties

### Safety

- bootstrapping connections

```
\A bn \in GOOD_BOOTSTRAPPING :
  /\ num_peers(bn) <= MAX_PEERS
  /\ b_blacklist[bn] \cap connections[bn] = {}
```

- bootstrapping messages

```
\A bn \in GOOD_BOOTSTRAPPING :
  { msg.from : msg \in b_messages } \subseteq connections[bn]
```

- phase consistency

```
\A bn \in GOOD_BOOTSTRAPPING :
  LET p == phase[bn] IN
  /\ p \notin Phase_init => num_peers(bn) >= MIN_PEERS
  /\ \E l \in Levels :
      p = major_phase(1..l) <=> l = highest_major_level(bn)
  /\ \E l \in Levels :
      p = apply_phase(1..l) <=> l = highest_major_level(bn)
```

- current_head consistency

```
\A bn \in GOOD_BOOTSTRAPPING :
  /\ 2 * Cardinality({ n \in good_conns(bn) : current_head[bn] \in good_headers(n) }) > Cardinality(good_conns(bn))
  /\ highest_major_level(bn) = 0 => current_head[bn] = gen_header
```

### Liveness

- fitness is monotonic increasing

```
\A bn \in GOOD_BOOTSTRAPPING :
  LET old_head  == current_head[bn]
      new_head  == current_head'[bn]
  IN
  [][ old_head /= new_head => old_head.fitness < new_head.fitness ]_vars
```

- allowable transitions

```
\* Init -> Search
\A bn \in GOOD_BOOTSTRAPPING :
  LET old_phase == phase[bn]
      new_phase == phase'[bn]
  IN
  [][ /\ old_phase \in Phase_init
      /\ old_phase /= new_phase
      =>
      /\ new_phase \in Phase_search
      /\ connections[bn] = {}
      /\ connections'[bn] /= {} ]_vars
```

```
\* Search -> Major
\A bn \in GOOD_BOOTSTRAPPING :
  LET old_phase == phase[bn]
      old_head  == current_head[bn]
      new_phase == phase'[bn]
      new_head  == current_head'[bn]
  IN
  [][ /\ old_phase \in Phase_search
      /\ old_phase /= new_phase
      =>
      /\ new_phase \in Phase_major
      /\ old_head.fitness < new_head.fitness
      /\ old_head.level < new_head.level ]_vars
```

```
\* Major -> { Major, Apply }
\A bn \in GOOD_BOOTSTRAPPING :
  LET old_phase == phase[bn]
      old_head  == current_head[bn]
      new_phase == phase'[bn]
      new_head  == current_head'[bn]
  IN
  \* Major -> Apply
  /\ [][ (old_phase \in Phase_major) => new_phase \in Phase_major \cup Phase_apply ]_vars
  \* Major -> Major
  /\ [][ \E l \in Levels :
          /\ old_phase = major_phase(1..l)
          /\ old_head.level = l
          /\ new_phase \in Phase_major
          =>
          \E k \in Levels :
              /\ k > l
              /\ new_head.level = k
              /\ new_phase = major_phase(1..k) ]_vars
```

```
\* Apply -> { Major, Search }
\A bn \in GOOD_BOOTSTRAPPING :
  LET old_phase == phase[bn]
      old_head  == current_head[bn]
      new_phase == phase'[bn]
      new_head  == current_head'[bn]
  IN
  \* Apply -> Search
  /\ [][ /\ old_phase \in Phase_apply
         /\ new_phase /= new_phase
          => new_phase \in Phase_search \cup Phase_major ]_vars
  \* Apply -> Major
  /\ [][ /\ old_phase \in Phase_apply
         /\ old_phase \in Phase_major
          =>
          /\ old_head.fitness < new_head.fitness
          /\ old_head.level < new_head.level ]_vars
```

- if progress can be made, it will be

```
\A bn \in GOOD_BOOTSTRAPPING :
  LET curr_hd == current_head[bn] IN
  \E hd \in major_headers(bn) :
      <>( \/ hd = curr_hd
          \/ hd.fitness < curr_hd.fitness )
```


## Phase diagram

[![](./phase_diagram.dot.svg)](./phase_diagram.dot.svg)

## Performance

### Memory

- will ultimately include memory usage estimates in specification and utilize model checking to improve performance
