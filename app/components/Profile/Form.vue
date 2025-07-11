<script setup lang="ts">
import { ref, reactive, computed } from 'vue';
import { useSupabase } from '~/composables/useSupabase';
import { toastStore } from '~/composables/useJuruTaniToast';

interface UserData {
  id: string;
  email?: string;
  full_name?: string;
  username?: string;
  phone?: string;
  address?: string;
  bio?: string;
  website?: string;
  birth_date?: string;
  avatar_url?: string;
}

const props = defineProps<{
  userData: UserData;
}>();

const emit = defineEmits<{
  update: [data: UserData];
  cancel: [];
}>();

const { supabase } = useSupabase();

// State
const loading = ref(false);
const imageFile = ref<File | null>(null);
const imagePreview = ref<string | null>(null);

// Form data with all fields
const formData = reactive({
  full_name: props.userData.full_name || '',
  username: props.userData.username || '',
  phone: props.userData.phone || '',
  address: props.userData.address || '',
  bio: props.userData.bio || '',
  website: props.userData.website || '',
  birth_date: props.userData.birth_date || '',
  avatar_url: props.userData.avatar_url || ''
});

// Validation rules
const validations = {
  username: (value: string) => {
    if (!value) return true; // Optional field
    return /^[a-zA-Z0-9_]+$/.test(value) || 'Username hanya boleh berisi huruf, angka, dan underscore';
  },
  website: (value: string) => {
    if (!value) return true; // Optional field
    try {
      new URL(value.startsWith('http') ? value : `https://${value}`);
      return true;
    } catch {
      return 'Format website tidak valid';
    }
  },
  phone: (value: string) => {
    if (!value) return true; // Optional field
    return /^[\d\s\-+()]+$/.test(value) || 'Format nomor telepon tidak valid';
  }
};

// Computed validation
const formErrors = computed(() => {
  const errors: Record<string, string> = {};
  
  Object.entries(validations).forEach(([field, validator]) => {
    const result = validator(formData[field as keyof typeof formData]);
    if (typeof result === 'string') {
      errors[field] = result;
    }
  });
  
  return errors;
});

const isFormValid = computed(() => Object.keys(formErrors.value).length === 0);

// Image handling
const handleImageSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  if (!file) return;

  const validTypes = ['image/jpeg', 'image/png', 'image/jpg', 'image/gif'];
  const maxSize = 2 * 1024 * 1024; // 2MB

  if (!validTypes.includes(file.type)) {
    toastStore.error('Format file tidak didukung. Gunakan JPG, PNG, atau GIF.');
    return;
  }

  if (file.size > maxSize) {
    toastStore.error('Ukuran file terlalu besar. Maksimal 2MB.');
    return;
  }

  imageFile.value = file;
  
  // Create preview
  const reader = new FileReader();
  reader.onload = (e) => {
    imagePreview.value = e.target?.result as string;
  };
  reader.readAsDataURL(file);
};

const resetImage = () => {
  imageFile.value = null;
  imagePreview.value = null;
};

// Avatar upload utility
const uploadAvatar = async (userId: string): Promise<string | null> => {
  if (!imageFile.value) return null;

  // console.log('Starting avatar upload process...');
  
  // Delete old avatar
  if (formData.avatar_url?.includes('avatars/')) {
    try {
      const oldPath = formData.avatar_url.split('avatars/')[1].split('?')[0];
      if (oldPath) {
        // console.log('Attempting to delete old avatar:', oldPath);
        await supabase.storage.from('avatars').remove([oldPath]);
        // console.log('Old avatar deleted successfully');
      }
    } catch (error) {
      console.warn('Error deleting old avatar:', error);
    }
  }

  // Upload new avatar
  const fileExt = imageFile.value.name.split('.').pop();
  const fileName = `avatar_${Date.now()}.${fileExt}`;
  const filePath = `${userId}/${fileName}`;

  // console.log('Uploading new avatar:', { fileName, filePath, fileSize: imageFile.value.size });

  const { data, error } = await supabase.storage
    .from('avatars')
    .upload(filePath, imageFile.value, {
      cacheControl: '3600',
      upsert: true
    });

  if (error) {
    console.error('Upload error:', error);
    throw new Error(`Gagal mengunggah gambar profil: ${error.message}`);
  }

  // console.log('Avatar uploaded successfully:', data);

  // Get public URL
  const { data: { publicUrl } } = supabase.storage
    .from('avatars')
    .getPublicUrl(filePath);

  if (!publicUrl) {
    throw new Error('Gagal mendapatkan URL gambar profil.');
  }

  return `${publicUrl}?t=${Date.now()}`;
};

