# Curve StableSwap: Pools

A Curve pool is a smart contract that implements the StableSwap invariant and thereby allows for the exchange of two or more tokens.

More broadly, Curve pools can be split into three categories:

1. ``Plain Pools``: A pool where two or more stablecoins are paired against each other.
2. ``Lending Pools``: A pool where two or more _wrapped_ tokens (e.g. ``cDAI``) are paired against one another, while the underlying is lent out on some other protocol.
3. ``Metapools``: A pool where a stablecoin is paired against the LP token from another pool.

Source code for Curve pools may be viewed on [GitHub](https://github.com/curvefi/curve-contract/tree/master/contracts).

!!! warning

    The API for plain, lending and metapools applies to all pools that are implemented based on [pool templates](https://github.com/curvefi/curve-contract/tree/master/contracts/pool-templates). When interacting with older Curve pools, there may be differences in terms of visibility, gas efficiency and/or variable naming. Furthermore, note that older contracts use ``vyper 0.1.x...`` and that the getters generated for public arrays changed between ``0.1.x`` and ``0.2.x`` to accept ``uint256`` instead of ``int128`` in order to handle the lookups.

    Please **do not** assume for a Curve pool to implement the API outlined in this section but verify this before interacting with a pool contract.

For information on code style please refer to the official [style guide](https://vyper.readthedocs.io/en/stable/style-guide.html).


# Plain Pools

The simplest Curve pool is a plain pool, which is an implementation of the StableSwap invariant for two or more tokens. The key characteristic of a plain pool is that the pool contract holds all deposited assets at **all** times.

An example of a Curve plain pool is [3Pool](https://github.com/curvefi/curve-contract/tree/master/contracts/pools/3pool), which contains the tokens ``DAI``, ``USDC`` and ``USDT``.

!!! note

    The API of plain pools is also implemented by lending and metapools.

The following Brownie console interaction examples are using [EURS](https://etherscan.io/address/0x0Ce6a5fF5217e38315f87032CF90686C96627CAA) Pool. The template source code for plain pools may be viewed on [GitHub](https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/base/SwapTemplateBase.vy).

### Getting Pool Info

Getter for the array of swappable coins within the pool.
=== "Vyper Code"

    ```vyper
    StableSwap.coins(i: uint256) → address: view
    ```
    
=== "Output"

    ```shell
    >>> pool.coin(0)
    '0xdB25f211AB05b1c97D595516F45794528a807ad8'
    ```

Getter for the pool balances array.
=== "Vyper Code"

    ```vyper
    StableSwap.balances(i: uint256) → uint256: view
    ```
    
=== "Output"

    ```shell
    >>> pool.balances(0)
    2918187395
    ```






