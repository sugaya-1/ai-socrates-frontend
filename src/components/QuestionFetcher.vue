<template>
  <div class="flex flex-col h-[85vh] max-w-3xl mx-auto bg-white rounded-xl shadow-2xl overflow-hidden">

    <!-- 1. 会話表示エリア (Chat History) -->
    <div ref="chatWindow" class="flex-grow p-6 space-y-6 overflow-y-auto custom-scrollbar">

      <!-- ロード中メッセージ -->
      <div v-if="isLoading" class="text-center text-gray-500 py-10">
        <svg class="animate-spin h-6 w-6 text-indigo-500 mx-auto mb-2" xmlns="http://www.w3.org/2000/svg" fill="none"
          viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor"
            d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z">
          </path>
        </svg>
        <p>AIソクラテスが知識を整理しています...</p>
      </div>

      <!-- 会話メッセージのループ -->
      <div v-for="(message, index) in history" :key="index" class="flex"
        :class="message.sender === 'user' ? 'justify-end' : 'justify-start'">

        <div
          :class="message.sender === 'user' ? 'bg-indigo-500 text-white rounded-br-none' : 'bg-gray-100 text-gray-800 rounded-tl-none'"
          class="max-w-[80%] px-4 py-3 rounded-xl shadow-md whitespace-pre-wrap">

          <!-- AI (質問/応答) -->
          <div v-if="message.sender === 'ai'">
            <span class="font-bold text-indigo-600 mr-2" v-if="message.type === 'question'">💡 質問:</span>
            <span class="font-bold text-gray-700 mr-2" v-else>🧠 AI:</span>
            <p class="mt-1" v-html="message.text"></p>

            <!-- ヒントと選択肢（初回質問時のみ表示） -->
            <div v-if="message.choices && message.choices.length > 0 && message.type === 'question'"
              class="mt-3 text-xs p-2 bg-gray-200 rounded-lg border border-gray-300">
              <p class="font-semibold text-gray-700 mb-1">ヒント（選択肢）:</p>
              <ul class="list-disc list-inside ml-4">
                <li v-for="choice in message.choices" :key="choice.id">
                  {{ choice.choice_text }}
                </li>
              </ul>
            </div>

            <!-- AI応答の正誤ステータス表示 -->
            <div v-if="message.is_correct !== undefined && message.type !== 'question'"
              class="mt-2 pt-2 border-t border-gray-300">
              <span class="text-xs font-semibold" :class="message.is_correct ? 'text-green-600' : 'text-orange-500'">
                {{ message.is_correct ? '✅ 正誤判定: 正解' : '⚠️ 正誤判定: 要検討' }}
              </span>
            </div>
          </div>

          <!-- ユーザー回答 -->
          <div v-else>
            <span class="font-bold">👤 あなた:</span>
            <p class="mt-1">{{ message.text }}</p>
          </div>
        </div>
      </div>

      <!-- エラー表示 (チャット形式) -->
      <div v-if="error" class="text-red-600 p-4 bg-red-50 border border-red-300 rounded-xl">
        エラー: {{ error.message }}
        <button @click="fetchInitialQuestion" class="ml-4 text-sm underline">問題を再取得</button>
      </div>
    </div>

    <!-- 2. 入力フォームエリア (Fixed Bottom) -->
    <div class="p-4 border-t border-gray-200 bg-white sticky bottom-0">

      <!-- ★修正: 通常の入力エリア (isSufficientがfalseの間だけ表示) ★ -->
      <textarea v-if="!isSufficient" v-model="inputAnswer" @keydown.enter.prevent.exact="handleSend"
        :disabled="isSending || !currentQuestionId"
        class="w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 transition duration-150 resize-none h-16"
        placeholder="あなたの考えを詳しく入力 (Enterで送信, Shift+Enterで改行)"></textarea>

      <!-- ★修正: 送信ボタン (isSufficientがfalseの間だけ表示) ★ -->
      <button v-if="!isSufficient" @click="handleSend" :disabled="isSending || !inputAnswer.trim() || !currentQuestionId" :class="{
        'opacity-50 cursor-not-allowed bg-gray-400': isSending || !inputAnswer.trim() || !currentQuestionId,
        'hover:bg-indigo-700': !isSending && inputAnswer.trim() && currentQuestionId
      }"
        class="mt-2 w-full py-3 bg-indigo-600 text-white font-semibold rounded-lg shadow-lg transition duration-200 flex items-center justify-center">
        <svg v-if="isSending" class="animate-spin h-5 w-5 mr-3" xmlns="http://www.w3.org/2000/svg" fill="none"
          viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor"
            d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z">
          </path>
        </svg>
        {{ isSending ? 'AIが思考中...' : '回答を送信' }}
      </button>

      <!-- ★追加: 次の問題へ進むボタン (isSufficientがtrueの時だけ表示) ★ -->
      <button v-else @click="fetchInitialQuestion"
        class="mt-2 w-full py-3 bg-green-500 text-white font-semibold rounded-lg shadow-lg transition duration-200 hover:bg-green-600 flex items-center justify-center space-x-2">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><polyline points="12 16 16 12 12 8"></polyline><line x1="8" y1="12" x2="16" y2="12"></line></svg>
        <span>理解完了！次の問題へ進む</span>
      </button>

    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue';
