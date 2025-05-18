<template>
  <form @submit.prevent="submitForm">
    <input v-model="form.title" placeholder="タイトル" class="block w-full mb-2 p-2 border" />
    <textarea
      v-model="form.content"
      placeholder="本文（改行対応）"
      class="block w-full mb-2 p-2 border"
      rows="5"
    ></textarea>
    <button type="submit" class="bg-blue-500 text-white px-4 py-2">
      {{ isEdit ? '更新' : '追加' }}
    </button>
  </form>
</template>

<script setup>
import { reactive, watch } from 'vue'

const props = defineProps({
  initialArticle: Object, // App.vue から渡される記事データ
  isEdit: Boolean
})
const emit = defineEmits(['submitArticle'])

const form = reactive({
  title: '',
  content: ''
})

// フォームに初期値を反映
watch(() => props.initialArticle, (newArticle) => {
  if (newArticle) {
    form.title = newArticle.title
    form.content = newArticle.content
  }
}, { immediate: true })

const submitForm = () => {
  if (!form.title || !form.content) {
    alert('全てのフィールドを入力してください')
    return
  }

  emit('submitArticle', { title: form.title, content: form.content })

  // フォーム初期化
  form.title = ''
  form.content = ''
}
</script>
