# sigpair-js

## Installation
Authenticate with auth token and setup registry
```bash
npm config set -- //registry.npmjs.org/:_authToken=YOU_NPM_AUTH_TOKEN
```

Install 

```bash
npm install https://github.com/silence-laboratories/sigpair-client-js 
```



## Example Usage

```typescript
import * as ed from "@noble/ed25519";
import {
  generatePartyKeys,
} from "@silencelaboratories/two-party-ecdsa-js";
import {
  SigpairClient,
  generateSigningKeys
} from "sigpair-client";

// Sigpair Admin in JS: https://github.com/silence-laboratories/sigpair-admin-v2
import {
  SigpairAdmin,
} from "sigpair-admin";

import * as ed from "@noble/ed25519";
import {
  generatePartyKeys,
} from "@silencelaboratories/two-party-ecdsa-js";
import {
  SigpairClient,
  generateSigningKeys
} from "sigpair-client";

// Sigpair Admin in JS: https://github.com/silence-laboratories/sigpair-admin-v2
// You can use an admin instance in any language, as long as it implements the Sigpair Admin API
import {
  SigpairAdmin,
} from "sigpair-admin";
async function main() {
	// Assuming you have a Sigpair Admin running locally at http://localhost:8080
	// Configured with the following admin token 
  const admin = new SigpairAdmin(
    "1ec3804afc23258f767b9d38825dc7ab0a2ea44ef4adf3254e4d7c6059c3b55a",
    "http://localhost:8080"
  );
  // Create a new user
  const userId = await admin.createUser("test");
  // Generate a signing keypair for the user using convience method. You can also generate a ed25519 keypair yourself.
  const keypair = await generateSigningKeys();
  // Generate a new user token for the user, for given user-id and public key and lifetime in seconds
  const userToken = admin.genUserToken(userId, keypair.publicKey, 60*60);

  // Create a new Sigpair Client
  const client = new SigpairClient(
    userToken,
    keypair.signingKey,
    "http://localhost:8080"
  );

  // Generate party keys for keygen
  const partyKeys = await generatePartyKeys();
  // Generate a new ECDSA keyshare 
  const keyshare = await client.genECDSAKey(partyKeys);
  console.log("Created keyshare: ", keyshare.data.key_id);

  // Generate a random message hash (ONLY for demo purposes. In practice the message MUST be hashed)
  const msgHash = ed.etc.bytesToHex(crypto.getRandomValues(new Uint8Array(32)));

  // Sign the message hash with the keyshare
  const sign = await client.genECDSASign(keyshare, msgHash);
  console.log("Signature: ", sign.sign);
  // Refresh the keyshare
  const key2 = await client.refreshECDSAKey(keyshare, partyKeys);
  console.log("Refreshed keyshare: ", key2.data.key_id);
  // Generate a new EdDSA keyshare
  const ed_key = await client.genEdDSAKey();
  console.log("Created EdDSA: ", ed_key.share.key_id);

  // Sign the message hash with the keyshare
  const sign2 = await client.genEdDSASign(ed_key, msgHash);
  console.log("EdDSA Signature: ", Buffer.from(sign2).toString("base64"));

  // Refresh the EdDSA keyshare
  const new_key = await client.refreshEdDSAKey(ed_key);
  console.log("Refreshed EdDSA: ", new_key.share.key_id);
}

main();
```



