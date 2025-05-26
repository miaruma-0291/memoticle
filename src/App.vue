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
          v-for="channel in channels"
          :key="channel.id"
          :class="{ active: currentChannel === channel.id }"
          @click="selectChannel(channel.id)"
        >
        <button
          class="delete-channel"
          @click.stop="deleteChannel(channel.id)"
          title="チャンネルを削除"
        >
          ×
        </button>
          {{ channel.name }}
        </li>
      </ul>
    </aside>

    <!-- ハンバーガーボタン（SP表示用） -->
    <button
      class="hamburger"
      :class="{ active: isSidebarOpen }"
      @click="toggleSidebar"
    >
      <span class="bar"></span><span class="bar"></span><span class="bar"></span>
    </button>

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
      <transition name="modal-fade">
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
      </transition>


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

const isSidebarOpen = ref(false)
const searchQuery = ref('')
const showModal = ref(false)
const editingIndex = ref(null)
const currentArticle = ref({ title: '', content: '' })

const channels = ref([
  { id: 1, name: '一般', articles: [] }
])
const currentChannel = ref(1)

// ローカルストレージから読み込み
onMounted(() => {
  const saved = localStorage.getItem('channels')
  if (saved) {
    channels.value = JSON.parse(saved)
  }
})

// チャンネル更新時にローカルストレージへ保存
watch(channels, (newVal) => {
  localStorage.setItem('channels', JSON.stringify(newVal))
}, { deep: true })

// 現在のチャンネルのデータを取得
const currentArticles = computed(() => {
  const channel = channels.value.find(c => c.id === currentChannel.value)
  return channel ? channel.articles : []
})

// 検索フィルタ処理（ひらがな対応）
const toHiragana = (str) => {
  return str.replace(/[\u30A1-\u30F6]/g, match =>
    String.fromCharCode(match.charCodeAt(0) - 0x60)
  )
}

const filteredArticles = computed(() => {
  const query = toHiragana(searchQuery.value.toLowerCase())
  return currentArticles.value.filter(article => {
    const title = toHiragana(article.title.toLowerCase())
    const content = toHiragana(article.content.toLowerCase())
    return title.includes(query) || content.includes(query)
  })
})

// モーダル関連処理
const openAddModal = () => {
  currentArticle.value = { title: '', content: '' }
  editingIndex.value = null
  showModal.value = true
}
const openEditModal = (article) => {
  const channel = channels.value.find(c => c.id === currentChannel.value)
  if (!channel) return

  const index = channel.articles.findIndex(a => a.id === article.id)
  if (index === -1) return

  currentArticle.value = { ...article }
  editingIndex.value = index
  showModal.value = true
}

const closeModal = () => {
  showModal.value = false
}

// 記事の追加・編集
const handleArticleSubmit = (article) => {
  const channel = channels.value.find(c => c.id === currentChannel.value)
  if (!channel) return

  if (editingIndex.value !== null) {
    // 編集時は既存記事の id を保持したまま更新
    article.id = channel.articles[editingIndex.value].id
    channel.articles[editingIndex.value] = article
  } else {
    // 新規追加時にユニークな id を付与
    article.id = Date.now()
    channel.articles.push(article)
  }
  closeModal()
}


// 記事の削除
const removeArticle = (article) => {
  const channel = channels.value.find(c => c.id === currentChannel.value)
  if (!channel) return

  const index = channel.articles.findIndex(a => a.id === article.id)
  if (index !== -1) {
    channel.articles.splice(index, 1)
  }
}


// チャンネル関連処理
const addChannel = () => {
  const name = prompt('新しいチャンネル名を入力してください')
  if (!name) return
  const id = Date.now()
  channels.value.push({ id, name, articles: [] })
  currentChannel.value = id
}
const selectChannel = (id) => {
  currentChannel.value = id
  isSidebarOpen.value = false
}


// チャンネル削除処理
const deleteChannel = (id) => {
  // チャンネルが1つだけなら削除を許可しない
  if (channels.value.length <= 1) {
    alert('これ以上チャンネルは削除できません。')
    return
  }

  if (!confirm('このチャンネルを削除しますか？')) return

  const index = channels.value.findIndex(c => c.id === id)
  if (index !== -1) {
    channels.value.splice(index, 1)

    // 削除したチャンネルが現在のチャンネルだった場合は切り替え
    if (currentChannel.value === id) {
      currentChannel.value = channels.value[0].id
    }
  }
}


// ハンバーガーメニュー
const toggleSidebar = () => {
  isSidebarOpen.value = !isSidebarOpen.value
}
</script>

<style src="./assets/style.css"></style>
