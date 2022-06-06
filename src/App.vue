<script setup>
import { onMounted, provide, ref, watch } from 'vue'
import Account from './components/Account.vue'
import Loan from './components/Loan.vue'
import Marketplace from './components/Marketplace.vue'
import Planet from './components/Planet.vue'
// import Profile from './components/Profile.vue'
import Server from './components/Server.vue'
import Ship from './components/Ship.vue'
import Sidemenu from './components/Sidemenu.vue'

const currentView = ref("Account")
// const token = ref("")

function setCurrentView(newView) {
  currentView.value = newView
}

provide('currentView', {
  currentView,
  setCurrentView
})
</script>

<template>
    <header class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0 shadow">
        <a class="navbar-brand col-md-3 col-lg-2 me-0 px-3" href="#">Space Traders HQ</a>
        <Server />
        <!-- <button class="navbar-toggler position-absolute d-md-none collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#sidebarMenu" aria-controls="sidebarMenu" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button> -->
        <!-- <input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search"> -->
        <div class="navbar-nav">
          <!-- <div class="nav-item text-nowrap">
            <Profile />
          </div> -->
        </div>
      </header>
      <div class="container-fluid">
        <div class="row">
          <nav id="sidebarMenu" class="col-md-3 col-lg-2 d-md-block bg-light sidebar collapse">
            <Sidemenu />
          </nav>

          <main class="col-md-9 ms-sm-auto col-lg-10 px-md-4">
            <div class="chartjs-size-monitor">
              <div class="chartjs-size-monitor-expand">
                <div class=""></div>
              </div>
              <div class="chartjs-size-monitor-shrink">
                <div class=""></div>
              </div>
            </div>
            <div v-if="currentView == 'Account'"><Account /></div>
            <div v-else-if="currentView == 'Loan'"><Loan /></div>
            <div v-else-if="currentView == 'Marketplace'"><Marketplace /></div>
            <div v-else-if="currentView == 'Ship'"><Ship /></div>
            <div v-else-if="currentView == 'Planet'"><Planet /></div>
            <div v-else>
              <p>Select one option from the menu on the left.</p>
            </div>
          </main>
        </div>
      </div>
</template>

<style>
</style>