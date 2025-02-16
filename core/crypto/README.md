# crypto

A cryptography library for the Odin language

## Supported

This library offers various algorithms implemented in Odin.
Please see the chart below for some of the options.

## Hashing algorithms

| Algorithm                                                                                                    |                  |
|:-------------------------------------------------------------------------------------------------------------|:-----------------|
| [BLAKE2B](https://datatracker.ietf.org/doc/html/rfc7693)                                                     | &#10004;&#65039; |
| [BLAKE2S](https://datatracker.ietf.org/doc/html/rfc7693)                                                     | &#10004;&#65039; |
| [SHA-2](https://csrc.nist.gov/csrc/media/publications/fips/180/2/archive/2002-08-01/documents/fips180-2.pdf) | &#10004;&#65039; |
| [SHA-3](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf)                                            | &#10004;&#65039; |
| [SHAKE](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf)                                            | &#10004;&#65039; |
| [SM3](https://datatracker.ietf.org/doc/html/draft-sca-cfrg-sm3-02)                                           | &#10004;&#65039; |
| legacy/[Keccak](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf)                                    | &#10004;&#65039; |
| legacy/[MD5](https://datatracker.ietf.org/doc/html/rfc1321)                                                  | &#10004;&#65039; |
| legacy/[SHA-1](https://datatracker.ietf.org/doc/html/rfc3174)                                                | &#10004;&#65039; |

#### High level API

Each hash algorithm contains a procedure group named `hash`, or if the algorithm provides more than one digest size `hash_<size>`\*.
Included in these groups are six procedures.
- `hash_string` - Hash a given string and return the computed hash. Just calls `hash_bytes` internally
- `hash_bytes` - Hash a given byte slice and return the computed hash
- `hash_string_to_buffer` - Hash a given string and put the computed hash in the second proc parameter. Just calls `hash_bytes_to_buffer` internally
- `hash_bytes_to_buffer` - Hash a given string and put the computed hash in the second proc parameter. The destination buffer has to be at least as big as the digest size of the hash
- `hash_stream` - Takes a stream from io.Stream and returns the computed hash from it
- `hash_file` - Takes a file handle and returns the computed hash from it. A second optional boolean parameter controls if the file is streamed (this is the default) or read at once (set to true)

\* On some algorithms there is another part to the name, since they might offer control about additional parameters.
For instance, `SHA-2` offers different sizes.
Computing a 512-bit hash is therefore achieved by calling `sha2.hash_512(...)`.

#### Low level API

The above mentioned procedures internally call three procedures: `init`, `update` and `final`.
You may also directly call them, if you wish.

#### Example

```odin
package crypto_example

// Import the desired package
import "core:crypto/blake2b"

main :: proc() {
    input := "foo"

    // Compute the hash, using the high level API
    computed_hash := blake2b.hash(input)

    // Variant that takes a destination buffer, instead of returning the computed hash
    hash := make([]byte, sha2.DIGEST_SIZE) // @note: Destination buffer has to be at least as big as the digest size of the hash
    blake2b.hash(input, hash[:])

    // Compute the hash, using the low level API
    ctx: blake2b.Context
    computed_hash_low: [blake2b.DIGEST_SIZE]byte
    blake2b.init(&ctx)
    blake2b.update(&ctx, transmute([]byte)input)
    blake2b.final(&ctx, computed_hash_low[:])
}
```
For example uses of all available algorithms, please see the tests within `tests/core/crypto`.

## Implementation considerations

- The crypto packages are not thread-safe.
- Best-effort is make to mitigate timing side-channels on reasonable
  architectures. Architectures that are known to be unreasonable include
  but are not limited to i386, i486, and WebAssembly.
- Some but not all of the packages attempt to santize sensitive data,
  however this is not done consistently through the library at the moment.
  As Thomas Pornin puts it "In general, such memory cleansing is a fool's
  quest."
- All of these packages have not received independent third party review.

## License

This library is made available under the BSD-3 license.