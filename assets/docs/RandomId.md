# Understanding Random IDs

![RandomID](https://github.com/puid/assets/blob/dev/pics/RandomID.png?raw=true)

Developers frequently need random strings in applications ranging from long-term (e.g., data store keys) to short-term (e.g. DOM IDs on a web page). These IDs are, of course, of secondary concern. No one wants to think about them much, they just want to be easy to generate.

But developers _should_ think about the random strings they use. The generation of random IDs is a design choice, and just like any other design choice, that choice should be explicit in nature and based on a familiar with why such choices are made. Yet a cursory review of random string libraries, as well as random string usage in many applications, yields a lack of clarity that belies careful consideration.

### Random Strings

#### What is a random string?

Although this may seem to have an obvious answer, there is actually a key, often overlooked subtlety: a random string ***is not random*** in and of itself. To understand this, we need to understand [entropy](https://en.wikipedia.org/wiki/Entropy_\(information_theory\)) as it relates to computers.

A somewhat simplistic statement for entropy from information theory is: _“entropy is a measure of uncertainty in the possible outcomes of an event”_. Given the base 2 system inherent in computers, this uncertainty naturally maps to a unit of bits (known as Shannon entropy). So we see statements like _“this random string has 128 bits of entropy”_. But here is the subtlety:

> A random string does not have inherent entropy

Rather, a random string represents _captured_ entropy, entropy that was produced by _some other_ process. For example, you cannot look at the hex string **’18f6303a’** and definitively say it has 32 bits of entropy. To see why, suppose you run the following code snippet (or [similar](https://gist.github.com/dingosky/5f5b99f212d0639eec542ac79b08b8be)) and get **’18f6303a’**:

```javascript
// JavaScript
const randId = () => Math.random() < 0.5 ? '18f6303a' : '1'
randId()
// => '18f6303a'
```

The entropy of the resulting string **’18f6303a’** is **1** bit. That’s it; **1** bit. The same entropy as when the outcome **’1'** is observed. In either case, there are two equally possible outcomes and the resulting entropy is therefore **1** bit. It’s important to have this clear understanding:

> Entropy is a measure in the uncertainty of an event, independent of the representation of that uncertainty

In information theory you would state the random process has two symbols, **18f6303a** and **1**, and the outcome is equally likely to be either symbol. Hence there is **1** bit of entropy in the process. The symbols don’t matter. It would be much more likely to see the symbols **T** and **F**, or **0** and **1**, or even **ON** and **OFF**, but regardless, the process _produces_ **1** bit of entropy and the symbols used to _represent_ that entropy do not effect the entropy itself.

###Generating random strings 

Random string generation can be viewed as the _transformation_ of entropy (typically provided by the computing system) into a character _representation_ of “captured” entropy. There are three key aspects in the generation of random strings: **entropy source**, **string characters** and resulting **captured randomness**.

#### Entropy source

Random string generators need an external source of entropy and typically use a system resource for that entropy. Regardless of the source, it is important to appreciate that the properties of the generated random strings depend on the characteristics of the source itself. For example, whether a random string is suitable for use as a secure token depends on the security characteristics of the entropy source, not on the string representation of the resulting token. Furthermore, the random strings can only be as random as the source.

#### String characters

The characters (symbols) used for a random string do not determine the resulting entropy. However, the number of unique characters available does. Under the assumption that each character is equally probable (which maximizes entropy) it is easy to show the entropy per character is a constant `log2(N)`, where `N` is of the number of characters available. This value represents the amount of source entropy “captured” per character.

#### Captured randomness

String randomness is a measure of the total captured entropy, i.e. the entropy per character times the number of characters in the string. The _quality_ of that randomness is directly tied to the process that serves as the entropy source. The _randomness_ depends on the number of available characters and the length of the string. And finally we can state:

> A random string is a character representation of captured entropy

### Random IDs

Now that we have a definition of a random string, let’s turn our attention to using these strings for random IDs. In this context, random IDs can be viewed as a bunch of random strings, no two of which are the same, i.e, all the IDs are unique.

#### Uniqueness

So, let’s simply generate a bunch of random strings for use as IDs. But there is a catch; a big catch:

> Random strings do not produce unique IDs

Recall that entropy is the measure of uncertainty in the possible outcomes of an event. It is critical that the uncertainty of each event is _independent_ of all prior events. This means two separate events can produce the same result (i.e., the same ID); otherwise the process isn’t random. You could, of course, compare each generated random string to all prior IDs and thereby achieve uniqueness. But some such post-processing must occur to ensure random IDs are truly unique.

Deterministic uniqueness checks, however, incur significant processing overhead and are rarely used. Instead, developers (knowingly?) relax the requirement that random IDs are truly, deterministically unique for a much lesser standard, one of probabilistic uniqueness. We “trust” that randomly generated IDs are unique by virtue of the chance of a repeated ID being very low.

And once again, we reach a point of subtlety. (And we thought random strings were easy!) The “trust” that randomly generated IDs are unique actually turns entropy as it’s been discussed thus far on it’s head. Instead of viewing entropy as a measure of uncertainty in the _generation_ of IDs, we consider entropy as a measure of the probability that no two IDs will be the same. To be sure, we want this probability to be very low, but for random strings it ***cannot be zero!*** And to be clear, _entropy is not such a measure_. Not directly anyway. Yes, the higher the entropy, the lower the probability, but it takes a bit of math to correlate the two in a proper manner.

Furthermore, the probable uniqueness of ID generation is always in some limited context. Consider IDs for a data store. You don’t care if a generated ID is the same as an ID used in another data store in another application in another company in a galaxy far, far away. You care that the IDs are (probably) unique within the context of your application.

To recap, random string generation does not produce unique IDs, but rather, IDs that are probably unique (within some context). That subtlety is critically important and fully at odds with a version 4 **UUID**. Why? Because being generated via a random process means a **UUID** _cannot be unique_. As a corollary, it can’t be _universal_ either. Though we don’t really need an ID to be universal anyway, the fact remains, a **UUID** isn’t **UU**.

#### ID Randomness

So what does the statement _“these IDs have 122 bits of entropy”_ actually mean? Entropy is a measure of uncertainty after all, and we’re concerned that our IDs be unique, probably unique anyway. So what does _“122 bits of entropy”_ mean for the probable uniqueness of IDs?

First, let’s be clear what it doesn’t mean. We’re concerned with uniqueness of a bunch of strings in a certain context. The randomness of _any one_ of those strings isn’t the real concern. Yes, we can say _“given 122 bits of entropy”_ each string has a probability of **1/2¹²²** of occurring. And yes, that certainly makes the occurrence of any particular string rare. But with respect to the uniqueness of random IDs, it simply isn’t “enough” to tell the whole story.

And here again we hit another subtlety. It turns out the question , as posed, is under-specified, i.e. it is not specific enough to be answered. To properly determine how entropy relates to the probable uniqueness of IDs, we need to specify _how many) IDs are to be generated in a certain context. Only then can we determine the probability of uniqueness. So our question really needs to be: given `N` bits of string entropy, what is the probability of generating a total of `T` unique IDs?

Fortunately, there is a mathematical correlation between entropy and the probability of uniqueness. This correlation is often explored via the [Birthday Paradox](https://en.wikipedia.org/wiki/Birthday_problem#Cast_as_a_collision_problem). Why paradox? Because the relationship, when cast as a problem of unique birthdays in some number of people, is initially quite surprising. But nonetheless, the relationship exists and is well-known.

At this point we can note that rather than say _“these IDs have N bits of entropy”_, we actually want to express ID randomness as _“generating a total T of these IDs has a risk R of a repeat”_.

### Efficiency

The efficiency of generating random IDs has no bearing on the statistical characteristics of the IDs themselves. But who doesn’t care about efficiency? Unfortunately, most random string generation, it seems.

#### Entropy source

As previously stated, random ID generation is basically a _transformation_ of an entropy source into a character _representation_ of captured entropy. But the entropy of the source and the entropy of the captured ID ***is not the same thing***.

To understand the difference, we’ll investigate an example that is, surprisingly, quite common. Consider the following strategy for generating random strings: using a fixed list of `k` characters, use a random uniform integer `i, 0 <= i < k`, as an index into the list to select a character. Repeat this `n` times, where `n` is the length of the desired string. In code this might look like (or [similar](https://gist.github.com/dingosky/86328fc8b51d6b3037087ab1a8d14b4f)):

```javascript
// JavaScript
const commonId = (n) => {
  const chars = 'abcdefghijklmnopqrstuvwxyz'
  let id = '';
  for (let i = 0; i < n; i++) {
    id += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return id
}
commonId(8)
// => 'btvjyxfe'
```

First, consider the amount of source entropy used in the code snippet above. Given each entropy source uses **IEEE 754** floats in generating random bits, let’s assume the process generates **53** bits of entropy (since that’s the number of significant bits in an **IEEE 754** double-precision float). So generating an **8** character ID above consumes `8*53 = 424` bits of source entropy.

Second, consider how much entropy was captured by the ID. Given there are **26** available characters, each character represents `log2(26) = 4.7` bits of entropy. So each generated ID represents `8*4.7 = 37.6` bits of entropy.

Hmmm. That means the ratio of ID entropy to source entropy is `37.6/424 = 0.09`, or a whopping **9%**. That’s not an efficiency most developers would be comfortable with. Granted, this is a particularly egregious example, but most random ID generation suffers such inefficient use of source entropy.

#### ID Characters

As previous noted, the entropy of a random string is equal to the entropy per character times the length of the string. Using this value leads to an easy calculation of _entropy representation efficiency_ (**ere**). We can define **ere** as the ratio of random string entropy to the number of bits required to represent the string. For example, the lower case alphabet has an entropy per character of **4.7**, so an ID of length **8** using those characters has **37.6** bits of entropy. Since each lower case character requires **1** byte, this leads to an ere of `37.6/64 = 0.59`, or **59%**. (Non-ASCII characters, of course, may occupy more than 1 byte, so the ere calculation would take that into account; but for the purposes of this discussion, we’ll stick with just ASCII).

The total entropy of a string is the product of the entropy per character times the string length, however, only if each character in the final string is equally probable. This is usually the case for random string generators, but there is a notable exception: the **version 4** string representation of a **UUID**. As defined in [RFC 4122, Section 4.4](https://www.rfc-editor.org/rfc/rfc4122#section-4.4), a **v4 UUID** uses a total of **32** hex and **4** hyphen characters. Although the hex characters can represent **4** bits of entropy each, **6** bits of the hex representation in a **UUID** are fixed, resulting in `32*4 – 6 = 122` bits of entropy (not **128**). The **4** fixed-position hyphen characters contribute zero entropy. So a **36** character **UUID** has an **ere** of `122/(36*8) = 0.40`, or **40%**. There are much more efficient ways to represent **122** bits of entropy.

### Overkill and Under Specify

![](https://github.com/puid/assets/blob/dev/pics/Overkill.png?raw=true)

#### Overkill

Random string generation is plagued by overkill and under specified usage. Consider the all too frequent use of **UUID**s as random strings. The rational is seemingly that the probability of a repeat using **UUID**s is low. Yes, it is admittedly low, but is that sufficient reason to use a **UUID** without further thought? For example, suppose **UUID**s are used as keys in a data store that will have at most a thousand items. What is the probability of a repeated database key in this case? It’s _1 in a nonillion_. That’s **10³⁰**, or 1 followed by 30 zeros, or about a million times the estimated number of stars in the universe. Really? Doesn’t that seem a bit overkill? Is that level of assurance really needed? And if so, why stop there? Why not concatenate two **UUID**s and get an even more ridiculous level of “assurance”?

Or why not be a bit more reasonable and think about the problem for a moment. Suppose you accept a _1 in 10¹⁵_ risk of repeat. That’s still a really low risk. Ah, but wait, to do that you can’t use a **UUID**, because **UUID** generation isn’t flexible. The characters are fixed, the representation is fixed, and the bits of entropy are fixed.

You could roll your sleeves up and generate the IDs by determining the actual amount of ID entropy required (it’s **68.76** bits), selecting some set of available characters (pick your favorite), calculate the string length necessary (which depends on the characters chosen), and finally generate the IDs as outlined in the earlier common ID generation scheme.

Whew, maybe that’s another reason developers tend to use **UUID**s. That seems like a lot of effort.

Ah, but there is another way. Well, for **Elixir**, **JavaScript**, **Swift** and **Python** anyway. The Probably Unique IDentifier (PUID) library makes generating context appropriate random IDs a snap. For example, to generate appropriate database IDs for the above specification (1000 IDs with a 1 in 10¹⁵ risk of repeat):

```elixir
# Elixir
iex> defmodule(DbId, do: use(Puid, total: 1_000, risk: 1.0e15))
iex> DbId.generate()
# => "sJmzTrFELLls"
```

```python
# Python
from puid import Puid
db_id = Puid(total=1000, risk=1e15)
db_id.generate()
# => 'tcDPzTAjoRcU'
```

```javascript
// JavaScript
const { puid } = require('puid-js')
const { generator: dbId } = puid({ total: 1000, risk: 1e15 })
dbId()
// => 'c1DVnnbI3RTr'
```

```swift
// Swift
let puid = Puid(total: 1_000, risk: 1e15)
try puid.generate()
// => "LR41qr8FcgVc"
```

The resulting IDs have **72** bits of entropy. But guess what? You don’t care. What you care is having explicitly stated you expect to have **1000** IDs and your level of repeat risk is _1 in a quadrillion_. It’s right there in the code. And as added bonus, the IDs are only **12** characters long, not **36**. Who doesn’t like ease, control and efficiency?

#### Under specify

Another head-scratcher in schemes that generate random strings is using an API that explicitly declares string length. Why is this troubling? Because that declaration doesn’t specify the actual amount of desired randomness, either needed or achieved. Suppose you are tasked with maintaining code that is using random IDs of **8** characters composed of lower alpha characters. Why are the IDs **8** characters long? Without code comments, you have no idea. And without knowing how many IDs are expected, you can’t determine the risk of a repeat, i.e., you can’t even make a statement about how random the random IDs actually are! Was **8** chosen for a reason, or just because it made the IDs look good?

Let’s investigate how random those IDs are by calculating the approximate repeat risk for an increasing total number of generated IDs:

Characters: **lower alpha**
Length:     **8**
Entropy:    **37.6**
Possible:   **208,827,064,576**

| Generated | Repeat Risk | Percent |
| --------: | ----------: | ------: |
|    10,000 |   0.000240  |   0.002 |
|    25,000 |   0.001500  |   0.015 |
|    50,000 |   0.005000  |   0.050 |
|   100,000 |   0.0237    |   2.37  |
|   250,000 |   0.1393    |  13.93  |
|   500,000 |   0.4512    |  45.12  |
|   537,400 |   0.5000    |  50.00  |
| 2,000,000 |   0.9999    |  99.99  |

And there’s the Birthday Paradox. Even though there are over **2** billion possible IDs, the chance of repeat is **50%** at a bit over half a million IDs, which is around **0.00026%** of the possible IDs! And at **2** million, or **1/1000** of all the possible IDs, your virtually guaranteed to have a repeat. Those numbers are just not evident in code that merely states a set of characters and a string length for the IDs.

The simple fact is, you don’t care how long a random string is, you care about the risk of repeat if you were to generate some total number of those strings for IDs. Look at the previous **PUID** code snippets. For a total of **1000** IDs, the risk of repeat is explicitly stated as **1/10¹⁵**. The total and risk are clearly specified. The fact the IDs are **12** characters long is, and should be, a by-product, not a specification.

![tl;dr](https://github.com/puid/assets/blob/dev/pics/tldr.png?raw=true)

### Random string

- A character representation of captured entropy

### Random IDs

- A bunch of random strings, used in a specific context, that are assumed to be unique

### Random ID generation

- Should provide full control of **entropy source**, **ID characters** and amount of **captured randomness**
- Should specify parameters for captured randomness (the goal), not string length (a by-product)

### UUIDs are

- Not UU: Universal isn’t needed and unique isn’t possible (since randomly generated)
- Not efficient: Requires string length of **36** characters when much more efficient options are available
- Rarely needed: Did you really need **122** bits of entropy?
- Inflexible: No control over **entropy source**, **ID characters** or amount of **captured randomness**
