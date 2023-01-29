# PUID
## Probably Unique IDentifier

**PUID** aims to be a general, flexible mechanism for creating random string for use as random IDs. Library implementations also strive to be fast and efficient.

For the purposes of **PUID**, random IDs are a bunch of random strings generated for use as IDs within a specified context. (See [Understanding Random IDs](https://github.com/puid/.github/blob/8ba9028f106cec9b3c2ebf0fd456d26a810df075/assets/docs/RandomId.md) for detailed overview.) As such, random ID generation can be viewed as a _transformation_ of some entropy source into a string _representation_ of captured entropy. A general purpose library for such transformation should provide flexible control over the **entropy source**, the **characters** used, and the resulting **captured randomness**.


### ID Generation

#### Entropy source

**PUID** provide full control over the bytes used as input into the transformation process, while also providing three default sources: **CSPRNG**, **PRNG** and **Fixed Bytes**.

- CSPRNG: Crytographically Secure Pseudo Random Number Generator. This entropy source is recommended for IDs used in context where security is of importance. Each library utilizes language specific mechanisms to access the crytographically secure random bytes provided by the underlying system.

- PRNG: Pseudo Random Number Generator. This entropy source also provides excellent randomess but may be susceptible to certain security related attacks. Again, each library utilizes language specific mechanisms to access randomness provided by the underlying system.

- Fixed Bytes: This entropy source is not random, but provides deterministic bytes useful for testing purposes. An application can use CSPRNG, PRNG, or a custom entropy source for production and configure the use of Fixed Bytes for deterministic testing.

#### Characters

**PUID** provides full control over the characters used in generated IDs. A number of predefined character sets are provided, and custom characters (including unicode) can also be specified. To maximize captured entropy, **PUID** enforces the characters used must be unique.

#### Captured randomness

The randomness of IDs is actually the ultimate metric in random ID generation. Curiously, many random ID generators don't even specify what that metric is or don't provide any direct way to specify its value. Given the importance of the metric, it should be explicitly clear how random random IDs are. **PUID** provides direct specification of the bits of entropy desired for IDs, as well as an intuitive way to specify desired randomness in a total number of generated IDs given a specific risk.

### Correctness

Each **PUID** library includes extensive test suites for correctness:

- Fixed Bytes entropy sources provide bit level inspection of the correctness in transforming source entropy into random IDs.

- Character distribution across large numbers of IDs is tested for uniform distribution using chi-square analysis. These tests are independently and directly performed with the Elixir and Python libraries. All libraries validate ID generation by using comparing output of chi-square tested data with expected output.

### Efficiency

The general, flexible use of **PUID** is its primary goals. However, careful consideration has been taken to make the library implementations fast and efficient as well:

- **PUID** produces IDs from its entropy source using a low-level bit slicing scheme that only uses the number of bits necessary to (conceptually) index into an array of input characters. For example, when using **26** lowercase alpha characters, such an index would require at most **5** bits, so **PUID** sliced input entropy bytes **5** bits at a time. This is in direct contrast to using, for example, a random integer produced from an IEEE 754 float that may well have **53** actual bits of entropy.

- The **PUID** bit-slicing process maintains unused bits for use in generating the next **puid**.

- **PUID** uses the absolute minimum number of bits to produce an ID. Consider, for example, using the **10** vowel characters `aeiouAEIOU` for IDs. The bit-slicing mechanism used by **PUID**  slices **4** bits at a time to produce a (conceptual) index to determine the next character. One bit-slicing strategy would be to inspect the index and, if greater than **9**, throw the **4** bits away and repeat with the next **4** bit slice. Although this would work, it actually wastes bits, so **PUID** takes the process one step further. As example, suppose the sliced bits were `11xx`, that is, the first two sliced bits are `11`. Regardless of the other **2** bits, the index from all **4** bits will be greater than **9**; therefore **PUID** only throws away the first **2** bits and uses the `xx` bits as the beginning of the next bit slice. In this example, the "bit saving" scheme saves **32%** of the bits that would be tossed.

- **PUID** libraries do not actually index into an array of characters, but rather encodes bit-sliced indexes directly for faster ID generation.

### Implementations

**PUID** libraries are currently available for:

- [Elixir](https://github.com/puid/Elixir)
- [JavaScript](https://github.com/puid/JavaScript)
- [Python](https://github.com/puid/Python)
- [Swift](https://github.com/puid/Swift)
