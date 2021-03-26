# 1: Write & deploy your first NFT smart contract

Learn to write an NFT smart contract on Celo!

# Introduction 
In this tutorial, we will create and deploy an NFT smart contract on the Celo network. Since Celo is a close cousin to Ethereum, it inherits Ethereum’s developer ecosystem and tooling. 

Part of what Celo inherits is the ERC-721 standard for NFTs. We will be writing and deploying a simple NFT smart contract to the Alfajores testnet using Solidity, Datahub and Truffle.

After finishing this tutorial, we will have an NFT smart contract we can keep building on. The possibilities are endless for NFTs, so this will serve as an introduction to get you started. 

# Prerequisites
Since Celo runs on the Ethereum Virtual Machine, we’ll use Truffle to compile our NFT smart contract. If you don’t have it installed, run the following command in your terminal: 
	
`npm install -g truffle`

It is optional (but recommended) to have completed the previous tutorials in the Celo pathway: 


[1. Connect to a Celo node with DataHub](https://learn.figment.io/network-documentation/celo/tutorial/1.connect)

[2. Create your first Celo account](https://learn.figment.io/network-documentation/celo/tutorial/2.account) 

[3. Query the Celo network](https://learn.figment.io/network-documentation/celo/tutorial/3.query)

[4. Submit your first transactions](https://learn.figment.io/network-documentation/celo/tutorial/4.transactions)

[5. Write and deploy your first Celo smart contract](https://learn.figment.io/network-documentation/celo/tutorial/5.smart-contract)

If you have not completed the above tutorials, you may have to backtrack for certain parts of this tutorial. 

# Project setup 
First, open the terminal and make a new project folder. We’ll call if myNFT:
 
`mkdir myNFT && cd myNFT`

Next, let’s initialize the project directory with NPM 

`npm init -y`

After it’s been initialized, we’ll need to install some additional packages. Here's an overview of them: 

 - ContractKit is a package created by the Celo team to aid in Celo development
 - Dotenv is used for reading environment variables in our code
 - And finally, Web3 library that helps facilitate interaction with the blockchain
 
 Install all of the above using: 

`npm install --save @celo/contractkit dotenv web3`

Next, we’re going to install Open Zeppelin’s solidity library. 

Open Zeppelin is a security company which has made several Solidity code samples that have been audited and tested extensively. They are used frequently throughout Solidity development. 

For this tutorial, we will use the ERC-721 (NFT) contract: 

`npm i @openzeppelin/contracts`

To end our setup, initialize the smart contract project with Truffle: 

`truffle init`

# Writing the smart contract

Now that we’re set up, we want to create the smart contract for our NFT. 
	
Open your project directory in your favorite text editor. In the **contracts folder**, delete the Migrations.sol file and create a new file called CoolNFT.sol

In that file, write the following: 

```
pragma solidity >=0.6.0 <0.8.0;
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract CoolNFT is ERC721 {
  uint256 public tokenCounter; 

  constructor() public ERC721("CoolNFT", "COOL") {
    tokenCounter = 0; 
  }

  function createCollectible(string memory tokenURI) public returns (uint256) {
    uint256 newItemId = tokenCounter; 
    _safeMint(msg.sender, newItemId); 
    _setTokenURI(newItemId, tokenURI); 
    tokenCounter = tokenCounter + 1;

    return newItemId; 
  }
}
```

This is a simple NFT smart contract written in Solidity. Let's break it down a bit. 

At the top of the file, we use inheritance (the *is* keyword in Solidity) to inherit from the ERC-721 OpenZeppelin contract: 

`` contract CoolNFT is ERC721 { ``

Next, we create a tokenCounter variable of type uint256. Uint256 is a type of unsigned integer in Solidity which can be 256 bits in size.

``  uint256 public tokenCounter; ``

Next, we create a function called **createCollectible()** which takes in a string variable named tokenURI as input and returns an integer of type uint256: 

``  function createCollectible(string memory tokenURI) public returns (uint256) {``

Inside the createCollectible() function, we set the id for the new NFT to be equal to the tokenCounter. We also use the **_safeMint()** function our contract inherits from OpenZeppelin's contract to mint an NFT: 

```   
 uint256 newItemId = tokenCounter;  
_safeMint(msg.sender, newItemId); 
```

**_safeMint()** will create an NFT owned by the user which called the contract (msg.sender), with an id of newItemId. 

After minting the NFT, we set the NFT to have the properties contained in tokenURI (usually a link and a title for the NFT). We also increase the NFT count, and return the id.

```
_setTokenURI(newItemId, tokenURI); 
tokenCounter = tokenCounter + 1;

return newItemId; 
```

When we use our smart contract to mint NFTs in part two, we will call the createCollectible() function with a tokenURI in order to mint NFTs.

Now that we’ve written the smart contract, it’s time to get started with the deployment setup. 

# Deployment setup

Create a .env file in the **root directory** of the `myNFT` folder. To do so, navigate to the root directory of your project and type the following command into your terminal: 

`touch .env`

Next, open the .env file in your text editor and add the following variable:

`REST_URL=https://celo-alfajores--rpc.datahub.figment.io/apikey/<YOUR API KEY>/`

Where YOUR API KEY is your DataHub API key. 

If this is unfamiliar to you, please read: [Connect to a Celo node with DataHub](https://learn.figment.io/network-documentation/celo/tutorial/1.connect).

Next, we’re going to need a Celo account to deploy from. We will need three things for deployment: 
 - A Celo account address
 - A Celo account private key
 - A Celo account [loaded with testnet funds](https://celo.org/developers/faucet)

If you do not have the above, please refer to: [Create your first Celo account](https://learn.figment.io/network-documentation/celo/tutorial/2.account) and follow the steps.

Once you have your account address and private key come back to resume this tutorial. 

Next, in your .env file create a new line for your private key like so: 

``PRIVATE_KEY=YOUR PRIVATE KEY!``

**Reminder: Make sure that your Celo account is loaded with testnet funds**

Now that your Celo account is setup, we can move on to the truffle deployment side of things. Replace the contents of **truffle-config.js** with the following: 

```
const ContractKit = require('@celo/contractkit');
const Web3 = require('web3');

require('dotenv').config({path: '.env'});

// Create connection to DataHub Celo Network node
const web3 = new Web3(process.env.REST_URL);
const client = ContractKit.newKitFromWeb3(web3);

// Initialize account from our private key
const account = web3.eth.accounts.privateKeyToAccount(process.env.PRIVATE_KEY);

// We need to add private key to ContractKit in order to sign transactions
client.addAccount(account.privateKey);

module.exports = {
  compilers: {
    solc: {
      version: "0.6.6",    // Fetch exact version from solc-bin (default: truffle's version)
    }
  },
  networks: {
    test: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*"
    },
    alfajores: {
      provider: client.connection.web3.currentProvider, // CeloProvider
      network_id: 44787  // latest Alfajores network id
    }
  }
};
```

The above config connects to the Alfajores test network, and resolves to deploy our contract with your account.

As the last part of the deployment setup, delete the **1_initial_migration.js** file inside of the **migrations folder**, and replace it with  a file called **1_deploy_coolnft.js**. In our new deploy file, write the following:

```
var CoolNFT = artifacts.require('CoolNFT')

module.exports = function(deployer) {
  deployer.deploy(CoolNFT)
};
```

# Deployment

We’re almost there! Run the following to check that you did everything correctly: 

`` truffle compile``

If things are working, you should see the following output: 

![Truffle compile result](https://i.imgur.com/hfjbJUZ.png)

Now that we’ve compiled the NFT smart contract, the last step is to deploy it. Run the following to deploy to the Alfajores testnet: 

 `truffle migrate --network alfajores`

You should see the following deployment output:  

![deployment output](https://i.imgur.com/qSkl1HM.png)


To see the deployed contract visually, you can go to the [Alfajores Celo Block Explorer](https://alfajores-blockscout.celo-testnet.org/), and input the Celo account address you generated earlier. You should now see a contract creation transaction!

![block explorer output](https://i.imgur.com/lMEeQc6.png)

# Conclusion

Congratulations! You just created your first NFT smart contract on the Celo network. 

Now that you've finished the tutorial, you should have a basic understanding of creating NFT smart contracts and deploying them. You have the basic skills to make more advanced NFTs now. Also, the possibilities for NFTs are endless! It's still early. You can use this code as a jumping off point for more complex NFTs 🥳

The complete source code for this tutorial can be found on [Github](https://github.com/alexreyes/Celo-NFT-Tutorial).

## Next steps
Creating and deploying an NFT smart contract is great, but what if you want to mint NFTs with our contract? 

In the next tutorial, we will dive deeper into the wonderful world of NFTs and we will interact with our NFT smart contract in order to mint new NFTs. 
