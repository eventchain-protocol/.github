# EventChain

EventChain is an open protocol for histories that anyone holding a copy can
check. A history is an **AOF**, an append-only JSON Lines file. Each line is
one signed entry that names the hash of the line before it. A copy verifies
offline, without asking the system that served it.

**[White paper](https://eventchain-protocol.github.io/)**
· [Specification appendix](https://eventchain-protocol.github.io/appendix-specification.html)

Everything here is version 0. The specification still has open points, and
outcomes may change when they are ruled on.

## Verify a history in your browser

Open <https://eventchain-protocol.github.io/verify/#sample>.

The page loads a four-entry sample with a heartbeat. It reports
**Valid: 4 entries** and **Heartbeat: match**. A heartbeat is a statement the
Hub signs naming the entry everyone should hold at a position. Drop your own
AOF on the page to check it instead. The page's security policy forbids every
network request, so the file never leaves your machine.

## Verify from the command line

Each [release](https://github.com/eventchain-protocol/eventchain/releases)
carries a prebuilt `eventchain-verify` that needs nothing installed. Set `f`
to your platform's file, then download it, check it against `SHA256SUMS`, and
run it on the sample history:

```sh
f=eventchain-verify-macos-aarch64  # or eventchain-verify-linux-x86_64, eventchain-verify-linux-aarch64
base=https://github.com/eventchain-protocol/eventchain/releases/latest/download
curl -fL -O "$base/$f" -O "$base/SHA256SUMS"
shasum -a 256 -c --ignore-missing SHA256SUMS  # on Linux: sha256sum -c --ignore-missing SHA256SUMS
chmod +x "$f"
curl -fLO https://raw.githubusercontent.com/eventchain-protocol/conformance/main/vectors/v0-lifecycle.jsonl
"./$f" aof v0-lifecycle.jsonl
```

It prints one JSON line per AOF line and exits 0:

```text
{"line":1,"outcome":"valid","event_id":"dBnWt41McCT7DYaltzQSAi5KH4NttgOuOGKU-xuSH8k"}
{"line":2,"outcome":"valid","event_id":"SlHtaGwdfQ_0t8FVR69nzBOfBBiPko486PU_rhhfwSg"}
{"line":3,"outcome":"valid","event_id":"d4xf0V-OjErAonXOTwrgrvVZIolIv61wcGQtVjB3YBQ"}
{"line":4,"outcome":"valid","event_id":"LkOkJDZAAs92ykSQA5yEf1d7tVxF8vYu_MV9YU5qV08"}
```

The `event_id` is the SHA-256 of the line's exact bytes. To see a failure,
run it on the same history with the last signature altered:

```sh
curl -fLO https://raw.githubusercontent.com/eventchain-protocol/conformance/main/vectors/signature/altered-signature.jsonl
"./$f" aof altered-signature.jsonl
```

```text
{"line":1,"outcome":"valid","event_id":"dBnWt41McCT7DYaltzQSAi5KH4NttgOuOGKU-xuSH8k"}
{"line":2,"outcome":"valid","event_id":"SlHtaGwdfQ_0t8FVR69nzBOfBBiPko486PU_rhhfwSg"}
{"line":3,"outcome":"valid","event_id":"d4xf0V-OjErAonXOTwrgrvVZIolIv61wcGQtVjB3YBQ"}
{"line":4,"outcome":"signature_invalid"}
```

The exit status is 0 when every line is valid, 1 when any is not, and 2 for
a usage or IO error. The macOS binary needs macOS 15 or later. Download it
with `curl` as above: a file downloaded in a browser is quarantined, and
Gatekeeper refuses to run it. On other platforms,
[build from source](https://github.com/eventchain-protocol/eventchain#quick-start).

## Run the whole stack

With Docker and `curl`, and no other toolchain:

```sh
git clone https://github.com/eventchain-protocol/hub && cd hub
./demo/run.sh
```

The script builds the images and starts a Hub, the server that accepts signed
entries and appends them to a topic's AOF. It signs nine entries, submits
them, and fetches the file and a heartbeat. Then both verifiers check what it
fetched. It ends with:

```text
== result: 9 passed, 0 failed
```

The first run compiles Haskell and Rust code, so it takes a while; later runs
reuse the images. The [Hub README](https://github.com/eventchain-protocol/hub#run-it-in-docker)
covers running it again from scratch and using its HTTP routes.

## What "valid" does and does not mean

A valid line is well formed, names the hash of the line before it, and
carries a signature that verifies under the key on the line. A valid
signature shows that a key signed these bytes. It does not show who holds
the key, that the statement is true, or when it was made. Neither verifier
checks Payload contents, key registries or OpenTimestamps proofs yet.

## Repositories

| Repository | What it holds |
|---|---|
| [eventchain](https://github.com/eventchain-protocol/eventchain) | Haskell libraries: the Producer that signs entries and the Verifier that checks them, with the `eventchain-verify` command |
| [verify](https://github.com/eventchain-protocol/verify) | An independent TypeScript verifier, for Node and the browser page above |
| [hub](https://github.com/eventchain-protocol/hub) | The reference Hub, in Rust. Work in progress, not for real data |
| [conformance](https://github.com/eventchain-protocol/conformance) | Test vectors, and a manifest of 51 cases giving the outcome the specification requires for each |
| [eventchain-protocol.github.io](https://github.com/eventchain-protocol/eventchain-protocol.github.io) | The rendered white paper |

The Haskell and TypeScript verifiers are written separately and share no
code. On the manifest's 29 AOF cases, both give every line its expected
outcome. That agreement is the evidence that they read the format the same
way.

All code is under the BSD 3-Clause License.
