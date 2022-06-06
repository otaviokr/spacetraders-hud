<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'
import OwnedShipList from './ship/OwnedShipList.vue';

// const props = defineProps(['token'])
const token = ref("")

const quantityToTrade = ref(-1)
const selectedShipId = ref("")
const selectedShip = ref({})
const availableGoods = ref([])

const ownedShips = ref([])

const cargoDetails = ref({})

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

function getAvailableGoods() {
    if (!selectedShipId) {
        return
    }

    let selectedShipLocation = ""

    if(localStorage.goods) {
        let localStorageGoods = JSON.parse(localStorage.getItem('goods'))
        cargoDetails.value = localStorageGoods
    }
    console.log("Cargo Details")
    console.log(cargoDetails)

    // Get ship details to know in which planet it is docked.
    axios.get("https://api.spacetraders.io/my/ships/" + selectedShipId.value, { params: { token: token.value } })
    .then((response) => {
        console.log(response)
        selectedShipLocation = response.data.ship.location

         axios.get("https://api.spacetraders.io/locations/" + selectedShipLocation + "/marketplace", { params: { token: token.value } })
        .then((response) => {
            console.log(response.data)
            availableGoods.value = response.data.marketplace
        })
        .catch(err => {
            console.log(err)
        });
    })
    .catch(err => {
        availableGoods.value = [{
            symbol: "ERROR!!! " + err.message
        }]
        console.log(err)
    });
}

function buyGood(shipId, goodId, quantity) {
    axios.post("https://api.spacetraders.io/my/purchase-orders", {shipId: shipId, good: goodId, quantity: quantity}, { params: { token: token.value } })
    .then((response) => {
        let cargo = {}
        cargo[response.data.order.good] = {
            price: response.data.order.pricePerUnit,
            quantity: response.data.order.quantity
        }

        if(localStorage.goods) {
            let localStorageGoods = JSON.parse(localStorage.getItem('goods'))
            if(localStorageGoods[response.data.order.good]) {
                if (localStorageGoods[response.data.order.good].price < cargo.price) {
                    localStorageGoods[response.data.order.good].price = cargo.price
                }
            } else {
                localStorageGoods[response.data.order.good] = cargo[response.data.order.good]
                console.log("new local json")
                console.log(localStorageGoods)
            }
            localStorage.setItem('goods', JSON.stringify(localStorageGoods));
        } else {
            localStorage.setItem('goods', JSON.stringify(cargo));
        }

        getOwnedShips()
        getAvailableGoods()
    })
    .catch(err => {
        console.log(err)
    });
}

function sellGood(shipId, goodId, quantity) {
    axios.post("https://api.spacetraders.io/my/sell-orders",{shipId: shipId, good: goodId, quantity: quantity}, { params: { token: token.value } })
    .then((response) => {
        if(localStorage.goods) {
            let localStorageGoods = JSON.parse(localStorage.getItem('goods'))
            if(localStorageGoods[response.data.order.good]) {
                if(localStorageGoods[response.data.order.good].quantity < quantity) {
                    localStorageGoods[response.data.order.good].quantity -= quantity
                } else {
                    delete localStorageGoods[response.data.order.good]
                }
            }
            localStorage.setItem('goods', JSON.stringify(localStorageGoods));
        }

        getOwnedShips()
        getAvailableGoods()
    })
    .catch(err => {
        console.log("ERROR!!! " + err)
        console.log(err)
    })
}

onMounted(() => {
    token.value = localStorage.token
    getOwnedShips()
})
</script>

<template>
    <div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
        <h1 class="h2">Marketplace</h1>
    </div>

    <h2>Buying for / selling from ship:
        <span v-if="selectedShipId">{{ selectedShip.manufacturer }} - {{ selectedShip.class }} @ {{ selectedShip.location }}</span>
        <span v-else>NOT SELECTED</span>
    </h2>
    <select v-model="selectedShipId" @change="getDetailsSelectedShip(); getAvailableGoods()">
        <option v-for="ship in ownedShips" :value="ship.id">{{ ship.manufacturer }} - {{ ship.class }} @ {{ ship.location }}</option>
    </select>
    <hr />

    <h2>Goods</h2>
    <div v-if="availableGoods.length > 0" class="table-responsive">
        <table class="table table-striped table-sm">
        <thead>
            <tr>
                <th scope="col">Symbol</th>
                <th scope="col">Available in Dock</th>
                <th scope="col">Volume Per Unit</th>
                <th scope="col">Price Per Unit</th>
                <th scope="col">Available in Cargo</th>
                <th scope="col">Paid Price Per Unit</th>
                <th scope="col">Interest</th>
                <th scope="col">Operation</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="good in availableGoods">
                <td>{{ good.symbol }}</td>
                <td>{{ good.quantityAvailable }}</td>
                <td>{{ good.volumePerUnit }}</td>
                <td>{{ good.pricePerUnit }}</td>
                <td><span v-if="cargoDetails[good.symbol]">{{ cargoDetails[good.symbol].quantity }}</span></td>
                <td><span v-if="cargoDetails[good.symbol]">{{ cargoDetails[good.symbol].price }}</span></td>
                <td>
                    <span v-if="!cargoDetails[good.symbol]" class="badge bg-info">New</span>
                    <span v-else-if="good.pricePerUnit < cargoDetails[good.symbol].price" class="badge bg-warning">Buy</span>
                    <span v-else class="badge bg-danger">Sell</span>
                </td>
                <td>
                    <input v-model="quantityToTrade" :placeholder="good.quantityAvailable" /> &nbsp;
                    <a class="btn btn-primary" href="#" @click="buyGood(selectedShipId, good.symbol, quantityToTrade)">Buy</a> &nbsp;
                    <a class="btn btn-primary" href="#" @click="sellGood(selectedShipId, good.symbol, quantityToTrade)">Sell</a>
                </td>
            </tr>
        </tbody>
        </table>
    </div>
    <p v-else-if="selectedShipId && availableGoods.length < 1">There is no goods available to buy in this dock station.</p>
    <p v-else>You must select a ship to see the avaible goods to buy.</p>
    <hr />
    <OwnedShipList />
</template>

<style>
</style>