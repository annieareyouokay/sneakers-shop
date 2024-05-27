<script setup>
import axios from 'axios'
import { inject, ref, computed } from 'vue'

import DrawerHead from './DrawerHead.vue'
import CartItemList from './CartItemList.vue'
import infoBlock from './infoBlock.vue'


const props = defineProps({
  totalPrice: Number,
  vatPrice: Number,
})
const isOrderCreating = ref(false)
const orderId = ref(null);
const { cart, closeDrawer } = inject('cart')

const createOrder = async () => {
  try {
    isOrderCreating.value = true
    const { data } = await axios.post(`https://70d04dcd5a23ce3c.mokky.dev/orders`, {
      items: cart.value,
      totalPrice: props.totalPrice.value
    })
    cart.value = []
    orderId.value = data.id;
    return data
  } catch (error) {
    console.error(error)
  } finally {
    isOrderCreating.value = false
  }
}
const cartIsEmpty = computed(() => cart.value.length === 0)
const buttonIsDisabled = computed(() => isOrderCreating.value || cartIsEmpty.value)
</script>

<template>
  <div @click="closeDrawer" class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    <DrawerHead />

    <div v-if="!totalPrice || orderId" class="flex h-full items-center">
      <infoBlock
      v-if="!totalPrice && !orderId"
        title="Корзина пустая"
        description="Добавьте хотя бы одну пару кроссовок, чтобы сделать заказ"
        image-url="/package-icon.png"
      />
      <infoBlock
        v-else-if="orderId"
        title="Ваш заказ оформлен"
        :description="`Заказ с номером №${orderId} успешно оформлен и будет передан в службу доставки`"
        image-url="/order-success-icon.png"
      />
    </div>
    <div v-else>
      <CartItemList />

      <div class="flex flex-col gap-4 my-7">
        <div class="flex gap-2">
          <span>Итого:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ totalPrice }} P</b>
        </div>
        <div class="flex gap-2">
          <span>Налог 5%:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ vatPrice }} P</b>
        </div>
        <button
          :disabled="buttonIsDisabled"
          @click="createOrder"
          class="transition bg-lime-500 w-full rounded-xl py-3 text-white disabled:bg-slate-300 hover:bg-lime-600 active:bg-lime-700 cursor-pointer"
        >
          Оформить заказ
        </button>
      </div>
    </div>
  </div>
</template>
