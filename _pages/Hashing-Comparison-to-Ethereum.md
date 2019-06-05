* Thinking

  For blockchain attacks like common 51% attack, it depend on hashing power, more likely to say hashes generate in an amount of time. To consider cost of an attack, the factors may look like following:
`f(cost to generate a hash, probability a hash be accepted)`.

  The first and second term is somehow independent, cost to generate a hash has no impact to hash accepted probability, however, if hashes generated too fast or too slow, then next time probability might be decreased or increased. When viewing the moment of generating a hash, these two terms can be viewed as independent to each other.

  A hash outcome cannot be predicted, it is only known after hash is calculated, `difficulty` is an index to adjust how likely a hash can be accepted, which is the second term in function.

  The first term is about how much money it takes to generate a hash, it's a more complex situation. Once a machine and algorithm is decided, the time to generate a hash is probably within a small a range. If the machine is running on AWS or other cloud platform, then cost of generating each hash can be computed.

* Manual Hashing Comparison

  - Bitmark Hashing

    [reference code for hash generation](https://gist.github.com/jamieabc/e1e973a8b330e4f062e5f3f31939c7b7)

    | EC2 Type    | Hardware Spec          | Machine Cost (USD/hour) | Time of 1000 H(s) | Hash Cost (USD) |
    |-------------|------------------------|-------------------------|-------------------|-----------------|
    | t2.xlarge   | 8 cores 2.3G, 16G mem  |                0.243    |               170 |           0.011 |
    | c5d.4xlarge | 16 cores 3.0G, 32G mem |                0.976    |                60 |           0.016 |

    Currently Bitmarkd hashing difficulty is 1, which means possibly a hash accepted is

    ```
      2^248 / 2^256 = 2^-8
    ```

    Statistically speaking, every 256 hashes generated will contain 1 valid hash.

    Now recorderd runs with 120 H/s, which means that another 120 H/s is needed for 51% attack.

    From data above, 1000 hashes in 60 seconds stands for 16 H/s, in other words, rent 10 c5d.4xlage
    instances running hashing parallelly provides 160 H/s, enough for 51% attack.

    The cost for each c5d.4xlarge is 0.976 USD/hour, rent 10 instances cost about 10 USD/hour.

    In cap, when current hashing power is X H/s, the cost for 51% attack is roughly `Max(X/16, 1)` USD/hour.

  - Ethereum

    Refer from [this page](https://f-a.nz/gist/ethereum-gpu-mining-on-aws-ec2-in-2017/)

    AWS EC2 instance type: `p3.16xlarge`

    Date: 2018/1/31

    GPU: 8x Nvidia Tesla V100 (5,120 CUDA cores)

    Monthly cost: $ 17,625

    Monthly revenue: $ 2,009

    Hash Rate: 724.64 MH / second

  - Summary

    |                           | Bitmark | Ethereum |
    |---------------------------|---------|----------|
    | cost of 1000 hashes (USD) |    4E-3 |   9.0E-9 |

* Market Comparison

  Use this [website](https://www.nicehash.com) as open market price benchmark.

  Since there's no hashing algorithm as argon2, use Monero's memory intensive algorithm as calculation. The website uses BTC as a payment, and price changes everyday, below table is a summarization on 2/15/2019.


   |          | Price (BTC) | Hash (per day) | Cost / Hash |
   |----------|-------------|----------------|-------------|
   | Monero   |       0.049 | 0.2808 MH      | 1.7 E-7     |
   | Ethereum |        3.18 | 0.0737 TH      | 4.31 E-11   |

  hashing cost of Monero vs Ethereum = 3944 times.