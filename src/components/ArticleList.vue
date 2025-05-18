<template>
  <div class="articleBlock">
    <div
      v-for="(article, index) in articles"
      :key="index"
      class="article border p-2 mb-2"
    >
      <h3>{{ article.title }}</h3>
      <p style="white-space: pre-wrap;">{{ article.content }}</p>
      <button @click="edit(index)" class="editBtn bg-yellow-400 text-white px-2 py-1 mr-2"><img src="/src/assets/img/ic_pencil.png" alt="アイコン" /></button>
      <button @click="remove(index)" class="deleteBtn bg-red-500 text-white px-2 py-1">×</button>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  articles: Array
})
const emit = defineEmits(['removeArticle', 'editArticle'])

const remove = (index) => {
  emit('removeArticle', index)
}
const edit = (index) => {
  emit('editArticle', index)
}
</script>
