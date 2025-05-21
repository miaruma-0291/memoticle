<template>
  <div class="wrapper">
    <!-- サイドバー -->
    <aside :class="['sidebar', { open: isSidebarOpen }]">
      <div class="channel-header">
        <h2>Channel</h2>
        <button @click="addChannel" class="add-channel">＋</button>
      </div>
      <ul class="channel-list">
        <li
          v-for="(channel, index) in channels"
          :key="index"
          :class="{ active: currentChannel === channel.id }"
          @click="selectChannel(channel.id)"
        >
          {{ channel.name }}
        </li>
      </ul>
    </aside>

    <!-- ハンバーガーボタン（SP表示用） -->
    <button class="hamburger" @click="toggleSidebar">☰</button>

    <!-- メインコンテンツ -->
    <div class="container">
      <h1>Memo-ticle</h1>
      <div class="separateLine"></div>

      <div class="mainFormBox">
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
      <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
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
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import ArticleForm from './components/ArticleForm.vue'
import ArticleList from './components/ArticleList.vue'

// サイドバー開閉状態
const isSidebarOpen = ref(false)

// チャンネル管理
const channels = ref([
  { id: 1, name: '一般', articles: [] },
  { id: 2, name: '仕事', articles: [] },
  { id: 3, name: '旅行', articles: [] },
])
const currentChannel = ref(channels.value[0].id)

// 記事の状態
const articles = ref([])
const showModal = ref(false)
const editingIndex = ref(null)
const currentArticle = ref({ title: '', content: '' })
const searchQuery = ref('')

// 初回マウント時にローカルストレージから読み込む
onMounted(() => {
  const saved = localStorage.getItem('articles')
  if (saved) {
    articles.value = JSON.parse(saved)
  }
})

// 記事が更新されるたびに保存
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

// 検索フィルター
const filteredArticles = computed(() => {
  // 検索クエリをひらがな＋小文字化して保持
  const query = toHiragana(searchQuery.value.toLowerCase())
  return articles.value.filter(article => {
    const title = toHiragana(article.title.toLowerCase())
    const content = toHiragana(article.content.toLowerCase())
    return title.includes(query) || content.includes(query)
  })
})

// チャンネルを追加
const addChannel = () => {
  const name = prompt('新しいチャンネル名を入力してください')
  if (!name) return
  const id = Date.now()
  channels.value.push({ id, name, articles: [] })
  currentChannel.value = id
}

// チャンネルを切り替え
const selectChannel = (id) => {
  currentChannel.value = id
  isSidebarOpen.value = false
}

// サイドバーの表示・非表示を切り替え
const toggleSidebar = () => {
  isSidebarOpen.value = !isSidebarOpen.value
}
</script>

<style src="./assets/style.css"></style>
