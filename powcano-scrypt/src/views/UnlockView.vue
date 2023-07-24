<template>
  <div class="about">
    <h1>This is the unlock page</h1>
    <!-- The TXID of the contract you are trying to unlock -->
    <!-- From the path that we are on '/:id' -->
    <p>TXID: {{ route.params.id }}</p>

    <!-- Message input -->
    <input type="text" v-model="unlockMessage" />
    <!-- Login button -->
    <button @click="login">Login</button>
    <!-- Deploy Contract -->
    <button @click="unlock">Unlock</button>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

const unlockMessage = ref(null)
const contractTXID = ref(route.params.id)

import { Helloworld } from '../contracts/helloworld';
import artifact from '../../artifacts/helloworld.json';
import { sha256, toByteString, DefaultProvider, SensiletSigner, } from 'scrypt-ts';

const signer = ref<SensiletSigner>();
const instance = ref<Helloworld>();

onMounted(() => {
  console.log('Unlock txid view mounted');
  Helloworld.loadArtifact(artifact)

  console.log('Helloworld', Helloworld);
  const provider = new DefaultProvider({});
  signer.value = new SensiletSigner(provider);
})



// login function
const login = async () => {
  console.log('Login function', instance.value, signer.value)
  // request authentication
  if (signer.value) {
    try {
      const { isAuthenticated, error } = await signer.value.requestAuth();
      if (!isAuthenticated) {
        // something went wrong, throw an Error with `error` message
        // throw new Error(error);
        alert('Authentication error: ' + error)
      }
      // authenticated
      // you can show user's default address
      const userAddress = await signer.value.getDefaultAddress();
      console.log('User address', userAddress);
      alert('Logged in user is: ' + userAddress)
      console.log('Connected to Scrypt smart contract instance', instance.value);

      if (contractTXID.value) {
        console.log('contractTXID', contractTXID.value[0])
        const tx = await signer.value.connectedProvider.getTransaction(contractTXID.value)
        instance.value = Helloworld.fromTx(tx, 0)

        console.log('instance', instance);

        const provider = new DefaultProvider({});
        signer.value = new SensiletSigner(provider);

        await instance.value.connect(signer.value);
      }

      // ...
    } catch (error) {
      alert('Sensilet error is: ' + error)
    }
  }

}

// deploy function
const unlock = async() => {
  console.log('Unlock function', instance.value, signer.value, unlockMessage.value)
  if(instance.value && unlockMessage.value) {
  // Call the 'unlock' method of our deployed HelloWorld contract instance
  const hexUnlockMessage = toByteString(unlockMessage.value, true)
  const { tx: callTx } = await instance.value.methods.unlock(hexUnlockMessage)
  console.log('Helloworld contract `unlock` called: ', callTx.id)
  }
}
</script>

