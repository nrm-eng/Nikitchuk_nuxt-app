<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold">Редагування категорії</h1>
      <UButton as="a" href="/categories/BlogCategoriesUi" color="secondary" icon="i-heroicons-arrow-left">
        Назад до списку
      </UButton>
    </div>

    <UForm :validate="validate" :state="state" class="space-y-4" @submit="onSubmit">
      <UFormField label="Назва категорії" name="title">
        <UInput v-model="state.title" />
      </UFormField>

      <UFormField label="Опис категорії" name="description">
        <UTextarea v-model="state.description" :rows="4" />
      </UFormField>

      <UFormField label="ID батьківської категорії" name="parent_id">
        <UInput v-model="state.parent_id" type="number" />
      </UFormField>

      <UButton type="submit" :loading="pending">
        Зберегти
      </UButton>
    </UForm>
  </div>
</template>

<script setup lang="ts">
import { reactive, watch } from 'vue'
import { useRoute, useFetch, useRouter } from '#imports'
import { z } from 'zod'
import type { FormError, FormSubmitEvent } from '@nuxt/ui'

// Інтерфейси
interface CategoryAPI {
  title: string;
  description: string | null;
  parent_id: number | null
}
interface CategoryState {
  title: string;
  description: string;
  parent_id: number | null
}

const route = useRoute()
const router = useRouter()
const toast = useToast()
const id = route.params.id as string

// Fetch поточної категорії
const { data: category, pending, error } = await useFetch<CategoryAPI>(
    () => `http://localhost/api/categories/${id}`,
    { method: 'GET', key: `category-${id}`, immediate: true }
)
// Стан форми
const state = reactive<CategoryState>({ title: '', description: '', parent_id: null })
watch(
    () => category.value,
    (cat) => {
      if (cat) {
        state.title = cat.title
        state.description = cat.description || ''
        state.parent_id = cat.parent_id
      }
    },
    { immediate: true }
)

// Zod-схема валідації
const CategorySchema = z.object({
  title: z.string().min(1, { message: 'Назва обовʼязкова' }),
  description: z.string(),
  parent_id: z.number().nullable()
})

// Функція validate з Zod
const validate = (s: Partial<CategoryState>): FormError<string>[] => {
  const result = CategorySchema.safeParse(s)
  if (result.success) return []

  return result.error.errors.map(e => ({
    name: e.path[0] as string,
    message: e.message,
  }))
}

// Функція для генерації slug
const charMap: Record<string,string> = {
  'а':'a','б':'b','в':'v','г':'h','ґ':'g','д':'d','е':'e','є':'ie','ж':'zh','з':'z',
  'и':'y','і':'i','ї':'i','й':'i','к':'k','л':'l','м':'m','н':'n','о':'o','п':'p',
  'р':'r','с':'s','т':'t','у':'u','ф':'f','х':'kh','ц':'ts','ч':'ch','ш':'sh','щ':'shch',
  'ь':'','ю':'iu','я':'ia'
}
function slugify(text: string): string {
  let slug = text.toLowerCase().split('').map(ch => charMap[ch] ?? (/[a-z0-9]/.test(ch) ? ch : ' ')).join('')
  return slug.trim().replace(/\s+/g,'-').replace(/-+/g,'-')
}

// Submit
async function onSubmit(event: FormSubmitEvent<CategoryState>) {
  try {
    const slug = slugify(state.title)
    await $fetch(`http://localhost/api/categories/${id}`, {
      method: 'PUT',
      body: { title: state.title, slug, description: state.description, parent_id: state.parent_id }
    })
    toast.add({ title: 'Успіх', description: 'Категорію оновлено.', color: 'success' })
    router.push('/categories/BlogCategoriesUi')
  } catch {
    toast.add({ title: 'Помилка', description: 'Не вдалося оновити категорію.', color: 'error' })
  }
}
</script>