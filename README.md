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
