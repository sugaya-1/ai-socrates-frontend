<template>
  <!--
    全体背景:
    1. 背景画像: bg-socrates.jpg
    2. 全体の文字色を青白い白(text-cyan-50)に変更してダークモードに対応
  -->
  <div
    class="flex items-center justify-center min-h-screen text-cyan-50 font-sans relative overflow-hidden bg-slate-900"
    :style="{
      backgroundImage: `url('/bg-socrates.jpg')`,
      backgroundSize: 'cover',
      backgroundPosition: 'center',
      backgroundAttachment: 'fixed',
      backgroundRepeat: 'no-repeat',
    }">

    <!-- 背景オーバーレイ:
         サイバー感を出すため、深い青と黒のグラデーションを重ねて
         背景画像を「暗い空間に浮かぶデータ」のように見せる
    -->
    <div
      class="absolute inset-0 bg-gradient-to-b from-slate-950/80 via-slate-900/70 to-indigo-950/70 backdrop-blur-[2px]">
    </div>

    <!-- メインカード:
         ダークなすりガラス(bg-slate-900/60)に変更。
         枠線(border)をシアン色で薄く発光させ、近未来的なデバイス感を演出。
         shadowで青白い光(glow)を追加。
    -->
    <div
      class="w-full max-w-2xl h-[90vh] bg-slate-900/60 backdrop-blur-md rounded-xl shadow-[0_0_40px_rgba(6,182,212,0.15)] flex flex-col overflow-hidden relative border border-cyan-500/30 z-10 transition-all duration-500">

      <!-- ヘッダー -->
      <header
        class="py-5 px-6 border-b border-cyan-500/20 bg-slate-900/50 z-10 shrink-0 text-center relative shadow-lg backdrop-blur-md">
        <!-- タイトルにテキストシャドウをつけて発光させる -->
        <h1
          class="text-2xl font-bold text-cyan-100 tracking-wide font-serif drop-shadow-[0_0_8px_rgba(34,211,238,0.5)]">
          AIソクラテス</h1>
        <p class="text-xs text-cyan-400/80 mt-1 font-medium tracking-[0.3em] uppercase">System: Philosophia / Online</p>
      </header>

      <!-- メインエリア -->
      <main ref="chatWindow" class="flex-1 overflow-y-auto p-6 space-y-8 custom-scrollbar relative bg-transparent">

        <!-- 開始画面 -->
        <div v-if="!hasStarted" class="flex flex-col items-center justify-center h-full space-y-10 fade-in pb-20">
          <div
            class="bg-slate-800/60 backdrop-blur-md text-cyan-50 p-8 rounded-lg shadow-[0_0_20px_rgba(6,182,212,0.1)] border border-cyan-500/30 max-w-[85%] text-base leading-loose relative font-serif">
            <!-- 吹き出しの尻尾 -->
            <div
              class="absolute -top-2 left-1/2 -translate-x-1/2 w-4 h-4 bg-slate-800/60 border-t border-l border-cyan-500/30 rotate-45">
            </div>
            <div class="mb-4 text-xs font-bold text-cyan-400 flex items-center justify-center gap-1 tracking-wider">
              <span class="animate-pulse">●</span> <span>INITIALIZING...</span>
            </div>
            <p class="text-center font-medium drop-shadow-md">叡智の探求者よ、こんにちは。<br>我と共に智への道を歩み始めようではないか。<br>準備はよろしいかな？</p>
          </div>

          <!-- 開始ボタン: 発光するサイバーボタン -->
          <button @click="startSession"
            class="group relative bg-cyan-900/40 text-cyan-50 text-lg font-serif font-bold py-3 px-12 rounded-lg overflow-hidden border border-cyan-400/50 transition-all duration-300 hover:bg-cyan-800/50 hover:shadow-[0_0_20px_rgba(34,211,238,0.4)] hover:-translate-y-0.5">
            <span class="relative z-10">智の海へ潜る</span>
            <!-- ボタン内の流れる光エフェクト -->
            <div
              class="absolute inset-0 -translate-x-full group-hover:animate-[shimmer_1.5s_infinite] bg-gradient-to-r from-transparent via-cyan-400/10 to-transparent z-0">
            </div>
          </button>
        </div>


        <!-- 完了画面 -->
        <div v-else-if="isCompleted"
          class="flex flex-col items-center justify-center h-full space-y-6 fade-in text-center p-8 bg-slate-900/40 backdrop-blur-sm rounded-xl m-4 border border-cyan-500/30">
          <div class="text-6xl mb-4 text-cyan-400 animate-pulse drop-shadow-[0_0_15px_rgba(34,211,238,0.6)]">✦</div>
          <h2 class="text-3xl font-bold text-cyan-50 font-serif drop-shadow-md">真理への到達</h2>
          <p class="text-cyan-200/80 leading-relaxed font-serif font-medium">全問クリアしました。<br>素晴らしい探究心です。</p>
          <button @click="resetSession"
            class="mt-6 text-cyan-400 hover:text-cyan-200 font-serif font-semibold underline decoration-1 underline-offset-4 transition-colors drop-shadow-sm">システム再起動</button>
        </div>

        <!-- チャット画面 -->
        <template v-else>
          <div v-for="(message, index) in history" :key="index" class="flex w-full fade-in group"
            :class="message.sender === 'user' ? 'justify-end' : 'justify-start'">

            <!-- 吹き出し -->
            <div :class="[
              'p-5 max-w-[88%] text-[15px] leading-relaxed shadow-lg relative rounded-xl backdrop-blur-sm border',
              message.sender === 'user'
                ? 'bg-cyan-900/60 text-cyan-50 rounded-tr-sm border-cyan-500/40 shadow-[0_0_10px_rgba(6,182,212,0.1)]' // ユーザー: 濃いシアン
                : 'bg-slate-800/70 text-cyan-50 rounded-tl-sm border-slate-600/50' // AI: 暗いグレー
            ]">
              <div v-if="message.sender === 'ai'"
                class="mb-2 text-xs font-bold text-cyan-400 flex items-center gap-1 select-none font-serif tracking-wider border-b border-cyan-500/20 pb-1">
                <span v-if="message.type === 'question'">❖ QUERY</span>
                <span v-else>SOCRATES.AI</span>
              </div>

              <!-- 本文 -->
              <div class="break-words" :class="[
                message.sender === 'ai'
                  ? 'prose prose-invert prose-p:text-cyan-50 prose-strong:text-cyan-300 prose-headings:text-cyan-200 prose-a:text-cyan-400 max-w-none prose-p:font-serif prose-headings:font-serif'
                  : 'whitespace-pre-wrap font-sans'
              ]" v-html="parseMarkdown(message.text)">
              </div>

              <!-- 選択肢 -->
              <div v-if="message.choices && message.choices.length > 0 && message.type === 'question'"
                class="mt-5 pt-3 border-t border-cyan-500/20">
                <p class="text-xs font-serif text-cyan-400/70 mb-3 text-center">- SELECT ANSWER -</p>
                <div class="flex flex-col space-y-2">
                  <span v-for="choice in message.choices" :key="choice.id"
                    class="text-sm bg-slate-900/50 hover:bg-cyan-900/40 px-4 py-3 rounded border border-cyan-500/30 text-cyan-100 shadow-sm hover:border-cyan-400 hover:text-cyan-50 transition-all duration-300 cursor-pointer font-serif hover:shadow-[0_0_10px_rgba(34,211,238,0.2)]">
                    {{ choice.choice_text }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <!-- ローディング -->
          <div v-if="isSending || isLoading" class="flex justify-start fade-in">
            <div
              class="p-4 rounded-xl rounded-tl-sm bg-slate-800/70 border border-slate-600/50 text-cyan-400 shadow-sm flex items-center min-h-[40px] backdrop-blur-sm">
              <div class="typing-indicator"><span></span><span></span><span></span></div>
            </div>
          </div>

          <!-- エラー -->
          <div v-if="error" class="flex justify-center my-4 fade-in">
            <div
              class="bg-red-900/60 text-red-200 text-sm px-4 py-3 rounded border border-red-500/50 shadow-[0_0_10px_rgba(239,68,68,0.2)] flex items-center gap-2 font-serif backdrop-blur-sm">
              <span>⚠ SYSTEM ERROR: {{ error.message }}</span>
            </div>
          </div>
        </template>

      </main>

      <!-- フッター -->
      <footer v-if="hasStarted && !isCompleted"
        class="p-5 bg-slate-900/80 backdrop-blur-md z-10 shrink-0 border-t border-cyan-500/20">
        <div v-if="!isSufficient" class="fade-in w-full">
          <div class="flex items-end space-x-3 relative">
            <textarea v-model="inputAnswer" @keydown.enter.prevent.exact="handleSend" :disabled="isSending || isLoading"
              rows="1" ref="textareaRef" @input="resizeTextarea"
              class="flex-1 p-4 pr-2 max-h-32 min-h-[56px] bg-slate-800/50 border border-cyan-500/30 rounded-lg focus:ring-1 focus:ring-cyan-500 focus:border-cyan-500 focus:outline-none transition-all resize-none disabled:bg-slate-900/50 disabled:text-slate-600 text-cyan-50 placeholder-cyan-400/50 text-base shadow-inner font-sans backdrop-blur-sm focus:bg-slate-800/80"
              placeholder="思考を形にしてみよう..."></textarea>

            <div class="flex space-x-2 pb-0.5">
              <!-- 神託ボタン -->
              <button @click="askOracle" :disabled="isSending || isLoading || !currentQuestionId"
                class="w-12 h-12 flex items-center justify-center bg-amber-600/80 text-white rounded-lg hover:bg-amber-500 active:scale-95 transition-all shadow-lg hover:shadow-[0_0_15px_rgba(245,158,11,0.4)] disabled:opacity-50 backdrop-blur-sm border border-amber-500/30"
                title="オラクル（ヒント）">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path
                    d="m12 3-1.912 5.813a2 2 0 0 1-1.275 1.275L3 12l5.813 1.912a2 2 0 0 1 1.275 1.275L12 21l1.912-5.813a2 2 0 0 1 1.275-1.275L12 3Z" />
                </svg>
              </button>
              <!-- 送信ボタン -->
              <button @click="handleSend" :disabled="isSending || !inputAnswer.trim() || !currentQuestionId"
                class="w-12 h-12 flex items-center justify-center bg-cyan-700/80 text-white rounded-lg hover:bg-cyan-600 active:scale-95 transition-all shadow-lg hover:shadow-[0_0_15px_rgba(6,182,212,0.4)] disabled:opacity-50 backdrop-blur-sm border border-cyan-500/30"
                title="送信">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <line x1="22" y1="2" x2="11" y2="13"></line>
                  <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
                </svg>
              </button>
            </div>
          </div>
        </div>
        <div v-else class="fade-in w-full">
          <!-- 次のトピックボタン -->
          <button @click="handleNextTopic"
            class="w-full bg-cyan-900/80 text-cyan-50 font-serif font-bold py-4 px-6 rounded-lg hover:bg-cyan-800 active:scale-[0.99] focus:outline-none transition-all shadow-lg hover:shadow-[0_0_20px_rgba(34,211,238,0.3)] flex items-center justify-center space-x-3 text-lg border border-cyan-500/40 backdrop-blur-sm">
            <span>次のシーケンスへ</span>
          </button>
        </div>
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue';
import axios from 'axios';
import { marked } from 'marked';

const chatWindow = ref(null);
const textareaRef = ref(null);
const hasStarted = ref(false);
const isSufficient = ref(false);
const isCompleted = ref(false);
const history = ref([]);
const isLoading = ref(false);
const error = ref(null);
const inputAnswer = ref('');
const isSending = ref(false);
const currentQuestionId = ref(null);
const currentTopicId = ref(1);
const nextTopicId = ref(null);

const parseMarkdown = (text) => {
  if (text == null || typeof text !== 'string') {
    return '';
  }

  try {
    let cleanText = text;
    cleanText = cleanText.replace(/\\(\*)/g, '$1');

    // サイバー感のある発光する青に変更
    cleanText = cleanText.replace(/\*\*\s*(.*?)\s*\*\*/g, '<strong class="text-cyan-300 font-bold drop-shadow-[0_0_5px_rgba(34,211,238,0.8)]">$1</strong>');

    return marked.parse(cleanText);

  } catch (e) {
    console.error("Markdown変換エラー:", e);
    return text;
  }
};

const startSession = async () => {
  hasStarted.value = true;
  history.value = [];
  await fetchNextQuestion();
};

const resetSession = () => {
  hasStarted.value = false;
  isCompleted.value = false;
  currentTopicId.value = 1;
  history.value = [];
};

const handleNextTopic = async () => {
  isSufficient.value = false;
  history.value.push({ sender: 'user', type: 'answer', text: '（次のトピックへ進む）' });
  if (nextTopicId.value) { currentTopicId.value = nextTopicId.value; } else { currentTopicId.value++; }
  await fetchNextQuestion();
};

const scrollToBottom = () => {
  nextTick(() => {
    if (chatWindow.value) {
      chatWindow.value.scrollTo({ top: chatWindow.value.scrollHeight, behavior: 'smooth' });
    }
  });
};

const resizeTextarea = () => {
  const el = textareaRef.value;
  if (el) {
    el.style.height = 'auto';
    el.style.height = `${Math.min(el.scrollHeight, 128)}px`;
  }
};

const fetchNextQuestion = async () => {
  isLoading.value = true;
  error.value = null;
  scrollToBottom();
  try {
    const API_URL = `http://localhost/api/topics/${currentTopicId.value}/next-question`;
    const response = await axios.get(API_URL);
    const data = response.data;
    currentQuestionId.value = data.id;
    nextTopicId.value = data.next_topic_id || null;
    history.value.push({ sender: 'ai', type: 'question', text: data.question_text, question_id: data.id, choices: data.choices });
  } catch (err) {
    if (err.response && err.response.status === 404) {
      isCompleted.value = true;
    } else {
      console.error("Error:", err);
      error.value = { message: '問題の取得に失敗しました。' };
    }
  } finally {
    isLoading.value = false;
    scrollToBottom();
  }
};

const handleSend = async () => {
  const text = inputAnswer.value.trim();
  if (isSending.value || !text || !currentQuestionId.value) return;
  inputAnswer.value = '';
  if (textareaRef.value) textareaRef.value.style.height = 'auto';
  isSending.value = true;
  error.value = null;
  history.value.push({ sender: 'user', type: 'answer', text: text, question_id: currentQuestionId.value });
  scrollToBottom();
  try {
    const API_URL = `http://localhost/api/questions/${currentQuestionId.value}/check`;
    const response = await axios.post(API_URL, { answer_text: text });
    const data = response.data;
    history.value.push({ sender: 'ai', type: 'feedback', text: data.explanation, question_id: currentQuestionId.value, is_correct: data.is_correct });
    if (data.is_sufficient === true || (data.explanation && data.explanation.includes('[CORRECT]'))) {
      isSufficient.value = true;
    }
  } catch (err) {
    console.error("Send Error:", err);
    error.value = { message: '通信エラーが発生しました。' };
  } finally {
    isSending.value = false;
    scrollToBottom();
  }
};

const askOracle = () => {
  inputAnswer.value = "知恵を借してくれないか？";
  handleSend();
};
</script>

<style scoped>
/* Google Fontsから美しい明朝体(Shippori Mincho)を読み込み */
@import url('https://fonts.googleapis.com/css2?family=Shippori+Mincho:wght@400;500;700&display=swap');

.font-serif {
  font-family: 'Shippori Mincho', serif;
}

/* スクロールバーのデザイン: サイバーブルーに変更 */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(15, 23, 42, 0.5);
  /* slate-900/50 */
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #0e7490;
  /* cyan-700 */
  border-radius: 20px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background-color: #06b6d4;
  /* cyan-500 */
}

/* フェードインアニメーション */
.fade-in {
  animation: fadeIn 0.4s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 3点リーダーアニメーション: シアン色に発光 */
.typing-indicator {
  display: flex;
  align-items: center;
  gap: 4px;
}

.typing-indicator span {
  height: 6px;
  width: 6px;
  background-color: #22d3ee;
  /* cyan-400 */
  border-radius: 50%;
  display: inline-block;
  animation: bounce 1.4s infinite ease-in-out both;
  box-shadow: 0 0 5px #22d3ee;
}

.typing-indicator span:nth-of-type(1) {
  animation-delay: -0.32s;
}

.typing-indicator span:nth-of-type(2) {
  animation-delay: -0.16s;
}

@keyframes bounce {

  0%,
  80%,
  100% {
    transform: scale(0);
  }

  40% {
    transform: scale(1.0);
  }
}

/* ボタン内の光のエフェクト */
@keyframes shimmer {
  0% {
    transform: translateX(-150%) skewX(-15deg);
  }

  100% {
    transform: translateX(150%) skewX(-15deg);
  }
}
</style>
