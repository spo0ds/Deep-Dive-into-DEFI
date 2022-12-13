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

This mapping allows to store which address are authorized to use the vault maybe for auction.

```solidity
function hope(address usr) external { can[msg.sender][usr] = 1; }
function nope(address usr) external { can[msg.sender][usr] = 0; }
```

Hope allows the caller to auction the particular address vault and nope denies it.

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

Collateral type mapped to the address wih the vault's data.

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

unbacked stablecoin (system debt, not belonging to any urn(specific vault).

```solidity
uint256 public debt;  // Total Dai Issued    [rad]
uint256 public vice;  // Total Unbacked Dai  [rad]
uint256 public Line;  // Total Debt Ceiling  [rad]
uint256 public live;  // Active Flag
```

debt: the sum of all dai (the total quantity of dai issued).
vice: the sum of all sin (the total quantity of system debt).
ilk.Art: the sum of all art in the urns for that ilk.
debt: is vice plus the sum of ilk.Art * ilk.rate across all ilk's.

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

This function start stability fee collection for a particular collateral type and only authenticated user can call this.

```solidity
function file(bytes32 what, uint data) external auth {
        require(live == 1, "Vat/not-live");
        if (what == "Line") Line = data;
        else revert("Vat/file-unrecognized-param");
    }
```

Line is the total debt ceiling for all the collateral types.So this function sets the debt ceiling for collateral.

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

This function modifies the user's collateral balance.Adds the user balance with the token that is supplied(wad).

```solidity
function flux(bytes32 ilk, address src, address dst, uint256 wad) external {
        require(wish(src, msg.sender), "Vat/not-allowed");
        gem[ilk][src] = _sub(gem[ilk][src], wad);
        gem[ilk][dst] = _add(gem[ilk][dst], wad);
    }
```

This function  transfer collateral between users.Requires an address to modify another address's gem or dai balance.Subtract the transferred collateral balance from the sender and adds to the destination.

```solidity
function move(address src, address dst, uint256 rad) external {
        require(wish(src, msg.sender), "Vat/not-allowed");
        dai[src] = _sub(dai[src], rad);
        dai[dst] = _add(dai[dst], rad);
    }
```

This function transfer stablecoin between users.Requires an address to modify another address's gem or dai balance.Subtracts the DAI balance of source and adds it to the destination.

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

This function modifies the vault.

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

First multiplication of ilk.Art and ilk.rate should be less than equals to ilk.line which means multiplication of total normalized stablecoin debt and accumulated rates should be less than or equals to debt ceiling.

And also debt should be less that or equals to the debt ceiling for all collateral types.

Or change in debt should be less than or equals to 0.

```solidity
require(either(both(dart <= 0, dink >= 0), tab <= _mul(urn.ink, ilk.spot)), "Vat/not-safe");
```

Either change in debt should be less than or equals to 0 or change in collateral should be more than or equals to 0.

Or calculated tab above should be less than or equals to the multiplication of collateral balance and collateral price with safety margin.

This ensures that the vault is safe or less risky than before.

```solidity
require(either(both(dart <= 0, dink >= 0), wish(u, msg.sender)), "Vat/not-allowed-u");
```

Change in debt should be <=0 and change in collateral should be >=0.

Or address should be allowed to modify the vault.

```solidity
require(either(dink <= 0, wish(v, msg.sender)), "Vat/not-allowed-v");
```

Either change in collateral should be <=0 or address v should be allowed to modify vault.


```solidity
require(either(dart >= 0, wish(w, msg.sender)), "Vat/not-allowed-w");
```

Either change in debt should be >=0 or address w should be allowed to modify vault.

```solidity
require(either(urn.art == 0, tab >= ilk.dust), "Vat/dust");
```

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

