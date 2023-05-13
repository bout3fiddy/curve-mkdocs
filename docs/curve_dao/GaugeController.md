 
## Overview

The “Gauge Controller” maintains a list of gauges and their types, with the weights of each gauge and type. In order to implement weight voting, GaugeController has to include parameters handling linear character of voting power each user has.

GaugeController records points (bias + slope) per gauge in vote_points, and _scheduled_ changes in biases and slopes for those points in vote_bias_changes and vote_slope_changes. New changes are applied at the start of each epoch week.

Per-user, per-gauge slopes are stored in vote_user_slopes, along with the power the user has used and the time their vote-lock ends.

The totals for slopes and biases for vote weight per gauge, and sums of those per type, are scheduled / recorded for the next week, as well as the points when voting power gets to 0 at lock expiration for some of users.

When a user changes their gauge weight vote, the change is scheduled for the next epoch week, not immediately. This reduces the number of reads from storage which must to be performed by each user: it is proportional to the number of weeks since the last change rather than the number of interactions from other users.

Each liquidity gauge is assigned a type within the gauge controller. Grouping gauges by type allows the DAO to adjust the emissions according to type, making it possible to e.g. end all emissions for a single type.

Currently active gauge types are as follows:  
Ethereum (stableswap pools): 0  
Fantom: 1  
Polygon (Matic): 2  
xDai: 4  
Ethereum (crypto pools): 5  
Arbitrum: 7  
Avalanche: 8  
Harmony: 9  

Types 3 and 6 have been deprecated.



