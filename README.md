# strava-connect

# **Strava to Blockchain Integration**

This project demonstrates how to connect Strava activity data with Ethereum-compatible blockchains like Ethereum or Polygon. The goal is to fetch user activity data from Strava and store or verify it on-chain using smart contracts.

## **Table of Contents**

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [1. Strava API Integration](#1-strava-api-integration)
  - [2. Smart Contract Setup](#2-smart-contract-setup)
  - [3. Web3 Integration](#3-web3-integration)
- [Optional Enhancements](#optional-enhancements)
  - [A. Gas Optimization](#a-gas-optimization)
  - [B. Tokenization (Rewards & Staking)](#b-tokenization-rewards--staking)
  - [C. Cheating Prevention](#c-cheating-prevention)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

---

## **Overview**

This project connects physical activity data, such as running or cycling, from Strava with Ethereum-compatible smart contracts (Ethereum, Polygon, etc.). The data can be stored or verified on the blockchain for various purposes like activity tracking, rewards, or staking systems.

Features include:
- Fetching activity data from Strava (distance, time, pace, etc.)
- Recording activity data on a blockchain using Solidity smart contracts.
- Optional token rewards or staking mechanics for users.
- Verification mechanisms to prevent cheating or data manipulation.

---

## **Prerequisites**

Before starting, ensure you have the following installed:
- [Node.js & npm](https://nodejs.org/en/download/)
- [Hardhat](https://hardhat.org/) or [Truffle](https://www.trufflesuite.com/)
- MetaMask (or any Ethereum-compatible wallet)
- Access to an Ethereum-compatible testnet (e.g., Goerli, Polygon Mumbai)
- [Strava Developer Account](https://developers.strava.com/)

---

## **Project Structure**

```
├── contracts/                   # Solidity smart contracts
│   ├── StravaActivityTracker.sol # Main contract to record activity data
│   ├── ActivityToken.sol         # (Optional) ERC20 contract for token rewards
│   └── StravaActivityStaking.sol # (Optional) Smart contract for staking activities
├── scripts/                     
│   └── deploy.js                 # Deployment script using Hardhat
├── src/                         
│   ├── components/               # React/React Native components
│   └── utils/                    # Utility functions (e.g., Web3 interaction, Strava API fetch)
├── README.md                     # This README file
├── package.json                  # Node.js project metadata
├── hardhat.config.js             # Hardhat configuration file
├── .env                          # Environment variables (Strava API keys, etc.)
```

---

## **Getting Started**

### **1. Strava API Integration**

You need to register an application on the Strava Developer Portal to access the Strava API. After registering, you'll receive your `client_id`, `client_secret`, and `redirect_uri` for OAuth authentication.

#### **Step 1: Install Dependencies**

Install required libraries for OAuth and making HTTP requests:
```bash
npm install react-native-app-auth node-fetch ethers dotenv ipfs-http-client
```

#### **Step 2: Implement OAuth Flow**

Create a function to authenticate the user and retrieve their Strava activity data:
```javascript
import { authorize } from 'react-native-app-auth';

const config = {
  issuer: 'https://www.strava.com/oauth/authorize',
  clientId: process.env.STRAVA_CLIENT_ID,
  clientSecret: process.env.STRAVA_CLIENT_SECRET,
  redirectUrl: 'yourapp://oauthredirect',
  scopes: ['activity:read'],
  serviceConfiguration: {
    authorizationEndpoint: 'https://www.strava.com/oauth/authorize',
    tokenEndpoint: 'https://www.strava.com/oauth/token',
  },
};

export const authenticateStrava = async () => {
  const authResponse = await authorize(config);
  return authResponse.accessToken;
};
```

#### **Step 3: Fetch Activity Data**

Use the access token to request activity data from Strava’s API:
```javascript
export const fetchActivities = async (accessToken) => {
  const response = await fetch(
    `https://www.strava.com/api/v3/athlete/activities?access_token=${accessToken}`
  );
  const activities = await response.json();
  return activities;
};
```

---

### **2. Smart Contract Setup**

Write a basic Solidity smart contract to store the user’s Strava activity data (distance and duration) on-chain.

#### **Solidity Contract (StravaActivityTracker.sol)**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract StravaActivityTracker {
    struct Activity {
        address user;
        uint256 distance;
        uint256 duration;
        uint256 timestamp;
    }

    mapping(address => Activity[]) public activities;

    event ActivityRecorded(address indexed user, uint256 distance, uint256 duration, uint256 timestamp);

    function recordActivity(uint256 _distance, uint256 _duration) external {
        activities[msg.sender].push(Activity({
            user: msg.sender,
            distance: _distance,
            duration: _duration,
            timestamp: block.timestamp
        }));
        
        emit ActivityRecorded(msg.sender, _distance, _duration, block.timestamp);
    }
}
```

---

### **3. Web3 Integration**

Once the contract is deployed, interact with it using `ethers.js` to record activities.

#### **Client-Side Code for Recording Activity**

```javascript
import { ethers } from 'ethers';

const provider = new ethers.providers.Web3Provider(window.ethereum);
const signer = provider.getSigner();

const contractAddress = 'YOUR_DEPLOYED_CONTRACT_ADDRESS';
const contractABI = [/* ABI from your Solidity contract */];

const contract = new ethers.Contract(contractAddress, contractABI, signer);

export const recordActivity = async (distance, duration) => {
  const tx = await contract.recordActivity(distance, duration);
  await tx.wait();
  console.log("Activity recorded successfully");
};
```

---

## **Optional Enhancements**

### **A. Gas Optimization**

To optimize gas usage, batch multiple activities into one transaction or store large data like GPS coordinates off-chain using IPFS.

#### **Batch Activity Recording**

```solidity
function recordActivities(uint256[] memory _distances, uint256[] memory _durations) external {
    require(_distances.length == _durations.length, "Mismatched arrays");

    for (uint256 i = 0; i < _distances.length; i++) {
        activities[msg.sender].push(Activity({
            user: msg.sender,
            distance: _distances[i],
            duration: _durations[i],
            timestamp: block.timestamp
        }));
    }

    emit ActivitiesRecorded(msg.sender, _distances.length);
}
```

#### **Store GPS Data on IPFS**

```javascript
import { create } from 'ipfs-http-client';

const ipfs = create('https://ipfs.infura.io:5001');

export const uploadGpsDataToIPFS = async (gpsData) => {
  const gpsBuffer = Buffer.from(JSON.stringify(gpsData));
  const result = await ipfs.add(gpsBuffer);
  return result.cid.toString(); // This is the IPFS CID
};
```

---

### **B. Tokenization (Rewards & Staking)**

Create an ERC20 token to reward users for completing activities or allow them to stake tokens on their own performance.

#### **ERC20 Token Contract**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract ActivityToken is ERC20 {
    address public activityContract;

    constructor() ERC20("ActivityToken", "ATK") {
        activityContract = msg.sender;
    }

    function mintReward(address user, uint256 amount) external {
        require(msg.sender == activityContract, "Only the activity contract can mint");
        _mint(user, amount);
    }
}
```

---

### **C. Cheating Prevention**

Implement P2P attestation via QR codes or NFC, or integrate Chainlink oracles for data validation.

#### **P2P Attestation with QR Codes**

Generate and scan QR codes to validate user activities.

```javascript
import QRCode from 'qrcode.react';

const generateQR = (userAddress) => {
  return <QRCode value={userAddress} size={128} />;
};
```

---

## **Deployment**

Deploy the smart contracts to an Ethereum-compatible network like Goerli or Polygon.

#### **Deploy Using Hardhat**

1. Configure the deployment script in `scripts/deploy.js`:
   ```javascript
   const StravaActivityTracker = await ethers.getContractFactory("StravaActivityTracker");
   const contract = await StravaActivityTracker.deploy();
   ```

2. Run the deployment:
   ```bash
   npx hardhat run scripts/deploy.js --network goerli
   ```

---

## **Contributing**

Contributions are welcome! Feel free to open issues or pull requests with improvements or suggestions.

---

## **License**

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---
