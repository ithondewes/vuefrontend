<template>
  <div class="card">
    <DataTable v-model:filters="filters" :value="equityHistory" paginator :rows="10" dataKey="id" filterDisplay="row" :loading="loading"
      :globalFilterFields="['date_custodia', 'value_custodia']" :rowHover="true">
      <template #header>
        <div class="flex justify-content-end flex-column sm:flex-row">
            <span class="p-input-icon-left mb-2">
                <i class="pi pi-search" />
                <InputText v-model="filters['global'].value" placeholder="Pesquisar" style="width: 100%" />
            </span>
        </div>
      </template>
      <template #empty> No data found. </template>
      <template #loading> Loading data. Please wait. </template>
      <Column field="date" header="Date" style="min-width: 12rem">
        <template #body="{ data }">
          {{ formatDate(data.date_custodia) }}
        </template>
      </Column>
      <Column field="value" header="Value" style="min-width: 12rem">
        <template #body="{ data }">
          {{ formatCurrency(data.value_custodia) }}
        </template>
      </Column>
    </DataTable>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { FilterMatchMode } from 'primevue/api';
import axios from 'axios';

const equityHistory = ref([]);
const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS }
});
const loading = ref(true);

onMounted(() => {
  axios.get("https://run.mocky.io/v3/d4a79840-93c0-4297-80bb-108c279377a3")
    .then((res) => {
      equityHistory.value = res.data.data.advisor_summary.equity_history.map(entry => ({
        date_custodia: new Date(entry.date),
        value_custodia: entry.value
      }));
      loading.value = false;
    })
    .catch((error) => {
      console.log(error);
      loading.value = false;
    });
});

const formatDate = (value) => {
  return value.toLocaleDateString('en-US', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric'
  });
};

const formatCurrency = (value) => {
  return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
};
</script>
