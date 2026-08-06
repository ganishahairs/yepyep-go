# Why Base?

Base offers low fees, fast transactions, and a strong focus on developers.  

I'm starting this repo to track my learning journey on this network.

### Useful Resources

- Base Documentation: https://docs.base.org  
- Base Bridge: https://bridge.base.org  
- Base Explorer: https://base.blockscout.com  

Collecting useful links as I go.

### Low Fees on Base

One of the biggest advantages of Base is the very low transaction cost.  

This makes it ideal for testing ideas without spending much.

### Adding setValue Function

```solidity
function setValue(uint256 newValue) public {
    value = newValue;
}

### Get Value Function

```solidity
function getValue() public view returns (uint256) {
    return value;
}

### ValueUpdated Event

```solidity
event ValueUpdated(address indexed sender, uint256 newValue);

function setValue(uint256 newValue) public {
    value = newValue;
    emit ValueUpdated(msg.sender, newValue);
}

### Constructor with Initial Value

```solidity
constructor(uint256 initialValue) {
    value = initialValue;
}

### Only Owner Modifier

```solidity
modifier onlyOwner() {
    require(msg.sender == owner, "Not the owner");
    _;
}

### OwnershipTransferred Event

```solidity
event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

function transferOwnership(address newOwner) public onlyOwner {
    emit OwnershipTransferred(owner, newOwner);
    owner = newOwner;
}

### Get Balance Function

```solidity
function getBalance(address user) public view returns (uint256) {
    return balances[user];
}

### Withdraw Event

```solidity
event Withdraw(address indexed user, uint256 amount);

function withdraw(uint256 amount) public {
    require(balances[msg.sender] >= amount, "Insufficient balance");
    balances[msg.sender] -= amount;
    payable(msg.sender).transfer(amount);
    emit Withdraw(msg.sender, amount);
}

### Understanding require()

`require()` is used to validate conditions.  

If the condition is false, the transaction reverts and any changes are undone.

### Using the Modifier

```solidity
function setMessage(string memory newMessage) public onlyOwner {
    message = newMessage;
}


### Understanding msg.value

`msg.value` represents the amount of ETH (in wei) sent with the transaction.

It is only available in payable functions.

### Receiving ETH

To accept ETH, a function must be marked as `payable`.  

Without it, the transaction will revert if ETH is sent.

### Receiving ETH

To accept ETH, a function must be marked as `payable`.  

Without it, the transaction will revert if ETH is sent.

### Payable Fallback

```solidity
fallback() external payable {
    balances[msg.sender] += msg.value;
}

### address vs address payable

- `address` → cannot receive ETH directly  
- `address payable` → can receive ETH  

You need to cast when necessary:
```solidity
payable(user).transfer(1 ether);

### Using a Struct

```solidity
mapping(address => User) public users;

function createUser() public {
    users[msg.sender] = User(msg.sender, 0, true);
}

### Array Length

```solidity
function getUserCount() public view returns (uint256) {
    return userList.length;
}

### Be Careful with Loops

Loops can consume a lot of gas if the array is large.  

Avoid unbounded loops when possible.

### Ternary Operator

```solidity
function getStatus() public view returns (string memory) {
    return msg.sender == owner ? "Owner" : "Not owner";
}

### Setting Enum Values

```solidity
function setStatus(Status newStatus) public {
    currentStatus = newStatus;
}

### Why Interfaces Are Useful

Interfaces allow contracts to interact with other contracts without knowing their full code.

They are essential for composability.

### Basic ERC20 Functions

The most important functions are:

- `balanceOf`
- `transfer`
- `approve`
- `transferFrom`
- `allowance`

### Transfer Function

```solidity
function transfer(address to, uint256 amount) public returns (bool) {
    require(balanceOf[msg.sender] >= amount, "Insufficient balance");
    balanceOf[msg.sender] -= amount;
    balanceOf[to] += amount;
    return true;
}

### burnFrom Function

```solidity
function burnFrom(address from, uint256 amount) public {
    require(allowance[from][msg.sender] >= amount, "Insufficient allowance");
    require(balanceOf[from] >= amount, "Insufficient balance");

    allowance[from][msg.sender] -= amount;
    balanceOf[from] -= amount;
    totalSupply -= amount;

    emit Transfer(from, address(0), amount);
}

### Complete ERC20 Skeleton

Core components so far:

- name, symbol, decimals  
- totalSupply  
- balanceOf  
- allowance  
- transfer / transferFrom  
- approve  
- mint / burn  


### Basic ERC721 Structure

```solidity
mapping(uint256 => address) public ownerOf;
mapping(address => uint256) public balanceOf;
uint256 public nextTokenId;

