<script setup>
// TODO Using hardedcoded location
import { onMounted, ref } from 'vue'
import axios from 'axios'
import OwnedShipList from './ship/OwnedShipList.vue';

// const props = defineProps(['token'])
const token = ref("")

const selectedShipId = ref("")
const selectedShip = ref({})
const availableLocation = ref([])
const myFlightPlans = ref([])

const ownedShips = ref([])

function getOwnedShips() {
    axios.get("https://api.spacetraders.io/my/ships", { params: { token: token.value } })
    .then((response) => {
        ownedShips.value = response.data.ships
    })
    .catch(err => {
        ownedShips.value = [{
            type: "ERROR!!! " + err
        }]
    });
}

function getDetailsSelectedShip() {
    console.log("Entered function...")
    console.log(ownedShips.value)
    if(ownedShips.length < 1) {
        return
    }
    for(let index in ownedShips.value) {
        console.log("Inside the loop...")
        console.log(selectedShipId.value)
        console.log(ownedShips.value[index])
        console.log("Inside the loop - testing the if...")
        if(ownedShips.value[index].id == selectedShipId.value) {
            selectedShip.value = ownedShips.value[index]
            console.log(selectedShip)
            return
        }
    }
}

function getAvailableLocations() {
    if (!selectedShipId) {
        return
    }

    let selectedShipSystem = ""

    // Get ship details to know in which planet it is docked.
    axios.get("https://api.spacetraders.io/my/ships/" + selectedShipId.value, { params: { token: token.value } })
    .then((response) => {
        console.log(response)
        selectedShipSystem = response.data.ship.location.split("-")[0]
        // availableGoods.value = response.data.marketplace

        // axios.get("https://api.spacetraders.io/systems/" + selectedShipSystem + "/locations", { params: { token: token.value, type: 'PLANET' } })
        axios.get("https://api.spacetraders.io/systems/" + selectedShipSystem + "/locations", { params: { token: token.value } })
        .then((response) => {
            availableLocation.value = response.data.locations
        })
        .catch(err => {
            console.log(err)
        });
    })
    .catch(err => {
        console.log(err)
    });
}

function defineFlightPlan(shipId, destination) {
    axios.post("https://api.spacetraders.io/my/flight-plans", {shipId: shipId, destination: destination}, { params: { token: token.value } })
    .then(() => {
        getOwnedShips()
        getAvailableLocations()
    })
    .catch(err => {
      console.log(err)
    });
}

function getFlightPlans() {
    console.log(ownedShips)
    for(const ship in ownedShips) {
        console.log(ship)
        if(ship.flightPlanId) {
            axios.get("https://api.spacetraders.io/my/flight-plans/" + ship.flightPlanId, { params: { token: token.value } })
            .then((response) => {
                myFlightPlans.value = myFlightPlans.value.concat(response.data)
            })
            .catch(err => {
                myFlightPlans.value = [{
                    symbol: "ERROR!!! " + err
                }]
            });
        }
    }
}

function doWarpJump() {
    // You can jump only when you have no active loans...
    axios.post("https://api.spacetraders.io/my/warp-jumps", {shipId: selectedShipId.value}, { params: { token: token.value } })
    .then(() => {
      getFlightPlans()
    })
    .catch(err => {
      console.log(err)
    });
}

onMounted(() => {
    token.value = localStorage.token
    getOwnedShips()
    // getFlightPlans()
    // getAvailableLocations()
})

</script>

<template>
    <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
        <h1 class="h2">Planets</h1>
        <div v-if="selectedShipId" class="btn-toolbar mb-2 mb-md-0">
        <button type="button" class="btn btn-sm btn-outline-secondary" @click="doWarpJump()">
            Warp drive engage
        </button>
        </div>
    </div>

    <h2>Planets nearby of ship:
        <span v-if="selectedShipId">{{ selectedShip.manufacturer }} - {{ selectedShip.class }} @ {{ selectedShip.location }}</span>
        <span v-else>NOT SELECTED</span>
    </h2>
    <select v-model="selectedShipId" @change="getDetailsSelectedShip(); getAvailableLocations()">
        <option v-for="ship in ownedShips" :value="ship.id">{{ ship.manufacturer }} - {{ ship.class }} @ {{ ship.location }}</option>
    </select>
    <hr />
    <div v-if="availableLocation.length > 0" class="table-responsive">
        <table class="table table-striped table-sm">
        <thead>
            <tr>
                <th scope="col">Name</th>
                <th scope="col">Symbol</th>
                <th scope="col">Type</th>
                <th scope="col">X</th>
                <th scope="col">Y</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="location in availableLocation">
                <td>{{ location.name }}</td>
                <td>{{ location.symbol }}</td>
                <td>{{ location.type }}</td>
                <td>{{ location.x }}</td>
                <td>{{ location.y }}</td>
                <td><a href="#" @click="defineFlightPlan(selectedShipId, location.symbol)">Define Flight Plan</a></td>
            </tr>
        </tbody>
        </table>
    </div>
    <p v-else>You must select a ship to see the planets nearby.</p>

    <OwnedShipList />
</template>

<style>
</style>