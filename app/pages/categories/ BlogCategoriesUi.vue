<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold">Категорії</h1>
      <div class="flex items-center gap-2">
        <UButton
            as="a"
            href="/"
            color="secondary"
            icon="i-heroicons-home"
        >
          Головна сторінка
        </UButton>
        <UButton
            as="a"
            href="create"
            color="primary"
            icon="i-heroicons-plus"
        >
          Додати категорію
        </UButton>
      </div>
    </div>

    <UCard>
      <UTable
          :data="categories"
          :columns="columns"
          :loading="status === 'pending'"
          stripe
          row-key="id"
          class="w-full"
      >
        <template #parentTitle-cell="{ row }">
          {{ row.original.parentTitle }}
        </template>
        <template #description-cell="{ row }">
          <span class="block truncate max-w-xs">
            {{ row.original.description || '—' }}
          </span>
        </template>
        <template #action-cell="{ row }">
          <UDropdownMenu :items="getDropdownActions(row.original)">
            <UButton
                icon="i-lucide-ellipsis-vertical"
                color="neutral"
                variant="ghost"
                aria-label="Дії"
            />
          </UDropdownMenu>
        </template>
      </UTable>

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

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import type { TableColumn, DropdownMenuItem } from '@nuxt/ui'
import { useClipboard } from '@vueuse/core'

// Тип з API
interface Category {
  id: number
  title: string
  description: string | null
  parent_id: number | null
}

// Тип для таблиці
interface CategoryTableRow {
  id: number
  title: string
  description: string
  parentTitle: string
}

const toast = useToast()
const { copy } = useClipboard()

const page = ref(1)
const itemsPerPage = ref(5)

// ——— 1. Завантажуємо сирі дані ———
const { data: res, status, refresh } = await useFetch<{ data: Category[] }>(
    'http://localhost/api/categories',
    { key: 'table-categories' }
)

// Будуємо мапу ID → title для швидкого lookup
const categoryMap = computed<Map<number,string>>(() => {
  const m = new Map<number,string>()
  for (const cat of res.value?.data || []) {
    m.set(cat.id, cat.title)
  }
  return m
})

// Трансформуємо в рядки таблиці з підставленням parentTitle
const allCategories = computed<CategoryTableRow[]>(() =>
    (res.value?.data || []).map(cat => ({
      id: cat.id,
      title: cat.title,
      description: cat.description ?? '',
      parentTitle:
          cat.parent_id === null
              ? 'Корінь'
              : (categoryMap.value.get(cat.parent_id) || '—')
    }))
)

const total = computed(() => allCategories.value.length)

const categories = computed(() => {
  const start = (page.value - 1) * itemsPerPage.value
  return allCategories.value.slice(start, start + itemsPerPage.value)
})


// Колонки
const columns: TableColumn<CategoryTableRow>[] = [
  { accessorKey: 'id', header: 'ID' },
  { accessorKey: 'parentTitle', header: 'Батьківська категорія', id: 'parentTitle' },
  { accessorKey: 'title', header: 'Назва' },
  { accessorKey: 'description', header: 'Опис', id: 'description' },
  { id: 'action' }
]

// Скидаємо сторінку при зміні даних
watch(allCategories, () => {
  page.value = 1
})

// Дії
function getDropdownActions(row: CategoryTableRow): DropdownMenuItem[][] {
  return [
    [
      {
        label: 'Копіювати ID',
        icon: 'i-lucide-copy',
        onSelect: () => {
          copy(row.id.toString())
          toast.add({ title: 'ID категорії скопійовано', color: 'success' })
        }
      }
    ],
    [
      {
        label: 'Редагувати',
        icon: 'i-lucide-edit',
        onSelect: () => {
          window.location.href = `/categories/${row.id}/edit`
        }
      },
      {
        label: 'Видалити',
        icon: 'i-lucide-trash',
        color: 'error',
        onSelect: async () => {
          try {
            await $fetch(`http://localhost/api/categories/${row.id}`, { method: 'DELETE' })
            toast.add({ title: 'Категорію видалено', color: 'success' })
            await refresh() // оновити useFetch
          } catch {
            toast.add({ title: 'Помилка при видаленні', color: 'error' })
          }
        }
      }
    ]
  ]
}
</script>