### NFT Transfer Event

```solidity
event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);

function transfer(address to, uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    ownerOf[tokenId] = to;
    balanceOf[msg.sender] -= 1;
    balanceOf[to] += 1;
    emit Transfer(msg.sender, to, tokenId);
}

### Approval Event

```solidity
event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId);

function approve(address to, uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    getApproved[tokenId] = to;
    emit Approval(msg.sender, to, tokenId);
}

### Burn Event

```solidity
function burn(uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    address owner = ownerOf[tokenId];
    delete ownerOf[tokenId];
    balanceOf[owner] -= 1;
    emit Transfer(owner, address(0), tokenId);
}

### Set Mint Price

```solidity
function setMintPrice(uint256 newPrice) public onlyOwner {
    mintPrice = newPrice;
}

### Protecting Mint with Pause

```solidity
function mint() public payable whenNotPaused {
    require(nextTokenId < maxSupply, "Max supply reached");
    require(msg.value >= mintPrice, "Insufficient payment");
    ownerOf[nextTokenId] = msg.sender;
    balanceOf[msg.sender] += 1;
    nextTokenId++;
}

### Total Minted View

```solidity
function totalMinted() public view returns (uint256) {
    return nextTokenId;
}

### Max per Transaction

```solidity
uint256 public maxPerTx = 5;

function batchMint(uint256 quantity) public payable {
    require(quantity <= maxPerTx, "Exceeds max per transaction");
    // rest of the mint logic
}

### Minted At Timestamp

```solidity
mapping(uint256 => uint256) public mintedAt;

function mint() public payable {
    // existing checks...
    mintedAt[nextTokenId] = block.timestamp;
    ownerOf[nextTokenId] = msg.sender;
    balanceOf[msg.sender] += 1;
    nextTokenId++;
}

### Get Full Token Info

```solidity
function getFullTokenInfo(uint256 tokenId) public view returns (
    address currentOwner,
    address minter,
    uint256 mintedTime
) {
    return (
        ownerOf[tokenId],
        originalMinter[tokenId],
        mintedAt[tokenId]
    );
}

### Level Up Function

```solidity
function levelUp(uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    tokenLevel[tokenId] += 1;
}

### Leveling Up with Experience

```solidity
uint256 public expPerLevel = 100;

function levelUp(uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    require(experience[tokenId] >= expPerLevel * tokenLevel[tokenId], "Not enough experience");
    require(tokenLevel[tokenId] < maxLevel, "Max level reached");
    
    tokenLevel[tokenId] += 1;
}

### Building tokenURI

```solidity
function tokenURI(uint256 tokenId) public view returns (string memory) {
    require(ownerOf[tokenId] != address(0), "Token does not exist");
    return string(abi.encodePacked(baseURI, Strings.toString(tokenId), ".json"));
}

### Why supportsInterface Matters

Many marketplaces and tools check `supportsInterface` to know if a contract is a valid ERC721.

Implementing it improves compatibility.

### NFT Collection Constructor

```solidity
constructor(string memory _name, string memory _symbol) {
    name = _name;
    symbol = _symbol;
    owner = msg.sender;
}

### Safer balanceOf

```solidity
function balanceOf(address owner) public view returns (uint256) {
    require(owner != address(0), "Zero address");
    return balanceOf[owner];
}

### Lock Token Function

```solidity
function lockToken(uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    tokenStatus[tokenId] = TokenStatus.Locked;
}

### Status Change Event

```solidity
event StatusChanged(uint256 indexed tokenId, TokenStatus newStatus);

function lockToken(uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    tokenStatus[tokenId] = TokenStatus.Locked;
    emit StatusChanged(tokenId, TokenStatus.Locked);
}

### Unstake Function

```solidity
function unstake(uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    require(isStaked[tokenId], "Not staked");
    
    isStaked[tokenId] = false;
    tokenStatus[tokenId] = TokenStatus.Normal;
}

### Claim Reward Function

```solidity
mapping(uint256 => uint256) public pendingRewards;

