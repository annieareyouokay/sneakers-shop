<script setup>
import debounce from 'lodash.debounce'
import axios from 'axios'
import { inject, onMounted, reactive, ref, watch } from 'vue'

import CardList from '../components/CardList.vue'

const items = ref([])
const { cart, addToCart, removeFromCart } = inject('cart')
const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

const onClickAdd = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}

const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
}, 350);

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = { item_id: item.id }
      item.isFavorite = true
      const { data } = await axios.post('https://70d04dcd5a23ce3c.mokky.dev/favorites', obj)
      item.favoriteId = data.id
    } else {
      await axios.delete(`https://70d04dcd5a23ce3c.mokky.dev/favorites/${item.favoriteId}`)
      item.favoriteId = null
      item.isFavorite = false
    }
  } catch (error) {
    console.error(error)
  }
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://70d04dcd5a23ce3c.mokky.dev/favorites`)
    items.value = items.value.map((obj) => {
      const favorite = favorites.find((favorite) => favorite.item_id === obj.id)

      if (!favorite) {
        return obj
      }

      return { ...obj, isFavorite: true, favoriteId: favorite.id }
    })
  } catch (error) {
    console.error(error)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }
    const { data } = await axios.get(`https://70d04dcd5a23ce3c.mokky.dev/items`, { params })
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    }))
  } catch (error) {
    console.error(error)
  }
}

onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []

  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

watch(filters, fetchItems)
watch(cart, () => {
  if (!cart.value.length) {
    items.value = items.value.map((item) => ({
      ...item,
      isAdded: false
    }))
  }
})
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-10">Все кроссовки</h2>

    <div class="flex gap-4">
      <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
        <option value="title">По названию</option>
        <option value="price">По цене (дешевые)</option>
        <option value="-price">По цене (дорогие)</option>
      </select>

      <div class="relative">
        <img class="absolute left-4 top-3" src="/search.svg" />
        <input
          @input="onChangeSearchInput"
          class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
          type="text"
          placeholder="Поиск..."
        />
      </div>
    </div>
  </div>

  <div class="mt-10">
    <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAdd" />
  </div>
</template>
