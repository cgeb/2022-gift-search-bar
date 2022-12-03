<script setup lang="ts">
import { ref, watch } from 'vue'

type Product = {
  id: number
  title: string
  price: number
}

const searchTerm = ref('')
const products = ref<Product[]>([])
const isLoading = ref(false)

const debounce = (func: Function, timeout: number) => {
  let timer: number
  return (...args) => {
    clearTimeout(timer)
    timer = setTimeout(() => {
      func.apply(this, args)
    }, timeout)
  }
}

const fetchProducts = async (term: string) => {
  const res = await fetch(`https://dummyjson.com/products/search?q=${term}`)
  const json = await res.json()
  return json.products as Product[]
}

const findProducts = debounce(async (term: string) => {
  if (!term) {
    products.value = []
    return
  }

  products.value = await fetchProducts(term)
  if (!products.value.length) {
    alert('No products found. Please try another search term.')
  }
  isLoading.value = false
}, 300)

watch(searchTerm, newTerm => {
  if (!newTerm) return

  isLoading.value = true
})

watch(searchTerm, newTerm => {
  findProducts(newTerm)
})
</script>

<template>
  <div class="w-full h-full flex flex-col gap-5 justify-center items-center">
    <h1 class="text-4xl font-bold">Gift Search Bar</h1>
    <input type="text" class="p-2 border-2 border-gray-dark" v-model="searchTerm" placeholder="Start typing..." />
    <ul v-if="!isLoading" class="list-disc">
      <li v-for="product in products" :key="product.id">{{ product.title }} - ${{ product.price }}</li>
    </ul>
    <p v-else>Please wait...</p>
  </div>
</template>
