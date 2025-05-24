<template>
    <div class="flex h-[75vh] bg-gray-50 rounded shadow p-4 gap-4">
      <!-- Видео: 75% -->
      <div class="w-3/4 bg-black flex items-center justify-center rounded">
        <video
          ref="video"
          class="w-full h-full object-contain"
          controls
          autoplay
          muted
        ></video>
      </div>

      <!-- Правая панель: 25% -->
      <div class="w-1/4 border border-gray-300 rounded p-4 flex flex-col overflow-y-auto">
        <!-- Телеметрия -->
        <div class="mb-4">
          <h2 class="text-lg font-semibold mb-2">Телеметрия</h2>
          <div class="text-sm text-gray-700 space-y-1">
            <div>Скорость: {{ telemetry.speed }} км/ч</div>
            <div>Температура: {{ telemetry.temp }} °C</div>
            <div>Статус: {{ isStreaming ? 'Активен' : 'Остановлен' }}</div>
          </div>
        </div>

        <!-- График -->
        <div class="mb-4">
          <h3 class="text-md font-semibold mb-2">Температура</h3>
          <LineChart :data="chartData" :options="chartOptions" />
        </div>

        <!-- Кнопки -->
        <div class="mt-auto space-y-2">
          <button
            @click="stopStream"
            class="w-full px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition"
          >
            Стоп трансляции
          </button>
          <button
            @click="restartStream"
            class="w-full px-4 py-2 border rounded text-primary border-primary hover:bg-primary/10 transition"
          >
            Перезапустить
          </button>
        </div>
      </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Hls from 'hls.js'
import { Line } from 'vue-chartjs'
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

// Регистрация компонентов Chart.js
ChartJS.register(Title, Tooltip, Legend, LineElement, CategoryScale, LinearScale, PointElement)

// Компонент графика
const LineChart = defineComponent({
  props: ['data', 'options'],
  components: { Line },
  template: `<Line :data="data" :options="options" />`,
})

const video = ref(null)
let hlsInstance = null
const streamUrl = 'https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8'

const isStreaming = ref(true)
const telemetry = ref({
  speed: 65,
  temp: 42,
})

// Имитация графика температуры
const chartData = ref({
  labels: ['0с', '5с', '10с', '15с', '20с'],
  datasets: [
    {
      label: 'Температура °C',
      data: [40, 42, 43, 42, 44],
      borderColor: '#3B82F6',
      backgroundColor: 'rgba(59,130,246,0.1)',
      tension: 0.4,
      fill: true,
    },
  ],
})

const chartOptions = {
  responsive: true,
  scales: {
    y: {
      beginAtZero: true,
    },
  },
}

// Поток
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
