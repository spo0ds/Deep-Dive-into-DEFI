**Smart contract module of MakerDAO**

The core module contains the state of the Maker Protocol and its central mechanisms while in normal operation
- Vat
- Cat
- Spotter

**[Vat](https://github.com/makerdao/dss/blob/master/src/vat.sol)**

```solidity
mapping (address => uint) public wards;
```

This is an address mapped to uint which stores whether an address is authenticated or not.If the address is authenticated, it can call protected functions.

```solidity
function rely(address usr) external auth { require(live == 1, "Vat/not-live"); wards[usr] = 1; }
function deny(address usr) external auth { require(live == 1, "Vat/not-live"); wards[usr] = 0; }
```

Rely function authenticates(adds to the whitelist) the address only if live is 1 where as deny function seizes that.Contracts have live = 1, indicating the system is running normally.

```solidity
modifier auth {
        require(wards[msg.sender] == 1, "Vat/not-authorized");
        _;
    }
```

This modifier implies that the address should be authenticated.

```solidity
mapping(address => mapping (address => uint)) public can;
```

This can is an approved allowance thing.

```solidity
function hope(address usr) external { can[msg.sender][usr] = 1; }
function nope(address usr) external { can[msg.sender][usr] = 0; }
```

Hope sets user to 1 means that user is authorized and nope means user is not authorized.

```solidity
function wish(address bit, address usr) internal view returns (bool) {
        return either(bit == usr, can[bit][usr] == 1);
    }
```
Check whether an address is authorized to modify another address's gem or dai balance.If bit == user means they're theselves so authorized or if bit has approved user so can is 1.

```solidity
// --- Data ---
    struct Ilk {
        uint256 Art;   // Total Normalised Debt     [wad]
        uint256 rate;  // Accumulated Rates         [ray]
        uint256 spot;  // Price with Safety Margin  [ray]
        uint256 line;  // Debt Ceiling              [rad]
        uint256 dust;  // Urn Debt Floor            [rad]
    }
```

Struct storing the collateral type data.

```solidity
struct Urn {
        uint256 ink;   // Locked Collateral  [wad]
        uint256 art;   // Normalised Debt    [wad]
    }
```

Struct storing  data for a specific Vault.

```solidity
mapping (bytes32 => Ilk)                       public ilks;
```

Mapping that stores an Ilk struct for each collateral type.Bytes32 is the id of the ilk.

```solidity
mapping (bytes32 => mapping (address => Urn )) public urns;
```

Collateral type mapped to the address wih the vault's data.Maybe you could have multpile urns to an ilk.Bytes32 is the id of the ilk then we've multiple addresses that point to multiple urns.

```solidity
mapping (bytes32 => mapping (address => uint)) public gem;  // [wad]
```

Collateral type mapped to the address wih the token kept as colleteral.This address is authenticated to call administrative function too.

```solidity
mapping (address => uint256)                   public dai;  // [rad]
```

Mapping that keeps track of how much DAI a vault has generated.It's a fixed point integer, with 45 decimal places.

```solidity
mapping (address => uint256)                   public sin;  // [rad]
```

Unbacked stablecoin (system debt, not belonging to any urn(specific vault).

```solidity
uint256 public debt;  // Total Dai Issued    [rad]
uint256 public vice;  // Total Unbacked Dai  [rad]
uint256 public Line;  // Total Debt Ceiling  [rad]
uint256 public live;  // Active Flag
```

debt: the sum of all dai (the total quantity of dai issued).
vice: the sum of all sin (the total quantity of system debt).
Line: the total debt ceiling for all collateral types.
live: cage flag

```solidity
constructor() public {
        wards[msg.sender] = 1;
        live = 1;
    }
```

Sets deployer to authenticate to use administrative function and and also indicates that system is running normally.

```solidity
function init(bytes32 ilk) external auth {
        require(ilks[ilk].rate == 0, "Vat/ilk-already-init");
        ilks[ilk].rate = 10 ** 27;
    }
```
Administrative function is setting up each ilk.

This function start stability fee collection for a particular collateral type and only authenticated user can call this.

```solidity
function file(bytes32 what, uint data) external auth {
        require(live == 1, "Vat/not-live");
        if (what == "Line") Line = data;
        else revert("Vat/file-unrecognized-param");
    }
```

Line is the total debt ceiling for all the collateral types.So this function sets the debt ceiling for collateral.If in bytes32, you pass in "Line", you can set the uint to Line else it'll revert.

```solidity
function file(bytes32 ilk, bytes32 what, uint data) external auth {
        require(live == 1, "Vat/not-live");
        if (what == "spot") ilks[ilk].spot = data;
        else if (what == "line") ilks[ilk].line = data;
        else if (what == "dust") ilks[ilk].dust = data;
        else revert("Vat/file-unrecognized-param");
    }
```

If bytes32 is spot, it sets the collateral price with safety margin, i.e. the maximum stablecoin allowed per unit of collateral for particular collateral.

If bytes32 is line, it sets the debt ceiling for a specific collateral type.

If bytes32 is dust,it sets the minimum possible debt of a Vault.

It is using bytes32 for string data because The EVM has a word-size of 32 bytes, so it is "optimized" for dealing with data in chunks of 32 bytes. (Compilers, such as Solidity, have to do more work and generate more bytecode when data isn't in chunks of 32 bytes, which effectively leads to higher gas cost.)

```solidity
function cage() external auth {
        live = 0;
    }
```

This function sets that the network is not acting normally.

```solidity
function slip(bytes32 ilk, address usr, int256 wad) external auth {
        gem[ilk][usr] = _add(gem[ilk][usr], wad);
    }
```

This function modifies the user's collateral balance.Adds the user balance with the token that is supplied(wad) as a collateral.Also might be subtracting the collateral because the data type is int.

```solidity
function flux(bytes32 ilk, address src, address dst, uint256 wad) external {
        require(wish(src, msg.sender), "Vat/not-allowed");
        gem[ilk][src] = _sub(gem[ilk][src], wad);
        gem[ilk][dst] = _add(gem[ilk][dst], wad);
    }
```

This function  transfer collateral between users.Requires an address to modify another address's gem or dai balance.Subtract the transferred collateral balance from the sender and adds to the destination.This is moving collateral between two vaults.

```solidity
function move(address src, address dst, uint256 rad) external {
        require(wish(src, msg.sender), "Vat/not-allowed");
        dai[src] = _sub(dai[src], rad);
        dai[dst] = _add(dai[dst], rad);
    }
```

This function transfer stablecoin between users.Requires an address to modify another address's gem or dai balance.Subtracts the DAI balance of source and adds it to the destination.They're moving arbitrarily collateral which can make system economically weak.

how does this function make sure this is all it need to update the stablecoin? There's no any calls to other contract as well.If you know the answer, send PR I'm willing to know.

The flux and move doesn't change the total circulating supply, it just change who owns it.

```solidity
function either(bool x, bool y) internal pure returns (bool z) {
        assembly{ z := or(x, y)}
    }
    function both(bool x, bool y) internal pure returns (bool z) {
        assembly{ z := and(x, y)}
    }
```

Function for implemeting add and or.

```solidity
 function frob(bytes32 i, address u, address v, address w, int dink, int dart) external {
        // system is live
        require(live == 1, "Vat/not-live");

        Urn memory urn = urns[i][u];
        Ilk memory ilk = ilks[i];
        // ilk has been initialised
        require(ilk.rate != 0, "Vat/ilk-not-init");

        urn.ink = _add(urn.ink, dink);
        urn.art = _add(urn.art, dart);
        ilk.Art = _add(ilk.Art, dart);

        int dtab = _mul(ilk.rate, dart);
        uint tab = _mul(ilk.rate, urn.art);
        debt     = _add(debt, dtab);

        // either debt has decreased, or debt ceilings are not exceeded
        require(either(dart <= 0, both(_mul(ilk.Art, ilk.rate) <= ilk.line, debt <= Line)), "Vat/ceiling-exceeded");
        // urn is either less risky than before, or it is safe
        require(either(both(dart <= 0, dink >= 0), tab <= _mul(urn.ink, ilk.spot)), "Vat/not-safe");

        // urn is either more safe, or the owner consents
        require(either(both(dart <= 0, dink >= 0), wish(u, msg.sender)), "Vat/not-allowed-u");
        // collateral src consents
        require(either(dink <= 0, wish(v, msg.sender)), "Vat/not-allowed-v");
        // debt dst consents
        require(either(dart >= 0, wish(w, msg.sender)), "Vat/not-allowed-w");

        // urn has no debt, or a non-dusty amount
        require(either(urn.art == 0, tab >= ilk.dust), "Vat/dust");

        gem[i][v] = _sub(gem[i][v], dink);
        dai[w]    = _add(dai[w],    dtab);

        urns[i][u] = urn;
        ilks[i]    = ilk;
    }
```

This function modifies the vault.Adding debt, removing debt, adding collateral and removing collateral are all being dumped into the same function.The reason to pull all in one function is that is makes the function versatile.Let's you want to add collateral and mint dai, you could do that.Let's say you need to put in more collateral because of volatility not in your favor without minting dai, you could do that.Let's say you want to take down the amount of dai that you've got etc.

```solidity
require(live == 1, "Vat/not-live");
```

Requires the system to be normal.

```solidity
Urn memory urn = urns[i][u];
Ilk memory ilk = ilks[i];
```

Stores the collateral type mapped to the address wih the vault's data of that bytes32 of that address u being passed as the parameter in a urn variable.

Stores the collateral data of that bytes32 into ilk.

```solidity
require(ilk.rate != 0, "Vat/ilk-not-init");
```

Requires the ilk to be initialized.

```solidity
urn.ink = _add(urn.ink, dink);
urn.art = _add(urn.art, dart);
ilk.Art = _add(ilk.Art, dart);
```

Sets the ink(locked collateral) by adding it with the dink(change in collateral) being passed.

Sets the art(normalized outstanding stablecoin debt) by adding it with the dart(change in debt).

Sets the Art(total normalized stablecoin debt) by adding it with the dart.

This simply means if we change our collateral amount, out stablecoin debt will also change.

```solidity
int dtab = _mul(ilk.rate, dart);
uint tab = _mul(ilk.rate, urn.art);
debt     = _add(debt, dtab);
```

Calculates dtab to be the multiplication of the initialized accumulated rates with the dart(change in debt).

Calculates tab to be the multiplication of initialized accumulated rates with the art(normalized outstanding stablecoin debt).

Total debt will be the addition of these two.

```solidity
require(either(dart <= 0, both(_mul(ilk.Art, ilk.rate) <= ilk.line, debt <= Line)), "Vat/ceiling-exceeded");
```
Whatever you're doing with the vault, it should either decrease the debt or decrease the debt but not go beyond the ceiling.

First multiplication of ilk.Art and ilk.rate should be less than equals to ilk.line which means multiplication of total normalized stablecoin debt and accumulated rates should be less than or equals to debt ceiling.

And also debt should be less that or equals to the debt ceiling for all collateral types.

Or change in debt should be less than or equals to 0.

```solidity
require(either(both(dart <= 0, dink >= 0), tab <= _mul(urn.ink, ilk.spot)), "Vat/not-safe");
```

We're checking like a concept of safeness.

Either change in debt should be less than or equals to 0 or change in collateral should be more than or equals to 0.

Or calculated tab above should be less than or equals to the multiplication of collateral balance and collateral price with safety margin.

This ensures that the vault is safe or less risky than before.

```solidity
require(either(both(dart <= 0, dink >= 0), wish(u, msg.sender)), "Vat/not-allowed-u");
```
More unsafe is more dai to the same amount of collateral which is some sort of measure of collateral ratio.Someone could repay our debt and make it less risky without any sort of approvals but if they're borrowing and making in more risky then they need approval.

Change in debt should be <=0 and change in collateral should be >=0.

Or address should be allowed to modify the vault.

```solidity
require(either(dink <= 0, wish(v, msg.sender)), "Vat/not-allowed-v");
```
v is the one who stands if the amount of collateral goes down.

Either change in collateral should be <=0 or address v should be allowed to modify vault.


```solidity
require(either(dart >= 0, wish(w, msg.sender)), "Vat/not-allowed-w");
```
You can't put in massive negative dart for kicks but you're going to need wish between w and the sender.

w is the one who stands to lose if the amount of dai goes down.

Either change in debt should be >=0 or address w should be allowed to modify vault.

```solidity
require(either(urn.art == 0, tab >= ilk.dust), "Vat/dust");
```
It's just checking that the debt is wither 0 or it's more than there's like sort of small dust amount left in the urn.

Non-dusty amount because it's not worth it for anyone to liquidate so no one's ever going to bother for it.It just ends of being kind of like dead.

Either normalized outstanding stablecoin debt should be 0 or tab calculated above should be greater then or equals to the minimum possible debt of the vault.

```solidity
gem[i][v] = _sub(gem[i][v], dink);
dai[w]    = _add(dai[w],    dtab);
```

Sets collateral amount of  the collateral type of the address v by subtracting it with change in collateral.

Set dai token of the address v by adding it with the dtab calculated above.

```solidity
urns[i][u] = urn;
ilks[i]    = ilk;
```

Stores the modified vault of that address u in the mapping.

Sets the collateral type with the collateral type that's being modified here.

```solidity
function fork(bytes32 ilk, address src, address dst, int dink, int dart) external {
        Urn storage u = urns[ilk][src];
        Urn storage v = urns[ilk][dst];
        Ilk storage i = ilks[ilk];

        u.ink = _sub(u.ink, dink);
        u.art = _sub(u.art, dart);
        v.ink = _add(v.ink, dink);
        v.art = _add(v.art, dart);

        uint utab = _mul(u.art, i.rate);
        uint vtab = _mul(v.art, i.rate);

        // both sides consent
        require(both(wish(src, msg.sender), wish(dst, msg.sender)), "Vat/not-allowed");

        // both sides safe
        require(utab <= _mul(u.ink, i.spot), "Vat/not-safe-src");
        require(vtab <= _mul(v.ink, i.spot), "Vat/not-safe-dst");

        // both sides non-dusty
        require(either(utab >= i.dust, u.art == 0), "Vat/dust-src");
        require(either(vtab >= i.dust, v.art == 0), "Vat/dust-dst");
    }
```

Fork is used to split the vault - binary approval or splitting/merging vaults.

```solidity
Urn storage u = urns[ilk][src];
Urn storage v = urns[ilk][dst];
Ilk storage i = ilks[ilk];
```

We load u, v and i  and we're doing it on storage.Why they've used storage?  If you know send the PR.

Maybe it's like the inefficiency of bringing such large structs out of storage because each slot of these in an urn would be it's own like sload.Maybe that cost outweighs the cost of reading and doing writing to individuals.

```solidity
u.ink = _sub(u.ink, dink);
u.art = _sub(u.art, dart);
v.ink = _add(v.ink, dink);
v.art = _add(v.art, dart);

uint utab = _mul(u.art, i.rate);
uint vtab = _mul(v.art, i.rate);
```

Calculating the respective variable.

```solidity
require(both(wish(src, msg.sender), wish(dst, msg.sender)), "Vat/not-allowed");
```

Intuitively funds are following from the source to the destination address but since we're dealing with the ints that can be counter-intuitive.You could be moving funds from the destination to the source because you subtract from the source and add to the destination but if you're putting in negative numbers then you'll be subtracting a negative number by adding a negative number i.e subtracting.

They kind of assumed that both sides.The main idea is either split into two or to merge all funds into one so then both sides will be taking on some kind of approval.



