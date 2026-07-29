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
