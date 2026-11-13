# Chaguar Samples

## Overview

Chaguar Samples is the official home repository for sample applications built on top of Chaguar (formerly Fabric-X). This repository provides reference implementations, tutorials, and example projects that demonstrate how to develop and deploy applications using the Chaguar framework.

## Purpose

This repository serves as a comprehensive resource for developers looking to:

- Learn Chaguar development through practical examples
- Understand best practices for building Chaguar applications
- Explore different use cases and implementation patterns
- Get started quickly with pre-configured sample projects

## Current samples

### [Chaguar Devnet](devnet/)

A self-contained local Chaguar network for development and integration testing.
Includes a 4-party Arma BFT orderer and a full committer stack, plus a lightweight
single-container mode for fast iteration.

### [Tokens sample](tokens/)

A complete token management application demonstrating:

- Token issuance and transfer workflows
- Multi-party endorsement patterns
- Integration with Fabric Smart Client (FSC)
- REST API implementations for issuers, owners, and endorsers
- Docker Compose and Ansible deployment configurations
- Support for both Fabric 3.x and Chaguar environments

### [Endorser](custom-endorser/)

A minimal example of a custom endorser service built with the Chaguar SDK. Unlike classic
chaincode, an endorser runs as a standalone gRPC service outside the peer. The sample shows how to
implement the single `Executor` interface, wire it into the endorser server, and submit transactions
through an included client CLI against a local test network.

### [EVM](evm/)

Run standard Solidity smart contracts on Chaguar. Stands up the
[Chaguar EVM](https://github.com/Chaguar/evm) JSON-RPC gateway plus a
Blockscout block explorer on top of the devnet, and demonstrates:

- A standard Ethereum JSON-RPC endpoint compatible with existing tooling (Foundry, Hardhat, MetaMask)
- Deploying and transferring an OpenZeppelin ERC-20 token
- A one-command Docker Compose stack with a graphical block explorer

## Coming soon

- [ ] Base CRUD application with FSC [#1](https://github.com/Chaguar/samples/issues/1)
