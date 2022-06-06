<script setup>
import { ref } from 'vue';
import axios from 'axios';

const currentStatus = ref("Unknown")

function checkStatus() {
    axios.get("https://api.spacetraders.io/game/status")
    .then((response) => {
        currentStatus.value = response.data.status;
    })
    .catch(err => {
        currentStatus.value = "ERROR!!! " + err.data;
    });
}
</script>


<template>
    <button @click="checkStatus">{{ currentStatus }}</button>
</template>

<style>
</style>