// Main submit handler
const handleSubmit = async () => {
  if (!isFormValid.value) {
    toastStore.error('Mohon perbaiki kesalahan pada form.');
    return;
  }

  loading.value = true;
  
  try {
    // Get authenticated user
    const { data: user } = await supabase.auth.getUser();
    // console.log("auth.uid():", user?.user?.id);
    // console.log("props.userData.id:", props.userData.id);
    
    if (!user?.user?.id) {
      console.error('No authenticated user found');
      toastStore.error('Anda harus login untuk memperbarui profil.');
      return;
    }

    if (user.user.id !== props.userData.id) {
      console.error('User ID mismatch:', {
        authUserId: user.user.id,
        profileUserId: props.userData.id
      });
      toastStore.error('Tidak dapat memperbarui profil pengguna lain.');
      return;
    }

    // Handle avatar upload
    let newAvatarUrl = formData.avatar_url;
    if (imageFile.value) {
      newAvatarUrl = await uploadAvatar(user.user.id);
      if (newAvatarUrl) {
        formData.avatar_url = newAvatarUrl;
        // console.log('New avatar URL:', newAvatarUrl);
      }
    }

    // Prepare update data
    const updates = {
      id: user.user.id,
      full_name: formData.full_name.trim(),
      username: formData.username.trim() || null,
      phone: formData.phone.trim() || null,
      address: formData.address.trim() || null,
      bio: formData.bio.trim() || null,
      website: formData.website.trim() || null,
      birth_date: formData.birth_date || null,
      avatar_url: newAvatarUrl,
      updated_at: new Date().toISOString()
    };

    // console.log('Updating profile with data:', updates);

    // Update profile
    const { data: updateData, error: updateError } = await supabase
      .from('profiles')
      .upsert(updates, {
        onConflict: 'id',
        ignoreDuplicates: false
      })
      .select();

    if (updateError) {
      console.error('Error updating profile:', updateError);
      throw new Error(`Gagal memperbarui profil: ${updateError.message}`);
    }

    // console.log('Profile updated successfully:', updateData);

    // Emit success
    emit('update', { ...updates, email: props.userData.email });
    toastStore.success('Profil berhasil diperbarui.');
    resetImage();
    
  } catch (error) {
    console.error('Exception updating profile:', error);
    toastStore.error(error instanceof Error ? error.message : 'Terjadi kesalahan saat memperbarui profil.');
  } finally {
    loading.value = false;
  }
};

const handleCancel = () => {
  resetImage();
  emit('cancel');
};

// Computed properties
const currentAvatar = computed(() => {
  return imagePreview.value || formData.avatar_url || '/profile.png';
});

const handleImageError = (event: Event) => {
  const target = event.target as HTMLImageElement;
  console.error('Image failed to load:', target.src);
  target.src = '/profile.png';
};

// Format website URL for display
const formatWebsiteUrl = (url: string) => {
  if (!url) return '';
  return url.startsWith('http') ? url : `https://${url}`;
};
</script>

