# MetaPool Factory

The metapool factory allows for permissionless deployment of Curve metapools. Source code for factory contracts may be 
viewed on [Github](https://github.com/curvefi/curve-factory).

# Organization

The metapool factory has several core components:

- The **factory** is the main contract used to deploy new metapools. It also acts a registry for finding the deployed pools 
  and querying information about them.
- **Pools** are deployed via a **proxy contract**. The implementation contract targetted by the proxy is determined according to 
  the base pool. This is the same technique used to create pools in Uniswap V1.
- **Deposit contracts** (“zaps”) are used for wrapping and unwrapping underlying assets when depositing into or withdrawing 
  from pools.

