# Introduction

## What is bootstrapping

Since Tezos is a permissionless blockchain, anyone can run a node and attempt to sync with other nodes on the network

It is also possible for a node to fail, go offline for some time, and then attempt to reconnect to the network

*Bootstrapping* is the process of a node syncing with the current state of the network

Nodes start the bootstrapping process after they have established sufficiently many secure connections (via handshaking)

## Why do we care

Bootstrapping nodes are particularly vulnerable to attacks since they do not have much info about the state

General goals of TezEdge:

- improve safety/security
- improve performance

We also aspire to codify and verify liveness properties for the TezEdge node

Using TLA+ to gain insights into space and memory usage is a future goal

## How we can help

- formal specification to clarify and codify design
- formulate and verify safety and liveness properties
- model checking (esp inductive invariants) to exhaustively verify design

## [Bootstrapping spec](./bootstrapping_spec.html)