function claimReward(uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    uint256 reward = calculateReward(tokenId);
    require(reward > 0, "No rewards");
    
    pendingRewards[tokenId] = 0;
    stakedAt[tokenId] = block.timestamp; // reset timer
    payable(msg.sender).transfer(reward);
}

### Set Reward Per Day

```solidity
function setRewardPerDay(uint256 newReward) public onlyOwner {
    rewardPerDay = newReward;
}

### Set Minimum Stake Time

```solidity
function setMinStakeTime(uint256 newMinTime) public onlyOwner {
    minStakeTime = newMinTime;
}

### Set Early Unstake Penalty

```solidity
function setEarlyUnstakePenalty(uint256 newPenalty) public onlyOwner {
    require(newPenalty <= 50, "Penalty too high");
    earlyUnstakePenalty = newPenalty;
}

### Update Staked Count on Unstake

```solidity
function unstake(uint256 tokenId) public {
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    require(isStaked[tokenId], "Not staked");

    isStaked[tokenId] = false;
    stakedCount[msg.sender] -= 1;
    tokenStatus[tokenId] = TokenStatus.Normal;
}

### Tracking Staked Tokens per User

```solidity
function _addStakedToken(address user, uint256 tokenId) internal {
    userStakedTokens[user].push(tokenId);
}

function _removeStakedToken(address user, uint256 tokenId) internal {
    // simple removal logic (swap and pop is common)
}

### Update totalStaked on Unstake

```solidity
function unstake(uint256 tokenId) public {
    // existing checks...
    isStaked[tokenId] = false;
    totalStaked -= 1;
    stakedCount[msg.sender] -= 1;
}

### Get Total Rewards Distributed

```solidity
function getTotalRewardsDistributed() public view returns (uint256) {
    return totalRewardsDistributed;
}

### Toggle Staking Pause

```solidity
function toggleStakingPause() public onlyOwner {
    stakingPaused = !stakingPaused;
}

### Protecting Withdraw

```solidity
function claimReward(uint256 tokenId) public noReentrant {
    // claim logic
}

### Learning Milestones

So far you have practiced:

- Structs, mappings and enums  
- Events  
- Access control  
- Payable functions  
- NFT standards  
- Staking mechanics  
- Basic security patterns  

### Breeding Cooldown

```solidity
mapping(uint256 => uint256) public lastBred;

uint256 public breedingCooldown = 7 days;

function breed(uint256 tokenId1, uint256 tokenId2) public {
    require(block.timestamp >= lastBred[tokenId1] + breedingCooldown, "Cooldown active");
    require(block.timestamp >= lastBred[tokenId2] + breedingCooldown, "Cooldown active");
    
    lastBred[tokenId1] = block.timestamp;
    lastBred[tokenId2] = block.timestamp;
    // rest of breeding logic
}

### Set Breeding Cost

```solidity
function setBreedingCost(uint256 newCost) public onlyOwner {
    breedingCost = newCost;
}

### Generating Attributes on Mint

```solidity
function mint() public payable {
    // existing checks...
    
    tokenAttributes[nextTokenId] = Attributes({
        strength: uint8((block.timestamp + nextTokenId) % 10) + 1,
        agility: uint8((block.timestamp + nextTokenId * 2) % 10) + 1,
        intelligence: uint8((block.timestamp + nextTokenId * 3) % 10) + 1
    });
    
    ownerOf[nextTokenId] = msg.sender;
    balanceOf[msg.sender] += 1;
    nextTokenId++;
}

### Attribute Cap

```solidity
uint8 public maxAttributeValue = 50;

function levelUp(uint256 tokenId) public {
    // existing logic...
    
    if (tokenAttributes[tokenId].strength < maxAttributeValue) {
        tokenAttributes[tokenId].strength += 1;
    }
    // same for other attributes
}

### Rarity on Mint Event

```solidity
event TokenMinted(address indexed to, uint256 indexed tokenId, string rarity);

function mint() public payable {
    // existing mint logic...
    
    string memory rarity = getRarity(nextTokenId - 1);
    emit TokenMinted(msg.sender, nextTokenId - 1, rarity);
}
### Calculating Player Score from NFTs

