## [I-0] Unchanged state variable should be declared constant or immutable.

Reading from storage is more expensive then reading from a constant or immutable variable.

Instance:
- `PuppyRaffle::raffleDuration` should be `immutable`
- `PuppyRaffle::commonImageUri` should be `constant`
- `PuppyRaffle::rareImageUri` should be `constant`
- `PuppyRaffle::legendaryImageUri` should be `constant`

## [L-0] Storage variable in a loop should be cached

Everytime you call `players.length` you read the storage as opposed to more gas.

```Diff
+  uint256 playersLength = players.length;
-  for (uint256 i = 0; i < players.length - 1; i++) {
+  for (uint256 i = 0; i < players.length - 1; i++) {
-            for (uint256 j = i + 1; j < players.length; j++) {
+             for (uint256 j = i + 1; j < players.length; j++) {
                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
            }
        }
```

## [L-1] `PuppyRaffle::getActivePlayerIndex` returns 0 for non-existant players and for players at index 0, causing a player on index 0 get incorrectly listed in non-existant players

**Description:**  If a player is in the `PuppyRaffle::players` array at index 0, this will return 0, but according to the natspec it will also return zero if the player is NOT in the array.

```solidity
function getActivePlayerIndex(address player) external view returns (uint256) {
        for (uint256 i = 0; i < players.length; i++) {
            if (players[i] == player) {
                return i;
            }
        }
        return 0;
    }
```

## [M-0] Smart contract wallets raffle winners if not have `receive` or `fallback` function then they blocked to start new contest means winners not get `prizePool`.

**Description:** In `PuppyRaffle::selectWinner` function the flow of select winner is (.call return -->> false -> your require(success,) -> revert, then `PuppyRaffle` get stucked) here you become a Denial of Service(DoS) vector.

```solidity
(bool success,) = winner.call{value: prizePool}("");
        require(success, "PuppyRaffle: Failed to send prize pool to winner");
        _safeMint(winner, tokenId);
    }
```

**Impact:** The `PuppyRaffle::selectWinner` function could revert many times, and make the contest reset difficult.
- Severity: MEDIUM
- LikelyHood: M/L

**Proof-of-Concept:** A behavior of `PuppyRaffle::selectWinner` function we notice here:

1. suppose 20 smart contract wallets enter in `PuppyRaffle` lottery, and one smart contract wallet enter without `receive` or `fallback` function.
2. Now the lottery ends and coincidentally the one who without `receive` or `fallback` function becomes winner of this contest
3. But in this step winner not get paid because `PuppyRaffle::selectWinner` function carries low-level call function to handle the `prizePool` ETH of winner.

**Recommended Mitigation:** Here is two ways to avoid this type of behavior of this `PuppyRaffle::selectWinner` function:

1. If we restrict this `PuppyRaffle::selectWinner` function with onlyOwner and smart contract wallets entrants not access it. But this type of solution lost trust of participants and Owner try to call this `PuppyRaffle::selectWinner` function when they want.
2. Create a mapping of addresses to uint256(payout) so winner can pull their funds themselves. putting the owness of the winner to claim their prize. (recommended)


## [I-1] This `PuppyRaffle::_isActivePlayer` is not used and should be removed

**Impact:** This `PuppyRaffle::_isActivePlayer` function use unnecessary gas because you don't use it anywhere so if better approach is that you should be removed this.

