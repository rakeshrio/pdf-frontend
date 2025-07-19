<template>
  <div class="w-full h-screen">
    <iframe
      v-if="pdfViewerSrc"
      id="document-preview"
      ref="pdfIframe"
      :src="pdfViewerSrc"
      width="100%"
      height="100%"
      title="document-preview"
    ></iframe>
    <button @click="handlePageClick">Go to Page 2 & Highlight 'Test'</button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

const pdfIframe = ref<HTMLIFrameElement | null>(null);
const selectedBlob = ref<string | null>(null);

const pdfViewerSrc = computed(() =>
  selectedBlob.value
    ? `/pdfjs-4.8.69-dist/web/viewer.html?file=${encodeURIComponent(selectedBlob.value)}`
    : null
);

function handlePageClick() {
  const viewerWindow = pdfIframe.value?.contentWindow;
  if (!viewerWindow) return;
  viewerWindow.postMessage({ type: 'navigateToPage', pageNumber: 2 }, '*');
  viewerWindow.postMessage(
    {
      type: 'find',
      query: 'Test',
      caseSensitive: false,
      highlightAll: true,
      findPrevious: false,
    },
    '*'
  );
}

async function loadDocumentBlob() {
  const response = await fetch('https://pdf-backend-demo.onrender.com/blob-pdf');
  const data = await response.blob();
  const blob = new Blob([data], { type: 'application/pdf' });
  selectedBlob.value = URL.createObjectURL(blob);
}

onMounted(() => {
  loadDocumentBlob();
});
</script>