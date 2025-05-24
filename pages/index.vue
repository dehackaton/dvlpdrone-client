<template>
  <div class="min-h-screen bg-white text-black p-4 space-y-4">
    <!-- Header -->
    <header class="flex justify-between items-center">
      <h1 class="text-2xl font-bold">Drone Tracker</h1>
      <nav class="flex space-x-4 text-gray-600">
        <button>Карта</button>
        <button>Видеострим</button>
        <button>Профиль</button>
      </nav>
    </header>

    <!-- Tabs -->
    <div class="flex space-x-2">
      <button class="bg-black text-white px-4 py-2 rounded-xl">Карта</button>
      <button class="bg-gray-200 text-black px-4 py-2 rounded-xl">Видеострим</button>
    </div>

    <!-- Main Content -->
    <main class="flex gap-4">
      <!-- Left Column: Search and Map -->
      <div class="flex-1 space-y-4">
        <!-- Search -->
        <div class="flex items-center bg-gray-100 p-2 rounded-xl">
          <span class="text-gray-500 mr-2">🔍</span>
          <input
            type="text"
            placeholder="Введите локацию"
            class="bg-transparent outline-none w-full"
          />
        </div>

        <!-- Map Container -->
        <div class="rounded-xl overflow-hidden shadow-md border h-[600px]">
          <ClientOnly>
            <div 
              id="map-container" 
              class="h-full w-full leaflet-container"
            ></div>
            <template #fallback>
              <div class="h-full flex items-center justify-center text-gray-500">
                Загрузка карты...
              </div>
            </template>
          </ClientOnly>
        </div>
      </div>

      <!-- Right Column: Info Panel -->
      <div class="w-[350px]">
        <div class="bg-white shadow-md rounded-xl border p-4 space-y-4">
          <div class="bg-cyan-400 rounded-xl p-4 text-white font-semibold">
            <div class="flex justify-between items-center">
              <div>
                <div>DJI Mavic Pro</div>
                <div class="text-sm">FHD high-Framerate Live Feed</div>
              </div>
              <img src="assets/1b20bbbb201d0ab7eeefd9b637c4d905dd8df894.png" alt="Drone" class="h-10">
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
{
  "latitude": 51.1605,
  "longitude": 71.4704,
  "altitude": 850,
  "name": "Астана",
  "address": "ул. Абая 58, Астана, Казахстан",
  "timezone": "Asia/Almaty",
  "point_type": "место отдыха",
  "country_code": "KZ"
}
            </pre>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { onMounted } from 'vue';

export default {
  setup() {
    onMounted(async () => {
      if (typeof window !== 'undefined') {
        // Динамический импорт Leaflet
        const L = await import('leaflet');
        await import('leaflet/dist/leaflet.css');
        
        // Фикс для иконок маркеров
        delete L.Icon.Default.prototype._getIconUrl;
        L.Icon.Default.mergeOptions({
          iconRetinaUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon-2x.png',
          iconUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon.png',
          shadowUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-shadow.png',
        });

        // Координаты центра карты (Астана)
        const center = [51.1605, 71.4704];
        
        // Инициализация карты
        const map = L.map('map-container').setView(center, 13);
        
        // Добавление слоя с тайлами
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
          attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
          maxZoom: 19,
        }).addTo(map);

        // Добавление маркера
        L.marker(center)
          .addTo(map)
          .bindPopup('Дрон здесь');

        // Обновление размеров карты после загрузки
        setTimeout(() => {
          map.invalidateSize();
        }, 100);
      }
    });
  }
}
</script>

<style>
/* Импорт CSS Leaflet через CDN */
@import url('https://unpkg.com/leaflet@1.7.1/dist/leaflet.css');

/* Стили для контейнера карты */
#map-container {
  height: 600px;
  width: 100%;
  z-index: 0;
}

/* Фикс для фона карты */
.leaflet-container {
  background-color: #e8f4f8 !important;
}

/* Адаптация под мобильные устройства */
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
