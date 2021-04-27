# oracles-v2

## Summary

Oracle client written in bash that utilizes secure scuttlebutt for offchain message passing along with signed price data to validate identity and authenticity on-chain.

## Design Goals

Goals of this new architecture are:
  1. Scalability
  2. Reduce costs by minimizing number of ethereum transactions and operations performed on-chain.
  3. Increase reliability during periods of network congestion
  4. Reduce latency to react to price changes
  5. Make it easy to on-board price feeds for new collateral types
  6. Make it easy to on-board new Oracles

## Architecture
Currently two main modules:

[Feed]
Each Feed runs a Feed client which pulls prices through Setzer, signs them with an ethereum private key, and broadcasts them as a message to the secure scuttlebutt network.

[Relay]
Relays monitor the gossiped messages, check for liveness, and homogenize the pricing data and signatures into a single ethereum transaction.

## Query Oracle Contracts

Query Oracle price Offchain   
```
rawStorage=$(seth storage <ORACLE_CONTRACT> 0x1)
seth --from-wei $(seth --to-dec ${rawStorage:34:32})
```

Query Oracle Price Onchain

```
seth --from-wei $(seth --to-dec $(seth call <ORACLE_CONTRACT> "read()(uint256)"))
```
This will require the address you are submitting the query from to be whitelisted in the Oracle smart contract.

## Install with Nix

Add Maker build cache:

```sh
nix run -f https://cachix.org/api/v1/install cachix -c cachix use maker
```

Then run the following to make the `omnia`, `ssb-server` and `install-omnia`
commands available in your user environment:

```
nix-env -i -f https://github.com/HOT-Protocol/oracles-v2/tarball/stable
```

Get the Scuttlebot private network keys (caps) from an admin and put it in a file
(e.g. called `secret-ssb-caps.json`). The file should have the JSON format:
`{ "shs": "<BASE64>", "sign": "<BASE64>" }`.

You can use the `install-omnia` command to install Omnia as a `systemd`
service, update your `/etc/omnia.conf`, `~/.ssb/config` and migrate a
Scuttlebot secret and gossip log.


To install and configure Omnia as a feed running with `systemd`:

```
install-omnia feed \
  --from         <ETHEREUM_ADDRESS> \
  --keystore     <KEYSTORE_PATH> \
  --password     <PASS_FILE_PATH> \
  --ssb-caps     <CAPS_JSON_PATH> \
  --ssb-external <PUBLICLY_REACHABLE_IP>
```

For more information about the install CLI:

```
install-omnia help
```

The installed Scuttlebot config can be found in `~/.ssb.config`, more details
about the [Scuttlebot config](https://github.com/ssbc/ssb-config#configuration).

## Development

To build from inside this repo, clone and run:

```
nix-build
```

You can then run `omnia` from `./result/bin/omnia`.

To get a development environment with all dependencies run:

```
nix-shell
cd omnia
./omnia.sh
```

Now you can start editing the `omnia` scripts and run them directly.

### Update dependencies

To update dependencies like `setzer-mcd` use `niv` e.g.:

```
nix-shell
niv show
niv update setzer-mcd
```

To update NodeJS dependencies edit the `nix/node-packages.json` file and run:

```
nix-shell
updateNodePackages
```

### Staging and release process

To create a release candidate (rc) for staging, typically after a PR has
passed its smoke and regression tests and is merged into `master`, checkout
`master` and run:

```
nix-shell --run "release minor"
```

This should bump the version of `omnia` by Semver version level `minor`
and create a new release branch with the resulting version
(e.g. `release/1.1`) and a tag with the suffix `-rc` which indicates a
release candidate that is ready for staging.

When a release candidate has been tested in staging and is deemed stable you can
run the same command in the release branch but without the Semver version level:

```
nix-shell --run release
```

This should add a Git tag to the current commit with its current version
(without suffix) and move the `stable` tag there also.
