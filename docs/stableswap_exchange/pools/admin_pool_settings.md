### Overview

The following are methods that may only be called by the pool admin (``owner``).

Additionally, some admin methods require a two-phase transaction process, whereby changes are committed in a first 
transaction and after a forced delay applied via a second transaction. The minimum delay after which a committed 
action can be applied is given by the constant pool attribute ``admin_actions_delay``, which is set to 3 days.

### Pool Ownership

!!! info "Initiate an ownership transfer of pool to ``_owner``"

    Callable only by the ownership admin. The ownership can not be transferred before ``transfer_ownership_deadline``, 
    which is the timestamp of the current block delayed by ``admin_actions_delay``.

    === "Vyper Code"
    
        ```vyper
        StableSwap.commit_transfer_ownership(_owner: address)
        ```

!!! info "Complete ownership transfer"

    Transfers ownership of the pool from current owner to the owner previously set via ``commit_transfer_ownership``.

    === "Vyper Code"
    
        ```vyper
        StableSwap.apply_transfer_ownership()
        ```
        
    !!! warning

        Pool ownership can only be transferred once.

!!! info "Revert ownership transfer"

    Reverts any previously committed transfer of ownership. This method resets the 
    ``transfer_ownership_deadline`` to ``0``.

    === "Vyper Code"
    
        ```vyper
        StableSwap.revert_transfer_ownership()
        ```

### Amplification Coefficient

The amplification coefficient ``A`` determines a pool’s tolerance for imbalance between the assets within it. 
A higher value means that trades will incur slippage sooner as the assets within the pool become imbalanced.

!!! note

    Within the pools, ``A`` is in fact implemented as ``1 / A`` and therefore a higher value implies that the pool will 
    be more tolerant to slippage when imbalanced.

The appropriate value for A is dependent upon the type of coin being used within the pool.

It is possible to modify the amplification coefficient for a pool after it has been deployed. However, it requires 
a vote within the Curve DAO and must reach a 15% quorum.

!!! info "Ramp amplification coefficient"

    Ramp ``A`` up or down by setting a new ``A`` to take effect at a future point in time.

    ``_future_A``: New future value of ``A``
    
    ``_future_time``: Timestamp at which new ``A`` should take effect

    === "Vyper Code"
    
        ```vyper
        StableSwap.ramp_A(_future_A: uint256, _future_time: uint256)
        ```

!!! info "Stop the ramping of amplification coefficient"

    Stop ramping ``A`` up or down and sets ``A`` to current ``A``.

    === "Vyper Code"
    
        ```vyper
        StableSwap.stop_ramp_A()
        ```

### Trade Fees

todo: hyperlink to fee collection and distribution
Curve pools charge fees on token swaps, where the fee may differ between pools. An admin fee is charged on the pool fee. 
For an overview of how fees are distributed, please refer to Fee Collection and Distribution.

!!! info "Commit new pool and admin fees for the pool"

    ``_new_fee``: New pool fee
    
    ``_new_admin_fee``: New admin fee (expressed as a percentage of the pool fee)

    The method commits new fee params: these fees do not take immediate effect.

    === "Vyper Code"
    
        ```vyper
        StableSwap.commit_new_fee(_new_fee: uint256, _new_admin_fee: uint256)
        ```

    !!! note

        Both the pool ``fee`` and the ``admin_fee`` are capped by the constants ``MAX_FEE`` and ``MAX_ADMIN_FEE``, 
        respectively. By default ``MAX_FEE`` is set at 50% and ``MAX_ADMIN_FEE`` at 100% (which is charged on the 
        ``MAX_FEE`` amount).


!!! info "Apply the previously committed new pool and admin fees for the pool"

    Apply the previously committed new pool and admin fees for the pool.

    === "Vyper Code"
    
        ```vyper
        StableSwap.apply_new_fee()
        ```
    
    !!! note

        Unlike ownership transfers, pool and admin fees may be set more than once.

!!! info "Reset any previously committed new fees"

    === "Vyper Code"
    
        ```vyper
        StableSwap.revert_new_parameters()
        ```

!!! info "Get the admin balance for a single coin in the pool"
    
    ``i``: Index of the coin to get admin balance for
    
    Returns the admin balance for coin ``i``.

    === "Vyper Code"
    
        ```vyper
        StableSwap.revert_new_parameters()
        ```

!!! info "Withdraw admin fees"
    
    Withdraws and transfers admin fees of the pool to the pool owner.

    === "Vyper Code"
    
        ```vyper
        StableSwap.withdraw_admin_fees()
        ```

!!! info "Donate all admin fees to the pool’s liquidity providers"

    === "Vyper Code"
    
        ```vyper
        StableSwap.donate_admin_fees()
        ```

    !!! note

        Older Curve pools do not implement this method.

### Kill a Pool

!!! info "Donate all admin fees to the pool’s liquidity providers"

    Pause a pool by setting the ``is_killed`` boolean flag to ``True``.
    
    This disables the following pool functionality: 
    - ``add_liquidity`` 
    - ``exchange`` 
    - ``remove_liquidity_imbalance`` 
    - ``remove_liquidity_one_coin``
    
    Hence, when paused, it is only possible for existing LPs to remove liquidity via ``remove_liquidity``.

    === "Vyper Code"
    
        ```vyper
        StableSwap.kill_me()
        ```

    !!! note

        Pools can only be killed within the first 30 days after deployment.

!!! info "Unpause a pool that was previously paused, re-enabling exchanges"

    === "Vyper Code"
    
        ```vyper
        StableSwap.unkill_me()
        ```