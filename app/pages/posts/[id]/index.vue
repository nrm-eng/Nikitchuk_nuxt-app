<template>
  <UContainer class="py-8">
    <!-- Loading State -->
    <div v-if="status === 'pending'" class="flex justify-center items-center min-h-[400px]">
      <UIcon name="i-heroicons-arrow-path" class="animate-spin text-2xl" />
      <span class="ml-2 text-base text-gray-700">Завантаження...</span>
    </div>

    <!-- Error State -->
    <UAlert
        v-else-if="status === 'error'"
        icon="i-heroicons-exclamation-triangle"
        color="error"
        variant="soft"
        title="Помилка"
        :description="errorMessage"
        class="mb-6"
    />

    <!-- Success State -->
    <div v-else>
      <!-- Navigation -->
      <div class="mb-6">
        <UButton to="/posts/BlogPostsUi" variant="ghost" icon="i-heroicons-arrow-left" class="mb-4">
          Повернутися до блогу
        </UButton>
      </div>

      <!-- Post Card -->
      <UCard class="mb-8">
        <template #header>
          <div class="space-y-2">
            <h1 class="text-2xl text-primary font-semibold">{{ post.title }}</h1>
          </div>
        </template>

        <div class="prose prose-gray dark:prose-invert max-w-none mb-6">
          <!-- Рендеримо content_html, якщо він є, інакше content_raw -->
          <div v-if="post.content_html" v-html="post.content_html"></div>
          <div v-else-if="post.content_raw" v-html="post.content_raw.replace(/\n/g, '<br>')"></div>
        </div>

        <template #footer>
          <div class="flex items-center gap-2 text-sm text-gray-500 dark:text-gray-400">
            <UIcon name="i-heroicons-calendar-days" class="text-lg" />
            <span>
      <strong>Опубліковано:</strong> {{ formattedPublishedAt }}</span>
          </div>
        </template>

      </UCard>

      <!-- Дії -->
      <div class="flex gap-2 justify-center">
        <UButton to="/posts/BlogPostsUi" variant="soft" icon="i-heroicons-arrow-left">
          Читати інші статті
        </UButton>
      </div>
    </div>
  </UContainer>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useRoute, createError, useHead } from '#imports'

interface Post {
  id: number
  category_id: number
  user_id: number
  slug: string
  title: string
  excerpt: string
  content_raw: string
  content_html: string
  is_published: 0 | 1
  published_at: string | null
  created_at: string
  updated_at: string | null
}

const route = useRoute()
const postId = route.params.id as string

const { data: postData, error, status } = await useFetch<Post>(
    `http://localhost/api/posts/${postId}`,
    { key: `post-${postId}` }
)

if (status.value === 'error' && error.value?.statusCode === 404) {
  throw createError({ statusCode: 404, statusMessage: 'Пост не знайдено' })
}


const post = computed(() => postData.value as Post)

const formatDate = (d: string | null) =>
    d
        ? new Date(d).toLocaleDateString('uk-UA', {
          year: 'numeric',
          month: 'long',
          day: 'numeric'
        })
        : '—'

const formattedPublishedAt = computed(() => formatDate(post.value.published_at))
const errorMessage = computed(() => error.value?.message ?? 'Сталася невідома помилка')

useHead({
  title: computed(() => (postData.value ? `${post.value.title} - Блог` : 'Завантаження...')),
  meta: [
    {
      name: 'description',
      content: computed(() =>
          postData.value ? post.value.excerpt : 'Завантаження статті...'
      )
    }
  ]
})
</script>