
## Notes

### Burn In

Ensure that Parity DevOps has run the new release on Westend, Kusama, and
Polkadot validators for at least 12 hours prior to publishing the release.

### Build Artifacts

Add any necessary assets to the release. They should include:

- Linux binary
- GPG signature of the Linux binary
- SHA256 of binary
- Source code
- Wasm binaries of any runtimes

The release notes should list:

- The priority of the release (i.e., how quickly users should upgrade) - this is
    based on the max priority of any *client* changes.
- Which native runtimes and their versions are included
- The proposal hashes of the runtimes as built with
    [srtool](https://gitlab.com/chevdor/srtool)
- Any changes in this release that are still awaiting audit

The release notes may also list:

- Free text at the beginning of the notes mentioning anything important
    regarding this release
- Notable changes (those labelled with B[1-9]-* labels) separated into sections


A runtime upgrade must bump the spec number. This may follow a pattern with the
client release (e.g. runtime v12 corresponds to v0.8.12, even if the current
runtime is not v11).

Any previous `on_runtime_upgrade` functions from old upgrades must be removed
to prevent them from executing a second time. The `on_runtime_upgrade` function
can be found in `runtime/<runtime>/src/lib.rs`.

### New Migrations

Ensure that any migrations that are required due to storage or logic changes
are included in the `on_runtime_upgrade` function of the appropriate pallets.

### Extrinsic Ordering

Offline signing libraries depend on a consistent ordering of call indices and
functions. Compare the metadata of the current and new runtimes and ensure that
the `module index, call index` tuples map to the same set of functions. In case
of a breaking change, increase `transaction_version`.

To verify the order has not changed, you may manually start the following [Github Action](https://github.com/paritytech/polkadot/actions/workflows/extrinsic-ordering-check-from-bin.yml). It takes around a minute to run and will produce the report as artifact you need to manually check.

The things to look for in the output are lines like:
  - `[Identity] idx 28 -> 25 (calls 15)` - indicates the index for `Identity` has changed
  - `[+] Society, Recovery` - indicates the new version includes 2 additional modules/pallets.
  - If no indices have changed, every modules line should look something like `[Identity] idx 25 (calls 15)`

Note: Adding new functions to the runtime does not constitute a breaking change
as long as the indexes did not change.

### Proxy Filtering

The runtime contains proxy filters that map proxy types to allowable calls. If
the new runtime contains any new calls, verify that the proxy filters are up to
date to include them.
