<template>
  <div class="card">
    <DataTable v-model:filters="filters" :value="groupedTransactions" paginator :rows="10" dataKey="id" filterDisplay="row" :loading="loading"
      :globalFilterFields="['name', 'date', 'value', 'type']" :rowHover="true">
      <template #header>
        <div class="flex justify-content-end flex-column sm:flex-row">
            <span class="p-input-icon-left mb-2">
                <i class="pi pi-search" />
                <InputText v-model="filters['global'].value" placeholder="Pesquisar" style="width: 100%" />
            </span>
        </div>
      </template>
      <template #empty> No transactions found. </template>
      <template #loading> Loading transaction data. Please wait. </template>
      <Column field="name" header="Name" style="min-width: 12rem">
        <template #body="{ data }">
          {{ data[0].name }}
        </template>
      </Column>
    </DataTable>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import { FilterMatchMode } from 'primevue/api';
import axios from 'axios';

const transactions = ref([]);
const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS }
});
const loading = ref(true);

onMounted(() => {
  axios.get("https://run.mocky.io/v3/d4a79840-93c0-4297-80bb-108c279377a3")
    .then((res) => {
      transactions.value = transformTransactions(res.data.data.clients_summary);
      loading.value = false;
    })
    .catch((error) => {
      console.log(error);
      loading.value = false;
    });
});

const transformTransactions = (clients) => {
  let transactions = [];
  clients.forEach(client => {
    if (client.latest_transactions && client.latest_transactions.length > 0) {
      client.latest_transactions.forEach(transaction => {
        transactions.push({
          id: transaction.id,
          name: client.name,
          date: new Date(transaction.date),
          value: transaction.value,
          type: transaction.type
        });
      });
    }
  });
  return transactions;
};

const groupedTransactions = computed(() => {
  const grouped = {};
  transactions.value.forEach(transaction => {
    if (!grouped[transaction.name]) {
      grouped[transaction.name] = [];
    }
    grouped[transaction.name].push(transaction);
  });
  return Object.values(grouped);
});
</script>