!!!note
    [Etherscan](https://etherscan.io/address/0x2F50D538606Fa9EDD2B11E2446BEb18C9D5846bB#code)  
    [Github](https://github.com/curvefi/curve-dao-contracts/blob/master/contracts/GaugeController.vy)



## READ ONLY FUNCTIONS

### `gauge_types`

!!! description "`GaugeController.gauge_types(_addr: address) -> int128`"

    Returns gauge type of a gauge address


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_addr` |  `address` | Gauge Addresses |

    ??? quote "Source code"

        ```python hl_lines="1 5"
        gauge_types_: HashMap[address, int128]

        @external
        @view
        def gauge_types(_addr: address) -> int128:
            """
            @notice Get gauge type for address
            @param _addr Gauge address
            @return Gauge type id
            """
            gauge_type: int128 = self.gauge_types_[_addr]
            assert gauge_type != 0

            return gauge_type - 1
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.gauge_types(0xbFcF63294aD7105dEa65aA58F8AE5BE2D9d0952A)
        0
        ```


### `gauge_relative_weight`

!!! description "`GaugeController.gauge_relative_weight(addr: address, time: uint256 = block.timestamp) -> uint256`"

    Returns the relative weight of a gauge. 


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_addr` |  `address` | Gauge Addresses |
    | `time` |  `uint256` | Timestamp |

    ??? quote "Source code"

        ```python hl_lines="3 27"
        @internal
        @view
        def _gauge_relative_weight(addr: address, time: uint256) -> uint256:
            """
            @notice Get Gauge relative weight (not more than 1.0) normalized to 1e18
                    (e.g. 1.0 == 1e18). Inflation which will be received by it is
                    inflation_rate * relative_weight / 1e18
            @param addr Gauge address
            @param time Relative weight at the specified timestamp in the past or present
            @return Value of relative weight normalized to 1e18
            """
            t: uint256 = time / WEEK * WEEK
            _total_weight: uint256 = self.points_total[t]

            if _total_weight > 0:
                gauge_type: int128 = self.gauge_types_[addr] - 1
                _type_weight: uint256 = self.points_type_weight[gauge_type][t]
                _gauge_weight: uint256 = self.points_weight[addr][t].bias
                return MULTIPLIER * _type_weight * _gauge_weight / _total_weight

            else:
                return 0


        @external
        @view
        def gauge_relative_weight(addr: address, time: uint256 = block.timestamp) -> uint256:
            """
            @notice Get Gauge relative weight (not more than 1.0) normalized to 1e18
                    (e.g. 1.0 == 1e18). Inflation which will be received by it is
                    inflation_rate * relative_weight / 1e18
            @param addr Gauge address
            @param time Relative weight at the specified timestamp in the past or present
            @return Value of relative weight normalized to 1e18
            """
            return self._gauge_relative_weight(addr, time)
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.gauge_relative_weight(0x555766f3da968ecbefa690ffd49a2ac02f47aa5f)
        27557442674450559
        ```
    !!! note
        Function can also be called without the input of a timestamp --> will take the present timestamp as input.  
        The value of relative weight is normalized to 1e18.


### `get_gauge_weight`

!!! description "GaugeController.get_gauge_weight(addr: address) -> uint256`"

    Getter function of current gauge weight.


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `addr` |  `address` | Gauge Addresses |

    ??? quote "Source code"

        ```python hl_lines="3"
        @external
        @view
        def get_gauge_weight(addr: address) -> uint256:
            """
            @notice Get current gauge weight
            @param addr Gauge address
            @return Gauge weight
            """
            return self.points_weight[addr][self.time_weight[addr]].bias
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.get_gauge_weight(0xbFcF63294aD7105dEa65aA58F8AE5BE2D9d0952A)
        1987873524145187062272000
        ```


### `get_type_weight` (look at this again)

!!! description "`GaugeController.get_type_weight(type_id: int128) -> uint256`"

    Returns gauge type of a gauge address


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `type_id` |  `int128` | Gauge Type |

    ??? quote "Source code"

        ```python hl_lines="0"
        @internal
        def _get_type_weight(gauge_type: int128) -> uint256:
            """
            @notice Fill historic type weights week-over-week for missed checkins
                    and return the type weight for the future week
            @param gauge_type Gauge type id
            @return Type weight
            """
            t: uint256 = self.time_type_weight[gauge_type]
            if t > 0:
                w: uint256 = self.points_type_weight[gauge_type][t]
                for i in range(500):
                    if t > block.timestamp:
                        break
                    t += WEEK
                    self.points_type_weight[gauge_type][t] = w
                    if t > block.timestamp:
                        self.time_type_weight[gauge_type] = t
                return w
            else:
                return 0


        @external
        @view
        def get_type_weight(type_id: int128) -> uint256:
            """
            @notice Get current type weight
            @param type_id Type id
            @return Type weight
            """
            return self.points_type_weight[type_id][self.time_type_weight[type_id]]
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.get_type_weight(0)
        1000000000000000000
        ```


### `get_total_weight`

!!! description "`GaugeController.get_total_weight() -> uint256`"

    Returns the current total (type-weighted) weight.

    ??? quote "Source code"

        ```python hl_lines="1 5"
        points_total: public(HashMap[uint256, uint256])  # time -> total weight

        @external
        @view
        def get_total_weight() -> uint256:
            """
            @notice Get current total (type-weighted) weight
            @return Total weight
            """
            return self.points_total[self.time_total]
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.get_total_weight()
        547873886536122498468683976000000000000000000
        ```


### `get_weights_sum_per_type`

!!! description "`GaugeController.get_weights_sum_per_type(type_id: int128) -> uint256`"

    Returns the sum of guage weight per type.

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `type_id` |  `int128` | Gauge Type |

    ??? quote "Source code"

        ```python hl_lines=5"
        points_sum: public(HashMap[int128, HashMap[uint256, Point]])  # type_id -> time -> Point

        @external
        @view
        def get_weights_sum_per_type(type_id: int128) -> uint256:
            """
            @notice Get sum of gauge weights per type
            @param type_id Type id
            @return Sum of gauge weights
            """
            return self.points_sum[type_id][self.time_sum[type_id]].bias
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.get_weights_sum_per_type(0)
        357345591048932206476271176
        ```


### `admin`

!!! description "`GaugeController.admin -> address: view`"

    Returns the admin address of the contract. Was initially set to `msg.sender` but was passed on to the CurveOwnershitAgent by calling `commit_transfer_ownership`. todo

    ??? quote "Source code"

        ```python hl_lines="1 13"
        admin: public(address)  # Can and will be a smart contract

        @external
        def __init__(_token: address, _voting_escrow: address):
            """
            @notice Contract constructor
            @param _token `ERC20CRV` contract address
            @param _voting_escrow `VotingEscrow` contract address
            """
            assert _token != ZERO_ADDRESS
            assert _voting_escrow != ZERO_ADDRESS

            self.admin = msg.sender
            self.token = _token
            self.voting_escrow = _voting_escrow
            self.time_total = block.timestamp / WEEK * WEEK
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.admin()
        0x40907540d8a6C65c637785e8f8B742ae6b0b9968
        ```

### `future_admin`

!!! description "`GaugeController.future_admin -> address: view`"

    Getter function of the future admin of GaugeController. Can be changed by calling `commit_transfer_ownership` by the admin.

    ??? quote "Source code"

        ```python hl_lines="1 10"
        future_admin: public(address)  # Can and will be a smart contract

        @external
        def commit_transfer_ownership(addr: address):
            """
            @notice Transfer ownership of GaugeController to `addr`
            @param addr Address to have ownership transferred to
            """
            assert msg.sender == self.admin  # dev: admin only
            self.future_admin = addr
            log CommitOwnership(addr)
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.future_admin()
        0x40907540d8a6C65c637785e8f8B742ae6b0b9968        
        ```


### `commit_transfer_ownership`

!!! description "`GaugeController.commit_transfer_ownership(addr: address)`"

    Function to change the admin of the contract. Can only be called by the admin.

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `addr` |  `address` | New Admin Address |

    ??? quote "Source code"

        ```python hl_lines="1 10"
        future_admin: public(address)  # Can and will be a smart contract

        @external
        def commit_transfer_ownership(addr: address):
            """
            @notice Transfer ownership of GaugeController to `addr`
            @param addr Address to have ownership transferred to
            """
            assert msg.sender == self.admin  # dev: admin only
            self.future_admin = addr
            log CommitOwnership(addr)
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.commit_transfer_ownership(todo)
        todo
        ```




### `apply_transfer_ownership`

!!! description "`GaugeController.admin -> address: view`"

    Returns the admin address of the contract.

    ??? quote "Source code"

        ```python hl_lines="1 5"
        event ApplyOwnership:
            admin: address

        @external
        def apply_transfer_ownership():
            """
            @notice Apply pending ownership transfer
            """
            assert msg.sender == self.admin  # dev: admin only
            _admin: address = self.future_admin
            assert _admin != ZERO_ADDRESS  # dev: admin not set
            self.admin = _admin
            log ApplyOwnership(_admin)
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.apply_transfer_ownership()

        ```





### ``

!!! description "`GaugeController.`"

    Returns gauge type of a gauge address


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_addr` |  `address` | Gauge Addresses |

    ??? quote "Source code"

        ```python hl_lines="1 5"
        gauge_types_: HashMap[address, int128]

        @external
        @view
        def gauge_types(_addr: address) -> int128:
            """
            @notice Get gauge type for address
            @param _addr Gauge address
            @return Gauge type id
            """
            gauge_type: int128 = self.gauge_types_[_addr]
            assert gauge_type != 0

            return gauge_type - 1
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.gauge_types(0xbFcF63294aD7105dEa65aA58F8AE5BE2D9d0952A)
        0
        ```

### ``

!!! description "`GaugeController.`"

    Returns gauge type of a gauge address


    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `_addr` |  `address` | Gauge Addresses |

    ??? quote "Source code"

        ```python hl_lines="1 5"
        gauge_types_: HashMap[address, int128]

        @external
        @view
        def gauge_types(_addr: address) -> int128:
            """
            @notice Get gauge type for address
            @param _addr Gauge address
            @return Gauge type id
            """
            gauge_type: int128 = self.gauge_types_[_addr]
            assert gauge_type != 0

            return gauge_type - 1
        ```

    === "Example"
        
        ```shell
        >>> GaugeController.gauge_types(0xbFcF63294aD7105dEa65aA58F8AE5BE2D9d0952A)
        0
        ```

