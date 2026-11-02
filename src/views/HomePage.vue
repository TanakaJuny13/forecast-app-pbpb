<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonRefresher,
  IonRefresherContent,
  IonList,
  IonListHeader,
  IonItem,
  IonLabel,
  IonIcon,
  IonSpinner,
} from '@ionic/vue';
import { timeOutline } from 'ionicons/icons';

interface HourlyRow {
  time: string;
  temperature_2m: number;
}

interface HourlyResponse {
  hourly: {
    time: string[];
    temperature_2m: number[];
  };
}

const API_URL = 'https://api.open-meteo.com/v1/forecast';

const rows = ref<HourlyRow[]>([]);
const loading = ref(false);
const error = ref('');

function toRows(data: HourlyResponse): HourlyRow[] {
  const { time, temperature_2m } = data.hourly;
  return time.map((t, i) => ({
    time: t.replace('T', ' '),
    temperature_2m: temperature_2m[i],
  }));
}

async function loadForecast() {
  loading.value = true;
  error.value = '';
  try {
    const response = await axios.get<HourlyResponse>(API_URL, {
      params: {
        latitude: -6.2,
        longitude: 106.8,
        hourly: 'temperature_2m',
        timezone: 'auto',
      },
    });

    rows.value = toRows(response.data);
    
  } catch (e) {
    console.error(e);
    if (axios.isAxiosError(e)) {
      error.value = e.response
        ? `Server error (${e.response.status}). Please try again later.`
        : 'Network error. Check your connection and try again.';
    } else {
      error.value = 'Could not read the forecast data.';
    }
  } finally {
    loading.value = false;
  }
}

async function onRefresh(event: CustomEvent) {
  await loadForecast();
  (event.target as HTMLIonRefresherElement).complete();
}

onMounted(loadForecast);
</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Forecast Jakarta Weather</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-refresher slot="fixed" @ionRefresh="onRefresh">
        <ion-refresher-content />
      </ion-refresher>

      <div v-if="loading && rows.length === 0" class="state">
        <ion-spinner name="crescent" />
        <p>Loading data...</p>
      </div>

      <div v-else-if="error" class="state">
        <p class="error">{{ error }}</p>
      </div>

      <ion-list v-else :inset="true" lines="full">
        <ion-list-header>
          <ion-label class="col-head">
            <span>time</span>
            <span>temperature_2m (°C)</span>
          </ion-label>
        </ion-list-header>

        <ion-item v-for="row in rows" :key="row.time">
          <ion-icon :icon="timeOutline" slot="start" class="time-icon" />
          <ion-label>{{ row.time }}</ion-label>
          <span slot="end" class="temp">{{ row.temperature_2m.toFixed(1) }} °C</span>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<style scoped>
.state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  height: 60%;
  padding: 0 16px;
  text-align: center;
  color: var(--ion-color-medium);
}

.error {
  color: var(--ion-color-danger);
}

.col-head {
  display: flex;
  justify-content: space-between;
  font-weight: 600;
}

.time-icon {
  font-size: 22px;
  color: var(--ion-color-medium);
}

.temp {
  font-size: 17px;
  font-weight: 600;
  color: var(--ion-color-primary);
}
</style>
