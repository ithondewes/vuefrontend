<script setup>
import { onMounted, reactive, ref, watch, onBeforeMount } from 'vue';
import CustomerService from '@/service/CustomerService';
import ProductService from '@/service/ProductService';
import { useLayout } from '@/layout/composables/layout';
import { FilterMatchMode, FilterOperator } from 'primevue/api';
import axios from 'axios';

const { isDarkTheme } = useLayout();

const dataCustodia = ref({
    labels: [],
    datasets: [
        {
            label: 'Valor',
            data: [],
            fill: true,
            tension: 0.4,
            borderColor: '#87CEEB',
            backgroundColor: 'rgba(135, 206, 235, 0.1)' 
        }
    ]
});

const products = ref(null);
const clienteCount = ref(null);
const patrimonioCustodia = ref(null);
const patrimonioMedio = ref(null);
const equityHistory = ref([]);

const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS }
});
const loading = ref(true);

onMounted(() => {
    productService.getProductsSmall().then((data) => (products.value = data));
    chartDataCarteira.value = setChartDataCarteira();
    chartDataPatrimonio.value = setChartDataPatrimonio();
    chartOptions.value = setChartOptions();

    axios.get("https://run.mocky.io/v3/dc63297f-8e6e-4998-8478-ac07bd390024")
    .then((res) => {
        const data = res.data.data.advisor_summary;
        clienteCount.value = data.client_count;
        console.log(clienteCount.value);
        patrimonioCustodia.value = formatCurrency(data.total_equity);
        patrimonioMedio.value = formatCurrency(data.average_equity);
        equityHistory.value = data.equity_history.map(entry => ({
            date_custodia: new Date(entry.date),
            value_custodia: entry.value
        }));
        transactions.value = transformTransactions(res.data.data.clients_summary);
        loading.value = false;

        // Atualizar dataCustodia com os dados obtidos
        dataCustodia.value.labels = equityHistory.value.map(entry => formatDate(entry.date_custodia));
        dataCustodia.value.datasets[0].data = equityHistory.value.map(entry => entry.value_custodia);
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

const items = ref([
    { label: 'Add New', icon: 'pi pi-fw pi-plus' },
    { label: 'Remove', icon: 'pi pi-fw pi-minus' }
]);
const lineOptions = ref(null);
const productService = new ProductService();

const formatCurrency = (value) => {
    return value.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
};

const chartOptionsCustodia = ref({
    plugins: {
        legend: {
            display: true,
            position: 'top'
        }
    },
    maintainAspectRatio: false,
    responsive: true,
    height: 350,
    scales: {
        y: {
            grid: {
                display: true,
                drawBorder: false
            },
            ticks: {
                callback: function(value, index, values) {
                    return value.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
                }
            }
        },
        x: {
            grid: {
                display: false,
                drawBorder: false
            }
        }
    }
});

const chartHeight = '350px';

const chartDataCarteira = ref();
const chartDataPatrimonio = ref();
const chartOptions = ref();

const setChartDataCarteira = () => {
    const documentStyle = getComputedStyle(document.documentElement);

    return {
        labels: ['C.A', 'C.B', 'C.C'],
        datasets: [
            {
                label: 'Valor',
                backgroundColor: documentStyle.getPropertyValue('--cyan-500'),
                borderColor: documentStyle.getPropertyValue('--cyan-500'),
                data: [45, 40, 10]
            }
        ]
    };
};

const setChartDataPatrimonio = () => {
    const documentStyle = getComputedStyle(document.documentElement);

    return {
        labels: ['C.A', 'C.B', 'C.C'],
        datasets: [
            {
                label: 'Valor',
                backgroundColor: documentStyle.getPropertyValue('--cyan-500'),
                borderColor: documentStyle.getPropertyValue('--cyan-500'),
                data: [30, 10, 10]
            }
        ]
    };
};

const setChartOptions = () => {
    const documentStyle = getComputedStyle(document.documentElement);
    const textColor = documentStyle.getPropertyValue('--text-color');
    const textColorSecondary = documentStyle.getPropertyValue('--text-color-secondary');
    const surfaceBorder = documentStyle.getPropertyValue('--surface-border');

    return {
        indexAxis: 'y',
        maintainAspectRatio: false,
        aspectRatio: 0.8,
        plugins: {
            legend: {
                labels: {
                    color: textColor
                }
            }
        },
        scales: {
            x: {
                ticks: {
                    color: textColorSecondary,
                    font: {
                        weight: 500
                    },
                    callback: function(value, index, values) {
                    return value + 'mi';
                    }
                },
                grid: {
                    display: false,
                    drawBorder: false
                }
            },
            y: {
                ticks: {
                    color: textColorSecondary
                },
                grid: {
                    color: surfaceBorder,
                    drawBorder: false
                }
            }
        }
    };
}

const transactions = ref([]);

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

const aporteStyle = {
  backgroundColor: 'var(--highlight-bg)',
  color: 'var(--highlight-text-color)',
  borderRadius: 'var(--border-radius)',
  padding: '0.5rem'
};

const resgateStyle = {
  backgroundColor: '#ffaca7',
  color: '#ffaca7',
  borderRadius: 'var(--border-radius)',
  padding: '0.5rem'
};

</script>

<style scoped>
.chart-container {
    max-height: 100%; 
    overflow-y: auto; /* Adiciona uma barra de rolagem vertical se necessário */
}

::v-deep(.p-datatable-frozen-tbody) {
    font-weight: bold;
}

::v-deep(.p-datatable-scrollable .p-frozen-column) {
    font-weight: bold;
}
</style>

<template>
<div class="grid">
    <div class="col-12 lg:col-6 xl:col-4">
        <div class="card">
            <div class="flex justify-content-between mb-3">
                <div>
                    <span class="block text-900 font-medium mb-3">Clientes</span>
                </div>
            </div>
            <div class="flex justify-content-between flex-column sm:flex-row">
                <span class="text-900">{{ clienteCount }}</span>
                <span style="background-color: var(--highlight-bg); color: var(--highlight-text-color); border-radius: var(--border-radius); padding: 0.25rem;" class="text-green-500 font-medium">+12%</span>
            </div>
        </div>
    </div>
    <div class="col-12 lg:col-6 xl:col-4">
        <div class="card">
            <div class="flex justify-content-between mb-3">
                <div>
                    <span class="block text-900 font-medium mb-3">Patrimônio Sob Custódia</span>
                </div>
            </div>
            <div class="flex justify-content-between flex-column sm:flex-row">
                <span class="text-900">{{ patrimonioCustodia }}</span>
                <span style="background-color: var(--highlight-bg); color: var(--highlight-text-color); border-radius: var(--border-radius); padding: 0.25rem;" class="text-green-500 font-medium">+12%</span>
            </div>
        </div>
    </div>
    <div class="col-12 lg:col-6 xl:col-4">
        <div class="card">
            <div class="flex justify-content-between mb-3">
                <div>
                    <span class="block text-900 font-medium mb-3">Patrimônio Médio</span>
                </div>
            </div>
            <div class="flex justify-content-between flex-column sm:flex-row">
                <span class="text-900">{{ patrimonioMedio }}</span>
                <span style="background-color: #ffaca7; color: var(--highlight-text-color); border-radius: var(--border-radius); padding: 0.25rem;" class="text-500 font-medium">-5%</span>
            </div>
        </div>
    </div>
    <div class="col-12 xl:col-12">
        <div class="card">
            <div class="flex align-items-center justify-content-between mb-4">
                <div class="flex align-items-center">
                    <h5>Evolução de Patrimônio Médio Sob Custódia</h5>
                    <div class="ml-2" style="width: 4rem; height: 2rem">
                        <i class="pi pi-exclamation-circle"></i>
                    </div>
                </div>
            </div>
            <div class="chart-container" style="height: 350px;">
                <Chart type="line" :data="dataCustodia" :options="chartOptionsCustodia" :style="{ height: chartHeight }"/>
            </div>
        </div>
    </div>
    <div class="col-12 xl:col-6">
        <div class="card">
            <div class="flex align-items-center justify-content-between mb-4">
                <div class="flex align-items-center">
                    <h5>Carteiras por Corretora</h5>
                    <div class="ml-2" style="width: 4rem; height: 2rem">
                        <i class="pi pi-exclamation-circle"></i>
                    </div>
                </div>
            </div>
            <Chart type="bar" :data="chartDataCarteira" :options="chartOptions" class="h-20rem"  />
        </div>
    </div>
    <div class="col-12 xl:col-6">
        <div class="card">
            <div class="flex align-items-center justify-content-between mb-4">
                <div class="flex align-items-center">
                    <h5>Patrimônio por Corretora</h5>
                    <div class="ml-2" style="width: 4rem; height: 2rem">
                        <i class="pi pi-exclamation-circle"></i>
                    </div>
                </div>
            </div>
            <Chart type="bar" :data="chartDataPatrimonio" :options="chartOptions" class="h-20rem"  />
        </div>
    </div>
</div>
    <div class="grid">
        <div class="col-12">
            <div class="card">
                <DataTable v-model:filters="filters" :value="transactions" paginator :rows="10" dataKey="id" filterDisplay="row" :loading="loading"
                    :globalFilterFields="['name', 'date', 'value', 'type']" :rowHover="true">
                    <template #header>
                        <div class="flex justify-content-between flex-column sm:flex-row">
                            <div>
                                <h5> Movimentações </h5>
                                <h7 style="color: #808080;"> Lista das últimas movimentações </h7>
                            </div>
                            <span class="p-input-icon-left mb-2">
                                <i class="pi pi-search" />
                                <InputText v-model="filters['global'].value" placeholder="Pesquisar" style="width: 100%" />
                            </span>
                        </div>
                    </template>
                    <template #empty> No transactions found. </template>
                    <template #loading> Loading transaction data. Please wait. </template>
                    <Column field="name" header="Nome" style="min-width: 12rem">
                        <template #body="{ data }">
                        {{ data.name }}
                        </template>
                    </Column>
                    <Column field="date" header="Data" style="min-width: 12rem">
                        <template #body="{ data }">
                        {{ formatDate(data.date) }}
                        </template>
                    </Column>
                    <Column field="value" header="Valor" style="min-width: 12rem">
                        <template #body="{ data }">
                        {{ formatCurrency(data.value) }}
                        </template>
                    </Column>
                    <Column field="type" header="Tipo" style="min-width: 12rem">
                        <template #body="{ data }">
                            <span 
                                :style="data.type.trim().toLowerCase() === 'aporte' ? aporteStyle : resgateStyle"
                                :class="{ 'text-green-500': data.type.trim().toLowerCase() === 'aporte', 'text-500': data.type.trim().toLowerCase() === 'resgate' }"
                            >
                                {{ data.type.trim().toLowerCase() === 'aporte' ? 'Aporte' : 'Resgate' }}
                            </span>
                        </template>
                    </Column>
                </DataTable>
            </div>
        </div>
    </div>
</template>
