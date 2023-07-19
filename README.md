Dedns is a public censorship-resistant, tamper-proof, 0-setup identity system.

Dedns is meant to be an alternative for applications that need to avoid the usual DNS&TLS-based identity due to its lack of censorship resistance, sovereign cryptographic security, and high barrier to entry (buying DNS name, configuring DNS server, and obtaining TLS certificate).

Unlike DNS&TLS is not a naming system. It sacrifices "Human-meaningful" from Zooko's triangle and uses only self-generated cryptographic identities to achieve two other properties: decentralization and security. 

In Dedns anyone can generate their public-key-based identity, and associate some additional data with it (e.g. connectivity information like public `IP:port`, DNS names, other cryptographic identities, etc.), which can be timestamped, signed over, and published on public servers.

This way Dedns identity can update such publically available information by uploading another signed document with a higher timestamp.

On a high level, a Dedns server is a KV-store mapping Dedns identities to the latest signed document associated with it.

## SPAM and abuse prevention with PoW

To prevent abuse, on the write path (creating or updating identity) each public server can require a certain amount of cash-hash-style POW associated with the identity. I.e. a `timestamp` & `nonce` so that `hash ("dedns" || pubkey || timestamp || nonce) < write_threshold`. The exact `write_threshold` can be adjusted dynamically on each server. The `timestamp` is only used to prevent re-using `nonce`s in the read path (see below).

On the read path (clients querying for a given identity), a server should require a `nonce` & `timestamp` pair, where `timestamp` is within a small window near the current Unix time, and `hash ("dedns" || pubkey  || timestamp || nonce) < read_threshold`. By using this `timestamp`-based construction each client querying for a key is simultaneously "mining" for a better PoW for a given identity, helping create a cryptographicly provable sign of its popularity and importance. Honest servers should collect the best PoW and expose it to allow its replication.

Assuming < 1K document size limits, 1GB of storage/memory would allow serving 1M Dedns identities, and this PoW construction would ensure that the most important and popular identities are always preserved.


*TODO*: write path probably should use the `timestamp` window as well, to phase out malicious spam identities over time.

Public servers are free to employ some additional forms of deciding which identities to hosts, and the protocol could include some standard way to provide user-feedback. E.g. captchas, LN payments, email checks etc. could help new users temporary bootstrap their identities, so that their users could help accumulate PoW for them. However only PoW is universally transferable between instances helping availability.

## Sharding through application-specific identity-masks

Dedns relies on an abundance of public servers, storing redundant information to increase availability. Each server trying to store the same copy of all the data is not the best scheme.

When Dedns is employed in applications, each application might require that the Dedns identity used in it match a certain pattern - e.g. starts with a fixed prefix. E.g. application XYZ generates and allows only Dedns identities starting with `0101` bytes. The XYZ application will just grind a little bit when generating the Dedns identity. This way it's possible to run Dedns server only hosting identities for a set of applications, effectively logically partitioning the identity space.

## Feedback

Please see [discussions page](https://github.com/dpc/dedns/discussions) and feel free to leave your feedback.
