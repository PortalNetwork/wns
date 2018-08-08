# Developer Guide

Implementations for registrars and local resolvers for the Wanchain Name Service.

For documentation of the WNS system, see [doc](https://github.com/wanchain/wns/tree/master/docs).

To run unit tests, clone this repository, and run:

    $ npm install
    $ npm test

## WNSRegistry.sol
Implementation of the WNS Registry, the central contract used to look up resolvers and owners for domains.

## FIFSRegistrar.sol
Implementation of a simple first-in-first-served registrar, which issues (sub-)domains to the first account to request them.

## Registrar.sol
Simplified version of the above, with no support for renewals. This is the current proposal for interim registrar of the WNS system until a permanent registrar is decided on.

## PublicResolver.sol
Simple resolver implementation that allows the owner of any domain to configure how its name should resolve. One deployment of this contract allows any number of people to use it, by setting it as their resolver in the registry.

# WNS Registry interface

The WNS registry is a single central contract that provides a mapping from domain names to owners and resolvers.

The WNS operates on 'nodes' instead of human-readable names; a human readable name is converted to a node using the namehash algorithm, which is as follows:

	def namehash(name):
	  if name == '':
	    return '\0' * 32
	  else:
	    label, _, remainder = name.partition('.')
	    return sha3(namehash(remainder) + sha3(label))

The registry's interface is as follows:

## owner(bytes32 node) constant returns (address)
Returns the owner of the specified node.

## resolver(bytes32 node) constant returns (address)
Returns the resolver for the specified node.

## setOwner(bytes32 node, address owner)
Updates the owner of a node. Only the current owner may call this function.

## setSubnodeOwner(bytes32 node, bytes32 label, address owner)
Updates the owner of a subnode. For instance, the owner of "foo.com" may change the owner of "bar.foo.com" by calling `setSubnodeOwner(namehash("foo.com"), sha3("bar"), newowner)`. Only callable by the owner of `node`.

## setResolver(bytes32 node, address resolver)
Sets the resolver address for the specified node.

# Resolver interface

Resolvers must implement one mandatory method, `has`, and may implement any number of other resource-type specific methods. Resolvers must `throw` when an unimplemented method is called.

## has(bytes32 node, bytes32 kind) constant returns (bool)

Returns true if the specified node has the specified record kind available. Record kinds are defined by each resolver type .

`has()` must return false iff the corresponding record type specific methods will throw if called.

## addr(bytes32 node) constant returns (address ret)

Implements the addr resource type. Returns the Ethereum address associated with a node if it exists, or `throw`s if it does not.


# Getting started
Install Truffle

	$ npm install -g ganache-cli

Launch the RPC client, for example TestRPC:

	$ ganache-cli

Deploy `ENS` and `FIFSRegistrar` to the private network, the deployment process is defined at [here](migrations/2_deploy_contracts.js):

	$ truffle migrate --network dev.fifs

alternatively, deploy the `HashRegistrar`:

	$ truffle migrate --network dev.auction

Testing all contract by TestRPC:

    $ truffle test

If you want test on testnet, remove all test file except TestIntegrate.js under test directory,
Then create four account,and deposit the accounts, and unlock them:

    $ personal.unlockAccount(eth.accounts[0], 'yourpasswd', 999999);
    $ personal.unlockAccount(eth.accounts[1], 'yourpasswd', 999999);
    $ personal.unlockAccount(eth.accounts[2], 'yourpasswd', 999999);
    $ personal.unlockAccount(eth.accounts[3], 'yourpasswd', 999999);

make a simple integate test on wanchain testnet:
    $ truffle test --network testnet

Check the truffle [documentation](http://truffleframework.com/docs/) for more information.