## Overview

Curve DAO Token ($CRV) is based on the ERC-20 token standard as defined at https://eips.ethereum.org/EIPS/eip-20.

!!! note
    Token Address: 0xD533a949740bb3306d119CC777fa900bA034cd52


Allocation of the Curve DAO Token:  
shareholders - 30%  
employees - 3%  
DAO-controller reserves - 5%  
Early users - 5%    
Inflation - 47%


The following Brownie console interaction examples are using the 
[CRV](https://etherscan.io/token/0xD533a949740bb3306d119CC777fa900bA034cd52) Token Contract.  
Source code of the contract can be found on [Github](https://github.com/curvefi/curve-dao-contracts/blob/master/contracts/ERC20CRV.vy).


## READ ONLY FUNCS/PUBLIC VARIABLES (HOW TO CALL THIS? ALSO HOW TO ORDER IT?)


### `name`

!!! description "`name -> String[64]`"

    Returns the name of the Token. Can be changed by calling the `set_name()` function (see below).

    ??? quote "Source code"

        ```python hl_lines="1 4 7 12"
        name: public(String[64])

        @external
        def __init__(_name: String[64], _symbol: String[32], _decimals: uint256):
            """
            @notice Contract constructor
            @param _name Token full name
            @param _symbol Token symbol
            @param _decimals Number of decimals for token
            """
            init_supply: uint256 = INITIAL_SUPPLY * 10 ** _decimals
            self.name = _name
            self.symbol = _symbol
            self.decimals = _decimals
            self.balanceOf[msg.sender] = init_supply
            self.total_supply = init_supply
            self.admin = msg.sender
            log Transfer(ZERO_ADDRESS, msg.sender, init_supply)

            self.start_epoch_time = block.timestamp + INFLATION_DELAY - RATE_REDUCTION_TIME
            self.mining_epoch = -1
            self.rate = 0
            self.start_epoch_supply = init_supply
        ```

    === "Example"
        
        ```shell
        >>> crv.name()
        Curve DAO Token
        ```



### `symbol`

!!! description "`symbol -> String[32]`"

    Returns the symbol of the Token.

    ??? quote "Source code"

        ```python hl_lines="1 4 8 13"
        symbol: public(String[32])

        @external
        def __init__(_name: String[64], _symbol: String[32], _decimals: uint256):
            """
            @notice Contract constructor
            @param _name Token full name
            @param _symbol Token symbol
            @param _decimals Number of decimals for token
            """
            init_supply: uint256 = INITIAL_SUPPLY * 10 ** _decimals
            self.name = _name
            self.symbol = _symbol
            self.decimals = _decimals
            self.balanceOf[msg.sender] = init_supply
            self.total_supply = init_supply
            self.admin = msg.sender
            log Transfer(ZERO_ADDRESS, msg.sender, init_supply)

            self.start_epoch_time = block.timestamp + INFLATION_DELAY - RATE_REDUCTION_TIME
            self.mining_epoch = -1
            self.rate = 0
            self.start_epoch_supply = init_supply
        ```

    === "Example"
        
        ```shell
        >>> crv.symbol()
        CRV
        ```


### `decimals`

!!! description "`decimals -> uint256`"

    Returns the decimals of the Token.

    ??? quote "Source code"

        ```python hl_lines="1 4 9 14"
        decimals: public(uint256)

        @external
        def __init__(_name: String[64], _symbol: String[32], _decimals: uint256):
            """
            @notice Contract constructor
            @param _name Token full name
            @param _symbol Token symbol
            @param _decimals Number of decimals for token
            """
            init_supply: uint256 = INITIAL_SUPPLY * 10 ** _decimals
            self.name = _name
            self.symbol = _symbol
            self.decimals = _decimals
            self.balanceOf[msg.sender] = init_supply
            self.total_supply = init_supply
            self.admin = msg.sender
            log Transfer(ZERO_ADDRESS, msg.sender, init_supply)

            self.start_epoch_time = block.timestamp + INFLATION_DELAY - RATE_REDUCTION_TIME
            self.mining_epoch = -1
            self.rate = 0
            self.start_epoch_supply = init_supply
        ```

    === "Example"
        
        ```shell
        >>> crv.decimals()
        18
        ```


### `minter`

!!! description "`minter -> address`"

    Returns the decimals of the Token.

    ??? quote "Source code"

        ```python hl_lines="1"
        minter: public(address)
        ```

    === "Example"
        
        ```shell
        >>> crv.minter()
        0xd061D61a4d941c39E5453435B6345Dc261C2fcE0
        ```


### `admin`

!!! description "`admin -> address`"

    Returns the admin address of the Token, which is the Curve Finance Ownership Agent (DAO).  
    Initial admin was the deployer of the contract (msg.sender), todo

    ??? quote "Source code"

        ```python hl_lines="1 17"
        admin: public(address)

        @external
        def __init__(_name: String[64], _symbol: String[32], _decimals: uint256):
            """
            @notice Contract constructor
            @param _name Token full name
            @param _symbol Token symbol
            @param _decimals Number of decimals for token
            """
            init_supply: uint256 = INITIAL_SUPPLY * 10 ** _decimals
            self.name = _name
            self.symbol = _symbol
            self.decimals = _decimals
            self.balanceOf[msg.sender] = init_supply
            self.total_supply = init_supply
            self.admin = msg.sender
            log Transfer(ZERO_ADDRESS, msg.sender, init_supply)

            self.start_epoch_time = block.timestamp + INFLATION_DELAY - RATE_REDUCTION_TIME
            self.mining_epoch = -1
            self.rate = 0
            self.start_epoch_supply = init_supply
        ```

    === "Example"
        
        ```shell
        >>> admin()
        0x40907540d8a6C65c637785e8f8B742ae6b0b9968
        ```


### `rate`

!!! description "`rate -> uint256`"

    Returns the current rate of emissions. Initial value of `rate` was 0 at deployment. Emissions were started by calling `update_mining_parameters()`.

    ??? quote "Source code"

        ```python hl_lines="1 22"
        rate: public(uint256)

        @external
        def __init__(_name: String[64], _symbol: String[32], _decimals: uint256):
            """
            @notice Contract constructor
            @param _name Token full name
            @param _symbol Token symbol
            @param _decimals Number of decimals for token
            """
            init_supply: uint256 = INITIAL_SUPPLY * 10 ** _decimals
            self.name = _name
            self.symbol = _symbol
            self.decimals = _decimals
            self.balanceOf[msg.sender] = init_supply
            self.total_supply = init_supply
            self.admin = msg.sender
            log Transfer(ZERO_ADDRESS, msg.sender, init_supply)

            self.start_epoch_time = block.timestamp + INFLATION_DELAY - RATE_REDUCTION_TIME
            self.mining_epoch = -1
            self.rate = 0
            self.start_epoch_supply = init_supply

        ```

    === "Example"
        
        ```shell
        >>> rate()
        6161965695807970181
        ```
    
    !!! note
        Rate displays emissions per second. Emissions per year -> 6.161965695807970181 * 86400 = 532393.8361178086 



### `avaliable_supply`

!!! description "`avaliably_supply -> uint256`"

    Returns the current number of tokens in existence (claimed or unclaimed)

    ??? quote "Source code"

        ```python hl_lines="0"
        @internal
        @view
        def _available_supply() -> uint256:
            return self.start_epoch_supply + (block.timestamp - self.start_epoch_time) * self.rate

        @external
        @view
        def available_supply() -> uint256:
            """
            @notice Current number of tokens in existence (claimed or unclaimed)
            """
            return self._available_supply()
        ```

    === "Example"
        
        ```shell
        >>> avaliable_supply()
        1953676805157446496269106603
        ```


### `totalSupply`

!!! description "`totalSupply -> uint256`"

    Returns the total number of tokens in existence.

    ??? quote "Source code"

        ```python hl_lines="1 4 9 14"
        @external
        @view
        def totalSupply() -> uint256:
            """
            @notice Total number of tokens in existence.
            """
            return self.total_supply
        ```

    === "Example"
        
        ```shell
        >>> totalSupply()
        18
        ```

### `mining_epoch`

!!! description "`mining_epoch -> int128`"

    Returns the total number of tokens in existence.

    ??? quote "Source code"

        ```python hl_lines="1 21 34"
        mining_epoch: public(int128)

        @external
        def __init__(_name: String[64], _symbol: String[32], _decimals: uint256):
            """
            @notice Contract constructor
            @param _name Token full name
            @param _symbol Token symbol
            @param _decimals Number of decimals for token
            """
            init_supply: uint256 = INITIAL_SUPPLY * 10 ** _decimals
            self.name = _name
            self.symbol = _symbol
            self.decimals = _decimals
            self.balanceOf[msg.sender] = init_supply
            self.total_supply = init_supply
            self.admin = msg.sender
            log Transfer(ZERO_ADDRESS, msg.sender, init_supply)

            self.start_epoch_time = block.timestamp + INFLATION_DELAY - RATE_REDUCTION_TIME
            self.mining_epoch = -1
            self.rate = 0
            self.start_epoch_supply = init_supply

        @internal
        def _update_mining_parameters():
            """
            @dev Update mining rate and supply at the start of the epoch
                Any modifying mining call must also call this
            """
            _rate: uint256 = self.rate
            _start_epoch_supply: uint256 = self.start_epoch_supply

            self.start_epoch_time += RATE_REDUCTION_TIME
            self.mining_epoch += 1

            if _rate == 0:
                _rate = INITIAL_RATE
            else:
                _start_epoch_supply += _rate * RATE_REDUCTION_TIME
                self.start_epoch_supply = _start_epoch_supply
                _rate = _rate * RATE_DENOMINATOR / RATE_REDUCTION_COEFFICIENT

            self.rate = _rate

            log UpdateMiningParameters(block.timestamp, _rate, _start_epoch_supply)
        ```

    === "Example"
        
        ```shell
        >>> mining_epoch()
        2
        ```


### `start_epoch_time`

!!! description "`start_epoch_time -> uint256`"

    Returns the total number of tokens in existence.

    ??? quote "Source code"

        ```python hl_lines="1 20"
        start_epoch_time: public(uint256)

        @external
        def __init__(_name: String[64], _symbol: String[32], _decimals: uint256):
            """
            @notice Contract constructor
            @param _name Token full name
            @param _symbol Token symbol
            @param _decimals Number of decimals for token
            """
            init_supply: uint256 = INITIAL_SUPPLY * 10 ** _decimals
            self.name = _name
            self.symbol = _symbol
            self.decimals = _decimals
            self.balanceOf[msg.sender] = init_supply
            self.total_supply = init_supply
            self.admin = msg.sender
            log Transfer(ZERO_ADDRESS, msg.sender, init_supply)

            self.start_epoch_time = block.timestamp + INFLATION_DELAY - RATE_REDUCTION_TIME
            self.mining_epoch = -1
            self.rate = 0
            self.start_epoch_supply = init_supply
        ```

    === "Example"
        
        ```shell
        >>> start_epoch_time()
        1660429048
        ```






## WRITE FUNCTIONS (HOW TO CALL THESE ONES?)

### `set_minter`

!!! description "`set_minter(_minter: address):`"

    Changes the minter address of the contract. Minter could only be set once (no change of address possible).

    ??? quote "Source code"

        ```python hl_lines="1 2 4 7"
        event SetMinter:
            minter: address
        
        minter: public(address)

        @external
        def set_minter(_minter: address):
            """
            @notice Set the minter address
            @dev Only callable once, when minter has not yet been set
            @param _minter Address of the minter
            """
            assert msg.sender == self.admin  # dev: admin only
            assert self.minter == ZERO_ADDRESS  # dev: can set the minter only once, at creation
            self.minter = _minter
            log SetMinter(_minter)

        ```

        === "Example"
        
        ```shell
        >>> set_admin()
        todo?A?FAS
        ```


### `set_admin`

!!! description "`set_admin(_minter: address):`"

    Changes the minter address of the contract. Minter could only be set once (no change of address possible).

    ??? quote "Source code"

        ```python hl_lines="1 2 4 7"
        event SetAdmin:
            admin: address
        
        admin: public(address)

        @external
        def set_admin(_admin: address):
            """
            @notice Set the new admin.
            @dev After all is set up, admin only can change the token name
            @param _admin New admin address
            """
            assert msg.sender == self.admin  # dev: admin only
            self.admin = _admin
            log SetAdmin(_admin)

        ```

    === "Example"
        
        ```shell
        >>> set_admin()
        todo
        ```

### `update_mining_parameters`

!!! description "`update_mining_parameters()`"

    function updates mining parameters --> reduces rate. internal function (cant be called directly -> need to be called via `update_mining_parameters`)  
    note to myself(!): `update_mining_parameters` calls `_update_mining_parameters` as it is a internal function which can only called this way.  
    following happens: at deployment rate = 0; start_epoch_supply = initial_supply (1_303_030_303); when successfully called (via `update_mining_parameters`) --> start_epoch_supply + RATE_REDUCTION_TIME and mining_epoch + 1 (which was set to -1 at initialization; so first time calling it actually sets mining_epoch to 0.); if statement to check if rate == 0 (if yes - which is true when called successfully once - then rate is INITIAL_RATE; if no _start_epoch_supply is rate mulitplied by RATE_REDUCTION_TIME (in seconds because rate is also denominated in seconds) to get the entire epoch supply); rate will also be updated and reduced by the RATE_REDUCTION_COEFFICIENT (2 ** (1/4) * 1e18); also start_epoch_time was "modified" at initialization of the contract with an INFLATION_DELAY of 86400s. event is triggered when called successfully.

    `assert block.timestamp >= self.start_epoch_time + RATE_REDUCTION_TIME` makes sure function can not be called to soon.
    this is fucking genius lol


    ??? quote "Source code"

        ```python hl_lines="1 22 46"
        event UpdateMiningParameters:
            time: uint256
            rate: uint256
            supply: uint256

        # Supply parameters
        INITIAL_SUPPLY: constant(uint256) = 1_303_030_303
        INITIAL_RATE: constant(uint256) = 274_815_283 * 10 ** 18 / YEAR  # leading to 43% premine
        RATE_REDUCTION_TIME: constant(uint256) = YEAR
        RATE_REDUCTION_COEFFICIENT: constant(uint256) = 1189207115002721024  # 2 ** (1/4) * 1e18
        RATE_DENOMINATOR: constant(uint256) = 10 ** 18
        INFLATION_DELAY: constant(uint256) = 86400

        # Supply variables
        mining_epoch: public(int128)
        start_epoch_time: public(uint256)
        rate: public(uint256)

        start_epoch_supply: uint256

        @internal
        def _update_mining_parameters():
            """
            @dev Update mining rate and supply at the start of the epoch
                Any modifying mining call must also call this
            """
            _rate: uint256 = self.rate
            _start_epoch_supply: uint256 = self.start_epoch_supply

            self.start_epoch_time += RATE_REDUCTION_TIME
            self.mining_epoch += 1

            if _rate == 0:
                _rate = INITIAL_RATE
            else:
                _start_epoch_supply += _rate * RATE_REDUCTION_TIME
                self.start_epoch_supply = _start_epoch_supply
                _rate = _rate * RATE_DENOMINATOR / RATE_REDUCTION_COEFFICIENT

            self.rate = _rate

            log UpdateMiningParameters(block.timestamp, _rate, _start_epoch_supply)


        @external
        def update_mining_parameters():
            """
            @notice Update mining rate and supply at the start of the epoch
            @dev Callable by any address, but only once per epoch
                Total supply becomes slightly larger if this function is called late
            """
            assert block.timestamp >= self.start_epoch_time + RATE_REDUCTION_TIME  # dev: too soon!
            self._update_mining_parameters()

        ```

    === "Example"
        
        ```shell
        >>> update_mining_parameters()
        todo
        ```

### `mint`

!!! description "`set_minter(_minter: address):`"

    Changes the minter address of the contract. Minter could only be set once (no change of address possible).

    ??? quote "Source code"

        ```python hl_lines="1 2 4 7"
        event SetAdmin:
            admin: address
        
        admin: public(address)

        @external
        def set_admin(_admin: address):
            """
            @notice Set the new admin.
            @dev After all is set up, admin only can change the token name
            @param _admin New admin address
            """
            assert msg.sender == self.admin  # dev: admin only
            self.admin = _admin
            log SetAdmin(_admin)

        ```

    === "Example"
        
        ```shell
        >>> mint()
        todo
        ```



### `burn`

!!! description "`todo`"

    Changes the minter address of the contract. Minter could only be set once (no change of address possible).

    ??? quote "Source code"

        ```python hl_lines="0"

        ```

    === "Example"
        
        ```shell
        >>> burn()
        todo
        ```

