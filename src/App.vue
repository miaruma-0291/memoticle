<template>
  <div class="container">

    <h1>Memo-ticle</h1>

    <div class="separateLine"></div>

    <div class="mainFormBox">
      <!-- 検索フォーム -->
      <input
        v-model="searchQuery"
        type="text"
        placeholder="検索ワードを入力"
        class="searchForm border p-2 mb-4 w-full"
      />
      
      <button @click="openAddModal" class="modalBtn bg-green-500 text-white px-4 py-2 mb-4">
        記事を追加
      </button>
    </div>

    <!-- モーダルウィンドウ -->
    <div v-if="showModal" class="modal-overlay">
      <div class="modal-content">
        <button @click="closeModal" class="close-btn">×</button>
        <ArticleForm
          :initialArticle="currentArticle"
          :isEdit="editingIndex !== null"
          @submitArticle="handleArticleSubmit"
        />
      </div>
    </div>

    <ArticleList
      :articles="filteredArticles"
      @removeArticle="removeArticle"
      @editArticle="openEditModal"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import ArticleForm from './components/ArticleForm.vue'
import ArticleList from './components/ArticleList.vue'

const articles = ref([])
const showModal = ref(false)
const editingIndex = ref(null)
const currentArticle = ref({ title: '', content: '' })
const searchQuery = ref('') // 検索ワードを保持

// 初回マウント時にローカルストレージから読み込む
onMounted(() => {
  const saved = localStorage.getItem('articles')
  if (saved) {
    articles.value = JSON.parse(saved)
  }
})

// articles が更新されるたびにローカルストレージへ保存
watch(articles, (newVal) => {
  localStorage.setItem('articles', JSON.stringify(newVal))
}, { deep: true })

const openAddModal = () => {
  currentArticle.value = { title: '', content: '' }
  editingIndex.value = null
  showModal.value = true
}

const openEditModal = (index) => {
  currentArticle.value = { ...articles.value[index] }
  editingIndex.value = index
  showModal.value = true
}

const closeModal = () => {
  showModal.value = false
}

// 記事の追加・編集
const handleArticleSubmit = (article) => {
  if (editingIndex.value !== null) {
    articles.value[editingIndex.value] = article
  } else {
    articles.value.push(article)
  }
  closeModal()
}

// 記事の削除
const removeArticle = (index) => {
  articles.value.splice(index, 1)
}

// ひらがなに正規化する関数（カタカナ→ひらがな）
const toHiragana = (str) => {
  return str.replace(/[\u30A1-\u30F6]/g, match =>
    String.fromCharCode(match.charCodeAt(0) - 0x60)
  )
}

// 検索結果に応じたフィルタリング処理
const filteredArticles = computed(() => {
  // 検索クエリをひらがな＋小文字化して保持
  const query = toHiragana(searchQuery.value.toLowerCase())

  // 記事一覧をフィルタリング
  return articles.value.filter(article => {
    const title = toHiragana(article.title.toLowerCase())
    const content = toHiragana(article.content.toLowerCase())
    return title.includes(query) || content.includes(query)
  })
})
</script>

<style src="./assets/style.css"></style>