```solidity
function calculatePlayerScore(address player) public view returns (uint256) {
    uint256 score = 0;
    // In a full implementation you would loop through the player's tokens
    // score += getPowerScore(tokenId) for each token
    return score;
}

### Cancel Listing

```solidity
function cancelListing(uint256 tokenId) public {
    require(listings[tokenId].seller == msg.sender, "Not the seller");
    listings[tokenId].active = false;
}

### Set Marketplace Fee

```solidity
function setMarketplaceFee(uint256 newFee) public onlyOwner {
    require(newFee <= 10, "Fee too high");
    marketplaceFee = newFee;
}
### Cancel Listing Event

```solidity
event ListingCancelled(uint256 indexed tokenId, address indexed seller);

function cancelListing(uint256 tokenId) public {
    require(listings[tokenId].seller == msg.sender, "Not the seller");
    listings[tokenId].active = false;
    emit ListingCancelled(tokenId, msg.sender);
}
### Accept Offer

```solidity
function acceptOffer(uint256 tokenId) public {
    Offer memory offer = offers[tokenId];
    require(ownerOf[tokenId] == msg.sender, "Not the owner");
    require(offer.active, "No active offer");

    offers[tokenId].active = false;

    // Transfer NFT to buyer
    ownerOf[tokenId] = offer.buyer;
    balanceOf[msg.sender] -= 1;
    balanceOf[offer.buyer] += 1;

    payable(msg.sender).transfer(offer.price);
}

### Get Offer Info

```solidity
function getOfferInfo(uint256 tokenId) public view returns (
    address buyer,
    uint256 price,
    bool active
) {
    Offer memory offer = offers[tokenId];
    return (offer.buyer, offer.price, offer.active);
}

### Applying Royalty on Sale

```solidity
function buyToken(uint256 tokenId) public payable {
    // existing checks...
    
    uint256 royaltyAmount = (item.price * royaltyPercentage) / 100;
    uint256 fee = (item.price * marketplaceFee) / 100;
    uint256 sellerAmount = item.price - royaltyAmount - fee;

    payable(royaltyReceiver).transfer(royaltyAmount);
    payable(item.seller).transfer(sellerAmount);
}
### supportsInterface for Royalty

```solidity
function supportsInterface(bytes4 interfaceId) public pure returns (bool) {
    return
        interfaceId == 0x80ac58cd || // ERC721
        interfaceId == 0x5b5e139f || // ERC721Metadata
        interfaceId == 0x2a55205a || // ERC2981
        interfaceId == 0x01ffc9a7;    // ERC165
}

### Code Organization Tips

As the contract grows it is a good idea to organize the code into sections:

- State variables  
- Events  
- Modifiers  
- Constructor  
- Minting functions  
- Staking functions  
- Marketplace functions  
- Admin functions  
- View functions  

### Minting with TokenURI

```solidity
function safeMint(address to, string memory uri) public onlyOwner {
    uint256 tokenId = nextTokenId;
    nextTokenId++;
    _safeMint(to, tokenId);
    _setTokenURI(tokenId, uri);
}

### Pausing with OpenZeppelin

```solidity
function pause() public onlyOwner {
    _pause();
}

function unpause() public onlyOwner {
    _unpause();
}

### Updating Royalty Info

```solidity
function setRoyalty(address receiver, uint96 feeNumerator) public onlyOwner {
    _setDefaultRoyalty(receiver, feeNumerator);
}

### Mint Function with OpenZeppelin

```solidity
function mint(string memory uri) public payable whenNotPaused nonReentrant {
    require(nextTokenId < maxSupply, "Max supply reached");
    require(msg.value >= mintPrice, "Insufficient payment");

    uint256 tokenId = nextTokenId;
    nextTokenId++;

    _safeMint(msg.sender, tokenId);
    _setTokenURI(tokenId, uri);
}

### Max Per Wallet

```solidity
uint256 public maxPerWallet = 5;
mapping(address => uint256) public mintedPerWallet;

function mint(string memory uri) public payable whenNotPaused nonReentrant {
    require(mintedPerWallet[msg.sender] < maxPerWallet, "Max per wallet reached");
    // rest of mint logic
    mintedPerWallet[msg.sender] += 1;
}

### Managing the Allowlist

```solidity
function setAllowlist(address user, bool status) public onlyOwner {
    allowlist[user] = status;
}

function setAllowlistEnabled(bool status) public onlyOwner {
    allowlistEnabled = status;
}