<template>
  <form class="bg-white rounded-lg shadow-md p-6 max-w-2xl mx-auto" @submit.prevent="handleSubmit">
    <h2 class="text-xl font-semibold mb-6">Edit Profil</h2>

    <!-- Avatar Upload -->
    <div class="mb-6">
      <label class="block text-gray-700 text-sm font-medium mb-2">Foto Profil</label>
      
      <div class="flex items-center">
        <div class="w-24 h-24 rounded-full overflow-hidden mr-4 bg-gray-100 flex-shrink-0">
          <img 
            :src="currentAvatar" 
            alt="Avatar Preview"
            class="w-full h-full object-cover"
            @error="handleImageError"
          >
        </div>
        
        <div class="flex flex-col">
          <input 
            id="avatar" 
            type="file" 
            accept="image/*" 
            class="hidden"
            @change="handleImageSelect"
          >
          <div class="flex space-x-2">
            <label 
              for="avatar" 
              class="cursor-pointer bg-blue-100 text-blue-600 px-3 py-2 rounded hover:bg-blue-200 transition-colors"
            >
              Pilih Gambar
            </label>
            <button 
              v-if="imagePreview" 
              type="button" 
              class="bg-gray-100 text-gray-600 px-3 py-2 rounded hover:bg-gray-200 transition-colors"
              @click="resetImage"
            >
              Batal
            </button>
          </div>
          <p class="text-xs text-gray-500 mt-1">
            Format: JPG, PNG, GIF. Maks. 2MB
          </p>
        </div>
      </div>
    </div>

    <!-- Form Grid -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
      <!-- Full Name -->
      <div class="md:col-span-2">
        <label for="full_name" class="block text-gray-700 text-sm font-medium mb-2">
          Nama Lengkap *
        </label>
        <input
          id="full_name"
          v-model="formData.full_name"
          type="text"
          required
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Masukkan nama lengkap Anda"
        >
      </div>

      <!-- Username -->
      <div>
        <label for="username" class="block text-gray-700 text-sm font-medium mb-2">
          Username
        </label>
        <input
          id="username"
          v-model="formData.username"
          type="text"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': formErrors.username }"
          placeholder="username_anda"
        >
        <p v-if="formErrors.username" class="text-red-500 text-xs mt-1">
          {{ formErrors.username }}
        </p>
      </div>

      <!-- Phone -->
      <div>
        <label for="phone" class="block text-gray-700 text-sm font-medium mb-2">
          Nomor Telepon
        </label>
        <input
          id="phone"
          v-model="formData.phone"
          type="tel"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': formErrors.phone }"
          placeholder="08123456789"
        >
        <p v-if="formErrors.phone" class="text-red-500 text-xs mt-1">
          {{ formErrors.phone }}
        </p>
      </div>

      <!-- Birth Date -->
      <div>
        <label for="birth_date" class="block text-gray-700 text-sm font-medium mb-2">
          Tanggal Lahir
        </label>
        <input
          id="birth_date"
          v-model="formData.birth_date"
          type="date"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
      </div>

      <!-- Website -->
      <div>
        <label for="website" class="block text-gray-700 text-sm font-medium mb-2">
          Website
        </label>
        <input
          id="website"
          v-model="formData.website"
          type="url"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{ 'border-red-500': formErrors.website }"
          placeholder="https://website-anda.com"
        >
        <p v-if="formErrors.website" class="text-red-500 text-xs mt-1">
          {{ formErrors.website }}
        </p>
      </div>

      <!-- Address -->
      <div class="md:col-span-2">
        <label for="address" class="block text-gray-700 text-sm font-medium mb-2">
          Alamat
        </label>
        <input
          id="address"
          v-model="formData.address"
          type="text"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Masukkan alamat lengkap Anda"
        >
      </div>

      <!-- Bio -->
      <div class="md:col-span-2">
        <label for="bio" class="block text-gray-700 text-sm font-medium mb-2">
          Bio
        </label>
        <textarea
          id="bio"
          v-model="formData.bio"
          rows="4"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none"
          placeholder="Ceritakan sedikit tentang diri Anda..."
        />
        <p class="text-xs text-gray-500 mt-1">
          {{ formData.bio.length }}/300 karakter
        </p>
      </div>
    </div>

    <!-- Buttons -->
    <div class="flex justify-end space-x-3 pt-4 border-t">
      <button
        type="button"
        class="px-6 py-2 border border-gray-300 rounded-md text-gray-700 hover:bg-gray-100 transition-colors"
        :disabled="loading"
        @click="handleCancel"
      >
        Batal
      </button>
      <button
        type="submit"
        class="px-6 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors disabled:opacity-50"
        :disabled="loading || !isFormValid"
      >
        <span v-if="loading" class="flex items-center">
          <svg class="animate-spin -ml-1 mr-2 h-4 w-4 text-white" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"/>
          </svg>
          Menyimpan...
        </span>
        <span v-else>Simpan Perubahan</span>
      </button>
    </div>
  </form>
</template>

<style scoped>
/* Custom scrollbar for textarea */
textarea::-webkit-scrollbar {
  width: 6px;
}

textarea::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

textarea::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}

textarea::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}
</style>