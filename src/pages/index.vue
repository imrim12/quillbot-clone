<script setup lang="ts">
import { openaiApi } from '@/api/openai'

const originalText = ref('')
const paraphrasedText = ref('')

async function handleParaphrase() {
  if (!originalText.value)
    return

  paraphrasedText.value = await openaiApi.getParaphraseFullContent(originalText.value)
}
</script>

<template>
  <div>
    <div style="display: flex;">
      <div>
        <textarea v-model="originalText" />
      </div>
      <div>
        {{ paraphrasedText }}
      </div>
    </div>
    <button @click="handleParaphrase">
      Paraphrase
    </button>
  </div>
</template>
