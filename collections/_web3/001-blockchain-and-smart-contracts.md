---
layout: post
title: "Blockchain and Smart Contracts: Fundamentals and Ethereum Development"
author: "Liberom"
date: 2025-01-06
category: "Web3"
toc: true
excerpt: "Master blockchain concepts: distributed ledgers, consensus mechanisms, smart contracts, and Ethereum development with Solidity."
---

# Blockchain and Smart Contracts: Fundamentals and Ethereum Development

Blockchain technology powers decentralized applications. Understanding core concepts and smart contracts is essential for Web3 development.

## Blockchain Fundamentals

### What is Blockchain?

A blockchain is a **distributed, immutable ledger** of transactions maintained by a network of computers (nodes).

**Key Characteristics**:
- **Distributed**: No single authority
- **Immutable**: Transactions cannot be altered (cryptographically secured)
- **Transparent**: All transactions visible to network participants
- **Decentralized**: Consensus-driven validation

### How Blockchain Works

```
1. User initiates transaction
   ↓
2. Transaction broadcast to network
   ↓
3. Nodes validate transaction
   ↓
4. Transaction included in block
   ↓
5. Block linked to previous blocks via cryptographic hash
   ↓
6. Consensus mechanism confirms block
   ↓
7. Block immutable on chain
```

### Consensus Mechanisms

**Proof of Work (PoW)**:
- Miners solve complex math puzzles
- High computational cost
- Bitcoin uses PoW
- Energy intensive but highly secure

```
Miner 1: Solve puzzle... ✓ Found!
Miner 2: Solve puzzle... ✗ Outpaced
Result: Miner 1 adds block, earns reward
```

**Proof of Stake (PoS)**:
- Validators chosen based on stake (coins locked)
- Energy efficient
- Ethereum 2.0 uses PoS
- Lower barriers to entry

```
Validator 1: Staked 32 ETH
Validator 2: Staked 32 ETH
Result: Random selection, add block, earn reward
```

## Ethereum Basics

### Ethereum Architecture

```
┌─────────────────────────────────────┐
│       Ethereum Network               │
├─────────────────────────────────────┤
│  • Distributed Ledger               │
│  • Smart Contracts (EVM)            │
│  • Decentralized Apps (dApps)       │
└─────────────────────────────────────┘
```

### Accounts

```solidity
// Two types of accounts:

// 1. Externally Owned Accounts (EOA)
// - Controlled by private keys
// - Can initiate transactions
// - Have balance (ETH)
Address: 0x742d35Cc6634C0532925a3b844Bc9e7595f

// 2. Contract Accounts
// - Controlled by smart contract code
// - Can receive and send transactions
// - Have associated code and storage
Address: 0x1234567890123456789012345678901234567890
Balance: 5 ETH
Code: [bytecode]
Storage: [state variables]
```

### Gas and Transactions

```solidity
// Every operation costs gas (computational units)
// Gas price measured in Gwei (10^-9 ETH)

Transaction {
    from: 0x742d...,
    to: 0x1234...,
    value: 1 ETH,
    gas: 21000,
    gasPrice: 50 Gwei,
    data: 0x [encoded function call]
}

// Cost = gas * gasPrice = 21000 * 50 Gwei = 0.00105 ETH
```

## Smart Contracts with Solidity

### Smart Contract Basics

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Simple contract
contract Counter {
    // State variable (stored on blockchain)
    uint256 public count = 0;

    // Function to increment counter
    function increment() public {
        count += 1;
    }

    // Read-only function (doesn't modify state)
    function getCount() public view returns (uint256) {
        return count;
    }

    // Pure function (no state access)
    function add(uint256 a, uint256 b) public pure returns (uint256) {
        return a + b;
    }
}
```

### Data Types

```solidity
pragma solidity ^0.8.0;

contract DataTypes {
    // Integers
    uint256 public largeNumber = 1000;
    int256 public signedNumber = -50;

    // Boolean
    bool public isActive = true;

    // Address
    address public owner = 0x742d35Cc6634C0532925a3b844Bc9e7595f;

    // String
    string public name = "John";

    // Arrays
    uint256[] public numbers;
    address[] public participants;

    // Mapping (key-value)
    mapping(address => uint256) public balances;
    mapping(address => bool) public whitelisted;

    // Struct
    struct User {
        string name;
        uint256 age;
        address wallet;
    }

    User[] public users;
}
```

### Functions and Modifiers

```solidity
pragma solidity ^0.8.0;

contract Functions {
    address public owner;
    uint256 public balance;

    constructor() {
        owner = msg.sender;  // sender of transaction
    }

    // Modifier - reusable access control
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;  // Execute function body
    }

    // Public function - accessible externally
    function deposit() public payable {
        balance += msg.value;
    }

    // Only owner can withdraw
    function withdraw(uint256 amount) public onlyOwner {
        require(amount <= balance, "Insufficient balance");
        balance -= amount;
        payable(msg.sender).transfer(amount);
    }

    // Internal function - only callable from within contract
    function _log(string memory message) internal pure {
        // Implementation
    }

    // Private function - only callable in this contract
    function _secret() private pure returns (string memory) {
        return "secret";
    }

    // View functions - read state (free)
    function getBalance() public view returns (uint256) {
        return balance;
    }
}
```

### Events and Logging

```solidity
pragma solidity ^0.8.0;

