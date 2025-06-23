<script setup lang="ts">
import { h, resolveComponent } from 'vue'
import { useHead } from '#imports'
import BlogPosts from "~/pages/posts/BlogPosts.vue";
import BlogPostsUi from "~/pages/posts/BlogPostsUi.vue";

useHead({
  title: 'Головна сторінка'
})

// Розв'язуємо компоненти Nuxt UI, які будемо використовувати
const UButton = resolveComponent('UButton')
const UCard = resolveComponent('UCard')
</script>

<template>
  <div class="flex-1 w-full h-screen flex items-center justify-center divide-y divide-(--ui-border-accented)">
    <!-- UCard з вирівнюванням тексту по центру -->
    <UCard class="w-96 p-6 text-center">
      <h1 class="text-2xl font-semibold mb-4">Головна сторінка</h1>

      <!-- Обгортка для кнопок зі space-y-4 -->
      <div class="flex flex-col items-center space-y-4">
        <NuxtLink to="/products">
          <UButton
              variant="flat"
              class="rounded-lg py-3 px-5 shadow-md transition-transform transform hover:scale-105 bg-gradient-to-r from-blue-500 to-indigo-500 hover:from-blue-600 hover:to-indigo-600 text-white"
          >
            Перейти до списку продуктів
          </UButton>
        </NuxtLink>

        <NuxtLink to="/posts/BlogPosts">
          <UButton
              variant="flat"
              class="rounded-lg py-3 px-5 shadow-md transition-transform transform hover:scale-105 bg-gradient-to-r from-blue-500 to-indigo-500 hover:from-blue-600 hover:to-indigo-600 text-white"
          >
            Перейти до BlogPosts
          </UButton>
        </NuxtLink>

        <NuxtLink to="/posts/BlogPostsUi">
          <UButton
              variant="flat"
              class="rounded-lg py-3 px-5 shadow-md transition-transform transform hover:scale-105 bg-gradient-to-r from-blue-500 to-indigo-500 hover:from-blue-600 hover:to-indigo-600 text-white"
          >
            BlogPostsUi
          </UButton>
        </NuxtLink>

        <NuxtLink to="/categories/BlogCategoriesUi">
          <UButton
              variant="flat"
              class="rounded-lg py-3 px-5 shadow-md transition-transform transform hover:scale-105 bg-gradient-to-r from-blue-500 to-indigo-500 hover:from-blue-600 hover:to-indigo-600 text-white"
          >
            BlogCategoriesUi
          </UButton>
        </NuxtLink>
      </div>
    </UCard>
  </div>
</template>
