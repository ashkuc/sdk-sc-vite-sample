<script setup lang="ts">

import {ExtensionTools, Ethereum, Polkadot} from '@unique-nft/utils/extension'
import {ethers} from 'ethers';
import {Address} from "@unique-nft/utils";
import {Sdk} from '@unique-nft/sdk';

import {bytecode, abi} from './assets/storage.contract'


const provider = new ethers.BrowserProvider(window.ethereum);

const sdk = new Sdk({ baseUrl: 'https://rest.unique.network/opal/v1' });

let contract;
let contractAddress = '0xd0D9CBb7B45B205eeD756a403Aec60bD50EA9411';

const switchToOpal = async () => {
  if (!ExtensionTools.Ethereum.currentChainIs.opal()) {
    await ExtensionTools.Ethereum.switchChainTo.opal();
  }

  const  { address, error } = await Ethereum.getOrRequestAccounts(true);

  if (error) {
    console.log(error);

    return;
  }

  const balance = ethers.formatEther(await provider.getBalance(address!));

  console.log([
    `Switched to OPAL`,
    `Address: ${address}`,
    `Mirror: ${Address.mirror.ethereumToSubstrate(address!)}`,
    `Balance is ${balance}`,
  ].join('\n\n'));
}

const deploy = async () => {
  const factory = new ethers.ContractFactory(abi, bytecode, await provider.getSigner());
  contract = await factory.deploy();
  await contract.waitForDeployment();

  contractAddress = await contract.getAddress();

  console.log(contractAddress);
}

const checkExists = async () => {
  const exists = await sdk.evm.contractExists({ contractAddress });

  console.dir(exists);
}

const callStore = async () => {
  const { accounts: [account]  } = await Polkadot.enableAndLoadAllWallets();

  const random = '0x' + Math.floor(Math.random() * 100).toString(16);

  const result = await sdk.evm.send.submitWaitResult({
    abi,
    funcName: 'store',
    address: account.address,
    contractAddress,
    args: [random as any],
  }, { signer: account.uniqueSdkSigner });

  console.dir(result);
}

const callRetrieve = async () => {
  const { accounts: [account] } = await Polkadot.enableAndLoadAllWallets();

  const result = await sdk.evm.call({ abi, contractAddress, address: account.address, funcName: 'retrieve' });

  console.dir(result);
}

</script>

<template>
  <div>
    <button @click="switchToOpal">Switch to Opal</button>
    <button @click="deploy">Deploy contract</button>
    <button @click="checkExists">Check contract exists</button>
    <button @click="callStore">Call Store</button>
    <button @click="callRetrieve">Call Retrieve</button>
  </div>
</template>

<style scoped>
button {
  display: block;
  margin: 30px;
}
</style>
