<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import Matter from 'matter-js'

// タイピングデータ
const text = ref('')
const completedSubText = ref('')
const remainingText = ref('')
const completedRoman = ref('')
const remainingRoman = ref('')
const count = ref(0) // 削除されたボールの総数

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

// 入力成功時の音（キータイプ音）
const okSound = new Audio('/assets/sound/ok.mp3')
// 入力失敗時の音
const ngSound = new Audio('/assets/sound/ng.mp3')
// ボール削除時の音
const removeSound = new Audio('/assets/sound/remove.mp3')

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
      playSound(ngSound)
      break
    case 'incomplete':
      playSound(okSound)
      addFallingLetter(key)
      break
    case 'complete':
      playSound(okSound)
      addFallingLetter(key)
      nextTopic() // 次のお題に切り替え
      break
    default:
      break
  }

  // テキストデータを更新
  updateTextData()
}

// 音を再生
function playSound(sound) {
  sound.currentTime = 0
  sound.play()
}

// テキストデータを更新
function updateTextData() {
  text.value = topics[currentTopicIndex.value].text
  completedSubText.value = typingText.completedText
  remainingText.value = typingText.remainingText
  completedRoman.value = typingText.completedRoman
  remainingRoman.value = typingText.remainingRoman
}

// Matter.jsのエンジンとワールド
let engine, render, world
let hole = null // 穴のオブジェクト
let constraint = null // 鎖の制約

// Matter.jsの初期化
function initializePhysics() {
  // エンジンの生成
  engine = Matter.Engine.create()
  world = engine.world

  // レンダリングの設定
  render = Matter.Render.create({
    element: document.getElementById('app'),
    engine: engine,
    options: {
      width: window.innerWidth,
      height: window.innerHeight,
      wireframes: false,
      background: '#f0f0f0',
    },
  })

  // マウス、マウス制約を生成
  const mouse = Matter.Mouse.create(document.getElementById('app'))
  const mouseConstraint = Matter.MouseConstraint.create(engine, {
    mouse: mouse,
    constraint: {
      render: {
        visible: false,
      },
    },
  })
  Matter.Composite.add(world, mouseConstraint)
  render.mouse = mouse

  // 壁（左）を追加
  const leftWall = Matter.Bodies.rectangle(0, window.innerHeight / 2, 20, window.innerHeight, {
    isStatic: true,
    collisionFilter: {
      group: -1, // 特定のグループに属する
    },
  })
  Matter.World.add(world, leftWall)

  // 壁（右）を追加
  const rightWall = Matter.Bodies.rectangle(
    window.innerWidth,
    window.innerHeight / 2,
    20,
    window.innerHeight,
    {
      isStatic: true,
      collisionFilter: {
        group: -1, // 特定のグループに属する
      },
    },
  )
  Matter.World.add(world, rightWall)

  // 床を追加
  const ground = Matter.Bodies.rectangle(
    window.innerWidth / 2,
    window.innerHeight,
    window.innerWidth,
    20,
    {
      isStatic: true,
      collisionFilter: {
        group: -1, // 特定のグループに属する
      },
    },
  )
  Matter.World.add(world, ground)

  // 穴を作成
  createHole()

  // レンダリングを実行
  Matter.Render.run(render)
  // エンジンを実行
  Matter.Runner.run(Matter.Runner.create(), engine)
}

// 文字を物理演算で落下させる
function addFallingLetter(letter) {
  letter = letter === '?' ? 'hatena' : letter
  const imgPath = `/assets/images/${letter}.png`
  const letterBody = Matter.Bodies.circle(Math.random() * window.innerWidth, 0, 20, {
    collisionFilter: {
      group: 1, // ボールは別のグループに属する
    },
    render: {
      sprite: {
        texture: imgPath,
        xScale: 0.3,
        yScale: 0.3,
      },
    },
  })

  letterBody.label = letter // 文字をラベルとして保持
  Matter.Composite.add(world, letterBody)
}

// 穴を作成する
function createHole() {
  if (hole) {
    Matter.World.remove(world, hole) // 既存の穴を削除
  }

  // 穴の初期位置
  const holeX = window.innerWidth - 100
  const holeY = 100

  // 穴の物理オブジェクト
  hole = Matter.Bodies.circle(holeX, holeY, 30, {
    isStatic: false,
    collisionFilter: {
      group: 0, // 穴は別のグループに属する
    },
    render: {
      fillStyle: '#000',
    },
  })
  Matter.World.add(world, hole)

  // 鎖（制約）を作成
  const anchor = { x: holeX, y: holeY } // 鎖の固定点
  constraint = Matter.Constraint.create({
    pointA: anchor,
    bodyB: hole,
    stiffness: 0.01, // 鎖の柔らかさ
    length: 0,
    render: {
      visible: true,
      lineWidth: 2,
      strokeStyle: '#555',
    },
  })
  Matter.World.add(world, constraint)

  // ボールが穴に落ちたかを監視
  Matter.Events.on(engine, 'collisionStart', (event) => {
    const pairs = event.pairs
    pairs.forEach((pair) => {
      if (pair.bodyA === hole || pair.bodyB === hole) {
        const ball = pair.bodyA === hole ? pair.bodyB : pair.bodyA
        if (ball.collisionFilter.group !== -1) {
          // 壁や床以外のオブジェクトを削除
          Matter.World.remove(world, ball)
          playSound(removeSound)
          count.value++ // 削除されたボールの総数を増加
        }
      }
    })
  })
}

// マウスで穴を動かす
function enableMouseControl() {
  const mouse = Matter.Mouse.create(render.canvas)
  const mouseConstraint = Matter.MouseConstraint.create(engine, {
    mouse: mouse,
    constraint: {
      stiffness: 0.2,
      render: {
        visible: false,
      },
    },
  })
  Matter.World.add(world, mouseConstraint)
}

onMounted(() => {
  initializeTypingText()
  initializePhysics()
  enableMouseControl()
  window.addEventListener('keydown', press)
})

onUnmounted(() => {
  window.removeEventListener('keydown', press)
  Matter.Render.stop(render)
  Matter.World.clear(world)
  Matter.Engine.clear(engine)
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
    <div class="text-display">
      <span>{{ text }}</span>
    </div>
    <!-- <div class="sub-text-display">
          <span class="completed">{{ completedSubText }}</span>
          <span class="remaining">{{ remainingText }}</span>
        </div> -->
    <div class="roman-display">
      <span class="completed-roman">{{ completedRoman }}</span>
      <span class="remaining-roman">{{ remainingRoman }}</span>
    </div>
    <div class="count-display">{{ count }}</div>
  </div>
</template>

<style scoped>
.typing-container {
  position: absolute;
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

.count-display {
  margin-top: 10px;
  font-size: 1.2rem;
  color: #333;
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
