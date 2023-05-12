## Overview

The Curve DAO token is the Token of the Curve Ecosystem.

!!! note
    Token Address = 0xD533a949740bb3306d119CC777fa900bA034cd52


The following Brownie console interaction examples are using the 
[CRV](https://etherscan.io/token/0xD533a949740bb3306d119CC777fa900bA034cd52) Token Contract.

## Curve DAO Token Info Methods

### `.balanceOf`

!!! description "`crv.balanceOf(arg0: address) → uint256`"

    Aggregates the balance of CRV Tokens of a wallet.

    Returns: crv balance (`uint256`) for address `arg0`.

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `arg0`       |  `address` | Wallet address |

    ??? quote "Source code"

        ```python hl_lines="1"
        balanceOf: public(HashMap[address, uint256])

        ```

    === "Example"
        
        ```shell
        >>> crv.balanceOf(0x5f3b5DfEb7B28CDbD7FAba78963EE202a494e2A2)
        '655,445,768.465357265214476736'
        ```

### `.balances`

!!! description "`StableSwap.balances(i: uint256) → uint256: view`"

    Getter for the pool balances array.

    Returns: Balance of coin (`uint256`) at index `i`.

    | Input      | Type   | Description |
    | ----------- | -------| ----|
    | `i`       |  `uint256` | Coin index |
        
    === "Example"
    
        ```shell
        >>> pool.balances(0)
        2918187395
        ```



