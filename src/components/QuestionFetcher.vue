<template>
  <div class="question-container">
    <h2 class="text-3xl font-bold mb-6 text-gray-800 border-b pb-2">AIソクラテスからの質問</h2>

    <!-- データを取得中の場合 -->
    <div v-if="isLoading" class="flex items-center space-x-2 text-blue-500">
      <svg class="animate-spin h-5 w-5 mr-3 text-indigo-500" xmlns="http://www.w3.org/2000/svg" fill="none"
        viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
        <path class="opacity-75" fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z">
        </path>
      </svg>
      <p>データをロード中...</p>
    </div>

    <!-- エラーが発生した場合 -->
    <p v-else-if="error" class="text-red-600 font-semibold p-4 border border-red-300 rounded-lg bg-red-50">
      エラーが発生しました。バックエンドサーバーが起動しているか確認してください。<br>
      詳細: {{ error.message }}
    </p>

    <!-- データが正常に取得できた場合 -->
    <div v-else-if="questionData">
      <p class="text-sm text-gray-500 mb-2 italic">トピック: コンピュータ基礎</p>

      <!-- 結果表示エリア -->
      <div v-if="isAnswered && !answerResult.isChecking && !answerResult.error"
        :class="answerResult.isCorrect ? 'bg-green-100 border-green-400' : 'bg-red-100 border-red-400'"
        class="p-4 mb-4 border-l-4 rounded-lg shadow-md font-semibold">

        <p :class="answerResult.isCorrect ? 'text-green-800' : 'text-red-800'">
          {{ answerResult.isCorrect ? '✨ 素晴らしい、正解です！' : '🤔 惜しいです。' }}
        </p>

        <!-- バックエンドから返されるAIの解説・壁打ちメッセージ -->
        <p v-if="answerResult.explanation" class="text-sm mt-2 text-gray-700">{{ answerResult.explanation }}</p>

        <!-- 次の問題ボタン -->
        <button v-if="isAnswered && !answerResult.isChecking" @click="fetchQuestion"
          class="mt-4 w-full py-3 bg-indigo-600 text-white font-semibold rounded-lg shadow-lg hover:bg-indigo-700 transition duration-200">
          次の問題へ進む
        </button>
      </div>

      <!-- 問題文 -->
      <h3 class="text-2xl font-semibold mb-4 text-gray-700 p-4 bg-indigo-100 rounded-lg shadow-inner">
        {{ questionData.question_text }}
      </h3>

      <!-- ★★★ 修正箇所: 選択肢をヒントとして復活させる ★★★ -->
      <h5 class="text-base font-semibold mt-4 mb-2 text-gray-600 border-b border-dashed pb-1">ヒント (選択肢)</h5>
      <ul class="list-disc list-inside text-sm text-gray-500 mb-4 ml-4 space-y-1">
        <li v-for="choice in questionData.choices" :key="choice.id">
          {{ choice.choice_text }}
        </li>
      </ul>
      <!-- ★★★ 修正箇所 ここまで ★★★ -->

      <h4 class="text-lg font-medium mt-6 mb-3 border-l-4 border-indigo-500 pl-2">あなたの考え（回答）を教えてください:</h4>

      <!-- ★★★ テキスト入力フォーム ★★★ -->
      <textarea v-model="userAnswerText" :disabled="isAnswered"
        class="w-full p-3 border border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 transition duration-150 resize-none h-24"
        placeholder="例: CPUはコンピュータの頭脳なので、それにあたります。"></textarea>

      <button @click="checkAnswer()" :disabled="isAnswered || !userAnswerText.trim()"
        :class="{ 'opacity-50 cursor-not-allowed': isAnswered || !userAnswerText.trim() }"
        class="mt-3 w-full py-3 bg-indigo-600 text-white font-semibold rounded-lg shadow-lg hover:bg-indigo-700 transition duration-200">
        {{ isAnswered ? '回答済み' : '回答を送信' }}
      </button>

    </div>

    <!-- データがまだない場合（初期状態） -->
    <p v-else class="text-gray-500 p-4 bg-white rounded-lg border">APIからデータを取得する準備ができました。サーバーを確認してください。</p>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

// 状態管理変数 (既存)
const questionData = ref(null);
const isLoading = ref(true);
const error = ref(null);

// ステップ6で追加・更新する状態変数
const answerResult = ref(null);   // 回答結果
const isAnswered = ref(false);    // 回答済みフラグ
const userAnswerText = ref('');   // ★追加: ユーザーの入力テキスト


/**
 * バックエンドAPIを呼び出し、問題データを取得する関数 (リセット処理を追加)
 */
const fetchQuestion = async () => {
  isLoading.value = true;
  error.value = null;
  // ★★★ ステップ6で追加: リセット処理 ★★★
  isAnswered.value = false;
  answerResult.value = null;
  userAnswerText.value = ''; // テキスト入力をリセット
  // ★★★★★★★★★★★★★★★★★★★★★★★★★★

  try {
    const API_URL = 'http://localhost/api/topics/1/next-question';
    const response = await axios.get(API_URL);
    questionData.value = response.data;

  } catch (err) {
    console.error("API呼び出しエラー:", err);
    if (err.response) {
      error.value = { message: `サーバーエラー (${err.response.status}): ${err.response.data.message || 'データ取得に失敗しました'}` };
    } else if (err.request) {
      error.value = { message: 'サーバーからの応答がありません。バックエンドが実行されているか確認してください。' };
    } else {
      error.value = { message: err.message };
    }
  } finally {
    isLoading.value = false;
  }
};


/**
 * ユーザーの回答(テキスト)をバックエンドに送信し、正誤判定を行う関数
 */
const checkAnswer = async () => {
  // 回答済み、入力が空、データがない場合は実行しない
  if (isAnswered.value || !userAnswerText.value.trim() || !questionData.value) return;

  isAnswered.value = true;
  answerResult.value = { isChecking: true };

  try {
    const questionId = questionData.value.id;
    const API_URL = `http://localhost/api/questions/${questionId}/check`;

    // answer_text を送信
    const response = await axios.post(API_URL, {
      answer_text: userAnswerText.value
    });

    // 成功した場合、正誤判定結果を格納
    answerResult.value = {
      isChecking: false,
      isCorrect: response.data.is_correct,
      explanation: response.data.explanation,
      user_answer: response.data.user_answer
    };

  } catch (err) {
    console.error("回答送信エラー:", err);
    isAnswered.value = false;
    answerResult.value = {
      isChecking: false,
      error: "回答の送信に失敗しました。"
    };
  }
};

onMounted(fetchQuestion);
</script>

<style scoped>
/* Tailwind CSSを使用することを想定したスタイル */
.question-container {
  padding: 24px;
  background-color: #ffffff;
  border-radius: 12px;
  max-width: 700px;
  width: 90%;
  margin: 40px auto;
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1);
}
</style>
