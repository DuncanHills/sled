# sled likes eating data! it's pre-alpha

<p>
  <img src="https://github.com/spacejam/rsdb/blob/master/art/tree_face.png" width="20%" height="auto" />
</p>

[![Build Status](https://travis-ci.org/spacejam/sled.svg?branch=master)](https://travis-ci.org/spacejam/sled)
[![crates.io](http://meritbadge.herokuapp.com/sled)](https://crates.io/crates/sled)
[![documentation](https://docs.rs/sled/badge.svg)](https://docs.rs/sled)

A pre-alpha modern embedded database.

```rust
extern crate sled;

let tree = sled::Config::default()
  .path(path)
  .tree();

// set and get
tree.set(k, v1);
assert_eq!(tree.get(&k), Some(v1));

// compare and swap
tree.cas(k, Some(v1), Some(v2));

// scan forward
let mut iter = tree.scan(k);
assert_eq!(iter.next(), Some((k, v2)));
assert_eq!(iter.next(), None);

// deletion
tree.del(&k);
```

# features

* ordered map API
* fully atomic single-key operations, supports CAS
* [zstd](https://github.com/facebook/zstd) compression (use the zstd build feature)
* cpu-scalable lock-free implementation
* SSD-optimized log-structured storage

# goals

1. don't make the user think. the interface should be obvious.
1. don't surprise users with performance traps.
1. don't wake up operators. bring reliability techniques from academia into real-world practice.
1. don't use so much electricity. our data structures should play to modern hardware's strengths.

# plans

* MVCC, transactions, merge operators and snapshots provided via a higher-level `Db` versioned-key interface
* beat [LSM trees](https://en.wikipedia.org/wiki/Log-structured_merge-tree)
  for reads and [traditional B+ trees](https://en.wikipedia.org/wiki/B%2B_tree) for writes
* forward-compatible binary format
* auto-tuning of internal parameters to squeeze out every drop of performance with
  zero operator effort
* bindings for other languages

# warnings

* quite young, there are lots of tests but don't bet a billion
  dollar business on it yet!
* the C API is likely to change rapidly
* the on-disk format is going to change in non-forward compatible ways
  before the `1.0.0` release! after that, we will always support
  forward migrations.
* has not yet received much attention for performance tuning,
  it has an extremely high theoretical performance but there
  is a bit of tuning to get there. currently only around 200k
  operations per second with mixed workloads, and 7 million/s
  for read-only workloads on tiny keys. this will be improving 
  dramatically soon!

# contribution welcome!

want to help advance the state of the art in open source embedded
databases? check out [CONTRIBUTING.md](CONTRIBUTING.md)!

# architecture

lock-free tree on a lock-free pagecache on a lock-free log. the pagecache scatters
partial page fragments across the log, rather than rewriting entire pages at a time
as B+ trees for spinning disks historically have. on page reads, we concurrently
scatter-gather reads across the log to materialize the page from its fragments.

the system is largely inspired by the Deuteronomy architecture, and aims to implement
the best features from RocksDB as well.

# References

* [The Bw-Tree: A B-tree for New Hardware Platforms](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf)
* [LLAMA: A Cache/Storage Subsystem for Modern Hardware](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/llama-vldb2013.pdf)
* [Deuteronomy: Transaction Support for Cloud Data](https://www.microsoft.com/en-us/research/publication/deuteronomy-transaction-support-for-cloud-data/)
* [The Design and Implementation of a Log-Structured File System](https://people.eecs.berkeley.edu/~brewer/cs262/LFS.pdf)