### Verifying Merkle Proof

```solidity
function mint(bytes32[] calldata proof, string memory uri) public payable {
    bytes32 leaf = keccak256(abi.encodePacked(msg.sender));
    require(MerkleProof.verify(proof, merkleRoot, leaf), "Invalid proof");
    // rest of mint logic
}

### Reveal Function

```solidity
function reveal(string memory newBaseURI) public onlyOwner {
    revealed = true;
    // optionally update baseURI here
}

### Is Revealed View

```solidity
function isRevealed() public view returns (bool) {
    return revealed;
}

### Useful Enumerable Functions

With `ERC721Enumerable` you get:

- `totalSupply()`  
- `tokenByIndex(uint256 index)`  
- `tokenOfOwnerByIndex(address owner, uint256 index)`  

These make frontend development much easier.

### Limiting Batch Size

```solidity
uint256 public maxBatchSize = 20;

function batchMint(address to, string[] memory uris) public onlyOwner {
    require(uris.length <= maxBatchSize, "Batch too large");
    // rest of the logic
}

### Phase-Based Mint Logic

```solidity
function mint(string memory uri, bytes32[] calldata proof) public payable {
    if (currentPhase == MintPhase.Closed) revert("Mint is closed");
    
    if (currentPhase == MintPhase.Allowlist) {
        // verify merkle proof
    } else if (currentPhase == MintPhase.Public) {
        require(msg.value >= mintPrice, "Insufficient payment");
    } else if (currentPhase == MintPhase.Free) {
        // free mint logic
    }
    
    // shared mint logic
}
### Reserved Supply View

```solidity
function getReservedInfo() public view returns (uint256 reserved, uint256 minted, uint256 remaining) {
    return (reservedSupply, reservedMinted, reservedSupply - reservedMinted);
}
### Reserved Supply for Team

