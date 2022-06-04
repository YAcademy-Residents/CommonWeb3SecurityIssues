# Common Web3 Security Issues

## External calls
- Check for reentrancy risks
- Check whether the external call specifies the 1. the proper function and 2. the proper function arguments in the proper order for that function.

## Swap is performed
- Check for frontrunning and sandwich attack of the swap. This can happen when interacting with any liquidity pool (Uniswap, Sushi, Curve, Balancer, etc.)
- If a one-sided swap is performed before depositing into the liquidity pool (often named a "zap"), there may be [dust left over](https://blog.alphaventuredao.io/onesideduniswap/) because the first one-sided swap alters the balance and price of the liquidity pool assets.

## transferFrom is used
- If tokens are transferred from a contract address, that contract must approve the tokens before the transferFrom is called, otherwise it will revert

## Overflows
- Safe math in Solidity 0.8.X does not protect against integer casting overflows and bitshift overflows

## Gas griefing / denial of service
- Check all `require` and `revert` logic in case of unintentional reverts causing the user loss of gas

## ecrecover
- Standard solidity ecrecover function is used without checking if the result is the zero address. Must check for zero address or use OZ `recover` from ECDSA library.

## ERC20
- If an ERC20 token list is used it must consider double entry point tokens, with past issues of this type [here](https://medium.com/chainsecurity/trueusd-compound-vulnerability-bc5b696d29e2) and [here](https://forum.balancer.fi/t/medium-severity-bug-found/3161)

## ERC721
- The ERC721 tokenId value must be unique. If the tokenId value is not a simple incrementing counter and instead uses a formula to calculate the tokenId, duplicate values may occur and lead to a revert
- If safeMint is used, check for reentrancy on the callback hook

## Access controls
- If any access controls are used (often in the form of modifiers) check functions that do not have any modifier to see if the modifier may have been forgotten.

## External price data is used
- If spot price of a liquidity pool is used... [just no](https://shouldiusespotpriceasmyoracle.com/)
- Chainlink oracle data may be stale. The roundId and timestamp should be checked.

## AMM/DEX
- Any purchase or swap function should have slippage protection. This is normally a user-specified function argument.

## Compound forks
- Compound does not strictly follow the checks-effects-interactions pattern to avoid reentrancy. This can lead to reentrancy problems [if tokens with callbacks](https://twitter.com/Hacxyk/status/1520370424680304640) (ERC721, ERC777, etc.) are used.
- Compound whitelists tokens and can avoid ERC20 tokens with unusual behavior. If a fork of compound does not whitelist tokens, issues can exist with [fee-on-transfer tokens](https://github.com/d-xo/weird-erc20#fee-on-transfer) among others.

## Proxies
<!-- markdown-link-check-disable-next-line -->
- UUPS proxies MUST be initialized after deployment. Forgetting to initialize the proxy led to [the first $10 million bug](https://medium.com/immunefi/wormhole-uninitialized-proxy-bugfix-review-90250c41a43a) from Immunefi and [many other past bugs](https://twitter.com/transmissions11/status/1527699663322697728).
- State variable layout must be followed when delegatecall is used, otherwise this [leads to problems](https://solidity-by-example.org/hacks/delegatecall)

## OpenZeppelin Upgradeable imports
- Many OpenZeppelin upgradeable contracts need to be initialized in the importing contract's constructor.

## Voting and rewards/yield
- Does the snapshot process prevent a user from flashloaning tokens to make a large number of votes or receive a lot of yield?
- Does the snapshot process prevent a user from voting (or redeeming yield), transferring tokens, and voting again (or redeeming yield)?
- If the voting happens off-chain, is proper planning and integration with snapshot.org applied? Which voting strategy is used? Will [voting delegation be added](https://docs.snapshot.org/guides/delegation#with-a-smart-contract) with the `setDelegate` function?

## Loan-related protocol
- Is there a way that a user can become undercollateralized, reducing the incentive to pay their debt?

## NFT projects
- If cryptopunks are supported (which requires custom code), it should protect against [this frontrun attack vector](https://blog.nftx.io/nftx-v2-punk-incident-post-mortem/).