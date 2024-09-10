<template>
  <div class="container mx-auto p-4 max-w-xl" data-theme="dark">
    <h1 class="text-2xl font-bold mb-4 text-center">Misskey Emoji Bulk Uploader</h1>
    <div class="bg-base-200 p-4 rounded-lg mb-4">
      <h2 class="text-lg font-semibold mb-2">Important Information:</h2>
      <ul class="list-disc list-inside text-sm">
        <li>絵文字の追加権限のあるAPIキーをご利用ください</li>
        <li>ロールでレートリミットを0にすることを推奨しています</li>
      </ul>
    </div>
    <form @submit.prevent="startUpload" class="space-y-2">
      <div class="flex flex-wrap -mx-2">
        <div class="w-full md:w-1/2 px-2 mb-2">
          <label class="label">
            <span class="label-text">Misskey Server URL:</span>
          </label>
          <input v-model="serverUrl" type="url" required 
                 placeholder="https://your-misskey-instance.com"
                 class="input input-bordered w-full input-sm">
        </div>
        <div class="w-full md:w-1/2 px-2 mb-2">
          <label class="label">
            <span class="label-text">API Key:</span>
          </label>
          <input v-model="apiKey" type="password" required 
                 placeholder="Your Misskey API key"
                 class="input input-bordered w-full input-sm">
        </div>
      </div>
      <div class="flex flex-wrap -mx-2">
        <div class="w-full md:w-1/2 px-2 mb-2">
          <label class="label">
            <span class="label-text">Emoji Category:</span>
          </label>
          <input v-model="category" type="text" 
                 placeholder="Optional category"
                 class="input input-bordered w-full input-sm">
        </div>
        <div class="w-full md:w-1/2 px-2 mb-2">
          <label class="label">
            <span class="label-text">Rate Limit (ms):</span>
          </label>
          <input v-model="rateLimit" type="number" required 
                 placeholder="Delay between uploads in ms" min="0"
                 class="input input-bordered w-full input-sm">
        </div>
      </div>
      <div class="mb-2">
        <label class="label">
          <span class="label-text">Select Emoji Files:</span>
        </label>
        <input type="file" @change="onFilesSelected" 
               accept="image/*" multiple required
               class="file-input file-input-bordered file-input-sm w-full">
      </div>
      
      <!-- Emoji Preview Grid with Lazy Loading -->
      <div v-if="selectedFiles.length > 0" class="mb-4">
        <div class="flex justify-between items-center mb-2">
          <h2 class="text-lg font-semibold">Selected Emojis: {{ remainingFiles.length }} remaining</h2>
          <button type="button" @click="removeAllEmojis" class="btn btn-error btn-xs">
            Remove All
          </button>
        </div>
        <div class="grid grid-cols-3 gap-2">
          <div v-for="(file, index) in remainingFiles" :key="index" 
               class="flex flex-col items-center relative">
            <button @click="removeEmoji(file)" class="btn btn-circle btn-xs absolute top-0 right-0 bg-error text-white">
              ×
            </button>
            <div class="w-16 h-16 bg-gray-200 rounded-lg mb-1 overflow-hidden">
              <img v-lazy="getFilePreview(file)" 
                   alt="Emoji preview" 
                   class="w-full h-full object-contain"
                   loading="lazy">
            </div>
            <input v-model="file.customName" @input="updateFileName(index)" 
                   class="input input-bordered input-xs w-full text-center"
                   :placeholder="getFileName(file)">
          </div>
        </div>
      </div>

      <div class="flex justify-between">
        <button type="submit" class="btn btn-primary btn-sm" :disabled="isUploading || isPaused">
          {{ isUploading ? 'Uploading...' : 'Upload Emojis' }}
        </button>
        <button type="button" class="btn btn-secondary btn-sm" @click="togglePause" :disabled="!isUploading && !isPaused">
          {{ isPaused ? 'Resume' : 'Pause' }}
        </button>
      </div>
    </form>
    <div v-if="messages.length > 0" class="mt-4">
      <h2 class="text-lg font-semibold mb-2">Upload Results:</h2>
      <ul class="list-disc list-inside">
        <li v-for="(msg, index) in messages" :key="index" class="text-sm">{{ msg }}</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const serverUrl = ref('')
const apiKey = ref('')
const category = ref('')
const rateLimit = ref(1000)
const selectedFiles = ref([])
const messages = ref([])
const isUploading = ref(false)
const isPaused = ref(false)

