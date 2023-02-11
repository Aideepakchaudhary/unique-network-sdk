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
    schema: {     // need to explore
      schemaName: 'unique',
      schemaVersion: '1.0.0',
      image: {
        urlTemplate: 'https://gateway.pinata.cloud/ipfs/{infix}'
      },
      coverPicture: {
        ipfsCid: 'QmNiBHiAhsjBXj5cXShDUc5q1dX23CJYrqGGPBNjQCCSXQ'
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

  const collectionId = 382

  const tokenResult = await sdk.tokens.create.submitWaitResult( {
    address: account.address,
    collectionId,
    data: {
      image: {
        ipfsCid: 'QmNiBHiAhsjBXj5cXShDUc5q1dX23CJYrqGGPBNjQCCSXQ'
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
  <br/><button @click = "getToken(382,1)">Get Token</button><br/>

</template>

<style scoped>

</style>
