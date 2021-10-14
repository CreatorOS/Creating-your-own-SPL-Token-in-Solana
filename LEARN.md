# Creating your own SPL Token in Solana
In this quest, we will be working on creating our own Token. Tokens can be compared to a fiat currency like INR, USD in the real world, but in crypto world we term currency as token/coin.

As this is a solana quest, hence we will solely work on creating a Token on Solana blockchain.

In the Solana system, SPL(Solana Program Library) token is basically a deposit account which is managed by the [Token](https://github.com/solana-labs/solana-program-library/tree/master/token) on-chain program(like smart contract).   
This “Token” on-chain program contains all the inbuilt methods to create Fungible and Non Fungible tokens along with Mint Authority, Associated Account.

Token program is written in rust primarily, but all the interfaces can be invoked by Web3.js library.

Cryptocurrency which are created natively on their own blockchain are called Coins.

Examples: ETH built on Ethereum, SOL built on Solana

Cryptocurrency which are created on top of existing blockchain are called Tokens.

Examples: Augur (REP) built on Ethereum blockchain, RAY built on Solana blockchain

It will be very fascinating in real world to create a currency on our names and build a use case around it. You can build your own token in the crypto world and build a use case around it. You can use your own token to incentivise the user for using your services or can tie up with vendors to provide them some discounts while using your tokens, having said that opportunity is endless.

What will you get after going through this quest ?

1. You will be able to build your own token in the Solana blockchain.
2. You will know the difference between minting and burning a token.
3. You will know the importance of mint authority and associated accounts.
4. You will know the difference between minting a token and transferring a token.
5. You will know what freezing authority is and its usage.

Prerequisites to work on this quest are as follows. All these have been covered in [https://questbook.app](https://questbook.app) should you want to visit them first.

1. Knowledge of crypto wallet (Phantom wallet specifically). 
2. A basic react app with wallet connectivity feature.
3. Solana cluster can be : local, devnet or testnet.

For better understanding, please go through our first quest on creating a wallet connection with react app.

\*Local solana cluster will not work on M1 macbook

Folder structure expected of the react app to run this quest

- Assuming app name as "DemoMint"
	- App-Name
		- node_modules
		- public
		- Src
			- utils
				- createNewMintAuthority.js
			- App.js
		- Index.js
		- Package.json

```

dependencies:{

   "react": "^17.0.2",  
   "react-dom": "^17.0.2",  
   "react-scripts": "4.0.3",  
   "@solana/web3.js": "^1.29.1",  
   "buffer-layout": "^1.2.0"

}  
   

```

Basic terminologies to digest before we jump into the code

1. Mint Authority : The authority who has the permission to mint or create the token. It's like an institution/ central authority in real world who can print the fiat currency like USD notes, INR notes.  
Mint authority is a key which can approve the transaction for the minting or burning by signing it.
2. Associated Account: Once you have your own token, then you must be willing to transfer/mint it to the user's account address but you can not just directly transfer the custom token. To transfer/mint the custom token first you need to create an account for the user which is mapped with Mint Authority, only then valid custom token transaction takes place. So any account which is mapped/associated with Mint Authority is termed as Associated Account.
3. Minting Token: When you try to print new currency notes in real world, it increases the supply of the notes which is already present in the market. At the same, if we need to increase the supply of our token in the market then we mint (print) the tokens into an associated account.  
Minting process will increase the total supply of the custom token.
4. Transfer Token: In real world when you want to lend some money to your friend, you do not ask the government to print more money and give it to you, so that you can lend. What you do is you lend your money whatever you have with you to your friend. This lending doesn't increase the total supply of the fiat currency notes. In the similar way, when a user wants to send custom tokens to other user, then user transfers tokens to friend's wallet without ever minting new tokens.  
This does not increase the total supply of the custom token.
5. Burning Token: In real world when the supply of currency notes gets higher than the requirements, then situation like Inflation occurs. If there is too much money in circulation, but very little things to do with the money the value of the coin decreases. To overcome this situation government generally do demonetization.   
Hence in similar way, excessive amount of tokens in the market can create the situation like inflation, so to avoid we can burn the  tokens in cryptworld.  
Burning a token means reducing the supply of the token by 1.
6. Freezing Authority: The Mint may also contain a freeze_authority which can be used to issue FreezeAccount instructions that will make the mint account unusable. Unusable account can not perform any action like minting or burning the tokens. This can be used in situations like where someone got access to our mint private key and trying to mint unlimited number of tokens, in this Freezing authority can issue instruction to make the mint account unusable.

Flow of custom token creation

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/c81c7c10-c5b4-407c-a76c-b8d30c3316be.jpg)
## Setting up the Mint
- To make it easy, all the below code is written in single file createNewMintAuthority.js

Let's create a new Keypair which we want to convert as a mint authority. A keypair identifies an account on Solana. 

A Keypair is a combination of public key and private key. Public key will be used to identify the Mint Authority over the internet and Private key can be used to sign the transactions.

```

let mint = new Account();

```

Next, we will focus on creating an instruction to map our new keypair to associate it with our wallet address.

We now need to associate the “mint” account we created with the wallet we are connected to, so that the wallet becomes the signer (owner) of the mint. 

Every action on Solana is a transaction. Creating accounts, deploying programs, transferring money etc. 

So first, we’ll create a transaction to create an account. By doing so, 

```

let transaction = new Transaction();

  transaction.add(

    SystemProgram.createAccount({

      fromPubkey: owner.publicKey,

      newAccountPubkey: mint.publicKey,

      lamports: await connection.getMinimumBalanceForRentExemption(

        MINT_LAYOUT.span,

      ),

      space: MINT_LAYOUT.span,

      programId: programIds.token,

    }),

  );

```

Few key variables to be used in the above function

- fromPubkey: The owner account with which we want to map the mint key
- newAccountPubkey: The mint keypair's public key, which will be used as token address
- lamports:  To create the account and make it rent free, minimum number of lamports are given by getMinimumBalanceForRentExemption utility function
- space: It is the minimum space which will be held by the mint account
- programId: This is the ID of the on-chain program of token, this on-chain contains all the interfaces to create the new token

Once we initialise the keypair as the mint authority then mint's public key will become the "custom token address” - that is the address of our custom token. Every currency on Solana is identified by an address. 

We need to provide decimals as well while intializing the new mint authority. Decimals act as a denomination for our custom tokens.

If we define the decimals as 9, 1 token is represented as 10^9. Managing decimals is harder than managing integers, and this is a common practice to maintain precision. 

So if you want to denote 0.001 tokens, you’ll define it as 0.001 \* 10^9 = 1000000.

```

transaction.add(

    initializeMint({

      mint: mint.publicKey,

      decimals,

      mintAuthority: owner.publicKey

    }),

  );

  let signers = \[mint\];

```

Few key variables to be used in above function

- mint: the mint's keypair public which will be converted to a token address
- decimals: decimals represents the denomination we want to have in the token
- mintAuthority: mintAuthority is the owner of this mint account who can perform actions like minting or burning of tokens.
- signers:  Array of signers for the transaction. We need to add the mint as the signer of the transaction, to map the mint with the wallet address. Without signer, the transaction will not be validated and will get failed.
## Signing the transactions
```

async function signAndSendTransaction(

  connection,

  transaction,

  wallet,

  signers,

  skipPreflight = false,

) {

  transaction.recentBlockhash = (

    await connection.getRecentBlockhash('max')

  ).blockhash;

  transaction.setSigners(

    wallet.publicKey,

    ...signers.map((s) => s.publicKey)

  );

  if (signers.length > 0) {

    transaction.partialSign(...signers);

  }

  transaction = await wallet.signTransaction(transaction);

  const rawTransaction = transaction.serialize();

                                                 

                                             

  return await connection.sendRawTransaction(rawTransaction, {

    skipPreflight,

    preflightCommitment: 'single'

  });

}

```

Few key variables to be used in above function

- transaction.recentBlockhash : As solana works on proof of history, hence every transaction in the solana blockchain requires the latest blockhash to be associated with the new transaction. Without the getRecentBlockhash, the validators will not be able to verify the transaction.
- wallet.publicKey: Every transaction needs a fee to be paid to the computers who are making the transaction possible. This parameter tells Solana who will be paying the fees for this transaction. 
- signers.map: Which are the accounts that will be touched by this transaction? That is defined by this list of signers. If any action is going to be taken on this account, it requires the signature using the account’s private key. This ensures that the program never updates the account of a user without the permission of the owner of that account. 
- transaction.partialSign : Any intermediate signer required in the transaction, requires to partially sign the transaction before the final transaction.
- transaction.serialize() : All the data must be stored on the blockchain. To keep the content format agnostic of the programming language used, the data is serialized before storing. 
- skipPreflight: It’s a bool data type
	- true: preflight transaction checks for the available methods before sending the transaction, which involves a very little latency.
	- false: (default value) : It is turned off by default to save the network bandwidth.
- preflightCommitment: Every transaction on Solana goes through a process of finalization. The longer the transaction has existed on the blockchain, less likely it is to get reverted. A transaction is reverted when the blockchain realizes later that the transaction was actually not supposed to be allowed. However, the process of finalization takes some time. Depending on the security required in your code, you can choose between single, confirmed and finalized security levels. Single means that there is atleast one “confirmation” given to the transaction by a validator on Solana.
## Making our transaction Blockchain compliant
LAYOUT class will help us to build the structure in the required format as same as the struct format required by the Token on-chain program to read the instructions and process it.

BufferLayout provides us the conversion of Layout to array of bytes called Encoding which can be transferred over the network and Decoding takes place in the solana on-chain program to find the instruction.

On-chain program perform the action based on the instructions it receives in the form of struct. It also provides us the options to define the exact data types required by the solana on-chain program to process. Like u8, blob, u16, struct

```

const LAYOUT = BufferLayout.union(BufferLayout.u8('instruction'));

LAYOUT.addVariant(

  0,

  BufferLayout.struct(\[

    BufferLayout.u8('decimals'),

    BufferLayout.blob(32, 'mintAuthority'),

    BufferLayout.u8('freezeAuthorityOption'),

    BufferLayout.blob(32, 'freezeAuthority'),

  \]),

  'initializeMint',

);

LAYOUT.addVariant(1, BufferLayout.struct(\[\]), 'initializeAccount');

LAYOUT.addVariant(

  7,

  BufferLayout.struct(\[BufferLayout.nu64('amount')\]),

  'mintTo',

);

LAYOUT.addVariant(

  8,

  BufferLayout.struct(\[BufferLayout.nu64('amount')\]),

  'burn',

);

LAYOUT.addVariant(9, BufferLayout.struct(\[\]), 'closeAccount');

LAYOUT.addVariant(

  12,

  BufferLayout.struct(\[BufferLayout.nu64('amount'), BufferLayout.u8('decimals')\]),

  'transferChecked',

);

```

- u8 : unsigned 8 bit integer, which means only positive integers will be supported of 8 bit example: BufferLayout.u8('decimals') means 'decimals' is a variable of unsigned 8 bit integer.
- struct: is equivalent to class but it's a very light weight data type as compared to classes in lower level languages like c\+\+,c.

```

Example: BufferLayout.struct(\[

    BufferLayout.u8('decimals'),

    BufferLayout.blob(32, 'mintAuthority'),

    BufferLayout.u8('freezeAuthorityOption'),

    BufferLayout.blob(32, 'freezeAuthority'),

  \]),  

```

- here we are defining the struct with the decimals, mintAuthority, freezeAuthorityOption and freezeAuthority variables with their respective data types.
- encodeTokenInstructionData: It's a utility function to encode each instruction in bytes array and make it compatible with on-chain solana programs.
- SYSVAR_RENT_PUBKEY: It's the public key of the system variable in rust, which is used to keep the on-chain program in memory by paying the minimum rent for the on-chain program.
- TOKEN_PROGRAM_ID : It is the public key of deployed on-chain program for the Solana Token, which has all the inbuilt functions to create the mint authority and associated accounts.
## Putting it all together
```

import \* as BufferLayout from 'buffer-layout';

import { SystemProgram, Transaction, PublicKey, Account, TransactionInstruction } from '@solana/web3.js';

import { SYSVAR_RENT_PUBKEY } from '@solana/web3.js';

const TOKEN_PROGRAM_ID = new PublicKey(

    'TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA',

  );

const programIds = {

      token: TOKEN_PROGRAM_ID,

}

const MINT_LAYOUT = BufferLayout.struct(\[

  BufferLayout.blob(44),

  BufferLayout.u8('decimals'),

  BufferLayout.blob(37),

\]);

const LAYOUT = BufferLayout.union(BufferLayout.u8('instruction'));

LAYOUT.addVariant(

  0,

  BufferLayout.struct(\[

    BufferLayout.u8('decimals'),

    BufferLayout.blob(32, 'mintAuthority'),

    BufferLayout.u8('freezeAuthorityOption'),

    BufferLayout.blob(32, 'freezeAuthority'),

  \]),

  'initializeMint',

);

LAYOUT.addVariant(1, BufferLayout.struct(\[\]), 'initializeAccount');

LAYOUT.addVariant(

  7,

  BufferLayout.struct(\[BufferLayout.nu64('amount')\]),

  'mintTo',

);

LAYOUT.addVariant(

  8,

  BufferLayout.struct(\[BufferLayout.nu64('amount')\]),

  'burn',

);

LAYOUT.addVariant(9, BufferLayout.struct(\[\]), 'closeAccount');

LAYOUT.addVariant(

  12,

  BufferLayout.struct(\[BufferLayout.nu64('amount'), BufferLayout.u8('decimals')\]),

  'transferChecked',

);

export const createNewMintAuthority = async (wallet, decimals=9, connection) =>{

  let mint = new Account();

  const signature = await createAndInitializeMint({

      connection,

      owner: wallet,

      mint,

      decimals,

    })

   

    await connection.confirmTransaction(signature);

    return {signature, mintAccount: mint};

}

async function createAndInitializeMint({

  connection,

  owner, // Wallet for paying fees and allowed to mint new tokens

  mint, // Account to hold token information

  decimals

}) {

  let transaction = new Transaction();

  transaction.add(

    SystemProgram.createAccount({

      fromPubkey: owner.publicKey, //the owner account with which we want to map the mint key

      newAccountPubkey: mint.publicKey, // the mint keypair's public key, which will be used as token address

      lamports: await connection.getMinimumBalanceForRentExemption(

        MINT_LAYOUT.span,

      ), // Important: To create the account and make it rent free, minimum number of lamports are given by getMinimumBalanceForRentExemption utility function

      space: MINT_LAYOUT.span, //Important: It is the minimum space which will be held by the mint acount

      programId: programIds.token, // This is the ID of the on-chain program of token, this on-chain contains all the interfaces to create the new token

    }),

  );

  transaction.add(

    initializeMint({

      mint: mint.publicKey, // the mint's keypair public which will be converted to a token address

      decimals, //decimals represents the denomination we want to have in the token

      mintAuthority: owner.publicKey, // mintAuthority is the owner of this mint account who can perform actions like minting or burning of tokens

    }),

  );

  let signers = \[mint\];

  return signAndSendTransaction(connection, transaction, owner, signers);

}

function initializeMint({

  mint,

  decimals,

  mintAuthority,

  freezeAuthority,

}) {

  let keys = \[

    { pubkey: mint, isSigner: false, isWritable: true },

    { pubkey: SYSVAR_RENT_PUBKEY, isSigner: false, isWritable: false },

  \];

  return new TransactionInstruction({

    keys,

    data: encodeTokenInstructionData({

      initializeMint: {

        decimals,

        mintAuthority: mintAuthority.toBuffer(),

        freezeAuthorityOption: !!freezeAuthority,

        freezeAuthority: (freezeAuthority || PublicKey.default).toBuffer(),

      },

    }),

    programId: programIds.token,

  });

}

async function signAndSendTransaction(

  connection,

  transaction,

  wallet,

  signers,

  skipPreflight = false,

) {

  transaction.recentBlockhash = (

    await connection.getRecentBlockhash('max')

  ).blockhash; // As solana works on proof of history, hence every transaction in the solana blockchain requires the latest blockhash to be associated

              // Without the getRecentBlockhash, the validators will not be able to verify the transaction

  transaction.setSigners(

    // fee payed by the wallet owner

    wallet.publicKey,

    ...signers.map((s) => s.publicKey), // we will add all the intermediate signers required for the transaction, in our case mint keypair is also required.

                                        // Hence we have added the mint signer from the signers array.

  );

  if (signers.length > 0) {  //Any intermediate signer required in the transaction, requires to partially sign the transaction before the final transaction.

    transaction.partialSign(...signers);

  }

  transaction = await wallet.signTransaction(transaction);

  const rawTransaction = transaction.serialize();  //Serialization is the process of converting an object into a stream of bytes,

                                                  //which can be used by on-chain programs to again de-serialize it to read the instructions

                                                  //and perform actions on it.

  return await connection.sendRawTransaction(rawTransaction, {

    skipPreflight,  //preflight transaction check checks for the available methods before sending the transaction, which involves a very little latency

    //that is why skipPreflight is generally kept false, to save the network bandwidth.

    preflightCommitment: 'single',

    //For preflight checks and transaction processing,

    //Solana nodes choose which bank state to query based on a commitment requirement set by the client.

    //The commitment describes how finalized a block is at that point in time. When querying the ledger state,

    //it's recommended to use lower levels of commitment to report progress and higher levels to ensure the state will not be rolled back.

    //For processing many dependent transactions in series, it's recommended to use "confirmed" commitment,

    //which balances speed with rollback safety. For total safety, it's recommended to use"finalized" commitment.

  });

}

const instructionMaxSpan = Math.max(

  ...Object.values(LAYOUT.registry).map((r) => r.span),

);

function encodeTokenInstructionData(instruction) {

  let b = Buffer.alloc(instructionMaxSpan);

  let span = LAYOUT.encode(instruction, b);

  return b.slice(0, span);

}

```

How to run this quest

1. Install phantom wallet chrome extension and add some SOL to the wallet.
2. Create a react app 
	1. npx create-react-app create_new_token
	2. cd create_new_token
3. Copy paste the below initiator code in the App.js to invoke createNewMintAuthority.js

```

import './App.css';

import { Connection } from "@solana/web3.js";

import \* as web3 from '@solana/web3.js';

import { createNewMintAuthority } from './utils/createNewMintAuthority';

import { useEffect, useState } from 'react';

const NETWORK = web3.clusterApiUrl("devnet");

const connection = new Connection(NETWORK);

const decimals = 9

function App() {

 const \[provider, setProvider\] = useState()

 const mintNewToken = async () =>{

   if(provider && !provider.isConnected){

       provider.connect()

     }

   try{

     const mintResult = await createNewMintAuthority(provider, decimals, connection)

     console.log(mintResult.signature,'--- signature of the transaction---')

     console.log(mintResult.mintAccount,'----mintAccount---')

   }catch(err){

   }

}

 useEffect(() => {

   if (provider) {

       provider.on("connect", async() => {

         console.log("wallet got connected")

         await mintNewToken()

       });

       provider.on("disconnect", () => {

         console.log("Disconnected from wallet");

       });

   }

 }, \[provider\]);

 useEffect(() => {

   if ("solana" in window && !provider) {

     setProvider(window.solana)

   }

 },\[\])

 return (

   

     

         Create new token

     

   

 );

}

export default App;

```

     5. To install the required dependencies:  “npm install” from the root folder of the project

     6. To run the project : “npm run start” from the root folder of the project

     7. Navigate to [http://localhost:3000](http://localhost:3000) 

     8. Click on the `Create new token` button to create your new token

     9. Open the chrome console and you should see your 

1. Signature
2. mintAccount

\*Local solana cluster will not work on M1 macbook
## What next?
What can you build taking this quest as a base ?

1. Your own custom token
2. A DApp which can have it's own token and you can distribute custom tokens based on the equivalent fiat currency.
3. Create a token which can be used in airdrop campaign.
4. Any DApp which required a minimum number of custom tokens to perform actions like playing games or watching a movie on the DApp.