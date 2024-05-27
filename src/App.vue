<script setup>
import { computed, provide, ref, watch } from 'vue'

import Header from './components/Header.vue'
import Drawer from './components/Drawer.vue'

const cart = ref([])
const drawerIsOpen = ref(false)

const totalPrice = computed(() => cart.value.reduce((acc, item) => acc + item.price, 0))
const vatPrice = computed(() => Math.round(totalPrice.value * 0.05))


const closeDrawer = () => {
  drawerIsOpen.value = false
}

const openDrawer = () => {
  drawerIsOpen.value = true
}

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1)
  item.isAdded = false
}

const addToCart = (item) => {
    cart.value.push(item);
    item.isAdded = true;
}

watch(
  cart,
  () => {
    localStorage.setItem('cart', JSON.stringify(cart.value))
  },
  { deep: true }
)

provide('cart', {
  cart,
  openDrawer,
  closeDrawer,
  removeFromCart,
  addToCart
})
</script>

<template>
  <Drawer
    v-if="drawerIsOpen"
    :total-price="totalPrice"
    :vat-price="vatPrice"
  />
  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <Header @open-drawer="openDrawer" :total-price="totalPrice" />
    <div class="p-10">
      <router-view></router-view>
    </div>
  </div>
</template>

<style scoped></style>
