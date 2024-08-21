# API for DID
Standard API for verifying and managing decentralized identifiers (DIDs) that allows other dApps to interact with your service for user authentication, you can use a RESTful API approach. This API will provide functionality for creating, verifying, and retrieving information about DIDs.

###1. Project Initialization
```
npm init -y
npm install express body-parser snarkyjs
```

###2. Creating the Server Application
Create a server.js file where the API will be defined:
```
const express = require('express');
const bodyParser = require('body-parser');
const { Field, Signature, PublicKey } = require('snarkyjs');
const { DIDContract } = require('./contracts/DIDContract'); // Importing the contract

const app = express();
app.use(bodyParser.json());

// Example DID storage
let didStorage = {}; // In a real implementation, this could be a database

// Initialize the smart contract
const contract = new DIDContract();

// Route for creating DID
app.post('/create-did', async (req, res) => {
  const { publicKey, did, signature } = req.body;

  try {
    const didField = Field.fromString(did);
    const sig = Signature.fromString(signature);
    const ownerKey = PublicKey.fromBase58(publicKey);

    // Call the smart contract to create the DID
    await contract.createDID(didField, sig);

    // Save the DID to storage
    didStorage[ownerKey.toBase58()] = did;

    res.status(200).send({ message: 'DID successfully created.' });
  } catch (error) {
    res.status(500).send({ error: 'Error creating DID.' });
  }
});

// Route for verifying DID
app.post('/verify-did', async (req, res) => {
  const { publicKey, did } = req.body;

  try {
    const didField = Field.fromString(did);
    const ownerKey = PublicKey.fromBase58(publicKey);

    // Call the smart contract to verify the DID
    const isValid = await contract.verifyDID(didField);

    res.status(200).send({ valid: isValid });
  } catch (error) {
    res.status(500).send({ error: 'Error verifying DID.' });
  }
});

// Route for retrieving DID information
app.get('/get-did-info/:publicKey', (req, res) => {
  const publicKey = req.params.publicKey;

  const did = didStorage[publicKey];

  if (did) {
    res.status(200).send({ did });
  } else {
    res.status(404).send({ error: 'DID not found.' });
  }
});

// Start the server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

###API Route Descriptions
#####POST /create-did

- Description: Creates a new DID for the specified public key.
- Parameters:
    - publicKey: The user's public key.
    - did: The decentralized identifier.
    - signature: A signature proving ownership of the DID.
- Response:
    - Success: { message: 'DID successfully created.' }
    - Error: { error: 'Error creating DID.' }

#####POST /verify-did

- Description: Verifies the validity of the DID for the specified public key.
- Parameters:
    - publicKey: The user's public key.
    - did: The decentralized identifier.
- Response:
    - Success: { valid: true/false }
    - Error: { error: 'Error verifying DID.' }

#####GET /get-did-info/:publicKey

- Description: Returns the information about the DID associated with the specified public key.
- Parameters:
    - publicKey: The user's public key.
- Response:
    - Success: { did: 'User's DID' }
    - Error: { error: 'DID not found.' }
 
This API allows other dApps to interact with your service for managing and verifying DIDs, ensuring secure and private user authentication. 