const remainingFiles = computed(() => selectedFiles.value.filter(file => !file.uploaded))

const onFilesSelected = (event) => {
  selectedFiles.value = Array.from(event.target.files).map(file => ({
    file,
    customName: '',
    uploaded: false
  }))
}

const getFilePreview = (fileObj) => {
  return URL.createObjectURL(fileObj.file)
}

const getFileName = (fileObj) => {
  return fileObj.file.name.split('.')[0].replace(/-/g, '_')
}

const updateFileName = (index) => {
  const file = selectedFiles.value[index]
  if (!file.customName) {
    file.customName = getFileName(file)
  } else {
    file.customName = file.customName.replace(/-/g, '_')
  }
}

const removeEmoji = (fileToRemove) => {
  selectedFiles.value = selectedFiles.value.filter(file => file !== fileToRemove)
}

const removeAllEmojis = () => {
  selectedFiles.value = []
  messages.value = []
}

const uploadFile = async (fileObj) => {
  const formData = new FormData()
  formData.append('file', fileObj.file)
  formData.append('force', 'true')
  formData.append('name', fileObj.file.name)

  try {
    const response = await $fetch(`${serverUrl.value}/api/drive/files/create`, {
      method: 'POST',
      body: formData,
      headers: {
        'Authorization': `Bearer ${apiKey.value}`
      }
    })
    return response.id
  } catch (error) {
    throw new Error(`Error uploading file ${fileObj.file.name}: ${error.message}`)
  }
}

const createEmoji = async (fileId, name) => {
  try {
    await $fetch(`${serverUrl.value}/api/admin/emoji/add`, {
      method: 'POST',
      body: {
        name: name,
        category: category.value || null,
        aliases: [],
        license: null,
        isSensitive: false,
        localOnly: false,
        roleIdsThatCanBeUsedThisEmojiAsReaction: [],
        fileId: fileId,
        i: apiKey.value
      },
      headers: {
        'Content-Type': 'application/json'
      }
    })
  } catch (error) {
    throw new Error(`Error creating emoji ${name}: ${error.message}`)
  }
}

const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms))

const uploadEmojis = async () => {
  while (remainingFiles.value.length > 0 && !isPaused.value) {
    const fileObj = remainingFiles.value[0]
    try {
      const fileId = await uploadFile(fileObj)
      const emojiName = (fileObj.customName || getFileName(fileObj)).replace(/-/g, '_')
      await createEmoji(fileId, emojiName)
      messages.value.push(`Successfully uploaded and created emoji: ${emojiName}`)
      selectedFiles.value = selectedFiles.value.filter(f => f !== fileObj)
    } catch (error) {
      messages.value.push(`Error uploading ${getFileName(fileObj)}: ${error.message}`)
      selectedFiles.value = [...selectedFiles.value.filter(f => f !== fileObj), fileObj]
    } finally {
      if (remainingFiles.value.length > 0) {
        await sleep(Number(rateLimit.value))
      }
    }
  }

  if (remainingFiles.value.length === 0) {
    isUploading.value = false
    messages.value.push('All upload attempts completed.')
  }
}

const startUpload = () => {
  if (selectedFiles.value.length === 0) {
    messages.value.push('Please select files to upload.')
    return
  }
  isUploading.value = true
  isPaused.value = false
  uploadEmojis()
}

const togglePause = () => {
  isPaused.value = !isPaused.value
  if (!isPaused.value && remainingFiles.value.length > 0) {
    uploadEmojis()
  }
}

// Lazy loading implementation
const lazyLoadObserver = ref(null)

const setupLazyLoading = () => {
  lazyLoadObserver.value = new IntersectionObserver((entries, observer) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target
        img.src = img.dataset.src
        observer.unobserve(img)
      }
    })
  }, {
    root: null,
    rootMargin: '0px',
    threshold: 0.1
  })
}

const vLazy = {
  mounted: (el, binding) => {
    el.dataset.src = binding.value
    lazyLoadObserver.value.observe(el)
  },
  unmounted: (el) => {
    lazyLoadObserver.value.unobserve(el)
  }
}

onMounted(() => {
  setupLazyLoading()
})

onUnmounted(() => {
  if (lazyLoadObserver.value) {
    lazyLoadObserver.value.disconnect()
  }
})
</script>