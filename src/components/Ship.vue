<script setup>
import { onMounted, ref } from 'vue';
import axios from 'axios';
import OwnedShipList from './ship/OwnedShipList.vue';

// const props = defineProps(['token'])
const token = ref("")
const availableShips = ref([]);

function getAvailableShips() {
    // FIXME We are using a hardcoded system code.
    axios.get("https://api.spacetraders.io/systems/OE/ship-listings", { params: { token: token.value } })
    .then((response) => {
        for(const ship of response.data.shipListings) {
            for(const location of ship.purchaseLocations) {
                availableShips.value.push({
                    class: ship.class,
                    location: location.location,
                    manufacturer: ship.manufacturer,
                    maxCargo: ship.maxCargo,
                    plating: ship.plating,
                    price: location.price,
                    speed: ship.speed,
                    system: location.system,
                    type: ship.type,
                    weapons: ship.weapons
                })
            }
        }
    })
    .catch(err => {
        availableShips.value = [{
            type: "ERROR!!! " + err
        }]
    });
}

function buyShip(location, type) {
    axios.get("https://api.spacetraders.io/my/loans", { params: { token: token.value, location: location, type: type } })
    .then((response) => {
        ownedShips.value = response.data.ship
    })
    .catch(err => {
        ownedShips.value = [{
            type: "ERROR!!! " + err
        }]
    });
}

onMounted(() => {
    token.value = localStorage.token
    getAvailableShips()
})

</script>

<template>
    <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
        <h1 class="h2">Ships</h1>
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

    <OwnedShipList />

    <h2>Ships vailable to buy</h2>
    <div class="table-responsive">
        <table class="table table-striped table-sm">
        <thead>
            <tr>
                <th scope="col">Manufacturer</th>
                <th scope="col">Class</th>
                <th scope="col">Type</th>
                <th scope="col">Speed</th>
                <th scope="col">Weapons</th>
                <th scope="col">Plating</th>
                <th scope="col">Max Cargo</th>
                <th scope="col">System</th>
                <th scope="col">Location</th>
                <th scope="col">Price</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="ship in availableShips">
                <td>{{ ship.manufacturer }}</td>
                <td>{{ ship.class }}</td>
                <td>{{ ship.type }}</td>
                <td>{{ ship.speed }}</td>
                <td>{{ ship.weapons }}</td>
                <td>{{ ship.plating }}</td>
                <td>{{ ship.maxCargo }}</td>
                <td>{{ ship.system }}</td>
                <td>{{ ship.location }}</td>
                <td>{{ ship.price }}</td>
                <td><a href="#" @click="buyShip(ship.location, ship.type)">Buy this ship</a></td>
            </tr>
        </tbody>
        </table>
    </div>
</template>

<style>
</style>