<script setup>
import { ref, onMounted } from 'vue'
import Hls from 'hls.js'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  LineElement,
  CategoryScale,
  LinearScale,
  PointElement,
} from 'chart.js'
import { Line } from 'vue-chartjs'

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  LineElement,
  CategoryScale,
  LinearScale,
  PointElement
)

const video = ref(null)
let hlsInstance = null
const streamUrl = 'https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8'

const isStreaming = ref(true)
const telemetry = ref({
  speed: 65,
  temp: 42,
})

const chartData = ref({
  labels: ['0с', '5с', '10с', '15с', '20с'],
  datasets: [
    {
      label: 'Температура °C',
      data: [40, 42, 43, 42, 44],
      borderColor: '#22d3ee', // cyan-400
      backgroundColor: 'rgba(34, 211, 238, 0.2)',
      tension: 0.3,
      fill: true,
    },
  ],
})

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  scales: {
    y: {
      beginAtZero: true,
      ticks: { color: '#000' },
      grid: { color: '#e5e7eb' },
    },
    x: {
      ticks: { color: '#000' },
      grid: { color: '#e5e7eb' },
    },
  },
  plugins: {
    legend: { labels: { color: '#000' } },
    title: { display: false },
  },
}

const startStream = () => {
  if (Hls.isSupported()) {
    hlsInstance = new Hls()
    hlsInstance.loadSource(streamUrl)
    hlsInstance.attachMedia(video.value)
    isStreaming.value = true
  } else if (video.value.canPlayType('application/vnd.apple.mpegurl')) {
    video.value.src = streamUrl
    isStreaming.value = true
  }
}

const stopStream = () => {
  if (hlsInstance) {
    hlsInstance.destroy()
    hlsInstance = null
  }
  video.value.pause()
  video.value.removeAttribute('src')
  video.value.load()
  isStreaming.value = false
}

const restartStream = () => {
  stopStream()
  setTimeout(() => startStream(), 500)
}

onMounted(() => {
  startStream()
})
</script>

<template>
  <div
    class="flex h-[75vh] rounded-xl shadow-md border border-gray-300 overflow-hidden bg-gray-50 p-4 gap-4"
  >
    <!-- Видео 75% -->
    <div class="w-3/4 bg-black rounded-lg flex items-center justify-center">
      <video
        ref="video"
        class="w-full h-full object-contain rounded-lg"
        controls
        autoplay
        muted
      ></video>
    </div>

    <!-- Правая панель 25% -->
    <div class="w-1/4 flex flex-col">
      <div class="mb-4">
        <h2 class="text-lg font-semibold mb-2">Телеметрия</h2>
        <div class="text-sm text-gray-700 space-y-1">
          <div>Скорость: {{ telemetry.speed }} км/ч</div>
          <div>Температура: {{ telemetry.temp }} °C</div>
          <div>Статус: {{ isStreaming ? 'Активен' : 'Остановлен' }}</div>
        </div>
      </div>

      <div class="mb-4 flex-grow">
        <h3 class="text-md font-semibold mb-2">Температура</h3>
        <div class="h-48">
          <Line :data="chartData" :options="chartOptions" />
        </div>
      </div>

      <div class="space-y-2 mt-auto">
        <button
          @click="stopStream"
          class="w-full px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition"
        >
          Стоп трансляции
        </button>
        <button
          @click="restartStream"
          class="w-full px-4 py-2 border rounded text-cyan-400 border-cyan-400 hover:bg-cyan-50 transition"
        >
          Перезапустить
        </button>
      </div>
    </div>
  </div>
</template>
