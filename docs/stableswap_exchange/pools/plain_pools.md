## Overview

The simplest Curve pool is a plain pool, which is an implementation of the StableSwap invariant for two or more tokens. 
The key characteristic of a plain pool is that the pool contract holds all deposited assets at **all** times.

An example of a Curve plain pool is [3Pool](https://github.com/curvefi/curve-contract/tree/master/contracts/pools/3pool), 
which contains the tokens ``DAI``, ``USDC`` and ``USDT``.

!!! note

    The API of plain pools is also implemented by lending and metapools.

The following Brownie console interaction examples are using 
[EURS](https://etherscan.io/address/0x0Ce6a5fF5217e38315f87032CF90686C96627CAA) Pool. The template source code for plain
pools may be viewed on [GitHub](https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/base/SwapTemplateBase.vy).

### Get Pool Info        

!!! info "Getter for the array of swappable coins within the pool"

    === "Vyper Code"
    
        ```vyper
        StableSwap.coins(i: uint256) → address: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.coin(0)
        '0xdB25f211AB05b1c97D595516F45794528a807ad8'
        ```

!!! info "Getter for the pool balances array"

    === "Vyper Code"
    
        ```vyper
        StableSwap.balances(i: uint256) → uint256: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.balances(0)
        2918187395
        ```

!!! info "Getter for the admin/owner of the pool contract"
    
    === "Vyper Code"
    
        ```vyper
        StableSwap.owner() → address: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.owner()
        '0xeCb456EA5365865EbAb8a2661B0c503410e9B347'
        ```

todo: add chapter hyperlink to LP token
!!! info "Getter for the LP token of the pool"

    === "Vyper Code"
    
        ```vyper
        StableSwap.lp_token() → address: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.lp_token()
        '0x194eBd173F6cDacE046C53eACcE9B953F28411d1'
        ```

    !!! note
    
        In older Curve pools ``lp_token`` may not be ``public`` and thus not visible.

todo: add hyperlink to amplification coefficient description
!!! info "Getter for the amplification coefficient of the pool"

    === "Vyper Code"
    
        ```vyper
        StableSwap.A() → uint256: view 
        ```
        
    === "Output"
    
        ```shell
        >>> pool.A()
        100
        ```

    !!! note
        
        The amplification coefficient is scaled by ``A_PRECISION`` (``=100``)

!!! info "Getter for the unscaled amplification coefficient of the pool"

    === "Vyper Code"
    
        ```vyper
        StableSwap.A_precise() → uint256: view 
        ```
        
    === "Output"
    
        ```shell
        >>> pool.A()
        10000
        ```

!!! info "Current virtual price of the pool LP token relative to the underlying pool assets"

    === "Vyper Code"
    
        ```vyper
        StableSwap.get_virtual_price() → uint256: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.get_virtual_price()
        1001692838188850782
        ```
    
    !!! note

        - The method returns ``virtual_price`` as an integer with ``1e18`` precision.
        - ``virtual_price`` returns a price relative to the underlying. You can get the absolute price
        by multiplying it with the price of the underlying assets.

!!! info "The pool swap fee"

    === "Vyper Code"
    
        ```vyper
        StableSwap.fee() → uint256: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.fee()
        4000000
        ```
    
    !!! note

        The method returns ``fee`` as an integer with ``1e10`` precision.

!!! info "The percentage of the swap fee that is taken as an admin fee"

    === "Vyper Code"
    
        ```vyper
        StableSwap.admin_Fee() → uint256: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.admin_fee()
        5000000000
        ```

    !!! note

        - The method returns an integer with with ``1e10`` precision.
        - Admin fee is set at 50% (``5000000000``) and is paid out to veCRV holders.


### Exchange Two Coins

!!! info "Get the amount of coin ``j`` one would receive for swapping ``_dx`` of coin ``i``"

    === "Vyper Code"
    
        ```vyper
        StableSwap.get_dy(i: int128, j: int128, _dx: uint256) → uint256: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.get_dy(0, 1, 100)
        996307731416690125
        ```

    !!! note

        Note: In this example,  the ``EURS Pool`` coins decimals for ``coins(0)`` and ``coins(1)`` are 
        ``2`` and ``18``, respectively.

!!! info "Perform an exchange between two coins"

    ``i``: Index value for the coin to send
    
    ``j``: Index value of the coin to receive
    
    ``_dx``: Amount of ``i`` being exchanged
    
    ``_min_dy``: Minimum amount of ``j`` to receive

    Returns the actual amount of coin ``j`` received. Index values can be found via the 
    ``coins`` public getter method.

    === "Vyper Code"
    
        ```vyper
        StableSwap.exchange(i: int128, j: int128, _dx: uint256, _min_dy: uint256) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> expected = pool.get_dy(0, 1, 10**2) * 0.99
        >>> pool.exchange(0, 1, 10**2, expected, {"from": alice})
        ```

### Add/Remove Liquidity

!!! info "Calculate addition or reduction in token supply from a deposit or withdrawal"

    ``_amounts``: Amount of each coin being deposited

    ``_is_deposit``: Set True for deposits, False for withdrawals
    
    Returns the expected amount of LP tokens received. This calculation accounts for slippage, but not fees.

    === "Vyper Code"
    
        ```vyper
        StableSwap.calc_token_amount(_amounts: uint256[N_COINS], _is_deposit: bool) → uint256: view
        ```
        
    === "Output"
    
        ```shell
        >>> pool.calc_token_amount([10**2, 10**18], True)
        1996887509167925969
        ```

!!! info "Deposit coins into the pool"

    ``_amounts``: List of amounts of coins to deposit

    ``_min_mint_amount``: Minimum amount of LP tokens to mint from the deposit
    
    Returns the amount of LP tokens received in exchange for the deposited tokens.

    === "Vyper Code"
    
        ```vyper
        StableSwap.add_liquidity(_amounts: uint256[N_COINS], _min_mint_amount: uint256) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> todo: add_liquidity console output example
        ```

!!! info "Withdraw coins from the pool"

    ``_amounts``: Quantity of LP tokens to burn in the withdrawal

    ``_min_mint_amount``: Minimum amounts of underlying coins to receive
    
    Returns a list of the amounts for each coin that was withdrawn.

    === "Vyper Code"
    
        ```vyper
        StableSwap.remove_liquidity(_amount: uint256, _min_amounts: uint256[N_COINS]) → uint256[N_COINS]
        ```
        
    === "Output"
    
        ```shell
        >>> todo: remove_liquidity console output example
        ```

!!! info "Withdraw coins from the pool in an imbalanced amount"

    ``_amounts``: List of amounts of underlying coins to withdraw

    ``_min_mint_amount``: Maximum amount of LP token to burn in the withdrawal
    
    Returns a list of the amounts for each coin that was withdrawn.

    === "Vyper Code"
    
        ```vyper
        StableSwap.remove_liquidity_imbalance(_amounts: uint256[N_COINS], _max_burn_amount: uint256) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> todo: remove_liquidity_imbalance console output example
        ```

!!! info "Calculate the amount received when withdrawing a single coin"

    ``_token_amount``: Amount of LP tokens to burn in the withdrawal

    ``i``: Index value of the coin to withdraw

    === "Vyper Code"
    
        ```vyper
        StableSwap.calc_withdraw_one_coin(_token_amount: uint256, i: int128) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> todo: calculate_withdraw_one_coin console output example
        ```

!!! info "Withdraw a single coin from the pool"

    ``_token_amount``: Amount of LP tokens to burn in the withdrawal

    ``i``: Index value of the coin to withdraw

    ``_min_amount``: Minimum amount of coin to receive

    Returns the amount of coin ``i`` received.

    === "Vyper Code"
    
        ```vyper
        StableSwap.remove_liquidity_one_coin(_token_amount: uint256, i: int128, _min_amount: uint256) → uint256
        ```
        
    === "Output"
    
        ```shell
        >>> todo: remove_liquidity_one_coin console output example
        ```