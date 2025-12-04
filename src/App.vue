<script setup>
// QuestionFetcherコンポーネントをインポート
import QuestionFetcher from './components/QuestionFetcher.vue';
</script>

<template>
  <div id="app" class="min-h-screen bg-gray-100 font-sans antialiased">
    <header class="relative bg-black shadow-lg overflow-hidden">
      <div class="absolute inset-0 bg-cover bg-center z-0" :style="{ backgroundImage: `url('/bg-3.jpg')` }"></div>

      <div class="absolute inset-0 bg-black/40 z-10"></div>

      <div class="relative z-20 max-w-7xl mx-auto py-10 px-4 sm:px-6 lg:px-8 w-full">
        <h1
          class="text-4xl md:text-5xl font-extrabold tracking-tight text-cyan-100 text-center font-serif drop-shadow-[0_0_8px_rgba(34,211,238,0.5)] mb-6">
          Agora Digitalis
        </h1>
        <p class="text-lg md:text-xl text-cyan-100 text-center font-serif drop-shadow-[0_0_8px_rgba(34,211,238,0.5)]">
          対話を通じてあなたの知識を深めます
        </p>
      </div>
    </header>


    <main class="">
      <!-- 作成した問題取得コンポーネントを配置 -->
      <QuestionFetcher />
    </main>

    <footer class="text-center text-sm text-gray-500 py-4  border-t">
      © 2025 AIソクラテス
    </footer>
  </div>
</template>

<style>
/* Vue/Tailwindプロジェクトの基本スタイル。Tailwindがクラスを適用します。 */
/* App.vueでTailwindのクラスを使用するために、このファイルに特別なCSSは不要です。 */
</style>
