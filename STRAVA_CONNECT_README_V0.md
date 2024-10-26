# **Strava to Blockchain Integration**

Connect your Strava activities with Ethereum-compatible blockchains, enabling new possibilities for fitness tracking, rewards, and challenges.

## **Quick Start**

1. Clone the repository: `git clone https://github.com/flowstake/strava-connect.git`
2. Install dependencies: `npm install`
3. Set up your environment variables in `.env`
4. Deploy the smart contract: `npx hardhat run scripts/deploy.js --network goerli`
5. Start the application: `npm start`

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
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Community and Support](#community-and-support)
- [License](#license)

## **Overview**

This project bridges the gap between physical activities tracked on Strava and blockchain technology. By storing activity data on Ethereum-compatible blockchains, we open up new possibilities for verifiable fitness tracking, tokenized rewards systems, and decentralized fitness challenges.

Key benefits include:
- Immutable record of activities
- Potential for tokenized rewards based on performance
- Creation of decentralized fitness challenges and competitions

Features include:
- Fetching activity data from Strava (distance, time, pace, etc.)
- Recording activity data on a blockchain using Solidity smart contracts
- Optional token rewards or staking mechanics for users
- Verification mechanisms to prevent cheating or data manipulation

## **Prerequisites**

Before starting, ensure you have the following installed:
- [Node.js](https://nodejs.org/en/download/) (v14.0.0 or later) & npm (v6.0.0 or later)
- [Hardhat](https://hardhat.org/) (v2.9.0 or later) or [Truffle](https://www.trufflesuite.com/) (v5.4.0 or later)
- MetaMask (or any Ethereum-compatible wallet)
- Access to an Ethereum-compatible testnet (e.g., Goerli, Polygon Mumbai)
- [Strava Developer Account](https://developers.strava.com/)

Node.js and npm are required to run the project and manage dependencies. Hardhat or Truffle are used for smart contract compilation, testing, and deployment. MetaMask is needed for interacting with the blockchain from your browser.

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
├── test/                        # Test files for smart contracts and components
├── README.md                     # This README file
├── package.json                  # Node.js project metadata
├── hardhat.config.js             # Hardhat configuration file
├── .env                          # Environment variables (Strava API keys, etc.)
```

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
  try {
    const authResponse = await authorize(config);
    return authResponse.accessToken;
  } catch (error) {
    console.error('Strava authentication failed:', error);
    throw error;
  }
};
```

#### **Step 3: Fetch Activity Data**

Use the access token to request activity data from Strava's API:
```javascript
export const fetchActivities = async (accessToken) => {
  try {
    const response = await fetch(
      `https://www.strava.com/api/v3/athlete/activities?access_token=\${accessToken}`
    );
    if (!response.ok) {
      throw new Error('Failed to fetch activities');
    }
    const activities = await response.json();
    return activities;
  } catch (error) {
    console.error('Error fetching activities:', error);
    throw error;
  }
};
```

Note: Strava API has rate limits. For most applications, this is 100 requests every 15 minutes, 1000 daily. Ensure your application respects these limits to avoid being blocked.

### **2. Smart Contract Setup**

Write a basic Solidity smart contract to store the user's Strava activity data (distance and duration) on-chain.

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

    function getActivitiesCount(address _user) public view returns (uint256) {
        return activities[_user].length;
    }
}
```

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
  try {
    const tx = await contract.recordActivity(distance, duration);
    await tx.wait();
    console.log("Activity recorded successfully");
  } catch (error) {
    console.error("Error recording activity:", error);
    throw error;
  }
};
```

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
  try {
    const gpsBuffer = Buffer.from(JSON.stringify(gpsData));
    const result = await ipfs.add(gpsBuffer);
    return result.cid.toString(); // This is the IPFS CID
  } catch (error) {
    console.error("Error uploading to IPFS:", error);
    throw error;
  }
};
```

### **B. Tokenization (Rewards & Staking)**

Create an ERC20 token to reward users for completing activities or allow them to stake tokens on their own performance.

#### **ERC20 Token Contract**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract ActivityToken is ERC20, Ownable {
    address public activityContract;

    constructor() ERC20("ActivityToken", "ATK") {
        activityContract = msg.sender;
    }

    function setActivityContract(address _activityContract) external onlyOwner {
        activityContract = _activityContract;
    }

    function mintReward(address user, uint256 amount) external {
        require(msg.sender == activityContract, "Only the activity contract can mint");
        _mint(user, amount);
    }
}
```

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

## **Deployment**

Deploy the smart contracts to an Ethereum-compatible network like Goerli or Polygon.

#### **Deploy Using Hardhat**

1. Configure the deployment script in `scripts/deploy.js`:
   ```javascript
   const main = async () => {
     const StravaActivityTracker = await ethers.getContractFactory("StravaActivityTracker");
     const contract = await StravaActivityTracker.deploy();
     await contract.deployed();
     console.log("StravaActivityTracker deployed to:", contract.address);
   }

   main()
     .then(() => process.exit(0))
     .catch((error) => {
       console.error(error);
       process.exit(1);
     });
   ```

2. Run the deployment:
   ```bash
   npx hardhat run scripts/deploy.js --network goerli
   ```

## **Testing**

To run the smart contract tests:

```bash
npx hardhat test
```

To run the frontend tests:

```bash
npm test
```

## **Troubleshooting**

Common issues and their solutions:

1. "Cannot connect to Strava API":
   - Ensure your Strava API credentials are correctly set in the .env file
   - Check if you've authorized the correct scopes for your Strava app

2. "Transaction failed":
   - Make sure you have enough ETH in your wallet for gas fees
   - Check if you're connected to the correct network in MetaMask

3. "IPFS upload failed":
   - Verify your internet connection
   - Ensure you're using a reliable IPFS node or service

## **Security Considerations**

1. Never store private keys or sensitive information in your source code or public repositories.
2. Use environment variables for API keys, contract addresses, and other sensitive data.
3. Implement proper access control in smart contracts to prevent unauthorized actions.
4. Consider using OpenZeppelin's security audited contracts for standard functionalities.
5. Regularly update dependencies to patch known vulnerabilities.
6. Implement rate limiting and other measures to prevent API abuse.

## **Roadmap**

Future plans for this project include:
- Integration with additional fitness tracking platforms
- Implementation of a governance token for community-driven development
- Creation of a decentralized marketplace for fitness challenges and rewards
- Enhanced data visualization and analytics dashboard
- Mobile app development for easier access and real-time tracking

## **Contributing**

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

Please ensure your code adheres to the project's coding standards and includes appropriate tests.

## **Community and Support**

For help, discussions, or contributions:
- Join our [Discord channel](https://discord.gg/your-discord)
- Open an issue on our [GitHub repository](https://github.com/your-username/strava-blockchain-integration/issues)
- Check out our [contribution guidelines](CONTRIBUTING.md)
- Follow us on [Twitter](https://twitter.com/your-project-twitter) for updates

## **License**

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
```

This comprehensive README now includes all the suggested additions and improvements, providing a clear guide for users to understand, set up, and contribute to the Strava to Blockchain Integration project.