```solidity
uint256 public reservedSupply = 50;
uint256 public reservedMinted = 0;

function teamMint(address to, string memory uri) public onlyOwner {
    require(reservedMinted < reservedSupply, "Reserved supply exhausted");
    require(nextTokenId < maxSupply, "Max supply reached");

    uint256 tokenId = nextTokenId;
    nextTokenId++;
    reservedMinted++;

    _safeMint(to, tokenId);
    _setTokenURI(tokenId, uri);
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract LevelStakingNFT is ERC721URIStorage, Ownable, ReentrancyGuard {
    uint256 public nextTokenId;
    uint256 public maxSupply = 500;

    mapping(uint256 => uint256) public level;
    mapping(uint256 => bool) public isStaked;
    mapping(uint256 => uint256) public stakedAt;

    constructor() ERC721("Level Staking NFT", "LSNFT") Ownable(msg.sender) {}

    function mint(string memory uri) external onlyOwner {
        require(nextTokenId < maxSupply, "Max supply reached");
        uint256 tokenId = nextTokenId++;
        level[tokenId] = 1;
        _safeMint(msg.sender, tokenId);
        _setTokenURI(tokenId, uri);
    }

    function stake(uint256 tokenId) external {
        require(ownerOf(tokenId) == msg.sender, "Not owner");
        require(!isStaked[tokenId], "Already staked");
        isStaked[tokenId] = true;
        stakedAt[tokenId] = block.timestamp;
    }

    function unstake(uint256 tokenId) external {
        require(ownerOf(tokenId) == msg.sender, "Not owner");
        require(isStaked[tokenId], "Not staked");
        isStaked[tokenId] = false;
    }

    function levelUp(uint256 tokenId) external {
        require(ownerOf(tokenId) == msg.sender, "Not owner");
        require(!isStaked[tokenId], "Token is staked");
        level[tokenId]++;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract DutchAuctionNFT is ERC721URIStorage, Ownable, ReentrancyGuard {
    uint256 public nextTokenId;
    uint256 public maxSupply = 500;

    uint256 public startPrice = 0.2 ether;
    uint256 public endPrice = 0.02 ether;
    uint256 public duration = 24 hours;
    uint256 public startTime;

    bool public auctionStarted;

    constructor() ERC721("Dutch Auction NFT", "DUTCH") Ownable(msg.sender) {}

    function startAuction() external onlyOwner {
        require(!auctionStarted, "Already started");
        auctionStarted = true;
        startTime = block.timestamp;
    }

    function getCurrentPrice() public view returns (uint256) {
        if (!auctionStarted) return startPrice;
        if (block.timestamp >= startTime + duration) return endPrice;

        uint256 elapsed = block.timestamp - startTime;
        uint256 priceDrop = ((startPrice - endPrice) * elapsed) / duration;
        return startPrice - priceDrop;
    }

    function mint(string memory uri) external payable nonReentrant {
        require(auctionStarted, "Auction not started");
        require(nextTokenId < maxSupply, "Max supply reached");

        uint256 price = getCurrentPrice();
        require(msg.value >= price, "Insufficient payment");

        uint256 tokenId = nextTokenId++;
        _safeMint(msg.sender, tokenId);
        _setTokenURI(tokenId, uri);

        // Refund excess payment
        if (msg.value > price) {
            payable(msg.sender).transfer(msg.value - price);
        }
    }

    function withdraw() external onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract ERC20PaymentNFT is ERC721URIStorage, Ownable, ReentrancyGuard {
    uint256 public nextTokenId;
    uint256 public maxSupply = 1000;
    uint256 public mintPrice = 100 * 10**18; // 100 tokens
    IERC20 public paymentToken;

    constructor(address _paymentToken) ERC721("ERC20 Payment NFT", "PAYNFT") Ownable(msg.sender) {
        paymentToken = IERC20(_paymentToken);
    }

    function mint(string memory uri) external nonReentrant {
        require(nextTokenId < maxSupply, "Max supply reached");

        bool success = paymentToken.transferFrom(msg.sender, address(this), mintPrice);
        require(success, "Payment failed");

        uint256 tokenId = nextTokenId++;
        _safeMint(msg.sender, tokenId);
        _setTokenURI(tokenId, uri);
    }

    function setMintPrice(uint256 newPrice) external onlyOwner {
        mintPrice = newPrice;
    }

    function withdrawTokens() external onlyOwner {
        uint256 balance = paymentToken.balanceOf(address(this));
        paymentToken.transfer(owner(), balance);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract MultiSeriesNFT is ERC721URIStorage, Ownable, ReentrancyGuard {
    uint256 public nextTokenId;

    struct Series {
        string name;
        uint256 maxSupply;
        uint256 minted;
        uint256 mintPrice;
        bool active;
    }

    mapping(uint256 => Series) public series;
    mapping(uint256 => uint256) public tokenSeries; // tokenId => seriesId
    uint256 public seriesCount;

    constructor() ERC721("Multi Series NFT", "SERIES") Ownable(msg.sender) {}

    function createSeries(
        string memory name,
        uint256 maxSupply,
        uint256 mintPrice
    ) external onlyOwner {
        series[seriesCount] = Series(name, maxSupply, 0, mintPrice, true);
        seriesCount++;
    }

    function mint(uint256 seriesId, string memory uri) external payable nonReentrant {
        Series storage s = series[seriesId];
        require(s.active, "Series not active");
        require(s.minted < s.maxSupply, "Series sold out");
        require(msg.value >= s.mintPrice, "Insufficient payment");

        uint256 tokenId = nextTokenId++;
        s.minted++;
        tokenSeries[tokenId] = seriesId;

        _safeMint(msg.sender, tokenId);
        _setTokenURI(tokenId, uri);
    }

    function setSeriesActive(uint256 seriesId, bool status) external onlyOwner {
        series[seriesId].active = status;
    }

    function getSeriesInfo(uint256 seriesId) external view returns (
        string memory name,
        uint256 maxSupply,
        uint256 minted,
        uint256 mintPrice,
        bool active
    ) {
        Series memory s = series[seriesId];
        return (s.name, s.maxSupply, s.minted, s.mintPrice, s.active);
    }

    function withdraw() external onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract RoleBasedNFT is ERC721URIStorage, AccessControl, ReentrancyGuard {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant MANAGER_ROLE = keccak256("MANAGER_ROLE");

    uint256 public nextTokenId;
    uint256 public maxSupply = 1000;
    uint256 public mintPrice = 0.02 ether;

    constructor() ERC721("Role Based NFT", "ROLE") {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
        _grantRole(MANAGER_ROLE, msg.sender);
    }

    function mint(address to, string memory uri) external onlyRole(MINTER_ROLE) {
        require(nextTokenId < maxSupply, "Max supply reached");
        uint256 tokenId = nextTokenId++;
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }

    function publicMint(string memory uri) external payable nonReentrant {
        require(nextTokenId < maxSupply, "Max supply reached");
        require(msg.value >= mintPrice, "Insufficient payment");

        uint256 tokenId = nextTokenId++;
        _safeMint(msg.sender, tokenId);
        _setTokenURI(tokenId, uri);
    }

    function setMintPrice(uint256 newPrice) external onlyRole(MANAGER_ROLE) {
        mintPrice = newPrice;
    }

    function withdraw() external onlyRole(DEFAULT_ADMIN_ROLE) {
        payable(msg.sender).transfer(address(this).balance);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC721URIStorage, AccessControl)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract GuildToken is ERC20, Ownable {
    constructor(address initialOwner) 
        ERC20("Guild Token", "GUILD") 
        Ownable(initialOwner) 
    {
        _mint(initialOwner, 1_000_000 * 10 ** decimals());
    }

    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

contract GuildStaking {
    using SafeERC20 for IERC20;

    IERC20 public immutable stakingToken;
    IERC20 public immutable rewardToken;

    mapping(address => uint256) public stakedBalance;
    mapping(address => uint256) public rewardDebt;
    uint256 public totalStaked;
    uint256 public rewardPerTokenStored;
    uint256 public lastUpdateTime;
    uint256 public rewardRate; // tokens por segundo

    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount);
    event RewardClaimed(address indexed user, uint256 amount);

    constructor(address _stakingToken, address _rewardToken, uint256 _rewardRate) {
        stakingToken = IERC20(_stakingToken);
        rewardToken = IERC20(_rewardToken);
        rewardRate = _rewardRate;
        lastUpdateTime = block.timestamp;
    }

    function updateReward(address account) internal {
        if (totalStaked > 0) {
            rewardPerTokenStored += ((block.timestamp - lastUpdateTime) * rewardRate * 1e18) / totalStaked;
        }
        lastUpdateTime = block.timestamp;

        if (account != address(0)) {
            uint256 earned = (stakedBalance[account] * (rewardPerTokenStored - rewardDebt[account])) / 1e18;
            rewardDebt[account] = rewardPerTokenStored;
            if (earned > 0) {
                rewardToken.safeTransfer(account, earned);
                emit RewardClaimed(account, earned);
            }
        }
    }

    function stake(uint256 amount) external {
        updateReward(msg.sender);
        stakingToken.safeTransferFrom(msg.sender, address(this), amount);
        stakedBalance[msg.sender] += amount;
        totalStaked += amount;
        emit Staked(msg.sender, amount);
    }

    function unstake(uint256 amount) external {
        require(stakedBalance[msg.sender] >= amount, "Not enough staked");
        updateReward(msg.sender);
        stakedBalance[msg.sender] -= amount;
        totalStaked -= amount;
        stakingToken.safeTransfer(msg.sender, amount);
        emit Unstaked(msg.sender, amount);
    }

    function claimReward() external {
        updateReward(msg.sender);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract GuildMultiSig {
    address[] public owners;
    mapping(address => bool) public isOwner;
    uint256 public required;

    struct Transaction {
        address to;
        uint256 value;
        bytes data;
        bool executed;
        uint256 numConfirmations;
    }

    Transaction[] public transactions;
    mapping(uint256 => mapping(address => bool)) public isConfirmed;

    event Deposit(address indexed sender, uint256 amount);
    event Submit(uint256 indexed txId);
    event Confirm(address indexed owner, uint256 indexed txId);
    event Execute(uint256 indexed txId);
    event Revoke(address indexed owner, uint256 indexed txId);

    modifier onlyOwner() {
        require(isOwner[msg.sender], "Not owner");
        _;
    }

    modifier txExists(uint256 txId) {
        require(txId < transactions.length, "Tx does not exist");
        _;
    }

    modifier notExecuted(uint256 txId) {
        require(!transactions[txId].executed, "Already executed");
        _;
    }

    modifier notConfirmed(uint256 txId) {
        require(!isConfirmed[txId][msg.sender], "Already confirmed");
        _;
    }

    constructor(address[] memory _owners, uint256 _required) {
        require(_owners.length > 0, "Owners required");
        require(_required > 0 && _required <= _owners.length, "Invalid required");

        for (uint256 i = 0; i < _owners.length; i++) {
            address owner = _owners[i];
            require(owner != address(0), "Invalid owner");
            require(!isOwner[owner], "Owner not unique");
            isOwner[owner] = true;
            owners.push(owner);
        }
        required = _required;
    }

    receive() external payable {
        emit Deposit(msg.sender, msg.value);
    }

    function submitTransaction(address to, uint256 value, bytes memory data) external onlyOwner {
        uint256 txId = transactions.length;
        transactions.push(Transaction({to: to, value: value, data: data, executed: false, numConfirmations: 0}));
        emit Submit(txId);
    }

    function confirmTransaction(uint256 txId) external onlyOwner txExists(txId) notExecuted(txId) notConfirmed(txId) {
        Transaction storage transaction = transactions[txId];
        transaction.numConfirmations += 1;
        isConfirmed[txId][msg.sender] = true;
        emit Confirm(msg.sender, txId);
    }

    function executeTransaction(uint256 txId) external onlyOwner txExists(txId) notExecuted(txId) {
        Transaction storage transaction = transactions[txId];
        require(transaction.numConfirmations >= required, "Not enough confirmations");

        transaction.executed = true;
        (bool success, ) = transaction.to.call{value: transaction.value}(transaction.data);
        require(success, "Tx failed");
        emit Execute(txId);
    }

    function revokeConfirmation(uint256 txId) external onlyOwner txExists(txId) notExecuted(txId) {
        require(isConfirmed[txId][msg.sender], "Not confirmed");
        Transaction storage transaction = transactions[txId];
        transaction.numConfirmations -= 1;
        isConfirmed[txId][msg.sender] = false;
        emit Revoke(msg.sender, txId);
    }

    function getOwners() external view returns (address[] memory) {
        return owners;
    }

    function getTransactionCount() external view returns (uint256) {
        return transactions.length;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract GuildFaucet is Ownable {
    using SafeERC20 for IERC20;

    IERC20 public immutable token;
    uint256 public amountPerClaim;
    uint256 public cooldown; // en segundos
    mapping(address => uint256) public lastClaim;

    event Claimed(address indexed user, uint256 amount);

    constructor(address _token, uint256 _amountPerClaim, uint256 _cooldown, address initialOwner) 
        Ownable(initialOwner) 
    {
        token = IERC20(_token);
        amountPerClaim = _amountPerClaim;
        cooldown = _cooldown;
    }

    function claim() external {
        require(block.timestamp >= lastClaim[msg.sender] + cooldown, "Cooldown active");
        require(token.balanceOf(address(this)) >= amountPerClaim, "Faucet empty");

        lastClaim[msg.sender] = block.timestamp;
        token.safeTransfer(msg.sender, amountPerClaim);
        emit Claimed(msg.sender, amountPerClaim);
    }

    function setAmountPerClaim(uint256 newAmount) external onlyOwner {
        amountPerClaim = newAmount;
    }

    function setCooldown(uint256 newCooldown) external onlyOwner {
        cooldown = newCooldown;
    }

    function withdraw(address to, uint256 amount) external onlyOwner {
        token.safeTransfer(to, amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/Ownable.sol";

contract GuildWhitelist is Ownable {
    mapping(address => bool) public isWhitelisted;
    uint256 public whitelistCount;

    event Added(address indexed account);
    event Removed(address indexed account);

    constructor(address initialOwner) Ownable(initialOwner) {}

    function add(address account) external onlyOwner {
        require(!isWhitelisted[account], "Already whitelisted");
        isWhitelisted[account] = true;
        whitelistCount++;
        emit Added(account);
    }

    function addBatch(address[] calldata accounts) external onlyOwner {
        for (uint256 i = 0; i < accounts.length; i++) {
            if (!isWhitelisted[accounts[i]]) {
                isWhitelisted[accounts[i]] = true;
                whitelistCount++;
                emit Added(accounts[i]);
            }
        }
    }

    function remove(address account) external onlyOwner {
        require(isWhitelisted[account], "Not whitelisted");
        isWhitelisted[account] = false;
        whitelistCount--;
        emit Removed(account);
    }

    function isAllowed(address account) external view returns (bool) {
        return isWhitelisted[account];
    }
}
