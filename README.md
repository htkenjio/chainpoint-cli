# Chainpoint CLI 

[![JavaScript Style Guide](https://cdn.rawgit.com/feross/standard/master/badge.svg)](https://github.com/feross/standard)

[![npm](https://img.shields.io/npm/l/chainpoint-cli.svg)](https://www.npmjs.com/package/chainpoint-cli)
[![npm](https://img.shields.io/npm/v/chainpoint-cli.svg)](https://www.npmjs.com/package/chainpoint-cli)

A Command Line Interface (CLI) for creating and verifying Chainpoint proofs. See https://chainpoint.org

The Chainpoint CLI lets you submit hashes to a Chainpoint Node on the Tierion Network. Nodes periodically aggregate hashes and send data to Tierion Core for anchoring the hash to public blockchains.

The CLI lets you retrieve and verify a Chainpoint proof. Each proof cryptographically proves the integrity and existence of data at a point in time.

The CLI also maintains a simple local database that keeps track of
every hash you submit, and stores and manages Chainpoint proofs
locally for easy retrieval, export, and verification.

## Installation

### Prerequisites

The CLI requires Node.js version 7.6.0 or higher.

It is recommended to use a modern version of Node.js and its companion `npm` tool when installing the CLI.

An excellent way of doing so is to use the [Node Version Manager](https://github.com/creationix/nvm/blob/master/README.md) (`nvm`). Here are the quick install steps to get yourself set up with a current version of Node.js and `npm` using `nvm`.

Install NVM (optional)

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.4/install.sh | bash
```

Once you run this installer its recommended to close your current terminal and open a new one to ensure the install takes effect.

Now you can verify the install with this command, which should return the text `nvm`:

```
command -v nvm
```

Install the current Node.js and `npm`

```
nvm install node
```

Check the versions of Node and `npm` installed once the installation completes successfully. The version numbers you see might vary.

```
$ node --version
v8.2.1
$ npm --version
5.3.0
```

Set this version of Node.js/`npm` to be your system default.

```
nvm alias default node
```

Confirm the version(s) of Node you've installed with `nvm`. The one that the arrow points to should not be `system` but one of the versions that `nvm` manages.

```
nvm list
        v6.11.2
->       v8.2.1
         system
default -> node (-> v8.2.1)
```

All set! Move on to installing the CLI.

### Installing

Ensure you have Node.js version `7.6.0` or higher. If not, install a modern version, perhaps using the `nvm` install instructions above.

```
node --version
v8.2.1
```

We recommend globally installing the CLI from the
[npmjs.com](https://www.npmjs.com/) package respository.

Installing the CLI assumes you already have the `npm` command line tool, which is installed alongside Node.js. You can learn how to install `npm` using the `nvm` Node Version Manager as shown above, or at the [npmjs.com/get-npm](https://www.npmjs.com/get-npm) page.

```
npm install -g --production chainpoint-cli
```

This will install the `chainpoint-cli` npm package, which installs
the `chp` command line tool. You can check that `chp` is installed
and available with the command:

```
which chp
```

**Pro Tip**: You'll note that many of the `curl` samples in this document have been 'prettified' with [jq](https://stedolan.github.io/jq/) for easier readability. It is a powerful tool that lets you work with and explore `JSON` on the command line in new and interesting ways. Its highly recommended to [install this tool](https://stedolan.github.io/jq/download/) following the instructions for your platform.

## Usage

You can get an overview of the CLI usage by typing the command (`chp`). The Node that the CLI will
communicate with will be chosen from those advertised as healthy on the network.

On first use, the CLI will create a `~/.chainpoint` directory
where it will store `cli.config`, and `chainpoint-proofs.db`
files. You can edit the `cli.config` file if you'd like to
permanently specify a Node of your own choosing. The database
file will be managed for you and it is not recommended to
modify it yourself. The database stores a record of every hash
you submit, which Node it was submitted to, and a copy of the
proofs.

```
$ chp
Usage: chp <command> [options] <argument>

Commands:
  submit   submit a hash to be anchored
  update   retrieve an updated proof for your hash(es), if available
  verify   verify a proof's anchor claims
  export   export a proof
  list     display the status of every hash in the local database
  show     show the proof for a hash_id_node
  delete   delete a hash from the local database
  version  show the CLI version

Options:
  -s, --server  specify server to use
                                  [string] [default: "http://SERVER"]
  -q, --quiet   suppress all non-error output                          [boolean]
  -j, --json    format all output as json                              [boolean]
  --help        show help                                              [boolean]

You must specify a command.
```

### Submitting a hash

First you'll need to generate a hash of a local file or data
of your choice. We recommended using the [SHA256](https://en.wikipedia.org/wiki/SHA-2) one-way cryptographic hash function. Other hash types will also be accepted as long as they are hex strings between 40 and 128 hex characters (`[0-9a-fA-F]`) in length.

The Tierion Network only sees a hash of your data, never the
original. It is not possible for us to learn anything about your
data from the hash.

To submit a hash, simply call:

```
chp submit 2e75eaf17b8345c67234dfa92e867541ef41dda08baa6f8d5464fac432950794
e8892e30-992a-11e7-a296-01e28520c246 | 2e75eaf17b8345c67234dfa92e867541ef41dda08baa6f8d5464fac432950794 | submitted
```

The output returned consists of three parts:

The `node_hash_id` is `e8892e30-992a-11e7-a296-01e28520c246` in this example. This type 1 UUID serves as a handle to retrieve a proof.

The original hash you submitted (`2e75eaf17b8345c67234dfa92e867541ef41dda08baa6f8d5464fac432950794`) is echoed back.

The action that occurred, `submitted` in this case, is returned.

You can also submit multiple hashes at once by calling `submit` with multiple hashes.

```
chp submit <hash> <hash> <hash>
```

### Updating a Proof

Once a hash has been submitted, it will first be anchored to the `Calendar` and a proof will be generated. A proof that anchors your hash to the Calendar is generally available within ten seconds or less.

```
 chp update e8892e30-992a-11e7-a296-01e28520c246
e8892e30-992a-11e7-a296-01e28520c246 | updated | cal
```

You can see that you call `chp update` and just pass the `node_hash_id` as well.

You'll see echoed back to you the status, where the `cal` at the end indicates that the proof is anchored to the `Calendar`. Later you will see other blockchain anchors become available, such as `btc` to indicate that a
hash was anchored to the Bitcoin blockchain.

You can also call `chp update --all` to update all proofs locally stored.

### Verifying a Proof

Verifying a proof submits it to the Node for cryptographic verification that the hash captured in the proof is anchored all the way up to either the Calendar or to public blockchains. The Calendar contains all of the information needed to verify a proof.

```
chp verify e8892e30-992a-11e7-a296-01e28520c246
e8892e30-992a-11e7-a296-01e28520c246 | verified | cal
```

You can see here that the proof represented by the `node_hash_id` provided is anchored to the Calendar (`cal`) level.

### Viewing a Proof

You can of course view a proof in its entirety by asking
to see the proof associated with a `node_hash_id`.

```
 chp show e8892e30-992a-11e7-a296-01e28520c246 | jq

{
  "@context": "https://w3id.org/chainpoint/v3",
  "type": "Chainpoint",
  "hash": "2e75eaf17b8345c67234dfa92e867541ef41dda08baa6f8d5464fac432950794",
  "hash_id_node": "e8892e30-992a-11e7-a296-01e28520c246",
  "hash_submitted_node_at": "2017-09-14T08:58:42Z",
  "hash_id_core": "e89f7550-992a-11e7-b9a4-013cacb0360d",
  "hash_submitted_core_at": "2017-09-14T08:58:42Z",
  "branches": [
    {
      "label": "cal_anchor_branch",
      "ops": [
        {
          "l": "node_id:e8892e30-992a-11e7-a296-01e28520c246"
        },
        {
          "op": "sha-256"
        },
        {
          "l": "core_id:e89f7550-992a-11e7-b9a4-013cacb0360d"
        },
        {
          "op": "sha-256"
        },
        {
          "l": "nist:1505379480:07be37c4ef9ef7a74dc16406eb054e02de3b89d23e1cbd8ff991bb811c0f2b3f596573849963554ff222c92ee643c2598e78022d4ab4af5df2422c836d89365c"
        },
        {
          "op": "sha-256"
        },
        {
          "l": "3876:1505379530:1:https://a.chainpoint.org:cal:3876"
        },
        {
          "r": "0108f16b4520cfc45399067480dce73107f758132622249ef3f50ce908e2f002"
        },
        {
          "op": "sha-256"
        },
        {
          "anchors": [
            {
              "type": "cal",
              "anchor_id": "3876",
              "uris": [
                "https://a.chainpoint.org/calendar/3876/hash"
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

You can see in this case I piped the output of the `show` sub-command to the `jq` program. This is just for viewing convenience so we can see the pretty-printed version of the proof.

### Other commands

`list` will show you a list of all hash IDs and hashes that have been submitted from this CLI client.

`export` allows you to export a proof in either JSON or binary file formats.

`delete <hash_id>` allows you to manually delete any hash from your local DB.

You can also get JSON output by passing in the `--json` flag. For example:

```
chp verify --json e8892e30-992a-11e7-a296-01e28520c246
```

### Help

You should note that each of the sub-commands also has its own help screen.

```
$ chp submit --help
Usage: submit [options] (<hash> <hash>... | <hash>,<hash>,... )

Options:
  -s, --server  specify server to use
                                  [string] [default: "http://SERVER"]
  -q, --quiet   suppress all non-error output                          [boolean]
  -j, --json    format all output as json                              [boolean]
  --help        show help                                              [boolean]
  ```
