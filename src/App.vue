<template>
  <main>
    <template v-if="startMenu">
      <h1>EscapeFromMalphite</h1>
      <button @click="startGame">ゲームスタート</button>
    </template>

    <template v-else-if="!gameover && !gameClear">
      <img class="teemo" src="./assets/ico_Teemo.jpg">
      <p class="word">レベル{{ level }}</p>
      <p class="word">{{ inputWord }}</p>
      <div class="malphite">
        <transition name="fade" mode="out-in">
          <img :key="resetKey" :src="currentImage" alt="malphite image"
            :class="{ 'reset-animation': resetAnimation, 'animated': !resetAnimation }">
        </transition>
      </div>
    </template>

    <template v-else-if="gameClear">
      <p>おめでとう！すべての単語を入力しました！</p>
      <p>ミスタイプ数: {{ missCount }}</p>
      <p>入力ができたワードの数: {{ wordCount }}</p>
      <button @click="resetGame">スタートメニューに戻る</button>
    </template>

    <template v-else>
      <p>マルファイトにやられてしまった!!</p>
      <p>ミスタイプ数: {{ missCount }}</p>
      <p>入力ができたワードの数: {{ wordCount }}</p>
      <button @click="resetGame">スタートメニューに戻る</button>
    </template>
  </main>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';

const startMenu = ref(true);
const gameover = ref(false);
const gameClear = ref(false);

const dataList1 = ref(['red', 'pink', 'blue', 'gold']);
const dataList2 = ref(['action', 'global', 'rhythm', 'academy']);
const dataList3 = ref(['actually', 'instance', 'distance', 'surrender']);
const dataList4 = ref(['difficulty', 'understand', 'convenient', 'celebration']);
const dataList5 = ref(['refrigerator', 'dramatically', 'unquestionably', 'congratulations']);
const currentDataList = ref(dataList1.value);
const level = ref(1);

const wordIndex = ref(0);
const dataIndex = ref(0);
const inputWord = ref('');
const resetKey = ref(0);
const resetAnimation = ref(false);

let timer;

// スタートメニュー
const startGame = () => {
  startMenu.value = false;
  nextWord();
};

// 次のワードに
const nextWord = () => {
  resetAnimationEffect(); // アニメーションのリセットを追加
  wordIndex.value = 0;
  inputWord.value = currentDataList.value[dataIndex.value];
  clearTimeout(timer);
  timer = setTimeout(() => {
    gameover.value = true;
  }, 3000);
};

const advanceLevel = () => {
  level.value++;
  if (level.value === 2) {
    currentDataList.value = dataList2.value;
  } else if (level.value === 3) {
    currentDataList.value = dataList3.value;
  } else if (level.value === 4) {
    currentDataList.value = dataList4.value;
  } else if (level.value === 5) {
    currentDataList.value = dataList5.value;
  }
  dataIndex.value = 0;  // dataIndexをリセット
  nextWord();
};

const resetAnimationEffect = () => {
  resetAnimation.value = true;
  setTimeout(() => {
    resetAnimation.value = false;
    resetKey.value++;  // keyをインクリメントして要素を再レンダリング
  }, 100); // 少しの遅延を設けることでアニメーションをリセット
};

//キーボード判定
const missCount = ref(0);
const wordCount = ref(0);

const handleKeydown = (e) => {
  if (e.key === currentDataList.value[dataIndex.value][wordIndex.value]) {
    wordIndex.value++;
    inputWord.value =
      '_'.repeat(wordIndex.value) +
      currentDataList.value[dataIndex.value].substring(wordIndex.value);
    if (wordIndex.value === currentDataList.value[dataIndex.value].length) {
      dataIndex.value++;
      wordCount.value++;
      clearTimeout(timer);
      if (dataIndex.value < currentDataList.value.length) {
        nextWord();
      } else {
        if (level.value < 5) {
          advanceLevel();  // レベルアップ処理を呼び出し
        } else {
          gameClear.value = true;
        }
      }
    }
  } else {
    missCount.value++;
  }
};

onMounted(() => {
  document.addEventListener('keydown', handleKeydown);
});

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeydown);
});

const images = [
  './assets/m1.png',
  './assets/m2.png',
  './assets/m3.png',
  './assets/m4.png',
  './assets/m5.png'
];
const currentImageIndex = ref(0);

// 0.2秒ごとに画像を切り替えてアニメーションにする
setInterval(() => {
  currentImageIndex.value = (currentImageIndex.value + 1) % images.length;
}, 200);

const currentImage = computed(() => images[currentImageIndex.value]);

const resetGame = () => {
  startMenu.value = true;
  gameover.value = false;
  gameClear.value = false;
  level.value = 1;
  wordIndex.value = 0;
  dataIndex.value = 0;
  inputWord.value = '';
  missCount.value = 0;
  wordCount.value = 0;
  currentDataList.value = dataList1.value;
  resetKey.value = 0;
  clearTimeout(timer);
};
</script>

<style scoped>
main {
  width: 800px;
  height: 500px;
  margin: 30px auto 0;
  text-align: center;
  position: relative;
  color: red;
  background-color: rgb(0, 255, 119);
}

h1 {
  padding: 60px;
}

button {
  width: 200px;
  height: 40px;
  margin-top: 60px;
}

.word {
  margin: 8px;
  padding: 16px;
  font-size: 40px;
}

p {
  padding: 20px;
  font-size: 36px;
  letter-spacing: 0.2em;
}

.teemo {
  position: absolute;
  bottom: 0;
  left: 0;
}

.malphite img {
  position: absolute;
  bottom: 0;
  right: 0;
}

.reset-animation {
  right: 0;
  animation: none;
}

.animated {
  animation: moveLeft 5s linear infinite, switchImages 1s steps(5) infinite;
}

@keyframes moveLeft {
  from {
    right: 0;
  }

  to {
    right: 100%;
  }
}

@keyframes switchImages {
  0% {
    content: url('./assets/m1.png');
  }

  20% {
    content: url('./assets/m2.png');
  }

  40% {
    content: url('./assets/m3.png');
  }

  60% {
    content: url('./assets/m4.png');
  }

  80% {
    content: url('./assets/m5.png');
  }

  100% {
    content: url('./assets/m1.png');
  }
}
</style>