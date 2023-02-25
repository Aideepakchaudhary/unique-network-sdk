<script setup lang="ts">
import {onMounted, ref} from "vue";
import {Sdk, TokenByIdResponse} from "@unique-nft/sdk";
import {IPolkadotExtensionAccount, Polkadot} from "@unique-nft/utils/extension";

const sdk = new Sdk({
  baseUrl: 'https://rest.unique.network/opal/v1'  // url for sdk.
})

const accountRef = ref<IPolkadotExtensionAccount | null>(null) // need to explore

onMounted(async() => {
  console.log("Hello, Unique!")
  const result = await Polkadot.enableAndLoadAllWallets()  // get all the wallet from polkadot extension

  accountRef.value = result.accounts[1]  // We'll be using 2nd account
})


const getMyBalance = async () => {    
   const account = accountRef.value    // we are using 2nd account from polkadot extension
   if(!account) {     // If there is no account in the extension
     throw new Error('no account')
   }

  const balance = await sdk.balance.get({    // this is called promise : the function will wait till the data get load.
      address: account.address,
    })
    console.log(balance)
}


const transferBalance = async () => {
  const account = accountRef.value
  
  if(!account) {
    throw new Error('no account')
  }

  const transferResult = await sdk.balance.transfer.submitWaitResult({    // transfer some balance.
    address: account.address,
    destination: '5EL9nwBv52s8WE1dpoGN9g7BfLqW2Uzn32cqPp873TvoWg8N',
    amount: 1,
  }, { 
    signer: account.uniqueSdkSigner     // make the transaction signed
  })

  console.log(transferResult)
  console.log(transferResult.parsed)

}

const transferToken = async () => {
  const account = accountRef.value

  if(!account) {
    throw new Error('no account')
  }

  const transferToken = await sdk.tokens.transfer.submitWaitResult({
    address: account.address,
    to: '5EL9nwBv52s8WE1dpoGN9g7BfLqW2Uzn32cqPp873TvoWg8N',
    collectionId: 467,
    tokenId: 1,
  }, {
    signer: account.uniqueSdkSigner   // Make the transaction signed
  })

  console.log(transferToken.parsed)
}

const createCollection = async () => {    // create a new collection
  const account = accountRef.value
  if(!account) {
    throw new Error('no account')
  }

  const collectionCreationResult = await sdk.collections.creation.submitWaitResult({
    address: account.address,
    name: 'Demo collection1',
    description: 'My test collection',
    tokenPrefix: 'DEEP',
    permissions: {
      nesting: {
        tokenOwner: true,
        collectionAdmin: true,
      }
    },
    schema: {     // need to explore
      schemaName: 'unique',
      schemaVersion: '1.0.0',
      image: {
        urlTemplate: 'https://gateway.pinata.cloud/ipfs/{infix}'
      },
      coverPicture: {
        ipfsCid: 'QmfMTg3LitMcKKCCWP7Qa2cE1pMpCn37XPWJajqQTAXwAn'
      },
      attributesSchemaVersion: '1.0.0',
      attributesSchema: {
        0: {
          name: {_: 'color'},
          type: 'string',
          optional: true,
          enumValues: {
            0: {_: 'red'},
            1: {_: 'green'},
            2: {_: 'blue'},
          }
        },
      }
    }
  }, {
    signer: account.uniqueSdkSigner
  })

  console.log(collectionCreationResult.parsed)
}


const mintToken = async () => {     // Mint token
  const account = accountRef.value
  if(!account) {
    throw new Error('no account')
  }

  const collectionId = 660

  const tokenResult = await sdk.tokens.create.submitWaitResult( {
    address: account.address,
    collectionId,
    data: {
      image: {
        ipfsCid: 'QmfMTg3LitMcKKCCWP7Qa2cE1pMpCn37XPWJajqQTAXwAn'
      },
      encodedAttributes: {
        0: 0
      }
    }
  }, {
    signer: account.uniqueSdkSigner
  })

  console.log(tokenResult.parsed)
}

const nestToken = async() => {
  const account = accountRef.value
  if(!account) {
    throw new Error('no account')
  }

  const nestTokenResult = await sdk.tokens.nest.submitWaitResult( {
    address: account.address,
    parent: {
      collectionId: 179, 
      tokenId: 1,  // this token will be the parent of token 1 -> collection 180
    },
    nested: {
      collectionId: 180,
      tokenId: 1,
    },
  }, {
    signer: account.uniqueSdkSigner
  })

  console.log(nestTokenResult.parsed)
}

const checkBundle = async() => {
  const account = accountRef.value
  if(!account) {
    throw new Error('no account')
  }

  const checkBundleResult: any = await sdk.tokens.getBundle({
    collectionId: 179,
    tokenId: 1,
  })

  console.log('getBundle', checkBundleResult)
}



const tokenRef = ref<TokenByIdResponse | null> (null)
const getToken = async (collectionId: number, tokenId: number) => {       // get a token
  const token = await sdk.tokens.get({
    collectionId,
    tokenId,
  })
  tokenRef.value = token

  console.log(token)
}


</script>

<template>
Hello Unique!

  <br/><button @click = "getMyBalance">Get My Balance</button><br/>
  <br/><button @click = "transferBalance">Transfer Balance</button><br/>
  <br/><button @click = "createCollection">Create Collection</button><br/>
  <br/><button @click = "mintToken">MintToken</button><br/>
  <br/><button @click = "getToken(467,1)">Get Token</button><br/>
  <br/><button @click = "transferToken">Transfer Token</button><br/>
  <br/><button @click = "nestToken">Nest Token</button><br/>
  <br/><button @click = "checkBundle">Get Bundle</button><br/>


</template>

<style scoped>

</style>
