IELE: Semantics of New Blockchain VM in K
==============================================

In this repository we provide a model of IELE in K.

### Structure

The file [iele-syntax.md](iele-syntax.md) contains the syntax definition of IELE, along with comments that guide the reader through the structure of the language and links to more detailed descriptions of various features. This file is a good starting point for getting familiar with the language.

The file [data.md](data.md) explains the basic data of IELE (including words and some datastructures over them).
This data is defined functionally.

[iele.md](iele.md) is the file containing the semantics of IELE.
This file contains the **configuration** (a map of the state), and a simple imperative execution machine which IELE lives on top of.
It deals with the semantics of opcodes and parsing/unparsing/assembling/disassembling.

[iele-gas.md](iele-gas.md) describes gas price computations. [iele-gas-summary.md](iele-gas-summary.md) summarizes them in a format readable by those who don't know K. 

Finally, the file [Design.md](Design.md) discusses the design rationale of IELE. It also provides more detailed descriptions of various IELE features, as well as differences and similarities with EVM and LLVM.

### Testing

[iele-testing.md](iele-testing.md) loads test-files from the [Ethereum Test Set](https://github.com/ethereum/tests) and executes them, checking that the output is correct.
If the output is correct, the entire configuration is cleared.
If any check fails, the configuration retains the failed check at the top of the `<k>` cell.

[iele-testing.md](iele-testing.md) is also capable of executing tests of arbitrary IELE transactions (subject to the fact that this first release of IELE is still
built on top of an ethereum-like network layer). The test format is based off of, but slightly different from, the ethereum BlockchainTest json test format.

The structure of the json files is as follows:

* The top level is a json object, containing a list of key value pairs whose keys are the names of the tests to be executed and whose values are json objects containing an individual test case. The top-level fields in each test case are described below:
  * "pre": a json object mapping account IDs (20-byte hexadecimal strings) to an object for each account representing the state of that account before the test starts.
  * "blocks": a list of blocks the test should execute, each containing the folowing fields:
    * "transactions": a list of transactions to be executed, each containing the following fields:
      * "nonce": the nonce of the sender of the transaction.
      * "function": the name of the function to be called. Should be the empty string if the "to" field is the empty string.
      * "gasLimit": the gas limit of the transaction.
      * "value": the amount of currency sent with the transaction.
      * "to": the account the transaction is calling a function on. If the transaction creates an account, this field should be the empty string.
      * "arguments": a list of integers representing the input arguments of the top-level function call of the transaction.
      * "contractCode": the contract being uploaded. Should be the empty string if the "to" field is NOT the empty string.
      * "gasPrice": the price to be paid per unit of gas used.
      * "secretKey" OR ("v" AND "r" AND "s"): the signature of the transaction (either as a private key or as an ECDSA signature of the remaining fields) used to sign the transaction.
    * "results": a list of results of each transaction in the block, each containing the following fields:
      * "out": a list of integers representing the output of the top-level function call of the transaction.
      * "status": the exit status of the top-level function call of the transaction.
      * "gas": the amount of gas remaining after the transaction executes.
      * "logs": the keccak256 hash of the rlp-encoded log entries generated by the transaction.
      * "refund": the gas refund after the transaction executes.
    * "blockHeader": a json object containing the partial block header of the current block, with the following fields:
      * "gasLimit": The block gas limit.
      * "number": The block number.
      * "difficulty": The block difficulty.
      * "timestamp": The timestamp of the block.
      * "coinbase": The block's beneficiary.
  * "network": The current network rules. Currently only one ruleset is supported, "Albe".
  * "blockhashes": because we don't specify the entire blocks, it is necessary to explicitly specify the information seen by the iele.blockhash function. This should be a list of 32-byte hashes ending in 0x00 representing the parentHash of the genesis block.
  * "postState": a json object mapping accoutn IDs (20-byte hexadecimal strings) to an object for each account representing the state of that account expected after the test finishes.
  
* Each account listed in the "pre" or "postState" fields has the following fields:
  * "nonce": the nonce of the account.
  * "balance": the balance of currency in the account.
  * "storage": a json object mapping integer storage keys to integer storage values.
  * "code": either "", representing an empty account, a string beginning with "0x", indicating a binary representation of an account, or a filename containing a IELE contract to be assembled and used as the code of the account.
  
* Each integer is encocded as a string in hexadecimal representing the big-endian representation of the integer. The empty string also can be used to represent the integer zero.

For example pure-IELE tests, see `tests/iele`.

Using the Definition
--------------------

### Installing

See [INSTALL.md](INSTALL.md).

### Testing

To execute a VM test, run `./vmtest $file` where `$file` is a JSON file containing the specification of the test.
To execute a Blockchain test, run `./blockchaintest $file` where `$file` is a JSON file containing the specification of the test.

To run only the pure-IELE tests, run `make iele-test`.

For testing the interprocess VM API, first start the vm server
by running `.build/vm/iele-vm 10000 127.0.0.1` in another shell or
in the background.
With the server running, run `make iele-test-node` to run the pure-IELE
tests on the vm server, or run individual tests using `.build/vm/iele-vm-test`.

To execute all currently passing tests, run `make test`. Note that the vm server should have been started for `make test` to succeed.

### Debugging

When executing a test, if a test fails, the intermediate state of the IELE VM is dumped on the command line. This can be used to provide
information about exactly what went wrong with the test, which can be used for debugging purposes. Generally speaking, each dumped state begins with
the string `` `<generatedTop>`(`<k>` `` followed by the current list of instructions to be executed by the VM in balanced parentheses. You can usually
determine what went wrong with the test by examining the first entry in the list (i.e., everything up to the first `~>`).

Some examples of common errors you may encounter are below, with each example followed by a description of the error.

```
loadTx(#token("966588469268559010541288244128342317224451555083","Int"))
```

In this example, either a transaction was signed incorrectly, or the sender of the transaction as specified by its signature does not exist.

```
`check__IELE-TESTING`(`_:__IELE-DATA`(#token("\"account\"","String"),`{_}_IELE-DATA`(`_,__IELE-DATA`(`_:__IELE-DATA`(#token("91343852333181432387730302044767688728495783936","Int"),`{_}_IELE-DATA`(`_,__IELE-DATA`(`_:__IELE-DATA`(#token("\"storage\"","String"),`_Map_`(`_|->_`(#token("0","Int"),#token("10001","Int")),`_|->_`(#token("2428090106599461928744973076844625336880384098059","Int"),#token("10000","Int")))),`.List{"_,__IELE-DATA"}`(.KList)))),`.List{"_,__IELE-DATA"}`(.KList)))))
```

Here account 91343852333181432387730302044767688728495783936 (in decimal) has different values for storage than expected by the test. Similar errors exist for the nonce, balance, and code of an account. You can refer to the `<storage>` cell in the dumped state for information on their actual values. The correct storage cell is nested within an `<account>` cell whose `<acctID>` is the address of the account in decimal.

```
`check__IELE-TESTING`(`_:__IELE-DATA`(#token("\"out\"","String"),`[_]_IELE-DATA`(`_,__IELE-DATA`(#token("\"0x01\"","String"),`.List{"_,__IELE-DATA"}`(.KList)))))
```

Here is a similar error, but the error states that the transaction returned an incorrect list of values. The actual return value can be found in the `<output>` cell.

If other errors occur that are not on this list, feel free to contact us on Gitter or Riot and we will be happy to assist in helping you understand. Future releases of IELE may also have improved debugging information making this easier to interpret.

Contacting Us
-------------

In the spirit of open source, community-driven development, we will be holding all IELE discussions on our channels:

* [#IELE:matrix.org](https://riot.im/app/#/room/#IELE:matrix.org) on [Riot](https://riot.im)
* [runtimeverification/iele-semantics](https://gitter.im/runtimeverification/iele-semantics) for issues/pull-requests.
