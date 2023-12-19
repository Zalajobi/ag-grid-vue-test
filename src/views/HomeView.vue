<template>
  <div class="ag-theme-quartz-dark" style="height: 500px;">
    <ag-grid-vue
        :columnDefs="columnDefs"
        :rowData="rowData"
        domLayout="autoHeight"
        @grid-ready="onGridReady"
        @filter-changed="onFilterChanged"
        :pagination="true"
    />
  </div>
</template>

<script setup lang="ts">
import {ref, onMounted, reactive, watch, onBeforeMount} from 'vue';
import { AgGridVue } from 'ag-grid-vue3';
import axios from 'axios';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-quartz.css';

interface RowData {
  // id: number,
  // time: Date,
  title: string,
  // platform: number,
  // brand: string,
  price: number,
  // rating_count: number,
  // average_rating: number,
  // estimated_sales_30d: number,
  // fulfillment: string,
  // weight: string,
  // seller_count: number,
  // asin: string,
  // sales_rank: number,
  // marketplace: string,
  // dimensions: string,
  // created_at: string,
  // ratings_increase_30d: number
}

const paginationPageSize = ref();
const cacheBlockSize = ref();
const rowModelType = ref('');

const startRow = ref(0);
const endRow = ref(10);
onBeforeMount(() => {
  rowModelType.value = 'serverSide';
  paginationPageSize.value = 5;
  cacheBlockSize.value = 5;
});

const authToken = `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImV4cCI6MTk4MzgxMjk5Nn0.EGIM96RAZx35lJzdJsyH-qQwv8Hdp7fsn3W0YpN81IU`

const rowData = ref<RowData[]>([]);
const filterParams = reactive({});
const gridApi:any = ref(null);

const columnDefs = [
  // { headerName: 'ID', field: 'id', sortable: true, filter: true },
  { headerName: 'Name', field: 'title', sortable: true, filter: true },
  // { headerName: 'Brand', field: 'brand', sortable: true, filter: true },
  { headerName: 'Price', field: 'price', sortable: true, filter: true },
  // { headerName: 'ASIN', field: 'asin', sortable: true, filter: true },
  // { headerName: 'Sales', field: 'estimated_sales_30d', sortable: true, filter: true },
  // { headerName: 'Sales Rank', field: 'sales_rank', sortable: true, filter: true },
  // { headerName: 'Sellers', field: 'seller_count', sortable: true, filter: true },
  // { headerName: 'Product Ratings', field: 'ratings_increase_30d', sortable: true, filter: true },
  // { headerName: 'Product Average Ratings', field: 'average_rating', sortable: true, filter: true },
  // { headerName: 'Weight', field: 'weight', sortable: true, filter: true },
  // { headerName: 'Time', field: 'time', sortable: true, filter: 'agDateColumnFilter' },
];

const axiosConfig = {
  headers: {
    Authorization: `Bearer ${authToken}`,
    // Add other headers if needed
  },
};

const fetchData = async (type: 'LOAD' | 'FILTER') => {
  const url = 'http://localhost:55321/functions/v1/rhino-api/v2/product-research/walmart';
  // const url = 'http://localhost:55321/functions/v1/rhino-api/v2/product-research/amazon';
  if (type === 'LOAD') {
    const response = await axios.post(url, {}, axiosConfig);
    console.log({
      dataInfo: response?.data
    })
    rowData.value = response?.data?.productData
  } else if (type === 'FILTER') {
    const response = await axios.post(url, filterParams, axiosConfig);
    rowData.value = response?.data?.productData
    console.log({
      dataInfo: response?.data?.productData
    })
  }
};

onMounted(() => {
  fetchData('LOAD');
});

watch(filterParams, () => {
  fetchData('FILTER');
}, { deep: true });

const onGridReady = (params:any) => {
  params.api.sizeColumnsToFit();
  gridApi.value = params.api;
};

const onFilterChanged = () => {
  const filterModel = gridApi.value.getFilterModel();
  console.log({filterModel});
  Object.assign(filterParams, filterModel);
};
</script>