import axios from 'axios';

// DOM要素への参照
const chatWindow = ref(null);

// 状態管理変数
const history = ref([]); // 会話履歴全体 (メッセージ配列)
const isLoading = ref(true);
const error = ref(null);
const inputAnswer = ref('');    // ユーザーの入力テキスト
const isSending = ref(false);   // 送信中フラグ
const isSufficient = ref(false); // ★追加: 問答が終了したかどうかのフラグ

// 現在の問題ID (会話のどの部分に属するかを識別するために必要)
const currentQuestionId = ref(null);

// トピックID (現在は固定)
const currentTopicId = 1;

/**
 * InteractionデータをフロントエンドのMessageフォーマットに変換するヘルパー関数
 */
const formatInteractions = (interactions) => {
  const formattedHistory = [];

  interactions.forEach(interaction => {
    // ユーザーの回答をプッシュ
    formattedHistory.push({
      sender: 'user',
      type: 'answer',
      text: interaction.user_answer,
      question_id: interaction.question_id,
    });

    // AIの応答をプッシュ
    formattedHistory.push({
      sender: 'ai',
      type: 'feedback',
      text: interaction.ai_response,
      question_id: interaction.question_id,
      is_correct: interaction.is_correct,
    });
  });
  return formattedHistory;
};


/**
 * チャットウィンドウを最下部までスクロールする
 */
const scrollToBottom = () => {
  nextTick(() => {
    const el = chatWindow.value;
    if (el) {
      el.scrollTop = el.scrollHeight;
    }
  });
};

/**
 * 過去の会話履歴をデータベースから取得する関数
 */
const fetchHistory = async (questionId) => {
  // ★修正: currentQuestionIdではなく、引数で渡されたIDを使う
  const tempQuestionId = questionId || 1;

  try {
    const API_URL = `http://localhost/api/questions/${tempQuestionId}/history`;
    const response = await axios.get(API_URL);
    return formatInteractions(response.data.history);
  } catch (err) {
    console.error("履歴取得エラー:", err);
    return [];
  }
};


/**
 * 最初の質問、または次の質問を取得し、会話履歴をリセットする
 */
const fetchInitialQuestion = async () => {
  isLoading.value = true;
  error.value = null;
  isSending.value = false;

  // ★修正: 新しい問題に移るので、履歴と終了フラグをリセット
  history.value = [];
  isSufficient.value = false;

  // 1. 次の質問を取得 (GET /api/topics/{topicId}/next-question)
  try {
    const API_URL = `http://localhost/api/topics/${currentTopicId}/next-question`;
    const response = await axios.get(API_URL);
    const data = response.data;

    currentQuestionId.value = data.id; // 現在の問題IDを更新

    // 2. 質問メッセージを作成して履歴に追加
    const questionMessage = {
      sender: 'ai',
      type: 'question',
      text: data.question_text,
      question_id: data.id,
      choices: data.choices // ヒントとして表示するために選択肢も保持
    };
    history.value.push(questionMessage);

    // 3. 過去の対話履歴をDBから取得し、会話に追記 (重要)
    // ★修正: 取得した新しい問題IDを使って履歴を取得する
    const pastInteractions = await fetchHistory(data.id);
    history.value.push(...pastInteractions);


  } catch (err) {
    console.error("問題取得エラー:", err);
    error.value = { message: err.response?.data?.message || '問題データの取得に失敗しました。' };
  }

  isLoading.value = false;
  scrollToBottom();
};


/**
 * ユーザーが回答を送信した際の処理 (初回回答、または継続的な対話の回答)
 */
const handleSend = async () => {
  if (isSending.value || !inputAnswer.value.trim() || !currentQuestionId.value) return;

  const answer = inputAnswer.value;
  inputAnswer.value = ''; // 入力エリアをクリア
  isSending.value = true;

  // 1. ユーザーの回答を履歴に追加
  history.value.push({
    sender: 'user',
    type: 'answer',
    text: answer,
    question_id: currentQuestionId.value
  });
  scrollToBottom();

  try {
    // 2. バックエンドAPIを呼び出し (POST /api/questions/{questionId}/check)
    const API_URL = `http://localhost/api/questions/${currentQuestionId.value}/check`;

    const response = await axios.post(API_URL, {
      answer_text: answer
    });

    const data = response.data;

    // 3. AIの応答を履歴に追加
    history.value.push({
      sender: 'ai',
      type: 'feedback',
      text: data.explanation,
      question_id: currentQuestionId.value,
      is_correct: data.is_correct, // 正誤判定結果も表示に利用
    });

    // ★追加: バックエンドからの終了合図をチェックし、フラグを立てる
    if (data.is_sufficient === true) {
      isSufficient.value = true;
    }

  } catch (err) {
    console.error("回答送信エラー:", err);
    error.value = { message: err.response?.data?.message || 'AI通信中にエラーが発生しました。' };
  } finally {
    isSending.value = false;
    scrollToBottom();
  }
};

onMounted(fetchInitialQuestion);
</script>

<style scoped>
/* スクロールバーのスタイルはブラウザによって異なるため、簡易的に定義 */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #ccc;
  border-radius: 3px;
}
</style>

