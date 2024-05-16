<script setup lang="ts">
// import { openaiApi } from '@/api/openai'

type SelectionStatus = 'initial' | 'tooltip' | 'popover'

const originalText = ref('She is very beautiful.')
const paraphrasedText = ref('She is very very <b>pretty</b>.')

async function handleParaphrase() {
  if (!originalText.value)
    return

  paraphrasedText.value = 'Hee Hee'
}

const inputRef = ref<HTMLElement>()

const status = ref<SelectionStatus>('initial')

const boundingRect = ref({ x: 0, y: 0 })

const replacedContent = ref('')

let selection: Selection | null = null

const mousePos = ref({ x: 0, y: 0 })

onMounted(() => {
  document.addEventListener('selectionchange', () => {
    selection = window.getSelection()

    if (!selection?.rangeCount || selection.toString().length === 0) {
      // tooltipVisible.value = false

      return
    }

    const range = selection.getRangeAt(0)

    const rect = range.getBoundingClientRect()

    boundingRect.value = {
      x: rect.right,
      y: rect.top,
    }
  })

  document.addEventListener('mousemove', (e) => {
    mousePos.value.x = e.clientX
    mousePos.value.y = e.clientY
  })
})

function handleMouseUp() {
  const selection = window.getSelection()

  if (selection?.toString().length === 0)
    return

  status.value = 'tooltip'
}

function isMouseInElement(element: HTMLElement) {
  const elementRect = element.getBoundingClientRect()

  return (
    mousePos.value.x >= elementRect.left
    && mousePos.value.x <= elementRect.right
    && mousePos.value.y >= elementRect.top
    && mousePos.value.y <= elementRect.bottom
  )
}

function reselectElement() {
  if (!selection?.anchorNode || !selection.focusNode)
    return

  const range = document.createRange()

  range.setStart(selection?.anchorNode, selection?.anchorOffset || 0)
  range.setEnd(selection?.focusNode, selection?.focusOffset || 0)

  selection.removeAllRanges()
  selection.addRange(range)
}

function handleBlur() {
  if (inputRef.value && isMouseInElement(inputRef.value))
    reselectElement()
}

let i = 0

function handleApply() {
  if (!selection?.rangeCount)
    return

  const range = selection.getRangeAt(0)

  range?.deleteContents()

  range?.insertNode(document.createTextNode(replacedContent.value))

  i++

  replacedContent.value = ''
  status.value = 'initial'
  selection = null
}

function handleOpenPopover() {
  status.value = 'popover'

  replacedContent.value = `Hello World ${i}`
}
</script>

<template>
  <div>
    <div>
      x: {{ mousePos.x }}
      y: {{ mousePos.y }}
    </div>
    <div style="display: flex;">
      Original:
      <div>
        <textarea v-model="originalText" />
      </div>
      Paraphrased:
      <div
        ref="inputRef"
        contenteditable
        style="border: 1px solid rgb(129, 129, 129); min-width: 200px; min-height: 120px;"
        @blur="handleBlur"
        @mouseup="handleMouseUp"
        v-html="paraphrasedText"
      />
    </div>
    <div
      v-if="status === 'tooltip'"
      :style="{
        position: 'fixed',
        top: `${boundingRect.y}px`,
        left: `${boundingRect.x}px`,
        backgroundColor: 'red',
        width: '20px',
        height: '20px',
      }"
      @click="handleOpenPopover"
    />
    <div
      v-else-if="status === 'popover'"
      :style="{
        position: 'fixed',
        top: `${boundingRect.y}px`,
        left: `${boundingRect.x}px`,
        backgroundColor: 'yellow',
        maxWidth: '100px',
        maxHeight: '100px',
        padding: '5px',
      }"
    >
      <div>
        {{ replacedContent }}
      </div>
      <button @click="handleApply">
        Apply
      </button>
    </div>
    <button @click="handleParaphrase">
      Paraphrase
    </button>
  </div>
</template>
