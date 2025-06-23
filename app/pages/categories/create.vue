<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-between items-center mb-6">
      <h1 class="text-3xl font-bold">Створення категорії</h1>
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
        Створити
      </UButton>
    </UForm>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { useRouter } from '#imports'
import { z } from 'zod'
import type { FormError, FormSubmitEvent } from '@nuxt/ui'

const router = useRouter()
const toast = useToast()
const pending = ref(false)

// Стан форми
const state = reactive({
  title: '',
  description: '',
  parent_id: null as number | null
})

// Zod-схема валідації
const CategorySchema = z.object({
  title: z.string().min(1, { message: 'Назва обовʼязкова' }),
  description: z.string(),
  parent_id: z.number().nullable()
})

// Валідація
const validate = (s: Partial<{
  title: string;
  description: string;
  parent_id: number | null;
}>): FormError<string>[] => {
  const result = CategorySchema.safeParse(s)
  if (result.success) return []

  return result.error.errors.map(e => ({
    name: e.path[0] as string,
    message: e.message,
  }))
}


// Транслітерація для slug
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
async function onSubmit(event: FormSubmitEvent<typeof state>) {
  try {
    pending.value = true
    const slug = slugify(state.title)
    await $fetch('http://localhost/api/categories', {
      method: 'POST',
      body: {
        title: state.title,
        slug,
        description: state.description,
        parent_id: state.parent_id
      }
    })
    toast.add({ title: 'Успіх', description: 'Категорію створено.', color: 'success' })
    router.push('/categories/BlogCategoriesUi')
  } catch {
    toast.add({ title: 'Помилка', description: 'Не вдалося створити категорію.', color: 'error' })
  } finally {
    pending.value = false
  }
}
</script>