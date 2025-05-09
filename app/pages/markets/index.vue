<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue';
import { useSupabase } from '~/composables/useSupabase';
import { CreateButton } from '#components';

const { supabase } = useSupabase();

// Data
const marketsList = ref([]);
const loading = ref(true);
const error = ref(null);

// Filter and pagination
const currentCategory = ref('all');
const categories = ['all', 'Hasil Pertanian', 'Alat Pertanian', 'Pupuk', 'Lainya'];
const currentPage = ref(1);
const pageSize = ref(12);
const totalPages = ref(1);

// Fetch markets with filters
const fetchMarkets = async () => {
  loading.value = true;
  error.value = null;

  try {
    let query = supabase
      .from('markets')
      .select(`
        *
      `, { count: 'exact' }) // hanya ambil kolom yang dibutuhkan
      .is('deleted_at', null)
      .is('archived_at', null)
      .order('created_at', { ascending: false })
      .range(
        (currentPage.value - 1) * pageSize.value,
        currentPage.value * pageSize.value - 1
      );

    if (currentCategory.value !== 'all') {
      query = query.eq('category', currentCategory.value);
    }

    const { data, error: fetchError, count } = await query;

    if (fetchError) {
      console.error('[Supabase Fetch Error]', fetchError);
      error.value = fetchError;
    } else {
      console.log('[Markets Fetched Data]', data); // ✅ Debug log
      marketsList.value = data || [];
      totalPages.value = Math.ceil((count || 0) / pageSize.value);
    }
  } catch (err) {
    console.error('[FetchMarkets Error]', err);
    error.value = err;
  } finally {
    loading.value = false;
  }
};


// Reset to page 1 when category changes
watch(currentCategory, () => {
  currentPage.value = 1;
  fetchMarkets();
});

// Refetch when page changes
watch(currentPage, () => {
  fetchMarkets();
});

// Initial fetch
onMounted(() => {
  fetchMarkets();
});
</script>

<template>
  <div class="markets-page container mx-auto px-4">
    <h1 class="text-2xl font-bold text-center my-8">Pasar Tani</h1>
    
    <!-- Category Filter -->
    <MarketsFilterCategory 
      :categories="categories" 
      :current-category="currentCategory" 
      @update:category="currentCategory = $event" 
    />
    
    <!-- Markets Content -->
    <div class="mt-8">
      <div v-if="loading" class="text-center py-10">
        <p class="text-gray-500">Memuat produk...</p>
      </div>
      
      <div v-else-if="error" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
        <p>Terjadi kesalahan saat memuat produk.</p>
      </div>
      
      <div v-else-if="marketsList.length === 0" class="text-center py-10">
        <p class="text-gray-500">Tidak ada produk tersedia untuk kategori ini.</p>
      </div>
      
      <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        <MarketsCardContent 
          v-for="product in marketsList" 
          :key="product.id" 
          :product="product" 
        />
      </div>
    </div>
    
    <!-- Pagination -->
    <Pagination 
      v-if="!loading && marketsList.length > 0" 
      :current-page="currentPage" 
      :total-pages="totalPages" 
      @prev="currentPage > 1 ? currentPage-- : null" 
      @next="currentPage < totalPages ? currentPage++ : null" 
      @goto="page => currentPage = page" 
    />
    <CreateButton />
  </div>
</template>