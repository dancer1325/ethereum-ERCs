---
eip: 55
title: Mixed-case checksum address encoding
author: Vitalik Buterin <vitalik.buterin@ethereum.org>, Alex Van de Sande <avsa@ethereum.org>
discussions-to: https://github.com/ethereum/eips/issues/55
type: Standards Track
category: ERC
status: Final
created: 2016-01-14
---

# Specification

``` python
import eth_utils


def checksum_encode(addr): # Takes a 20-byte binary address as input
    hex_addr = addr.hex()
    checksummed_buffer = ""

    # Treat the hex address as ascii/utf-8 for keccak256 hashing
    hashed_address = eth_utils.keccak(text=hex_addr).hex()

    # Iterate over each character in the hex address
    for nibble_index, character in enumerate(hex_addr):

        if character in "0123456789":
            # We can't upper-case the decimal digits
            checksummed_buffer += character
        elif character in "abcdef":
            # Check if the corresponding hex digit (nibble) in the hash is 8 or higher
            hashed_address_nibble = int(hashed_address[nibble_index], 16)
            if hashed_address_nibble > 7:
                checksummed_buffer += character.upper()
            else:
                checksummed_buffer += character
        else:
            raise eth_utils.ValidationError(
                f"Unrecognized hex character {character!r} at position {nibble_index}"
            )

    return "0x" + checksummed_buffer


def test(addr_str):
    addr_bytes = eth_utils.to_bytes(hexstr=addr_str)
    checksum_encoded = checksum_encode(addr_bytes)
    assert checksum_encoded == addr_str, f"{checksum_encoded} != expected {addr_str}"


test("0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed")
test("0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359")
test("0xdbF03B407c01E7cD3CBea99509d93f8DDDC8C6FB")
test("0xD1220A0cf47c7B9Be7A2E6BA89F429762e7b9aDb")

```

* | English, (TODO: where does it appear)
  * the address is converted -- to -- hex
  * if the `i`th digit == letter (_Example:_ `abcdef`) & lowercase hexadecimal address's hash's `4*i`th bit == 1 -> print it | uppercase
  * OTHERWISE -> print it | lowercase

# Rationale

* Benefits
  - BACKWARDS compatible -- with -- MANY hex parsers / accept mixed case
    - -> easily introduced | time
  - length == 40 characters
  - 15 check bits / address /
    - net probability / randomly generated address mistyped accidentally pass a check == 0.0247%
* vs
  * ICAP
    * ~50x improvement
  * 4-byte check code
    * NOT good 

# Implementation

```js
const createKeccakHash = require('keccak')

function toChecksumAddress (address) {
  address = address.toLowerCase().replace('0x', '')
  var hash = createKeccakHash('keccak256').update(address).digest('hex')
  var ret = '0x'

  for (var i = 0; i < address.length; i++) {
    if (parseInt(hash[i], 16) >= 8) {
      ret += address[i].toUpperCase()
    } else {
      ret += address[i]
    }
  }

  return ret
}
```

```
> toChecksumAddress('0xfb6916095ca1df60bb79ce92ce3ea74c37c5d359')
'0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359'
```

* Keccak256 hash's input
  * == lowercase hexadecimal string (== hex address encoded -- as -- ASCII)

  ```
  var hash = createKeccakHash('keccak256').update(Buffer.from(address.toLowerCase(), 'ascii')).digest()
  ```

# Test Cases

```
# All caps
0x52908400098527886E0F7030069857D2E4169EE7
0x8617E340B3D01FA5F11F306F4090FD50E238070D
# All Lower
0xde709f2102306220921060314715629080e2fb77
0x27b1fdb04752bbc536007a920d24acb045561c26
# Normal
0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed
0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359
0xdbF03B407c01E7cD3CBea99509d93f8DDDC8C6FB
0xD1220A0cf47c7B9Be7A2E6BA89F429762e7b9aDb
```
