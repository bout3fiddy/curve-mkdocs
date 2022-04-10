### Overview

A metapool is a pool where a stablecoin is paired against the LP token from another pool, a so-called _base_ pool.

For example, a liquidity provider may deposit ``DAI`` into 
[3Pool](https://etherscan.io/address/0xbebc44782c7db0a1a60cb6fe97d0b483032ff1c7#code) and in exchange receive the 
pool’s LP token ``3CRV``. The ``3CRV`` LP token may then be deposited into the 
[GUSD metapool](https://etherscan.io/address/0x4f062658EaAF2C1ccf8C8e36D6824CDf41167956), which contains the 
coins ``GUSD`` and ``3CRV``, in exchange for the metapool’s LP token gusd3CRV. The obtained LP token may then be staked 
in the metapool’s liquidity gauge for ``CRV`` rewards.

Metapools provide an opportunity for the base pool liquidity providers to earn additional trading fees by depositing 
their LP tokens into the metapool. Note that the ``CRV`` rewards received for staking LP tokens into the pool’s liquidity 
gauge may differ for the base pool’s liquidity gauge and the metapool’s liquidity gauge. For details on liquidity 
gauges and protocol rewards, please refer to Liquidity Gauges and Minting CRV.

!!! note

    Metapools also implement the ABI from plain pools.

### Get Pool Info

!!! info "Get the coins of the base pool"

    === "Vyper Code"
    
        ```vyper
        StableSwap.base_coins(i: uint256) → address: view
        ```
        
    === "Output"
    
        ```shell
        >>> metapool.base_coins(0)
        '0x6B175474E89094C44Da98b954EedeAC495271d0F'
        >>> metapool.base_coins(1)
        '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48'
        >>> metapool.base_coins(2)
        '0xdAC17F958D2ee523a2206206994597C13D831ec7'
        ```

!!! info "Get the coins of the metapool"

    === "Vyper Code"
    
        ```vyper
        StableSwap.coins(i: uint256) → address: view
        ```
        
    === "Output"
    
        ```shell
        >>> metapool.coins(0)
        '0x056Fd409E1d7A124BD7017459dFEa2F387b6d5Cd'
        >>> metapool.coins(1)
        '0x6c3F90f043a72FA612cbac8115EE7e52BDe6E490'
        ```

        In this console example, ``coins(0)`` is the metapool’s coin (``GUSD``) and ``coins(1)`` is the LP token of 
        the base pool (``3CRV``).

!!! info "Get the current price of the base pool LP token relative to the underlying base pool assets"

    Note that the base pool’s virtual price is only fetched from the base pool if the cached price has expired. A 
    fetched based pool virtual price is cached for 10 minutes (``BASE_CACHE_EXPIRES: constant(int128) = 10 * 60``).


    === "Vyper Code"
    
        ```vyper
        StableSwap.coins(i: uint256) → address: view
        ```
        
    === "Output"
    
        ```shell
        >>> metapool.base_virtual_price()
        1014750545929625438
        ```

        In this console example, ``coins(0)`` is the metapool’s coin (``GUSD``) and ``coins(1)`` is the LP token of 
        the base pool (``3CRV``).

!!! info "Get the timestamp at which the base pool virtual price was last cached"

    === "Vyper Code"
    
        ```vyper
        StableSwap.base_cache_update() → uint256: view
        ```
        
    === "Output"
    
        ```shell
        >>> metapool.base_cache_updated()
        1616583340
        ```

### Exchange Two Coins

Similar to lending pools, on metapools exchanges can be made either between the coins the metapool actually holds 
(another pool’s LP token and some other coin) or between the metapool’s underlying coins. In the context of a metapool, 
**underlying** coins refers to the metapool’s coin and any of the base pool’s coins. The base pool’s LP token is **not** 
included as an underlying coin.

For example, the GUSD metapool would have the following:

Coins: ``GUSD``, ``3CRV`` (3Pool LP)

Underlying coins: ``GUSD``, ``DAI``, ``USDC``, ``USDT``


!!! note

    While metapools contain public getters for ``coins`` and ``base_coins``, there exists no getter for obtaining a list 
    of all underlying coins.

!!! info "Perform an exchange between two (non-underlying) coins in the metapool"

    Index values can be found via the ``coins`` public getter method.

    ``i``: Index value for the coin to send
    
    ``j``: Index value of the coin to receive
    
    ``_dx``: Amount of ``i`` being exchanged
    
    ``_min_dy``: Minimum amount of ``j`` to receive
    
    Returns the actual amount of coin ``j`` received.

    === "Vyper Code"
    
        ```vyper
        StableSwap.exchange(i: int128, j: int128, _dx: uint256, _min_dy: uint256) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> lending_pool.exchange()
        todo: console output
        ```

!!! info "Perform an exchange between two underlying tokens"

    Index values are the ``coins`` followed by the ``base_coins``, where the base pool LP token is **not** included as a 
    value.

    ``i``: Index value for the underlying coin to send
    
    ``j``: Index value of the underlying coin to receive
    
    ``_dx``: Amount of ``i`` being exchanged
    
    ``_min_dy``: Minimum amount of ``j`` to receive
    
    Returns the actual amount of coin ``j`` received.

    === "Vyper Code"
    
        ```vyper
        StableSwap.exchange_underlying(i: int128, j: int128, _dx: uint256, _min_dy: uint256) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> lending_pool.exchange_underlying()
        todo: console output
        ```

The template source code for metapools may be viewed on [GitHub](https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/meta/SwapTemplateMeta.vy).

