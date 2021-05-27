# Introduction

## What is bootstrapping

Tezos is a permissionless blockchain, anyone can run a node and attempt to sync with other nodes on the network

It is also possible for a node to fail, go offline for some time, and then attempt to reconnect to the network

*Bootstrapping* is the process of a node syncing with the current state of the network

Nodes start the bootstrapping process after they have established sufficiently many secure connections (via *handshaking*)

## Why do we care

Bootstrapping nodes are particularly vulnerable to attacks since they do not have much info about the current state

General goals of Viable Systems and TezEdge:

- improve safety/security
- ensure liveness
- improve performance
- simplify complex algorithms
- improve transparency and build relationships within the Tezos ecosystem

We use TLA+ to express and verify safety and liveness properties for the TezEdge node

## How we can help

- formal specifications to clarify, simplify, and codify algorithms
- formulate and verify safety and liveness properties
- model checking (esp using *inductive invariants*) to exhaustively verify design and properties

## [Bootstrapping spec](./bootstrapping_spec.html)
