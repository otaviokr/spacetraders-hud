<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'

const token = ref("")
const ownedShips = ref([])

function getOwnedShips() {
    console.log(token.value)
    axios.get("https://api.spacetraders.io/my/ships", { params: { token: token.value } })
    .then((response) => {
        // ownedShips.value = response.data.ships
        for(const index in response.data.ships) {
            let ship = response.data.ships[index]
            if(ship.flightPlanId) {
                axios.get("https://api.spacetraders.io/my/flight-plans/" + ship.flightPlanId, { params: { token: token.value } })
                .then((response) => {
                    ownedShips.value.push({
                        manufacturer: ship.manufacturer,
                        class: ship.class,
                        type: ship.type,
                        speed: ship.speed,
                        weapons: ship.weapons,
                        plating: ship.plating,
                        cargo: ship.cargo,
                        maxCargo: ship.maxCargo,
                        location: "Reaching " + response.data.flightPlan.destination + " in " + response.data.flightPlan.timeRemainingInSeconds + "s",
                        spaceAvailable: ship.spaceAvailable
                    })
                })
                .catch(err => {
                    console.log(err)
                });
            } else {
                ownedShips.value.push({
                    manufacturer: ship.manufacturer,
                    class: ship.class,
                    type: ship.type,
                    speed: ship.speed,
                    weapons: ship.weapons,
                    plating: ship.plating,
                    cargo: ship.cargo,
                    maxCargo: ship.maxCargo,
                    location: ship.location,
                    spaceAvailable: ship.spaceAvailable,
                    x: ship.x,
                    y: ship.y
                })
            }
            console.log(ship)
            console.log(ownedShips)
        }
    })
    .catch(err => {
        console.log(err)
    });
}

onMounted(() => {
    token.value = localStorage.token
    getOwnedShips()
})
</script>

<template>
<h2>Owned ships</h2>
<div v-if="ownedShips.length > 0" class="table-responsive">
    <table class="table table-striped table-sm">
    <thead>
        <tr>
            <th scope="col">Manufacturer</th>
            <th scope="col">Class</th>
            <th scope="col">Type</th>
            <th scope="col">Speed</th>
            <th scope="col">Weapons</th>
            <th scope="col">Plating</th>
            <th scope="col">Cargo (Qtt / Total)</th>
            <th scope="col">Max Cargo</th>
            <th scope="col">Location</th>
            <th scope="col">Space Available</th>
            <th scope="col">Coordinates</th>
        </tr>
    </thead>
    <tbody>
        <tr v-for="ship in ownedShips">
            <td>{{ ship.manufacturer }}</td>
            <td>{{ ship.class }}</td>
            <td>{{ ship.type }}</td>
            <td>{{ ship.speed }}</td>
            <td>{{ ship.weapons }}</td>
            <td>{{ ship.plating }}</td>
            <td>
                <ul v-if="ship.cargo && ship.cargo.length > 0">
                    <li v-for="good in ship.cargo">{{ good.good }} ({{ good.quantity }} / {{ good.totalVolume }})</li>
                </ul>
                <span v-else>No cargo</span>
            </td>
            <td>{{ ship.maxCargo }}</td>
            <td>{{ ship.location }}</td>
            <td>{{ ship.spaceAvailable }}</td>
            <td v-if="ship.x && ship.y">{{ ship.x }}, {{ ship.y }}</td>
            <td v-else>Flying (uncertain)</td>
        </tr>
    </tbody>
    </table>
</div>
<p v-else>You don't own a ship. Why not buy a cheap one?</p>
</template>