<template>
  <div>
    <Head>
      <Title>Блог пости</Title>
    </Head>

    <div class="min-h-screen bg-gray-900 text-white">
      <header class="bg-gray-800 shadow-sm">
        <div class="container mx-auto px-4 py-6">
          <h1 class="text-3xl font-bold">Блог пости</h1>
        </div>
      </header>

      <main class="container mx-auto px-4 py-8">
        <PostsTableComponent />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import PostsTableComponent from "../../components/PostsTableComponent.vue";
</script>