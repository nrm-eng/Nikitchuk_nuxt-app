<template>

  <div class="container">

    <div class="flex justify-center">

      <div class="w-full">

        <nav class="navbar bg-gray-100">

          <a href="/admin/blog/posts/create" class="">Додати</a>

        </nav>

        <div class="card">

          <div class="card-body">

            <table class="table table-auto">

              <thead>

              <tr>

                <th>#</th>

                <th>Автор</th>

                <th>Категорія</th>

                <th>Заголовок</th>

                <th>Дата публікації</th>

              </tr>

              </thead>

              <tbody>

              <tr v-for="post in posts">

                <td>{{ post.id }}</td>

                <td>{{ post.user.name }}</td>

                <td>{{ post.category.title }}</td>

                <td><a :href="'/admin/blog/posts/' + post.id + '/edit'">{{ post.title }}</a></td>

                <td>{{ post.published_at }}

                </td>

              </tr>

              </tbody>

            </table>

          </div>

        </div>

      </div>

    </div>

  </div>

</template>



<script setup lang="ts">

interface User {
  name: string
}

interface Category {
  title: string
}

interface Post {
  id: number
  title: string
  published_at: string
  user: User
  category: Category
}

const posts = ref<Post[]>([])

const getPosts = () => {

  $fetch<{ success: boolean; data: Post[] }>('http://127.0.0.1/api/blog/posts')
      .then(response => {
        posts.value = response.data
      })
      .catch(error => {
        console.error('Помилка при завантаженні постів:', error)
      })
}

getPosts()


</script>