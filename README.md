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
