<script setup>
import { onMounted, ref } from 'vue';
import axios from 'axios';

// const props = defineProps(['token'])
const token = ref("")

const availableLoans = ref([]);
const takenLoans = ref([])

function getAvailableLoans() {
    axios.get("https://api.spacetraders.io/types/loans", { params: { token: token.value } })
    .then((response) => {
        availableLoans.value = response.data.loans
    })
    .catch(err => {
        availableLoans.value = [{
            type: "ERROR!!! " + err
        }]
    });
}

function takeLoan(loanType) {
    axios.get("https://api.spacetraders.io/my/loans", { params: { token: token.value, type: loanType } })
    .then((response) => {
        takenLoans.value = response.data.loan
    })
    .catch(err => {
        takenLoans.value = [{
            type: "ERROR!!! " + err
        }]
    });
}

function pendingLoan() {
    axios.get("https://api.spacetraders.io/my/loans", { params: { token: token.value } })
    .then((response) => {
        takenLoans.value = response.data.loans
    })
    .catch(err => {
        takenLoans.value = [{
            type: "ERROR!!! " + err
        }]
    });
}

onMounted(() => {
    token.value = localStorage.token
    getAvailableLoans()
    pendingLoan()
})

</script>

<template>
    <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
        <h1 class="h2">Loans</h1>
        <div class="btn-toolbar mb-2 mb-md-0">
        <div class="btn-group me-2">
            <button type="button" class="btn btn-sm btn-outline-secondary">Share</button>
            <button type="button" class="btn btn-sm btn-outline-secondary">Export</button>
        </div>
        <button type="button" class="btn btn-sm btn-outline-secondary dropdown-toggle">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-calendar" aria-hidden="true"><rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect><line x1="16" y1="2" x2="16" y2="6"></line><line x1="8" y1="2" x2="8" y2="6"></line><line x1="3" y1="10" x2="21" y2="10"></line></svg>
            This week
        </button>
        </div>
    </div>

    <h2>Available loans</h2>
    <div class="table-responsive">
        <table class="table table-striped table-sm">
        <thead>
            <tr>
                <th scope="col">Type</th>
                <th scope="col">Rate</th>
                <th scope="col">Collateral?</th>
                <th scope="col">Days to Pay</th>
                <th scope="col">Amount</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="loan in availableLoans">
                <td>{{ loan.type }}</td>
                <td>{{ loan.rate }}</td>
                <td v-if="loan.collateralRequired">Yes</td><td v-else>No</td>
                <td>{{ loan.termInDays }}</td>
                <td>{{ loan.amount }}</td>
                <td><a href="#" @click="takeLoan(loan.type)">Take this loan</a></td>
            </tr>
        </tbody>
        </table>
    </div>

    <h2>Loans taken</h2>
    <div v-if="takenLoans.length > 0" class="table-responsive">
        <table class="table table-striped table-sm">
        <thead>
            <tr>
                <th scope="col">Status</th>
                <th scope="col">Type</th>
                <th scope="col">Due Date</th>
                <th scope="col">Amount</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="loan in takenLoans">
                <td>{{ loan.status }}</td>
                <td>{{ loan.type }}</td>
                <td>{{ loan.due }}</td>
                <td>{{ loan.repaymentAmount }}</td>
                <td><a href="#" @click="takeLoan(loan.id)">Pay it off</a></td>
            </tr>
        </tbody>
        </table>
    </div>
    <p v-else>You don´t have any active loan to pay.</p>
</template>

<style>
</style>