contract EventExample {
    // Event declaration
    event Transfer(address indexed from, address indexed to, uint256 amount);
    event Approval(address indexed owner, address indexed spender, uint256 amount);

    mapping(address => uint256) public balances;

    function transfer(address to, uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");

        balances[msg.sender] -= amount;
        balances[to] += amount;

        // Emit event
        emit Transfer(msg.sender, to, amount);
    }

    function approve(address spender, uint256 amount) public {
        // Emit approval event
        emit Approval(msg.sender, spender, amount);
    }
}
```

## Token Standards

### ERC-20 (Fungible Tokens)

```solidity
pragma solidity ^0.8.0;

interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint256 amount);
    event Approval(address indexed owner, address indexed spender, uint256 amount);
}

// ERC-20 Token Implementation
contract MyToken is IERC20 {
    mapping(address => uint256) public balances;
    mapping(address => mapping(address => uint256)) public allowances;

    uint256 public totalSupply = 1000000 * 10 ** 18;
    string public name = "My Token";
    string public symbol = "MTK";

    constructor() {
        balances[msg.sender] = totalSupply;
    }

    function balanceOf(address account) public view override returns (uint256) {
        return balances[account];
    }

    function transfer(address to, uint256 amount) public override returns (bool) {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        balances[to] += amount;
        emit Transfer(msg.sender, to, amount);
        return true;
    }
}
```

### ERC-721 (NFTs)

```solidity
pragma solidity ^0.8.0;

interface IERC721 {
    function balanceOf(address owner) external view returns (uint256);
    function ownerOf(uint256 tokenId) external view returns (address);
    function transferFrom(address from, address to, uint256 tokenId) external;
    function approve(address to, uint256 tokenId) external;
    function getApproved(uint256 tokenId) external view returns (address);

    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
    event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId);
}

// NFT Implementation
contract MyNFT is IERC721 {
    mapping(uint256 => address) public owners;
    mapping(address => uint256) public balances;

    uint256 public nextTokenId = 1;

    function mint(address to) public {
        owners[nextTokenId] = to;
        balances[to] += 1;
        emit Transfer(address(0), to, nextTokenId);
        nextTokenId += 1;
    }

    function transferFrom(address from, address to, uint256 tokenId) public override {
        require(owners[tokenId] == from, "Not owner");
        owners[tokenId] = to;
        balances[from] -= 1;
        balances[to] += 1;
        emit Transfer(from, to, tokenId);
    }
}
```

## Smart Contract Security

### Common Vulnerabilities

```solidity
// VULNERABILITY: Reentrancy Attack
// BAD
function withdraw(uint256 amount) public {
    require(balances[msg.sender] >= amount);

    // Danger: Attacker can call withdraw again
    (bool success, ) = payable(msg.sender).call{value: amount}("");
    require(success);

    balances[msg.sender] -= amount;  // After transfer!
}

// GOOD: Checks-Effects-Interactions pattern
function withdraw(uint256 amount) public {
    require(balances[msg.sender] >= amount);
    balances[msg.sender] -= amount;  // Modify state first
    (bool success, ) = payable(msg.sender).call{value: amount}("");
    require(success);
}

// VULNERABILITY: Integer Overflow
// BAD (pre-0.8.0)
uint256 a = type(uint256).max;
uint256 b = a + 1;  // Wraps to 0

// GOOD (Solidity 0.8.0+)
// Overflow automatically reverts

// VULNERABILITY: Unchecked External Calls
// BAD
address(externalContract).call(abi.encodeWithSignature("transfer(...)"));

// GOOD
(bool success, ) = address(externalContract).call(...);
require(success, "External call failed");
```

### Security Best Practices

```solidity
pragma solidity ^0.8.0;

// GOOD: Secure contract pattern
contract SecureContract {
    mapping(address => uint256) public balances;
    bool private locked;

    modifier nonReentrant() {
        require(!locked, "No reentrancy");
        locked = true;
        _;
        locked = false;
    }

    // Use safeTransfer for ERC-20
    function depositToken(address token, uint256 amount) public {
        require(amount > 0, "Amount must be > 0");
        // Use OpenZeppelin's SafeERC20
        IERC20(token).transferFrom(msg.sender, address(this), amount);
        balances[msg.sender] += amount;
    }

    // Reentrancy-safe withdrawal
    function withdraw(uint256 amount) public nonReentrant {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        (bool success, ) = payable(msg.sender).call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

## Development Tools

### Hardhat - Smart Contract Development

```javascript
// hardhat.config.js
module.exports = {
  solidity: {
    version: "0.8.0",
  },
  networks: {
    goerli: {
      url: `https://goerli.infura.io/v3/${INFURA_KEY}`,
      accounts: [PRIVATE_KEY],
    },
  },
};

// test/MyContract.js
const { expect } = require("chai");

describe("MyContract", function () {
  it("Should deploy and work", async function () {
    const MyContract = await ethers.getContractFactory("MyContract");
    const contract = await MyContract.deploy();

    expect(await contract.getCount()).to.equal(0);

    await contract.increment();
    expect(await contract.getCount()).to.equal(1);
  });
});
```

## Conclusion

Blockchain and smart contract essentials:
- Understand consensus mechanisms
- Learn Solidity syntax and patterns
- Follow security best practices
- Use established token standards (ERC-20, ERC-721)
- Test contracts thoroughly
- Use established libraries (OpenZeppelin)
- Avoid reentrancy and overflow vulnerabilities
- Consider gas optimization

