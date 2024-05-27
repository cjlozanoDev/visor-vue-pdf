<script setup>
import { ref } from 'vue'
import * as pdfjsLib from 'pdfjs-dist'
import 'pdfjs-dist/build/pdf.worker.min.mjs'

const viewerContainer = ref(null)
const pdfCanvas = ref(null)

const pdfBase64 = ref(null)

const handleFileUpload = (event) => {
  const file = event.target.files[0]

  const reader = new FileReader()
  reader.readAsDataURL(file)
  reader.onload = () => {
    pdfBase64.value = reader.result.split(',')[1]

    loadPdf()
  }
  reader.onerror = (error) => {
    console.error('Error reading file:', error)
  }
}

const base64ToUint8Array = (base64) => {
  var raw = atob(base64)
  var uint8Array = new Uint8Array(raw.length)
  for (var i = 0; i < raw.length; i++) {
    uint8Array[i] = raw.charCodeAt(i)
  }
  return uint8Array
}

const loadPdf = async () => {
  const canvas = pdfCanvas.value
  const context = canvas.getContext('2d')

  const loadingTask = pdfjsLib.getDocument({ data: base64ToUint8Array(pdfBase64.value) })

  const pdf = await loadingTask.promise

  const pageNumber = 1
  const page = await pdf.getPage(pageNumber)

  // Obtenemos el tamaño del contenedor
  const containerWidth = viewerContainer.value.clientWidth

  // Obtener el viewport y escalar según el ancho del contenedor
  const viewport = page.getViewport({ scale: 1 })
  const scale = containerWidth / viewport.width
  const scaledViewport = page.getViewport({ scale })

  // Ajustar el tamaño del canvas al tamaño escalado
  canvas.height = scaledViewport.height
  canvas.width = scaledViewport.width

  // Renderizar la página en el canvas
  const renderContext = {
    canvasContext: context,
    viewport: scaledViewport
  }

  await page.render(renderContext).promise
}
</script>

<template>
  <div>
    <header>
      <h1>Visor pdf</h1>
      <input type="file" @change="handleFileUpload" accept="application/pdf" />
    </header>
    <main>
      <div id="pdfViewer" ref="viewerContainer">
        <canvas ref="pdfCanvas"></canvas>
      </div>
    </main>
  </div>
</template>

<style scoped>
#pdfViewer {
  width: 800px;
  border: 1px solid black;
  margin: auto;
}
canvas {
  width: 100%;
}
</style>
