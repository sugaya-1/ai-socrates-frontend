<script setup>
// QuestionFetcherコンポーネントをインポート
import QuestionFetcher from './components/QuestionFetcher.vue';
</script>

<template>
  <div id="app" class="min-h-screen bg-gray-100 font-sans antialiased">
    <header class="bg-indigo-600 shadow-lg">
      <div class="max-w-7xl mx-auto py-5 px-4 sm:px-6 lg:px-8">
        <h1 class="text-4xl font-extrabold tracking-tight text-white text-center">
          <span class="mr-2">💡</span> AIソクラテス
        </h1>
        <p class="text-center text-indigo-200 mt-1">対話を通じてあなたの知識を深めます</p>
      </div>
    </header>

    <main class="py-10">
      <!-- 作成した問題取得コンポーネントを配置 -->
      <QuestionFetcher />
    </main>

    <footer class="text-center text-sm text-gray-500 py-4 mt-10 border-t">
      © 2024 AIソクラテス開発プロジェクト
    </footer>
  </div>
</template>

<style>
/* Vue/Tailwindプロジェクトの基本スタイル。Tailwindがクラスを適用します。 */
/* App.vueでTailwindのクラスを使用するために、このファイルに特別なCSSは不要です。 */
</style>
