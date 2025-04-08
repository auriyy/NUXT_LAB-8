<template>
    <div class="p-4 space-y-4">
        <h1 class="text-2xl font-bold">Список продуктів</h1>
    </div>

    <div class="p-4 space-y-4">
      <label for="search" class="sr-only">Пошук:</label>
      <input
        type="text"
        v-model="searchQuery"
        placeholder="Пошук..."
      />
  
      <table class="w-full border-collapse text-sm shadow-sm">
        <thead>
          <tr class="text-left">
            <th class="p-2 w-[110px]">Фото</th>
            <th class="p-2 cursor-pointer" @click="sortBy('title')">
              Назва
              <span v-if="sortField === 'title'">{{ sortAsc ? '⬆️' : '⬇️' }}</span>
            </th>
            <th class="p-2 cursor-pointer" @click="sortBy('description')">
              Опис
              <span v-if="sortField === 'description'">{{ sortAsc ? '⬆️' : '⬇️' }}</span>
            </th>
            <th class="p-2 cursor-pointer" @click="sortBy('price')">
              Ціна
              <span v-if="sortField === 'price'">{{ sortAsc ? '⬆️' : '⬇️' }}</span>
            </th>
            <th class="p-2 cursor-pointer" @click="sortBy('rating')">
              Оцінка
              <span v-if="sortField === 'rating'">{{ sortAsc ? '⬆️' : '⬇️' }}</span>
            </th>
            <th class="p-2 cursor-pointer" @click="sortBy('brand')">
              Бренд
              <span v-if="sortField === 'brand'">{{ sortAsc ? '⬆️' : '⬇️' }}</span>
            </th>
            <th class="p-2 cursor-pointer" @click="sortBy('category')">
              Категорія
              <span v-if="sortField === 'category'">{{ sortAsc ? '⬆️' : '⬇️' }}</span>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="item in paginated"
            :key="item.id"
            class="border-b transition"
          >
            <td class="p-2">
              <img
                :src="item.thumbnail"
                alt="thumb"
                class="w-[90px] h-[90px] object-cover rounded"
              />
            </td>
            <td class="p-2 font-medium">{{ item.title }}</td>
            <td class="p-2 text-gray-100 max-w-[300px] truncate">
              {{ item.description }}
            </td>
            <td class="p-2 font-semibold text-gray-400">€{{ item.price }}</td>
            <td
              class="p-2 font-bold"
              :class="{
                'text-red-500': item.rating < 4.5,
                'text-green-600': item.rating >= 4.5
              }"
            >
              {{ item.rating }}
            </td>
            <td class="p-2">{{ item.brand }}</td>
            <td class="p-2">{{ item.category }}</td>
          </tr>
        </tbody>
      </table>
  
      <div class="flex justify-center border-t border-(--ui-border) pt-4">
        <UPagination
          :total="sorted.length"
          :page="currentPage"
          :items-per-page="perPage"
          @update:page="val => currentPage = val"
        />
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, computed, onMounted } from 'vue'
  definePageMeta({
  title: 'Список продуктів'  
})
  
  const products = ref<any[]>([])
  const searchQuery = ref('')
  const currentPage = ref(1)
  const perPage = 5
  const sortField = ref('')
  const sortAsc = ref(true)
  
  const loadProducts = async () => {
      const res = await fetch('https://dummyjson.com/products?limit=200')
      const data = await res.json()
      products.value = data.products || []
    }
  
  
  onMounted(loadProducts)
  
  const filtered = computed(() => {
    const query = searchQuery.value.toLowerCase()
    if (!query) return products.value
  
    return products.value.filter(product =>
      Object.values(product).some(val =>
        String(val).toLowerCase().includes(query)
      )
    )
  })
  
  const sortBy = (field: string) => {
    sortField.value = sortField.value === field ? field : field;
    sortAsc.value = sortField.value === field ? !sortAsc.value : true;
    currentPage.value = 1;
  };
  
  const sorted = computed(() => {
    if (!sortField.value) return filtered.value;
  
    return filtered.value.slice().sort((a, b) => {
      const FirstField = a[sortField.value];
      const SecondField = b[sortField.value];
      
    if (sortAsc.value) {
      if (FirstField > SecondField) return 1;
      if (FirstField < SecondField) return -1;
  
      return 0;
    } else {
      if (FirstField < SecondField) return 1;
      if (FirstField > SecondField) return -1;
      return 0;
    }
      });
    });
  
  const paginated = computed(() => {
    const startIndex = (currentPage.value - 1) * perPage;
    return sorted.value.slice(startIndex, startIndex + perPage);
  });
  
  </script>  