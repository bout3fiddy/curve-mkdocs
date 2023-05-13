## Overview


Minter Contract is deployed on the Ethereum Mainnet at:  0xd061D61a4d941c39E5453435B6345Dc261C2fcE0

[Etherscan](https://etherscan.io/address/0xd061D61a4d941c39E5453435B6345Dc261C2fcE0#code)  
[Github](https://github.com/curvefi/curve-dao-contracts/blob/master/contracts/Minter.vy).




## ???

### `mint`

!!! description "`Minter.mint(gauge_addr: address)`"

    Function to mint CRV form a gauge.


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `gauge_addr` |  `address` | Gauge Addresses |

    ??? quote "Source code"

        ```python hl_lines="1 7 18 23"
        event Minted:
            recipient: indexed(address)
            gauge: address
            minted: uint256

        @internal
        def _mint_for(gauge_addr: address, _for: address):
            assert GaugeController(self.controller).gauge_types(gauge_addr) >= 0  # dev: gauge is not added

            LiquidityGauge(gauge_addr).user_checkpoint(_for)
            total_mint: uint256 = LiquidityGauge(gauge_addr).integrate_fraction(_for)
            to_mint: uint256 = total_mint - self.minted[_for][gauge_addr]

            if to_mint != 0:
                MERC20(self.token).mint(_for, to_mint)
                self.minted[_for][gauge_addr] = total_mint

                log Minted(_for, gauge_addr, total_mint)


        @external
        @nonreentrant('lock')
        def mint(gauge_addr: address):
            """
            @notice Mint everything which belongs to `msg.sender` and send to them
            @param gauge_addr `LiquidityGauge` address to get mintable amount from
            """
            self._mint_for(gauge_addr, msg.sender)
        ```

    === "Example"
        
        ```shell
        >>> Minter.mint(todo)
        'todo'
        ```

### `mint_many`

!!! description "`Minter.mint_many(gauge_addrs: address[8])`"

    Function to mint CRV from multiple gauges.


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `gauge_addrs` |  `address` | Gauge Addresses |

    ??? quote "Source code"

        ```python hl_lines="1 7 18 23"
        event Minted:
            recipient: indexed(address)
            gauge: address
            minted: uint256

        @internal
        def _mint_for(gauge_addr: address, _for: address):
            assert GaugeController(self.controller).gauge_types(gauge_addr) >= 0  # dev: gauge is not added

            LiquidityGauge(gauge_addr).user_checkpoint(_for)
            total_mint: uint256 = LiquidityGauge(gauge_addr).integrate_fraction(_for)
            to_mint: uint256 = total_mint - self.minted[_for][gauge_addr]

            if to_mint != 0:
                MERC20(self.token).mint(_for, to_mint)
                self.minted[_for][gauge_addr] = total_mint

                log Minted(_for, gauge_addr, total_mint)


        @external
        @nonreentrant('lock')
        def mint_many(gauge_addrs: address[8]):
            """
            @notice Mint everything which belongs to `msg.sender` across multiple gauges
            @param gauge_addrs List of `LiquidityGauge` addresses
            """
            for i in range(8):
                if gauge_addrs[i] == ZERO_ADDRESS:
                    break
                self._mint_for(gauge_addrs[i], msg.sender)
        ```

    === "Example"
        
        ```shell
        >>> Minter.mint_many(todo)
        'todo'
        ```


### `mint_for`

!!! description "`Minter.mint_for(gauge_addr: address, _for: address)`"

    Mint tokens for a different address. This function can only be called when the caller has previously been approved by `for` using `toggle_approve_mint`.


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `gauge_addrs` |  `address` | Gauge Addresses |

    ??? quote "Source code"

        ```python hl_lines="1 7 18 23"
        event Minted:
            recipient: indexed(address)
            gauge: address
            minted: uint256

        @internal
        def _mint_for(gauge_addr: address, _for: address):
            assert GaugeController(self.controller).gauge_types(gauge_addr) >= 0  # dev: gauge is not added

            LiquidityGauge(gauge_addr).user_checkpoint(_for)
            total_mint: uint256 = LiquidityGauge(gauge_addr).integrate_fraction(_for)
            to_mint: uint256 = total_mint - self.minted[_for][gauge_addr]

            if to_mint != 0:
                MERC20(self.token).mint(_for, to_mint)
                self.minted[_for][gauge_addr] = total_mint

                log Minted(_for, gauge_addr, total_mint)


        @external
        @nonreentrant('lock')
        def mint_for(gauge_addr: address, _for: address):
            """
            @notice Mint tokens for `_for`
            @dev Only possible when `msg.sender` has been approved via `toggle_approve_mint`
            @param gauge_addr `LiquidityGauge` address to get mintable amount from
            @param _for Address to mint to
            """
            if self.allowed_to_mint_for[msg.sender][_for]:
                self._mint_for(gauge_addr, _for)
        ```

    === "Example"
        
        ```shell
        >>> Minter.mint_for(todo)
        'todo'
        ```


### `toggle_approve_mint`

!!! description "`Minter.toggle_approve_mint(minting_user: address)`"

    Function to toggle approval for `minting_user` to mint CRV on behalf of the caller .

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `minting_user` |  `address` | Address |

    ??? quote "Source code"

        ```python hl_lines="1 7 18 23"
        @external
        def toggle_approve_mint(minting_user: address):
            """
            @notice allow `minting_user` to mint for `msg.sender`
            @param minting_user Address to toggle permission for
            """
            self.allowed_to_mint_for[minting_user][msg.sender] = not self.allowed_to_mint_for[minting_user][msg.sender]
        ```

    === "Example"
        
        ```shell
        >>> Minter.toggle_approve_mint(todo)
        'todo'
        ```

### `allowed_to_mint_for`

!!! description "`Minter.allowed_to_mint_for(arg0: address, arg1: address) -> bool: view`"

    Getter method to check if `minter` has been approved to call `mint_for` on behalf of `for`.


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `arg0` |  `address` | Address |
    | `arg1` |  `address` | Address |

    ??? quote "Source code"

        ```python hl_lines="1"
        allowed_to_mint_for: public(HashMap[address, HashMap[address, bool]])
        ```

    === "Example"
        
        ```shell
        >>> Minter.allowed_to_mint_for(0x7a16fF8270133F063aAb6C9977183D9e72835428, 0x0E33Be39B13c576ff48E14392fBf96b02F40Cd34)
        false
        ```

