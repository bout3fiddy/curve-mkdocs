### Overview

Curve pools may contain lending functionality, whereby the underlying tokens are lent out on other protocols 
(e.g., Compound or Yearn). Hence, the main difference to a plain pool is that a lending pool does not hold 
the underlying token itself, but a **wrapped** representation of it.

Currently, Curve supports the following lending pools:

``aave``: [Aave pool](https://www.curve.fi/aave), with lending on [Aave](https://www.aave.com)

``busd``: [BUSD](https://www.curve.fi/busd) pool, with lending on [yearn.finance](https://www.yearn.finance)

``compound``: [Compound](https://www.curve.fi/compound) pool, with lending on [Compound](https://compound.finance/)

``ib``: [Iron Bank pool](https://curve.fi/ib), with lending on [Cream](https://v1.yearn.finance/lending)

``pax``: [PAX](https://curve.fi/pax) pool, with lending on [yearn.finance](https://www.yearn.finance)

``usdt``: [USDT pool](https://curve.fi/usdt), with lending on [Compound](https://www.curve.fi/compound)

``y``: [Y pool](https://curve.fi/y), with lending on [yearn.finance](https://www.yearn.finance)

An example of a Curve lending pool is 
[Compound Pool](https://github.com/curvefi/curve-contract/tree/master/contracts/pools/compound), 
which contains the wrapped tokens ``cDAI`` and ``cUSDC``, while the underlying tokens ``DAI`` and ``USDC`` are lent out 
on Compound. Liquidity providers of the Compound Pool therefore receive interest generated on Compound in addition to 
fees from token swaps in the pool.

Implementation of lending pools may differ with respect to how wrapped tokens accrue interest. There are two main types 
of wrapped tokens that are used by lending pools:

``cToken-style tokens``: These are tokens, such as interest-bearing cTokens on Compound (e.g., ``cDAI``) or on yTokens 
                         on Yearn, where interest accrues as the rate of the token increases.

``aToken-style tokens``: These are tokens, such as aTokens on AAVE (e.g., ``aDAI``), where interest accrues as the 
balance of the token increases.

The template source code for lending pools may be viewed on GitHub.

!!! Note

    Lending pools also implement the ABI from plain pools.

### Get Pool Info

!!! info "Getter for the array of underlying coins within the pool"

    === "Vyper Code"
    
        ```vyper
        StableSwap.underlying_coins(i: uint256) → address: view
        ```
        
    === "Output"
    
        ```shell
        >>> lending_pool.coins(0)
        '0x5d3a536E4D6DbD6114cc1Ead35777bAB948E3643'
        >>> lending_pool.coins(1)
        '0x39AA39c021dfbaE8faC545936693aC917d5E7563'
        ```

### Exchange Two Coins

Like plain pools, lending pools have the ``exchange`` method. However, in the case of lending pools, calling ``exchange`` 
performs a swap between two wrapped tokens in the pool.

For example, calling ``exchange`` on the Compound Pool, would result in a swap between the wrapped tokens ``cDAI`` and ``cUSDC``.

!!! info "Perform an exchange between two underlying tokens"

    Perform an exchange between two underlying tokens. Index values can be found via the ``underlying_coins`` public 
    getter method.

    ``i``: Index value for the underlying coin to send
    
    ``j``: Index value of the underlying coin to receive
    
    ``_dx``: Amount of ``i`` being exchanged
    
    ``_min_dy``: Minimum amount of ``j`` to receive
    
    Returns the actual amount of coin ``j`` received.

    === "Vyper Code"
    
        ```vyper
        StableSwap.exchange_underlying(i: int128, j: int128, dx: uint256, min_dy: uint256) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> lending_pool.exchange_underlying()
        todo: console output
        ```

    !!! note

        Older Curve lending pools may not implement the same signature for ``exchange_underlying``. For instance, Compound 
        pool does not return anything for ``exchange_underlying`` and therefore costs more in terms of gas.

### Add/Remove Liquidity

The function signatures for adding and removing liquidity to a lending pool are mostly the same as for a plain pool. 
However, for lending pools, liquidity is added and removed in the wrapped token, not the underlying.

In order to be able to add and remove liquidity in the underlying token (e.g., remove DAI from Compound Pool instead of 
``cDAI``) there exists a ``Deposit<POOL>.vy`` contract (e.g., ([DepositCompound.vy](https://github.com/curvefi/curve-contract/blob/master/contracts/pools/compound/DepositCompound.vy)).

!!! warning

    Older Curve lending pools (e.g., Compound Pool) do not implement all plain pool methods for adding and removing 
    liquidity. For instance, ``remove_liquidity_one_coin`` is not implemented by Compound Pool).

Some newer pools (e.g., [IB](https://github.com/curvefi/curve-contract/blob/master/contracts/pools/ib/StableSwapIB.vy)) 
have a modified signature for ``add_liquidity`` and allow the caller to specify whether the deposited liquidity is in 
the wrapped or underlying token.

!!! Deposit coins into the pool

    Perform an exchange between two underlying tokens. Index values can be found via the ``underlying_coins`` public 
    getter method.

    ``_amounts``: List of amounts of coins to deposit

    ``_min_mint_amount``: Minimum amount of LP tokens to mint from the deposit
    
    ``_use_underlying`` If ``True``, deposit underlying assets instead of wrapped assets.
    
    Returns amount of LP tokens received in exchange for the deposited tokens.

    === "Vyper Code"
    
        ```vyper
        StableSwap.add_liquidity(_amounts: uint256[N_COINS], _min_mint_amount: uint256, _use_underlying: bool = False) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> lending_pool.add_liquidity()
        todo: console output
        ```