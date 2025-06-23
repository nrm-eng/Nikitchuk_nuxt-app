<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold">Редагування посту</h1>
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
        Зберегти
      </UButton>
    </UForm>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, watch } from 'vue'
import { useRoute, useFetch, useRouter } from '#imports'
import { z } from 'zod'
import type { FormError} from '@nuxt/ui'

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
interface PostAPI {
  title: string
  slug: string
  content_raw: string
  content_html: string
  category_id: number | null
}

interface PostState {
  title: string
  slug: string
  content_raw: string
  content_html: string
  category_id?: number
}

interface CategoryAPI {
  id: number
  title: string
}

const route = useRoute()
const router = useRouter()
const toast = useToast()
const id = route.params.id as string

// Fetch поточного посту
const { data: post, pending } = await useFetch<PostAPI>(
    () => `http://127.0.0.1/api/blog/posts/${id}`,
    { method: 'GET', key: `post-${id}`, immediate: true }
)

// Fetch списку категорій
const { data: catsResponse } = await useFetch<{ data: CategoryAPI[] }>(
    'http://127.0.0.1/api/categories',
    { key: 'post-categories' }
)
const categoriesList = ref<CategoryAPI[]>([])
watch(
    () => catsResponse.value,
    (res) => { if (res?.data) categoriesList.value = res.data },
    { immediate: true }
)

// Стан форми
const state = reactive<PostState>({
  title: '',
  slug: '',
  content_raw: '',
  content_html: '',
  category_id: undefined
})

// Ініціалізація з API
watch(
    () => post.value,
    (p) => {
      if (!p) return
      state.title = p.title
      state.slug = p.slug
      state.content_raw = p.content_raw
      state.content_html = p.content_html
      state.category_id = p.category_id ?? undefined
    },
    { immediate: true }
)

// Генерація slug та копіювання raw в html перед сабмітом
async function onSubmit() {
  try {
    const slug = slugify(state.title)
    const content_html = state.content_raw

    await $fetch(`http://127.0.0.1/api/blog/posts/${id}`, {
      method: 'PUT',
      body: {
        title: state.title,
        slug,
        content_raw: state.content_raw,
        content_html,
        category_id: state.category_id
      }
    })

    toast.add({ title: 'Успіх', description: 'Пост оновлено.', color: 'success' })
    router.push('/posts/BlogPostsUi')
  } catch {
    toast.add({ title: 'Помилка', description: 'Не вдалося оновити пост.', color: 'error' })
  }
}

// Zod-схема валідації
const PostSchema = z.object({
  title: z.string().min(1, { message: 'Заголовок обовʼязковий' }),
  content_raw: z.string().min(1, { message: 'Контент обовʼязковий' }),
  category_id: z.number().optional()
})


const validate = (s: Partial<PostState>): FormError[] => {
  const result = PostSchema.safeParse(s)
  if (result.success) return []

  return result.error.errors.map((e) => ({
    name: e.path[0] as string,
    message: e.message,
  }))
}

</script>