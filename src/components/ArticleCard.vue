<template>
  <div class="border p-4 rounded">
    <div v-if="!isEditing">
      <h2 class="font-bold">{{ article.title }}</h2>
      <p class="text-gray-700">{{ article.content }}</p>
      <button @click="isEditing = true" class="text-blue-500 mr-2">編集</button>
      <button @click="$emit('delete')" class="text-red-500">削除</button>
    </div>
    <div v-else>
      <input v-model="editForm.title" class="border p-1 mr-2" />
      <input v-model="editForm.content" class="border p-1 mr-2" />
      <button @click="saveEdit" class="text-green-500 mr-2">保存</button>
      <button @click="isEditing = false" class="text-gray-500">キャンセル</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
const props = defineProps(['article'])
const emit = defineEmits(['delete', 'edit'])

const isEditing = ref(false)
const editForm = ref({ ...props.article })

const saveEdit = () => {
  emit('edit', { ...editForm.value })
  isEditing.value = false
}
</script>
