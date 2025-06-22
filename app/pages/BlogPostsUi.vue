<script setup lang="ts">
import { h } from 'vue'
import type { TableColumn } from '@nuxt/ui'

// Тип з API
interface Blog {
  id: number
  title: string
  published_at: string | null
  user?: { name: string }
  category?: { title: string }
}

// Тип для таблиці (flat)
interface PostTableRow {
  id: number
  title: string
  author: string
  category: string
  publishedAt: string
}

// Пагінація відповідно до документації
const page = ref(1)
const itemsPerPage = ref(10)

// Fetch з трансформацією
const { data: postsResponse, status } = await useFetch<{ data: Blog[] }>('http://127.0.0.1/api/blog/posts', {
  key: 'table-posts'
})

const allPosts = computed<PostTableRow[]>(() =>
    postsResponse.value?.data.map((post) => ({
      id: post.id,
      title: post.title,
      author: post.user?.name ?? 'Невідомо',
      category: post.category?.title ?? 'Без категорії',
      publishedAt: post.published_at
          ? new Date(post.published_at).toLocaleDateString('uk-UA')
          : 'Не опубліковано'
    })) ?? []
)

// Загальна кількість записів
const total = computed(() => allPosts.value.length)

// Пагіновані пости для поточної сторінки
const posts = computed(() => {
  const start = (page.value - 1) * itemsPerPage.value
  const end = start + itemsPerPage.value
  return allPosts.value.slice(start, end)
})

// Колонки таблиці
const columns: TableColumn<PostTableRow>[] = [
  {
    accessorKey: 'id',
    header: '#'
  },
  {
    accessorKey: 'author',
    header: 'Автор'
  },
  {
    accessorKey: 'category',
    header: 'Категорія'
  },
  {
    accessorKey: 'title',
    header: 'Заголовок',
    cell: ({ row }) =>
        h(
            'a',
            {
              href: `/admin/blog/posts/${row.original.id}/edit`,
              class: 'text-blue-600 hover:text-blue-800 underline'
            },
            row.original.title
        )
  },
  {
    accessorKey: 'publishedAt',
    header: 'Дата публікації'
  }
]

// Скидаємо на першу сторінку при зміні даних
watch(allPosts, () => {
  page.value = 1
})
</script>

<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold">Блог пости</h1>
      <UButton
          as="a"
          href="/admin/blog/posts/create"
          color="primary"
          icon="i-heroicons-plus"
      >
        Додати пост
      </UButton>
    </div>

    <UCard>
      <UTable
          :data="posts"
          :columns="columns"
          :loading="status === 'pending'"
          stripe
          row-key="id"
          class="w-full"
      />

      <!-- UPagination відповідно до документації -->
      <template v-if="total > itemsPerPage">
        <div class="flex justify-center mt-6 p-4 border-t">
          <UPagination
              v-model:page="page"
              :total="total"
              :items-per-page="itemsPerPage"
              :sibling-count="2"
              show-edges
              show-controls
              color="neutral"
              variant="outline"
              active-color="primary"
              active-variant="solid"
              size="md"
          />
        </div>
      </template>
    </UCard>

    <!-- Інформація про пагінацію -->
    <div class="flex justify-between items-center mt-4 text-sm text-gray-600">
      <div>
        Показано {{ Math.min(itemsPerPage, total - (page - 1) * itemsPerPage) }} з {{ total }} записів
      </div>
      <div>
        Сторінка {{ page }} з {{ Math.ceil(total / itemsPerPage) }}
      </div>
    </div>
  </div>
</template>