<script setup>
import { onMounted, reactive, ref } from 'vue';
import axios from 'axios';

const token = ref("")

const account = reactive({
    credits: 0,
    joinedAt: "",
    shipCount: 0,
    structureCount: 0,
    username: ""
});

function getAccountDetails() {
    axios.get("https://api.spacetraders.io/my/account", { params: { token: token.value } })
    .then((response) => {
        account.credits = response.data.user.credits;
        account.joinedAt = response.data.user.joinedAt;
        account.shipCount = response.data.user.shipCount;
        account.structureCount = response.data.user.structureCount;
        account.username = response.data.user.username;
    })
    .catch(err => {
        currentStatus.value = "ERROR!!! " + err;
    });
}

function saveToken() {
    localStorage.token = token.value
}

onMounted(() => {
    token.value = localStorage.token
    getAccountDetails();
})

</script>

<template>
    <h2>Account Details</h2>
    <form class="row mb-3">
        <label for="userToken" class="col-sm-1 col-form-label">Token</label>
        <div class="col-sm-3">
            <input type="text" class="form-control" :id="token" placeholder="Enter your token">
        </div>
        <div class="col-sm-3">
            <button type="submit" class="btn btn-primary mb-3" @click="saveToken()">Save token</button>
        </div>
    </form>
    <hr />
    <div class="ul">
        <ul class="list-group">
            <li class="list-group-item">Name: {{ account.username }}</li>
            <li class="list-group-item">Joined at {{ account.joinedAt }}</li>
            <li class="list-group-item">Total credits: {{ account.credits }}</li>
            <li class="list-group-item">Total ships: {{ account.shipCount }}</li>
            <li class="list-group-item">Total structures: {{ account.structureCount }}</li>
        </ul>
    </div>
</template>

<style>
</style>