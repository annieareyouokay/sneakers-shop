<script setup>
import axios from 'axios';
import { inject, onMounted, ref } from 'vue';

import CardList from '@/components/CardList.vue';

const favorites = ref([]);
const { addToCart, removeFromCart } = inject('cart');

const onClickAdd = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

onMounted(async () => {
    try {
        const { data } = await axios.get('https://70d04dcd5a23ce3c.mokky.dev/favorites?_relations=items');
        favorites.value = data.map((element) => {
            return {
                ...element.item,
                favoriteId: element.id,
                isFavorite: true
            }
        });
    } catch (error) {
        console.error(error);
    }
});
</script>


<template>
  <div class="mt-10">
    <CardList :items="favorites" @add-to-cart="onClickAdd"/>
  </div>
</template>