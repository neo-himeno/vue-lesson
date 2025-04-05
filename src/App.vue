<script setup>
import { onMounted, onUnmounted, ref } from 'vue'

// タイピングデータ
const text = ref('')
const completedSubText = ref('')
const remainingText = ref('')
const completedRoman = ref('')
const remainingRoman = ref('')

// お題リスト
const topics = [
  { text: '今日の天気は晴れ。', subText: 'きょうのてんきははれ。' },
  { text: 'サッカーのワールドカップ。', subText: 'サッカーのワールドカップ。' },
  { text: 'オリンピックが楽しみ。', subText: 'オリンピックがたのしみ。' },
  { text: '映画の新作が公表。', subText: 'えいがのしんさくがこうひょう。' },
  { text: '今日のニュースを見た？', subText: 'きょうのニュースをみた？' },
  { text: '消費税が上がるらしい。', subText: 'しょうひぜいがあがるらしい。' },
  { text: '電車が遅れている。', subText: 'でんしゃがおくれている。' },
  { text: '金メダルを目指そう！', subText: 'きんメダルをめざそう！' },
  { text: '海外旅行が復活。', subText: 'かいがいりょこうがふっかつ。' },
  { text: '新型スマホが発売。', subText: 'しんがたスマホがはつばい。' },
]
const currentTopicIndex = ref(0) // 現在のお題のインデックス

// キータイプ音の準備
const keyTypeSound = new Audio('/sound/type.mp3')

// TypingTextインスタンス
let typingText = null

// 初期化処理
function initializeTypingText() {
  const topic = topics[currentTopicIndex.value].subText
  typingText = new window.TypingText(topic)

  // 初期状態を反映
  updateTextData()
}

// 次のお題に切り替える
function nextTopic() {
  currentTopicIndex.value = (currentTopicIndex.value + 1) % topics.length
  initializeTypingText()
}

// キー入力処理
function press(event) {
  const key = event.key
  const state = typingText.inputKey(key)

  switch (state) {
    case 'unmatch':
      console.log('miss') // ミスした場合の処理
      break
    case 'incomplete':
      playKeyTypeSound() // 入力中の音を再生
      break
    case 'complete':
      playKeyTypeSound() // 完成時も音を再生
      nextTopic() // 次のお題に切り替え
      break
    default:
      break
  }

  // テキストデータを更新
  updateTextData()
}

// キータイプ音を再生
function playKeyTypeSound() {
  keyTypeSound.currentTime = 0
  keyTypeSound.play()
}

// テキストデータを更新
function updateTextData() {
  text.value = topics[currentTopicIndex.value].text
  completedSubText.value = typingText.completedText
  remainingText.value = typingText.remainingText
  completedRoman.value = typingText.completedRoman
  remainingRoman.value = typingText.remainingRoman
}

// イベントリスナーの登録と解除
onMounted(() => {
  initializeTypingText()
  window.addEventListener('keydown', press)
})

onUnmounted(() => {
  window.removeEventListener('keydown', press)
})
</script>

<template>
  <div class="typing-container">
    <div class="progress-bar">
      <div
        class="progress"
        :style="{
          width: `${(completedSubText.length / (completedSubText.length + remainingText.length)) * 100}%`,
        }"
      ></div>
    </div>
    <div class="text-display">{{ text }}</div>
    <!-- <div class="sub-text-display">
      <span class="completed">{{ completedSubText }}</span>
      <span class="remaining">{{ remainingText }}</span>
    </div> -->
    <div class="roman-display">
      <span class="completed-roman">{{ completedRoman }}</span>
      <span class="remaining-roman">{{ remainingRoman }}</span>
    </div>
  </div>
</template>

<style scoped>
.typing-container {
  text-align: center;
  height: fit-content;
  min-width: 500px;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.progress-bar {
  width: 100%;
  height: 10px;
  background-color: #e0e0e0;
  border-radius: 5px;
  overflow: hidden;
  margin-bottom: 20px;
}

.progress {
  height: 100%;
  background-color: #4caf50;
  transition: width 0.3s ease;
}

.text-display {
  font-size: 1.8rem;
  margin-bottom: 10px;
  color: #333;
}

.sub-text-display {
  font-size: 1.4rem;
  margin-bottom: 10px;
  color: #333;
}

.roman-display {
  font-size: 1.2rem;
  color: #666;
  margin-bottom: 20px;
}

.completed {
  color: #4caf50;
}

.remaining {
  color: #333;
}

.completed-roman {
  color: #81c784;
}

.remaining-roman {
  color: #999;
}
</style>
