<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold">Створити новий пост</h1>
      <UButton as="a" href="/posts/BlogPostsUi" color="secondary" icon="i-heroicons-arrow-left">
        Назад до списку постів
      </UButton>
    </div>

    <UForm :validate="validate" :state="state" class="space-y-4" @submit="onSubmit">
      <UFormField label="Заголовок" name="title">
        <UInput v-model="state.title" />
      </UFormField>

      <UFormField label="Контент" name="content_raw">
        <UTextarea v-model="state.content_raw" :rows="6" />
      </UFormField>

      <UFormField label="Категорія" name="category_id">
        <USelect
            v-model="state.category_id"
            :items="categoriesList.map(cat => ({ label: cat.title, value: cat.id }))"
            item-value-key="value"
            item-label-key="label"
            placeholder="Оберіть категорію"
            class="w-64"
        />
      </UFormField>

      <UButton type="submit" :loading="pending">
        Створити
      </UButton>
    </UForm>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, onMounted } from 'vue'
import { useFetch, useRouter } from '#imports'
import { z } from 'zod'
import type { FormError, FormSubmitEvent } from '@nuxt/ui'

// Функція для генерації slug
const charMap: Record<string,string> = {
  'а':'a','б':'b','в':'v','г':'h','ґ':'g','д':'d','е':'e','є':'ie','ж':'zh','з':'z',
  'и':'y','і':'i','ї':'i','й':'i','к':'k','л':'l','м':'m','н':'n','о':'o','п':'p',
  'р':'r','с':'s','т':'t','у':'u','ф':'f','х':'kh','ц':'ts','ч':'ch','ш':'sh','щ':'shch',
  'ь':'','ю':'iu','я':'ia'
}
function slugify(text: string): string {
  return text
      .toLowerCase()
      .split('')
      .map(ch => charMap[ch] ?? (/^[a-z0-9]$/.test(ch) ? ch : ' '))
      .join('')
      .trim()
      .replace(/\s+/g, '-')
      .replace(/-+/g, '-')
}

// Інтерфейси
interface CategoryAPI {
  id: number
  title: string
}
interface PostState {
  title: string
  content_raw: string
  slug: string
  content_html: string
  category_id?: number
  user_id: number
}

const router = useRouter()
const toast = useToast()

// Fetch списку категорій
const { data: catsResponse, pending } = await useFetch<{ data: CategoryAPI[] }>(
    'http://localhost/api/categories',
    { key: 'post-categories' }
)
const categoriesList = ref<CategoryAPI[]>([])
onMounted(() => {
  if (catsResponse.value?.data) categoriesList.value = catsResponse.value.data
})

// Стан форми
const state = reactive<PostState>({
  title: '',
  content_raw: '',
  slug: '',
  content_html: '',
  category_id: undefined,
  user_id: 2
})

// Zod-схема валідації
const PostSchema = z.object({
  title: z.string().min(1, { message: 'Заголовок обовʼязковий' }),
  content_raw: z.string().min(1, { message: 'Контент обовʼязковий' }),
  category_id: z.number().optional(),
})

const validate = (s: Partial<PostState>): FormError<string>[] => {
  const result = PostSchema.safeParse(s)
  if (result.success) return []

  return result.error.errors.map(e => ({
    name: e.path[0] as string,
    message: e.message,
  }))
}


// Submit
async function onSubmit(event: FormSubmitEvent<PostState>) {
  try {
    const slug = slugify(state.title)
    const content_html = state.content_raw

    await $fetch('http://127.0.0.1/api/blog/posts', {
      method: 'POST',
      body: {
        title: state.title,
        slug,
        content_raw: state.content_raw,
        content_html,
        category_id: state.category_id,
        user_id: state.user_id
      }
    })

    toast.add({ title: 'Успіх', description: 'Пост створено.', color: 'success' })
    router.push('/posts/BlogPostsUi')
  } catch {
    toast.add({ title: 'Помилка', description: 'Не вдалося створити пост.', color: 'error' })
  }
}
</script>