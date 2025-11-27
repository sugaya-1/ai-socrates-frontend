<template>
  <div class="flex items-center justify-center min-h-screen bg-gray-100 text-gray-800 font-sans">

    <!-- メインのチャットウィンドウ -->
    <div
      class="w-full max-w-2xl h-[90vh] bg-white rounded-3xl shadow-2xl flex flex-col overflow-hidden relative border border-gray-200">

      <!-- ヘッダー -->
      <header class="py-4 px-6 border-b border-gray-100 bg-white z-10 shrink-0 text-center">
        <h1 class="text-xl font-bold text-gray-800 tracking-tight">AIソクラテス</h1>
        <p class="text-xs text-gray-500 mt-1 font-medium">汝自身を知れ</p>
      </header>

      <!-- チャットメッセージ表示エリア -->
      <main ref="chatWindow" class="flex-1 overflow-y-auto p-6 space-y-6 custom-scrollbar relative bg-white">

        <!-- 1. スタート画面 (未開始時) -->
        <div v-if="!hasStarted" class="flex flex-col items-center justify-center h-full space-y-10 fade-in pb-20">
          <!-- 挨拶バブル -->
          <div
            class="bg-slate-50 text-slate-800 p-6 rounded-2xl shadow-sm border border-slate-100 max-w-[85%] text-base leading-relaxed relative">
            <div class="absolute -top-2 left-5 w-4 h-4 bg-slate-50 border-t border-l border-slate-100 rotate-45"></div>
            <div class="mb-3 text-xs font-bold text-slate-500 flex items-center gap-1">
              <span>🧠 ソクラテス</span>
            </div>
            <p>良き探求者よ、こんにちは。<br>我と共に知への道を歩み始めようではないか。<br>準備はよろしいかな？</p>
          </div>

          <!-- 始めようボタン -->
          <button @click="startSession"
            class="bg-blue-500 text-white text-lg font-bold py-3 px-12 rounded-full shadow-lg hover:bg-blue-600 hover:shadow-xl hover:-translate-y-0.5 transition duration-200 ease-in-out">
            始めよう
          </button>
        </div>

        <!-- 2. 全問クリア画面 -->
        <div v-else-if="isCompleted"
          class="flex flex-col items-center justify-center h-full space-y-6 fade-in text-center p-8">
          <div class="text-6xl mb-4 animate-bounce">🎉</div>
          <h2 class="text-2xl font-bold text-gray-800">素晴らしい！</h2>
          <p class="text-gray-600 leading-relaxed">全問クリアしました。<br>おめでとうございます！</p>
          <button @click="resetSession"
            class="mt-6 text-blue-500 hover:text-blue-700 font-semibold underline decoration-2 underline-offset-4">最初からやり直す</button>
        </div>

        <!-- 3. チャット履歴 -->
        <template v-else>
          <div v-for="(message, index) in history" :key="index" class="flex w-full fade-in group"
            :class="message.sender === 'user' ? 'justify-end' : 'justify-start'">

            <!-- バブル本体 -->
            <div :class="[
              'p-4 max-w-[85%] text-[15px] leading-relaxed shadow-sm relative rounded-2xl',
              message.sender === 'user'
                ? 'bg-blue-500 text-white rounded-tr-sm'
                : 'bg-slate-100 text-slate-800 rounded-tl-sm'
            ]">
              <!-- ラベル -->
              <div v-if="message.sender === 'ai'"
                class="mb-1.5 text-xs font-bold text-slate-500/80 flex items-center gap-1 select-none">
                <span v-if="message.type === 'question'">💡 問い</span>
                <span v-else>ソクラテス</span>
              </div>

              <!-- テキスト -->
              <div class="whitespace-pre-wrap break-words" v-html="message.text"></div>

              <!-- 選択肢 -->
              <div v-if="message.choices && message.choices.length > 0 && message.type === 'question'"
                class="mt-4 pt-3 border-t border-slate-200/60">
                <p class="text-xs font-semibold text-slate-500 mb-2">選択肢:</p>
                <div class="flex flex-col space-y-1.5">
                  <span v-for="choice in message.choices" :key="choice.id"
                    class="text-sm bg-white px-3 py-1.5 rounded-lg border border-slate-200 text-slate-700 shadow-sm">
                    {{ choice.choice_text }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <!-- AI思考中 -->
          <div v-if="isSending || isLoading" class="flex justify-start fade-in">
            <div
              class="p-4 rounded-2xl rounded-tl-sm bg-slate-100 text-slate-800 shadow-sm flex items-center min-h-[40px]">
              <div class="typing-indicator"><span></span><span></span><span></span></div>
            </div>
          </div>

          <!-- エラー表示 -->
          <div v-if="error" class="flex justify-center my-4 fade-in">
            <div
              class="bg-red-50 text-red-600 text-sm px-4 py-3 rounded-xl border border-red-100 shadow-sm flex items-center gap-2">
              <span>{{ error.message }}</span>
            </div>
          </div>
        </template>

      </main>

      <!-- フッターエリア -->
      <footer v-if="hasStarted && !isCompleted" class="p-4 bg-white z-10 shrink-0 border-t border-gray-100">
        <!-- 入力フォーム -->
        <div v-if="!isSufficient" class="fade-in w-full">
          <div class="flex items-end space-x-2 relative">
            <textarea v-model="inputAnswer" @keydown.enter.prevent.exact="handleSend" :disabled="isSending || isLoading"
              rows="1" ref="textareaRef" @input="resizeTextarea"
              class="flex-1 p-4 pr-2 max-h-32 min-h-[56px] bg-white border border-gray-300 rounded-2xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 focus:outline-none transition-all resize-none disabled:bg-gray-50 disabled:text-gray-400 text-gray-700 placeholder-gray-400 text-base shadow-sm"
              placeholder="回答を入力..."></textarea>
            <div class="flex space-x-2 pb-0.5">
              <button @click="askOracle" :disabled="isSending || isLoading || !currentQuestionId"
                class="w-12 h-12 flex items-center justify-center bg-amber-400 text-white rounded-xl hover:bg-amber-500 active:scale-95 transition-all shadow-md disabled:opacity-50"
                title="神託">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                  <path
                    d="m12 3-1.912 5.813a2 2 0 0 1-1.275 1.275L3 12l5.813 1.912a2 2 0 0 1 1.275 1.275L12 21l1.912-5.813a2 2 0 0 1 1.275-1.275L21 12l-5.813-1.912a2 2 0 0 1-1.275-1.275L12 3Z" />
                </svg>
              </button>
              <button @click="handleSend" :disabled="isSending || !inputAnswer.trim() || !currentQuestionId"
                class="w-12 h-12 flex items-center justify-center bg-blue-500 text-white rounded-xl hover:bg-blue-600 active:scale-95 transition-all shadow-md disabled:opacity-50"
                title="送信">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                  <line x1="22" y1="2" x2="11" y2="13"></line>
                  <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
                </svg>
              </button>
            </div>
          </div>
        </div>
        <!-- 次へボタン -->
        <div v-else class="fade-in w-full">
          <button @click="handleNextTopic"
            class="w-full bg-sky-500 text-white font-bold py-4 px-6 rounded-2xl hover:bg-sky-600 active:scale-[0.99] focus:outline-none transition-all shadow-md flex items-center justify-center space-x-3 text-lg">
            <span>次のトピックへ進む</span>
          </button>
        </div>
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue';
import axios from 'axios';

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
/* スクロールバーのデザイン */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #cbd5e1;
  border-radius: 20px;
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

/* 3点リーダー（思考中）アニメーション */
.typing-indicator {
  display: flex;
  align-items: center;
  gap: 4px;
}

.typing-indicator span {
  height: 6px;
  width: 6px;
  background-color: #94a3b8;
  border-radius: 50%;
  display: inline-block;
  animation: bounce 1.4s infinite ease-in-out both;
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
</style>
