<script lang="ts">

</script>
<template>
  <UContainer class="py-8">
   
    <div class="mx-auto mb-12 max-w-2xl text-center">
        <h2 class="text-3xl md:text-4xl font-bold mb-4 text-green-800 dark:text-green-300">
          Forum Diskusi JuruTani
        </h2>
        <p class="text-gray-600 dark:text-gray-300">
          Tempat diskusi dan tanya jawab bagi petani, penyuluh, dan pakar pertanian Indonesia.
        </p>
      </div>

    <div class="grid md:grid-cols-2 lg:grid-cols-4 gap-6 pb-12">
      <!-- Diskusi Penyuluh -->
      <UCard class="discussion-card hover:shadow-lg transition-all duration-200">
        <template #header>
          <div class="flex items-center">
            <div class="bg-primary/10 p-3 rounded-full mr-3">
              <UIcon name="i-heroicons-chat-bubble-left-ellipsis" class="w-6 h-6 text-primary" />
            </div>
            <h2 class="text-lg font-semibold text-primary">Bicara dengan Penyuluh</h2>
          </div>
        </template>

        <p class="text-gray-600 dark:text-gray-300 mb-6 leading-relaxed text-sm">
          Tanyakan langsung kepada penyuluh pertanian seputar permasalahan di lapangan.
        </p>

        <template #footer>
          <UButton 
            to="/discussions/instructor" 
            color="green" 
            size="sm" 
            block
            class="font-medium"
          >
            Mulai Diskusi
          </UButton>
        </template>
      </UCard>

      <!-- Diskusi Pakar -->
      <UCard class="discussion-card hover:shadow-lg transition-all duration-200">
        <template #header>
          <div class="flex items-center">
            <div class="bg-blue-100 dark:bg-blue-900/20 p-3 rounded-full mr-3">
              <UIcon name="i-heroicons-shield-check" class="w-6 h-6 text-blue-600 dark:text-blue-400" />
            </div>
            <h2 class="text-lg font-semibold text-blue-600 dark:text-blue-400">Tanya ke Pakar</h2>
          </div>
        </template>

        <p class="text-gray-600 dark:text-gray-300 mb-6 leading-relaxed text-sm">
          Diskusi untuk masalah teknis atau penyakit tanaman bersama ahli pertanian dari tim JuruTani.
        </p>

        <template #footer>
          <UButton 
            to="/discussions/expert" 
            color="blue" 
            size="sm" 
            block
            class="font-medium"
          >
            Konsultasi Pakar
          </UButton>
        </template>
      </UCard>

      <!-- Diskusi Komunitas -->
      <UCard class="discussion-card hover:shadow-lg transition-all duration-200">
        <template #header>
          <div class="flex items-center">
            <div class="bg-yellow-100 dark:bg-yellow-900/20 p-3 rounded-full mr-3">
              <UIcon name="i-heroicons-users" class="w-6 h-6 text-yellow-600 dark:text-yellow-400" />
            </div>
            <h2 class="text-lg font-semibold text-yellow-600 dark:text-yellow-400">Forum Komunitas</h2>
          </div>
        </template>

        <p class="text-gray-600 dark:text-gray-300 mb-6 leading-relaxed text-sm">
          Temui sesama petani, berbagi tips dan pengalaman langsung dari lapangan pertanian dan peternakan.
        </p>

        <template #footer>
          <UButton 
            to="/discussions/group" 
            color="yellow" 
            size="sm" 
            block
            class="font-medium"
          >
            Gabung Komunitas
          </UButton>
        </template>
      </UCard>

      <!-- Room Chat -->
      <UCard class="discussion-card hover:shadow-lg transition-all duration-200">
        <template #header>
          <div class="flex items-center">
            <div class="bg-purple-100 dark:bg-purple-900/20 p-3 rounded-full mr-3">
              <UIcon name="i-heroicons-chat-bubble-oval-left-ellipsis" class="w-6 h-6 text-purple-600 dark:text-purple-400" />
            </div>
            <h2 class="text-lg font-semibold text-purple-600 dark:text-purple-400">Room Chat</h2>
          </div>
        </template>

        <p class="text-gray-600 dark:text-gray-300 mb-6 leading-relaxed text-sm">
          Chat real-time dengan sesama petani dan penyuluh dalam berbagai room diskusi tematik.
        </p>

        <template #footer>
          <UButton 
            to="/room-chat" 
            color="violet" 
            size="sm" 
            block
            class="font-medium"
          >
            Masuk Room Chat
          </UButton>
        </template>
      </UCard>
    </div>

     
  </UContainer>
</template>

<style scoped>
.text-primary {
  color: #2E7D32;
}

.discussion-card {
  border: 1px solid #E8F5E8;
}

.discussion-card:hover {
  border-color: #C8E6C9;
}

@media (prefers-color-scheme: dark) {
  .discussion-card {
    border-color: #2E7D32;
  }
  
  .discussion-card:hover {
    border-color: #4CAF50;
  }
}
</style>