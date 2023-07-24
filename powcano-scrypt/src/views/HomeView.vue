<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <!-- Message input -->
    <input type="text" v-model="message" />
    <!-- Login button -->
    <button @click="login">Login</button>
    <!-- Deploy Contract -->
    <button @click="deploy">Deploy</button>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { Helloworld } from '../contracts/helloworld';
import artifact from '../../artifacts/helloworld.json';
import { sha256, toByteString, DefaultProvider, SensiletSigner, } from 'scrypt-ts';

const signer = ref<SensiletSigner>();
const instance = ref<Helloworld>();
const message = ref<string>('');

onMounted(() => {
  console.log('Home view mounted');

  Helloworld.loadArtifact(artifact);



  console.log('Helloworld', Helloworld);

  const provider = new DefaultProvider({});
  signer.value = new SensiletSigner(provider);
});



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
      // ...
    } catch (error) {
      alert('Sensilet error is: ' + error)
    }
  }

}

// deploy function
const deploy = async() => {
  console.log('Deploy function', instance.value, signer.value)

  // create a new instance of the contract
  const messageInBytes = toByteString(message.value, true)
  instance.value = new Helloworld(sha256(messageInBytes))

  // connect the instance to the signer & deploy the contract
  if(instance.value && signer.value) {
    await instance.value.connect(signer.value);

    console.log('Scrypt smart contract instance', instance.value);

    // deploy the contract and lock up 42 satoshis in it
    const deployTx = await instance.value.deploy(42)
    console.log('Helloworld contract deployed: ', deployTx.id)
    alert('Helloworld contract deployed: ' + deployTx.id)
  }

}
</script>
