<template>
  <div class="flex space-x-2">
    <button @click="activeTab = 'map'" :class="[
      'px-4 py-2 rounded-xl',
      activeTab === 'map' ? 'bg-black text-white' : 'bg-gray-200 text-black'
    ]">
      Карта
    </button>
    <button @click="activeTab = 'stream'" :class="[
      'px-4 py-2 rounded-xl',
      activeTab === 'stream' ? 'bg-black text-white' : 'bg-gray-200 text-black'
    ]">
      Видеострим
    </button>
  </div>

  <!-- Search -->
  <div v-if="activeTab === 'map'" class="flex items-center bg-gray-100 p-2 rounded-xl mt-5">
    <span class="text-gray-500 mr-2">🔍</span>
    <input type="text" placeholder="Введите локацию" class="bg-transparent outline-none w-full" />
  </div>

  <div class="flex h-[87vh] rounded-xl shadow-md border border-gray-300 overflow-hidden bg-gray-50 p-4 gap-4 mt-5">

    <!-- Main Content -->
    <main v-if="activeTab === 'map'" class="flex gap-4 w-full">
      <!-- Map Column -->
      <div class="flex-[3] space-y-4">
        <!-- Map Container -->
        <div class="rounded-xl overflow-hidden shadow-md border h-[600px]">
          <ClientOnly>
            <div id="map-container" class="h-full w-full leaflet-container" />
            <template #fallback>
              <div class="h-full flex items-center justify-center text-gray-500">
                Загрузка карты...
              </div>
            </template>
          </ClientOnly>
        </div>
      </div>

      <!-- Info Panel -->
      <div class="flex-[1] min-w-[300px] max-w-[400px]">
        <div class="bg-white shadow-md rounded-xl border p-4 space-y-4">
          <div class="bg-cyan-400 rounded-xl p-4 text-white font-semibold">
            <div class="flex justify-between items-center">
              <div>
                <div>DJI Mavic Pro</div>
                <div class="text-sm">FHD high-Framerate Live Feed</div>
              </div>
              <img src="public/1b20bbbb201d0ab7eeefd9b637c4d905dd8df894.png" alt="Drone" class="h-10" />
            </div>
          </div>

          <div>
            <div class="font-semibold">Местоположение</div>
            <p>ул. Абая 58, Астана, Казахстан</p>
          </div>

          <div>
            <div class="font-semibold">Основные координаты:</div>
            <p>Широта: 51.1605</p>
            <p>Долгота: 71.4704</p>
          </div>

          <div>
            <div class="font-semibold">Геоинформация точки:</div>
            <pre class="bg-gray-100 p-2 rounded text-sm">
        "latitude": 51.1605,
        "longitude": 71.4704,
        "altitude": 850,
      </pre>
          </div>
        </div>
      </div>
    </main>

    <!-- Stream Tab -->

    <stream-monitor v-else />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import streamMonitor from '~/components/stream-monitor.vue'

const activeTab = ref('map')

onMounted(async () => {
  if (typeof window !== 'undefined') {
    const L = await import('leaflet')
    await import('leaflet/dist/leaflet.css')

    const droneIcon = L.icon({
      iconUrl: "./drone.png",
      iconSize: [50, 50],
      iconAnchor: [20, 20],
      popupAnchor: [0, -20],
    })

    delete L.Icon.Default.prototype._getIconUrl
    L.Icon.Default.mergeOptions({
      iconRetinaUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon-2x.png',
      iconUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon.png',
      shadowUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-shadow.png',
    })

    const center = [51.1605, 71.4704]
    const map = L.map('map-container').setView(center, 12)

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors',
      maxZoom: 19,
    }).addTo(map)

    const restrictedZone = L.polygon([
      [51.1300, 71.4200],
      [51.1300, 71.5000],
      [51.1100, 71.5000],
      [51.1100, 71.4200]
    ], {
      color: 'red',
      fillColor: '#f03',
      fillOpacity: 0.4
    }).addTo(map)

    restrictedZone.bindPopup('🚫 Запретная зона для дронов')

    L.marker(center, { icon: droneIcon }).addTo(map).bindPopup('Местоположение дрона')

    setTimeout(() => {
      map.invalidateSize()
    }, 100)
  }
})
</script>

<style>
@import url('https://unpkg.com/leaflet@1.7.1/dist/leaflet.css');

#map-container {
  height: 600px;
  width: 100%;
  z-index: 0;
}

.leaflet-container {
  background-color: #e8f4f8 !important;
}

@media (max-width: 768px) {
  .flex {
    flex-direction: column;
  }

  .w-\[350px\] {
    width: 100%;
    margin-top: 1rem;
  }
}
</style>
