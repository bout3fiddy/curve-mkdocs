# Overview

The Curve DAO Token can be locked to aquire veCRV (voting-escorwed CRV Tokens) voting power.  
1 CRV locked for 4 years = 1.00 veCRV  
1 CRV locked for 3 years = 0.75 veCRV  
1 CRV locked for 2 years = 0.50 veCRV  
1 CRV locked for 1 years = 0.25 veCRV  

!!! note
    Voting Escrow Address: 0x5f3b5DfEb7B28CDbD7FAba78963EE202a494e2A2


The following Brownie console interaction examples are using the 
[CRV](https://etherscan.io/token/0x5f3b5DfEb7B28CDbD7FAba78963EE202a494e2A2) Token Contract.

# Voting Escrow Info Methods

### `vecrv.locked_end`

!!! description "`vecrv.locked_end(_addr: address) -> uint256:`"

    Returns a epoch (`uint256`) when '_addr's lock finishes

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_addr`       |  `address` | Timestamp |

    ??? quote "Source code"

        ```python

        locked: public(HashMap[address, LockedBalance])
        epoch: public(uint256)

        @external
        @view
        def locked__end(_addr: address) -> uint256:
            """
            @notice Get timestamp when `_addr`'s lock finishes
            @param _addr User wallet
            @return Epoch time of the lock end
            """
            return self.locked[_addr].end
        ```

    === "Example"
        
        ```shell
        >>> vecrv.locked_end(0x7a16fF8270133F063aAb6C9977183D9e72835428)
        '1808956800'
        ```

### `vecrv.balanceOf`

!!! description "`vecrv.balanceOf(addr: address, _t: uint256 = block.timestamp) -> uint256:`"

    Returns the voting power (`uint256`) of an address at a certain timestamp.

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_addr`       |  `address` | Address |
    | `_t`       |  `uint256` | Timestamp |

    ??? quote "Source code"

        ```python

        @external
        @view
        def balanceOf(addr: address, _t: uint256 = block.timestamp) -> uint256:
            """
            @notice Get the current voting power for `msg.sender`
            @dev Adheres to the ERC20 `balanceOf` interface for Aragon compatibility
            @param addr User wallet address
            @param _t Epoch time to return voting power at
            @return User voting power
            """
            _epoch: uint256 = self.user_point_epoch[addr]
            if _epoch == 0:
                return 0
            else:
                last_point: Point = self.user_point_history[addr][_epoch]
                last_point.bias -= last_point.slope * convert(_t - last_point.ts, int128)
                if last_point.bias < 0:
                    last_point.bias = 0
                return convert(last_point.bias, uint256)

        ```

    === "Example"
        
        ```shell
        >>> vecrv.balanceOf(0x7a16fF8270133F063aAb6C9977183D9e72835428, 1683742945)
        '26990828983175164776061315'
        ```


### `vecrv.balanceOfAt`

!!! description "`vecrv.balanceOf(addr: address, _t: uint256 = block.timestamp) -> uint256:`"

    Returns the voting power (`uint256`) of an address at a certain block.

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_addr`    |  `address` | Address |
    | `_t`       |  `uint256` | Block |

    ??? quote "Source code"

        ```python

        @external
        @view
        def balanceOfAt(addr: address, _block: uint256) -> uint256:
            """
            @notice Measure voting power of `addr` at block height `_block`
            @dev Adheres to MiniMe `balanceOfAt` interface: https://github.com/Giveth/minime
            @param addr User's wallet address
            @param _block Block to calculate the voting power at
            @return Voting power
            """
            # Copying and pasting totalSupply code because Vyper cannot pass by
            # reference yet
            assert _block <= block.number

            # Binary search
            _min: uint256 = 0
            _max: uint256 = self.user_point_epoch[addr]
            for i in range(128):  # Will be always enough for 128-bit numbers
                if _min >= _max:
                    break
                _mid: uint256 = (_min + _max + 1) / 2
                if self.user_point_history[addr][_mid].blk <= _block:
                    _min = _mid
                else:
                    _max = _mid - 1

            upoint: Point = self.user_point_history[addr][_min]

            max_epoch: uint256 = self.epoch
            _epoch: uint256 = self.find_block_epoch(_block, max_epoch)
            point_0: Point = self.point_history[_epoch]
            d_block: uint256 = 0
            d_t: uint256 = 0
            if _epoch < max_epoch:
                point_1: Point = self.point_history[_epoch + 1]
                d_block = point_1.blk - point_0.blk
                d_t = point_1.ts - point_0.ts
            else:
                d_block = block.number - point_0.blk
                d_t = block.timestamp - point_0.ts
            block_time: uint256 = point_0.ts
            if d_block != 0:
                block_time += d_t * (_block - point_0.blk) / d_block

            upoint.bias -= upoint.slope * convert(block_time - upoint.ts, int128)
            if upoint.bias >= 0:
                return convert(upoint.bias, uint256)
            else:
                return 0
        ```

    === "Example"
        
        ```shell
        >>> vecrv.balanceOfAt(todo)
        'todo'
        ```

### `vecrv.totalSupply`

!!! description "`vecrv.totalSupply(t: uint256 = block.timestamp) -> uint256:`"

    Returns the total supply (`uint256`) of vecrv (= total voting power) at a certain timestamp.  
    Can also use `vecrv.totalSupply` with no arguments --> fills in current timestamp.

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `t`       |  `uint256` | Timestamp |

    ??? quote "Source code"

        ```python hl_lines="5"

        @external
        @view
        def totalSupply(t: uint256 = block.timestamp) -> uint256:
            """
            @notice Calculate total voting power
            @dev Adheres to the ERC20 `totalSupply` interface for Aragon compatibility
            @return Total voting power
            """
            _epoch: uint256 = self.epoch
            last_point: Point = self.point_history[_epoch]
            return self.supply_at(last_point, t)

        ```

    === "Example"
        
        ```shell
        >>> vecrv.totalSupply(1683742945)
        586046376093059723137909505
        ```


### `vecrv.totalSupplyAt`

!!! description "`vecrv.totalSupplyAt(_block: uint256) -> uint256:`"

    Returns the total supply (`uint256`) of vecrv (= total voting power) at a certain block.  

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_block`       |  `uint256` | Block |

    ??? quote "Source code"

        ```python

        @external
        @view
        def totalSupplyAt(_block: uint256) -> uint256:
            """
            @notice Calculate total voting power at some point in the past
            @param _block Block to calculate the total voting power at
            @return Total voting power at `_block`
            """
            assert _block <= block.number
            _epoch: uint256 = self.epoch
            target_epoch: uint256 = self.find_block_epoch(_block, _epoch)

            point: Point = self.point_history[target_epoch]
            dt: uint256 = 0
            if target_epoch < _epoch:
                point_next: Point = self.point_history[target_epoch + 1]
                if point.blk != point_next.blk:
                    dt = (_block - point.blk) * (point_next.ts - point.ts) / (point_next.blk - point.blk)
            else:
                if point.blk != block.number:
                    dt = (_block - point.blk) * (block.timestamp - point.ts) / (block.number - point.blk)
            # Now dt contains info on how far are we beyond point

            return self.supply_at(point, point.ts + dt)

        ```

    === "Example"
        
        ```shell
        >>> vecrv.totalSupply(1683742945)
        586046376093059723137909505
        ```



### `vecrv.token`

!!! description "`vecrv.token() -> address: view `"

    Voting-Escrow Token address  


    ??? quote "Source code"

        ```python hl_lines="1 4 11 17"
        token: public(address)

        @external
        def __init__(token_addr: address, 
                    _name: String[64], 
                    _symbol: String[32], 
                    _version: String[32]
                ):
            """
            @notice Contract constructor
            @param token_addr `ERC20CRV` token address
            @param _name Token name
            @param _symbol Token symbol
            @param _version Contract version - required for Aragon compatibility
            """
            self.admin = msg.sender
            self.token = token_addr
            self.point_history[0].blk = block.number
            self.point_history[0].ts = block.timestamp
            self.controller = msg.sender
            self.transfersEnabled = True

            _decimals: uint256 = ERC20(token_addr).decimals()
            assert _decimals <= 255
            self.decimals = _decimals

            self.name = _name
            self.symbol = _symbol
            self.version = _version
        ```

    === "Example"
        
        ```shell
        >>> vecrv.token()
        0xD533a949740bb3306d119CC777fa900bA034cd52
        ```

### `vecrv.name`

!!! description "`vecrv.name() -> String[64]: view `"

    voting-escrow Token name  


    ??? quote "Source code"

        ```python hl_lines="1 5 12 27"
        name: public(String[64])

        @external
        def __init__(token_addr: address, 
                    _name: String[64], 
                    _symbol: String[32], 
                    _version: String[32]
                ):
            """
            @notice Contract constructor
            @param token_addr `ERC20CRV` token address
            @param _name Token name
            @param _symbol Token symbol
            @param _version Contract version - required for Aragon compatibility
            """
            self.admin = msg.sender
            self.token = token_addr
            self.point_history[0].blk = block.number
            self.point_history[0].ts = block.timestamp
            self.controller = msg.sender
            self.transfersEnabled = True

            _decimals: uint256 = ERC20(token_addr).decimals()
            assert _decimals <= 255
            self.decimals = _decimals

            self.name = _name
            self.symbol = _symbol
            self.version = _version
        ```

    === "Example"
        
        ```shell
        >>> vecrv.name()
        Vote-escrowed CRV
        ```

### `vecrv.symbol`

!!! description "`vecrv.symbol() -> String[32]: view `"

    Voting-Escrow Token symbol


    ??? quote "Source code"

        ```python hl_lines="1 6 13 28"
        symbol: public(String[32])

        @external
        def __init__(token_addr: address, 
                    _name: String[64], 
                    _symbol: String[32], 
                    _version: String[32]
                ):
            """
            @notice Contract constructor
            @param token_addr `ERC20CRV` token address
            @param _name Token name
            @param _symbol Token symbol
            @param _version Contract version - required for Aragon compatibility
            """
            self.admin = msg.sender
            self.token = token_addr
            self.point_history[0].blk = block.number
            self.point_history[0].ts = block.timestamp
            self.controller = msg.sender
            self.transfersEnabled = True

            _decimals: uint256 = ERC20(token_addr).decimals()
            assert _decimals <= 255
            self.decimals = _decimals

            self.name = _name
            self.symbol = _symbol
            self.version = _version
        ```

    === "Example"
        
        ```shell
        >>> vecrv.symbol()
        veCRV
        ```

### `vecrv.version`

!!! description "`vecrv.version() -> String[32]: view`"

    enabels transfership of vecrv token. is needed for compatibility with Aragon's view methods.

    ??? quote "Source code"

        ```python hl_lines="1 7 29"
        version: public(String[32])

        @external
        def __init__(token_addr: address, 
                    _name: String[64], 
                    _symbol: String[32], 
                    _version: String[32]
                ):
            """
            @notice Contract constructor
            @param token_addr `ERC20CRV` token address
            @param _name Token name
            @param _symbol Token symbol
            @param _version Contract version - required for Aragon compatibility
            """
            self.admin = msg.sender
            self.token = token_addr
            self.point_history[0].blk = block.number
            self.point_history[0].ts = block.timestamp
            self.controller = msg.sender
            self.transfersEnabled = True

            _decimals: uint256 = ERC20(token_addr).decimals()
            assert _decimals <= 255
            self.decimals = _decimals

            self.name = _name
            self.symbol = _symbol
            self.version = _version

        ```

    === "Example"
        
        ```shell
        >>> vecrv.version()
        veCRV_1.0.0
        ```


### `vecrv.transfersEnabled`

!!! description "`vecrv.transfersEnabled() -> boolean: view`"

    enabels transfership of vecrv token. is needed for compatibility with Aragon's view methods.

    ??? quote "Source code"

        ```python hl_lines="1 3 21"
        # Aragon's view methods for compatibility
        controller: public(address)
        transfersEnabled: public(bool)

        @external
        def __init__(token_addr: address, 
                    _name: String[64], 
                    _symbol: String[32], 
                    _version: String[32]
                ):
            """
            @notice Contract constructor
            @param token_addr `ERC20CRV` token address
            @param _name Token name
            @param _symbol Token symbol
            @param _version Contract version - required for Aragon compatibility
            """
            self.admin = msg.sender
            self.token = token_addr
            self.point_history[0].blk = block.number
            self.point_history[0].ts = block.timestamp
            self.controller = msg.sender
            self.transfersEnabled = True

            _decimals: uint256 = ERC20(token_addr).decimals()
            assert _decimals <= 255
            self.decimals = _decimals

            self.name = _name
            self.symbol = _symbol
            self.version = _version

        ```

    === "Example"
        
        ```shell
        >>> vecrv.transfersEnabeled()
        True
        ```

    !!! note
        `transfersEnabled` is set to `true` and can't be changed!



### `vecrv.decimals`

!!! description "`vecrv.decimals() -> uint256: view`"

    Returns the total supply (`uint256`) of vecrv (= total voting power) at a certain block.  


    ??? quote "Source code"

        ```python hl_lines="1 15 23 24 25"
        decimals: public(uint256)

        @external
        def __init__(token_addr: address, 
                    _name: String[64], 
                    _symbol: String[32], 
                    _version: String[32]
                ):
            """
            @notice Contract constructor
            @param token_addr `ERC20CRV` token address
            @param _name Token name
            @param _symbol Token symbol
            @param _version Contract version - required for Aragon compatibility
            """
            self.admin = msg.sender
            self.token = token_addr
            self.point_history[0].blk = block.number
            self.point_history[0].ts = block.timestamp
            self.controller = msg.sender
            self.transfersEnabled = True

            _decimals: uint256 = ERC20(token_addr).decimals()
            assert _decimals <= 255
            self.decimals = _decimals

            self.name = _name
            self.symbol = _symbol
            self.version = _version

        ```

    === "Example"
        
        ```shell
        >>> vecrv.decimals()
        18
        ```


### `vecrv.admin`

!!! description "`vecrv.admin() -> address: view`"

    Returns todo


    ??? quote "Source code"

        ```python hl_lines="1 17"
        admin: public(address)  # Can and will be a smart contract
        future_admin: public(address)

        @external
        def __init__(token_addr: address, 
                    _name: String[64], 
                    _symbol: String[32], 
                    _version: String[32]
                ):
            """
            @notice Contract constructor
            @param token_addr `ERC20CRV` token address
            @param _name Token name
            @param _symbol Token symbol
            @param _version Contract version - required for Aragon compatibility
            """
            self.admin = msg.sender
            self.token = token_addr
            self.point_history[0].blk = block.number
            self.point_history[0].ts = block.timestamp
            self.controller = msg.sender
            self.transfersEnabled = True

            _decimals: uint256 = ERC20(token_addr).decimals()
            assert _decimals <= 255
            self.decimals = _decimals

            self.name = _name
            self.symbol = _symbol
            self.version = _version
        

        ```

    === "Example"
        
        ```shell
        >>> vecrv.admin()
        0x40907540d8a6C65c637785e8f8B742ae6b0b9968
        ```


### `vecrv.future_admin`

!!! description "`vecrv.future_admin() -> address: view`"

    Returns todo  


    ??? quote "Source code"

        ```python hl_lines="2 40"
        admin: public(address)  # Can and will be a smart contract
        future_admin: public(address)

        @external
        def __init__(token_addr: address, 
                    _name: String[64], 
                    _symbol: String[32], 
                    _version: String[32]
                ):
            """
            @notice Contract constructor
            @param token_addr `ERC20CRV` token address
            @param _name Token name
            @param _symbol Token symbol
            @param _version Contract version - required for Aragon compatibility
            """
            self.admin = msg.sender
            self.token = token_addr
            self.point_history[0].blk = block.number
            self.point_history[0].ts = block.timestamp
            self.controller = msg.sender
            self.transfersEnabled = True

            _decimals: uint256 = ERC20(token_addr).decimals()
            assert _decimals <= 255
            self.decimals = _decimals

            self.name = _name
            self.symbol = _symbol
            self.version = _version
        

        @external
        def commit_transfer_ownership(addr: address):
            """
            @notice Transfer ownership of VotingEscrow contract to `addr`
            @param addr Address to have ownership transferred to
            """
            assert msg.sender == self.admin  # dev: admin only
            self.future_admin = addr
            log CommitOwnership(addr)   

        ```

    === "Example"
        
        ```shell
        >>> vecrv.future_admin()
        0x40907540d8a6C65c637785e8f8B742ae6b0b9968
        ```




# HOW TO CALL THIS????

### `vecrv.commit_transfer_ownership`
!!! description "`vecrv.commit_transfer_ownership(addr: address):`"

    todo

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `addr`       |  `address` | Address |

    ??? quote "Source code"

        ```python hl_lines="1 8 15"
        event CommitOwnership:
            admin: address

        admin: public(address)  # Can and will be a smart contract
        future_admin: public(address)

        @external
        def commit_transfer_ownership(addr: address):
            """
            @notice Transfer ownership of VotingEscrow contract to `addr`
            @param addr Address to have ownership transferred to
            """
            assert msg.sender == self.admin  # dev: admin only
            self.future_admin = addr
            log CommitOwnership(addr)   

        ```

    === "Example"
        
        ```shell
        >>> vecrv.commit_transfer_ownership(todo):
        todo
        ```

    !!! note
        This function can only be called by the admin of the contract.


### `vecrv.apply_transfer_ownership`
!!! description "`vecrv.apply_transfer_ownership():`"

    todo

    ??? quote "Source code"

        ```python hl_lines="1 8 10 16"
        event ApplyOwnership:
            admin: address

        admin: public(address)  # Can and will be a smart contract
        future_admin: public(address)

        @external
        def apply_transfer_ownership():
            """
            @notice Apply ownership transfer
            """
            assert msg.sender == self.admin  # dev: admin only
            _admin: address = self.future_admin
            assert _admin != ZERO_ADDRESS  # dev: admin not set
            self.admin = _admin
            log ApplyOwnership(_admin)  

        ```

    === "Example"
        
        ```shell
        >>> vecrv.apply_transfer_ownership():
        
        ```


### `vecrv.deposit_for`
!!! description "`vecrv.deposit_for(_addr: address, _value: uint256):`"

    todo! anyone can lock for another wallet?? i remember people said this is a bug and is fixed somehow. but how? 

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_addr`       |  `address` | todo |
    | `_value_` |  `uint256` | todo |

    ??? quote "Source code"

        ```python hl_lines="0"
        
        @external
        @nonreentrant('lock')
        def deposit_for(_addr: address, _value: uint256):
            """
            @notice Deposit `_value` tokens for `_addr` and add to the lock
            @dev Anyone (even a smart contract) can deposit for someone else, but
                cannot extend their locktime and deposit for a brand new user
            @param _addr User's wallet address
            @param _value Amount to add to user's lock
            """
            _locked: LockedBalance = self.locked[_addr]

            assert _value > 0  # dev: need non-zero value
            assert _locked.amount > 0, "No existing lock found"
            assert _locked.end > block.timestamp, "Cannot add to expired lock. Withdraw"

            self._deposit_for(_addr, _value, 0, self.locked[_addr], DEPOSIT_FOR_TYPE)

        ```

    === "Example"
        
        ```shell
        >>> vecrv.apply_transfer_ownership():
        
        ```


### `vecrv.create_lock`
!!! description "`vecrv.create_lock(_value: uint256, _unlock_time: uint256):`"

    todo

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_value`       |  `uint256` | Amount to deposit |
    | `_unlock_time` |  `uint256` | Epoch when tokens unlock |

    ??? quote "Source code"

        ```python hl_lines="0"

        @external
        @nonreentrant('lock')
        def create_lock(_value: uint256, _unlock_time: uint256):
            """
            @notice Deposit `_value` tokens for `msg.sender` and lock until `_unlock_time`
            @param _value Amount to deposit
            @param _unlock_time Epoch time when tokens unlock, rounded down to whole weeks
            """
            self.assert_not_contract(msg.sender)
            unlock_time: uint256 = (_unlock_time / WEEK) * WEEK  # Locktime is rounded down to weeks
            _locked: LockedBalance = self.locked[msg.sender]

            assert _value > 0  # dev: need non-zero value
            assert _locked.amount == 0, "Withdraw old tokens first"
            assert unlock_time > block.timestamp, "Can only lock until time in the future"
            assert unlock_time <= block.timestamp + MAXTIME, "Voting lock can be 4 years max"

            self._deposit_for(msg.sender, _value, unlock_time, _locked, CREATE_LOCK_TYPE) 

        ```

    === "Example"
        
        ```shell
        >>> vecrv.apply_transfer_ownership():
        
        ```

### `vecrv.increase_amount`
!!! description "`vecrv.increase_amount(_value: uint256):`"

    todo

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_value`       |  `uint256` | Amount |

    ??? quote "Source code"

        ```python hl_lines="0"

        @external
        @nonreentrant('lock')
        def increase_amount(_value: uint256):
            """
            @notice Deposit `_value` additional tokens for `msg.sender`
                    without modifying the unlock time
            @param _value Amount of tokens to deposit and add to the lock
            """
            self.assert_not_contract(msg.sender)
            _locked: LockedBalance = self.locked[msg.sender]

            assert _value > 0  # dev: need non-zero value
            assert _locked.amount > 0, "No existing lock found"
            assert _locked.end > block.timestamp, "Cannot add to expired lock. Withdraw"

            self._deposit_for(msg.sender, _value, 0, _locked, INCREASE_LOCK_AMOUNT)

        ```

    === "Example"
        
        ```shell
        >>> vecrv.increase_amount(todo):
        
        ```


### `vecrv.increase_unlock_time`
!!! description "`vecrv.increase_unlock_time(_unlock_time: uint256):`"

    todo

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_unlock_time` |  `uint256` | Extend Unlock-Epoch |

    ??? quote "Source code"

        ```python hl_lines="0"

        @external
        @nonreentrant('lock')
        def increase_unlock_time(_unlock_time: uint256):
            """
            @notice Extend the unlock time for `msg.sender` to `_unlock_time`
            @param _unlock_time New epoch time for unlocking
            """
            self.assert_not_contract(msg.sender)
            _locked: LockedBalance = self.locked[msg.sender]
            unlock_time: uint256 = (_unlock_time / WEEK) * WEEK  # Locktime is rounded down to weeks

            assert _locked.end > block.timestamp, "Lock expired"
            assert _locked.amount > 0, "Nothing is locked"
            assert unlock_time > _locked.end, "Can only increase lock duration"
            assert unlock_time <= block.timestamp + MAXTIME, "Voting lock can be 4 years max"

            self._deposit_for(msg.sender, 0, unlock_time, _locked, INCREASE_UNLOCK_TIME)

        ```

    === "Example"
        
        ```shell
        >>> vecrv.increase_unlock_time(todo):
        todo
        ```

### `vecrv.withdraw`
!!! description "`vecrv.withdraw()`"

    todo

    ??? quote "Source code"

        ```python hl_lines="0"

        @external
        @nonreentrant('lock')
        def withdraw():
            """
            @notice Withdraw all tokens for `msg.sender`
            @dev Only possible if the lock has expired
            """
            _locked: LockedBalance = self.locked[msg.sender]
            assert block.timestamp >= _locked.end, "The lock didn't expire"
            value: uint256 = convert(_locked.amount, uint256)

            old_locked: LockedBalance = _locked
            _locked.end = 0
            _locked.amount = 0
            self.locked[msg.sender] = _locked
            supply_before: uint256 = self.supply
            self.supply = supply_before - value

            # old_locked can have either expired <= timestamp or zero end
            # _locked has only 0 end
            # Both can have >= 0 amount
            self._checkpoint(msg.sender, old_locked, _locked)

            assert ERC20(self.token).transfer(msg.sender, value)

            log Withdraw(msg.sender, value, block.timestamp)
            log Supply(supply_before, supply_before - value)

        ```

    === "Example"
        
        ```shell
        >>> vecrv.withdraw():
        todo
        ```

### `vecrv.change_controller`
!!! description "`vecrv.changeController(_newController: address):`"

    Simple dummy method required for Aragon compatibility.

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_newController` |  `address` | New Controller Address |

    ??? quote "Source code"

        ```python hl_lines="0"

        @external
        def changeController(_newController: address):
            """
            @dev Dummy method required for Aragon compatibility
            """
            assert msg.sender == self.controller
            self.controller = _newController
        ```
