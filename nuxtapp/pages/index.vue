<script setup lang="ts">
import { ref, computed, h, resolveComponent } from 'vue'
import { useFetch } from '@vueuse/core'
import type { TableColumn } from '@nuxt/ui'

definePageMeta({
  title: 'Список продуктів'
})

interface Product {
  id: number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
  [key: string]: any
}

const UButton = resolveComponent('UButton')

const searchQuery = ref('')
const currentPage = ref(1)
const perPage = 5
const sortField = ref('')
const sortAsc = ref(true)

const { data } = await useFetch('https://dummyjson.com/products?limit=200').get().json()

const products = computed<Product[]>(() => data.value?.products || [])

const filtered = computed(() => {
  const q = searchQuery.value.toLowerCase()
  if (!q) return products.value

  return products.value.filter((product: Product) =>
    Object.values(product).some(val =>
      String(val).toLowerCase().includes(q)
    )
  )
})

const sortBy = (field: string) => {
  sortField.value = sortField.value === field ? field : field
  sortAsc.value = sortField.value === field ? !sortAsc.value : true
  currentPage.value = 1
}

const sorted = computed(() => {
  if (!sortField.value) return filtered.value

  return [...filtered.value].sort((a: Product, b: Product) => {
    const aVal = a[sortField.value]
    const bVal = b[sortField.value]

    if (sortAsc.value) {
      return aVal > bVal ? 1 : aVal < bVal ? -1 : 0
    } else {
      return aVal < bVal ? 1 : aVal > bVal ? -1 : 0
    }
  })
})

const paginated = computed(() => {
  const start = (currentPage.value - 1) * perPage
  return sorted.value.slice(start, start + perPage)
})

const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'thumbnail',
    header: 'Photo',
    cell: ({ row }) =>
      h('img', {
        src: row.getValue('thumbnail'),
        class: 'w-[90px] h-[90px] object-cover rounded border border-gray-700'
      })
  },
  {
    accessorKey: 'title',
    header: () =>
      h(
        UButton,
        {
          color: 'blue',
          variant: 'ghost',
          class: 'text-blue-300',
          icon:
            sortField.value === 'title'
              ? sortAsc.value
                ? 'i-lucide-arrow-up'
                : 'i-lucide-arrow-down'
              : 'i-lucide-arrow-up-down',
          onClick: () => sortBy('title')
        },
        () => 'Title'
      )
  },
  {
    accessorKey: 'description',
    header: () =>
      h(
        UButton,
        {
          color: 'blue',
          variant: 'ghost',
          class: 'text-blue-300',
          icon:
            sortField.value === 'description'
              ? sortAsc.value
                ? 'i-lucide-arrow-up'
                : 'i-lucide-arrow-down'
              : 'i-lucide-arrow-up-down',
          onClick: () => sortBy('description')
        },
        () => 'Description'
        ),
    cell: ({ row }) =>
      h('div', { class: 'text-gray-400 max-w-[300px] truncate' }, row.getValue('description'))
  },
  {
    accessorKey: 'price',
    header: () =>
      h(
        UButton,
        {
          color: 'blue',
          variant: 'ghost',
          class: 'text-blue-300',
          icon:
            sortField.value === 'price'
              ? sortAsc.value
                ? 'i-lucide-arrow-up'
                : 'i-lucide-arrow-down'
              : 'i-lucide-arrow-up-down',
          onClick: () => sortBy('price')
        },
        () => 'Price'
      ),
    cell: ({ row }) =>
      h('span', { class: 'text-gray-300 font-semibold' }, `€${row.getValue('price')}`)
  },
  {
    accessorKey: 'rating',
    header: () =>
      h(
        UButton,
        {
          color: 'blue',
          variant: 'ghost',
          class: 'text-blue-300',
          icon:
            sortField.value === 'rating'
              ? sortAsc.value
                ? 'i-lucide-arrow-up'
                : 'i-lucide-arrow-down'
              : 'i-lucide-arrow-up-down',
          onClick: () => sortBy('rating')
        },
        () => 'Rating'
      ),
    cell: ({ row }) =>
      h(
        'span',
        {
          class: [
            'font-bold',
            (row.getValue('rating') as number) < 4.5 ? 'text-red-400' : 'text-green-400'
          ]
        },
        `⭐ ${row.getValue('rating')}`
      )
  },
  {
    accessorKey: 'brand',
    header: () =>
      h(
        UButton,
        {
          color: 'blue',
          variant: 'ghost',
          class: 'text-blue-300',
          icon:
            sortField.value === 'brand'
              ? sortAsc.value
                ? 'i-lucide-arrow-up'
                : 'i-lucide-arrow-down'
              : 'i-lucide-arrow-up-down',
          onClick: () => sortBy('brand')
        },
        () => 'Brand'
      )
  },
  {
    accessorKey: 'category',
    header: () =>
      h(
        UButton,
        {
          color: 'blue',
          variant: 'ghost',
          class: 'text-blue-300',
          icon:
            sortField.value === 'category'
              ? sortAsc.value
                ? 'i-lucide-arrow-up'
                : 'i-lucide-arrow-down'
              : 'i-lucide-arrow-up-down',
          onClick: () => sortBy('category')
        },
        () => 'Category'
      )
  }
]
</script>

<template>
  <div class="p-4 space-y-6 bg-black min-h-screen text-gray-200">
    <h1 class="text-2xl font-bold flex justify-center text-blue-400">Список продуктів</h1>

    <div class="flex justify-center">
      <UInput v-model="searchQuery" icon="i-lucide-search" placeholder="Пошук" class="max-w-md" />
    </div>

    <UTable :data="paginated" :columns="columns" class="bg-gray-900 rounded-lg shadow-sm" />

    <div class="flex justify-center">
      <UPagination
        :total="sorted.length"
        :page="currentPage"
        :items-per-page="perPage"
        @update:page="val => (currentPage = val)"
      />
    </div>
  </div>
</template>
