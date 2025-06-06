## mkp224o - vanity address generator for ed25519 anon hidden services

This tool generates vanity ed25519 (hidden service version 3[^1][^2],
formely known as proposal 224) onion addresses.

### Requirements for building

* C99 compatible compiler (gcc and clang should work)
* libsodium (including headers)
* GNU make
* GNU autoconf (to generate configure script, needed only if not using release tarball)
* UNIX-like platform (currently tested in Linux and OpenBSD, but should
  also build under cygwin and msys2).

For debian-like linux distros, this should be enough to prepare for building:

```bash
apt install gcc libc6-dev libsodium-dev make autoconf
```

### Building

Run `./autogen.sh` to generate a configure script, if there isn't one already.

Run `./configure` to generate a makefile.
On \*BSD platforms you may need to specify extra include/library paths:
`./configure CPPFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib"`.

On AMD64 platforms, you probably also want to pass something like
`--enable-amd64-51-30k` to the configure script invocation for faster key generation;
run `./configure --help` to see all available options.

Finally, `make` to start building (`gmake` in \*BSD platforms).

### Usage

mkp224o needs one or more filters to work.
You may specify them as command line arguments,
eg `./mkp224o test`, or load them from file with `-f` switch.

It makes directories with secret/public keys and hostnames
for each discovered service. By default, the working directory is the current
directory, but that can be overridden with `-d` switch.

Use `-s` switch to enable printing of statistics, which may be useful
when benchmarking different ed25519 implementations on your machine.

Use `-h` switch to obtain all available options.

I highly recommend reading [OPTIMISATION.txt][OPTIMISATION] for
performance-related tips.

### FAQ and other useful info

* How do I generate address?

  Once compiled, run it like `./mkp224o test`, and it will try creating
  keys for onions starting with "test" in this example; use `./mkp224o
  -d hs_keys test` to not litter current directory and put all
  discovered keys in directory named "hs_keys".

  If no files are generated use the `./mkp224o -y` to show the private and public keys in yaml format.
  Then use [yaml2hs](https://github.com/cl0ten/yaml2hs) python script to convert the base64 strings to key files.


* How do I make anon use generated keys?

  Copy the keys to the hidden service folder (though technically only `hs_ed25519_secret_key` is required)
  to where you want your service keys to reside:

  ```bash
  sudo cp test54as6d54....anon/* /var/lib/anon/hidden_service
  ```

  You may need to adjust ownership and permissions:

  ```bash
  sudo chown -R anon: /var/lib/anon/hidden_service
  sudo chmod -R u+rwX,og-rwx /var/lib/anon/hidden_service
  ```

  Then edit `anonrc` and add new service with that folder.\
  After reload/restart anon should pick it up.

* How long is it going to take?

  Because of probablistic nature of brute force key generation, and
  varience of hardware it's going to run on, it's hard to make promisses
  about how long it's going to take, especially when the most of users
  want just a few keys.\
  See [this issue][#27] for very valuable discussion about this.\
  If your machine is powerful enough, 6 character prefix shouldn't take
  more than few tens of minutes, if using batch mode (read
  [OPTIMISATION.txt][OPTIMISATION]) 7 characters can take hours
  to days.\
  No promisses though, it depends on pure luck.

* Will this work with onionbalance?

  It appears that onionbalance supports loading usual
  `hs_ed25519_secret_key` key so it should work.

* Is there a docker image?

  No, not for this fork

### Acknowledgements & Legal

To the extent possible under law, the author(s) have dedicated all
copyright and related and neighboring rights to this software to the
public domain worldwide. This software is distributed without any
warranty.
You should have received a copy of the CC0 Public Domain Dedication
along with this software. If not, see [CC0][].

* The whole fork is based on the original https://github.com/cathugger/mkp224o

* `keccak.c` is based on [Keccak-more-compact.c][keccak.c]
* `ed25519/{ref10,amd64-51-30k,amd64-64-24k}` are adopted from
  [SUPERCOP][]
* `ed25519/ed25519-donna` adopted from [ed25519-donna][]
* Idea used in `worker_fast()` is stolen from [horse25519][]
* base64 routines and initial YAML processing work contributed by
  Alexander Khristoforov (heios at protonmail dot com)
* Passphrase-based generation code and idea used in `worker_batch()`
  contributed by [foobar2019][]

[OPTIMISATION]: ./OPTIMISATION.txt
[#27]: https://github.com/cathugger/mkp224o/issues/27
[keccak.c]: https://github.com/XKCP/XKCP/blob/master/Standalone/CompactFIPS202/C/Keccak-more-compact.c
[CC0]: https://creativecommons.org/publicdomain/zero/1.0/
[SUPERCOP]: https://bench.cr.yp.to/supercop.html
[ed25519-donna]: https://github.com/floodyberry/ed25519-donna
[horse25519]: https://github.com/Yawning/horse25519
[foobar2019]: https://github.com/foobar2019
[^1]: https://spec.torproject.org/rend-spec/index.html
[^2]: https://gitlab.torproject.org/tpo/core/torspec/-/raw/main/attic/text_formats/rend-spec-v3.txt
