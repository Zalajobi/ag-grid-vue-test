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
import { ref, onMounted, reactive, watch } from 'vue';
import { AgGridVue } from 'ag-grid-vue3';
import axios from 'axios';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-quartz.css';

interface RowData {
  id: number,
  time: string,
  title: string,
  platform: number,
  brand: string,
  price: number,
  rating_count: number,
  average_rating: number,
  estimated_sales_30d: number,
  fulfillment: string,
  weight: string,
  seller_count: number,
  asin: string,
  sales_rank: number,
  marketplace: string,
  dimensions: string,
  created_at: string,
}

const authToken = `insert your auth token here`

const rowData = ref<RowData[]>([]);
const filterParams = reactive({});
const gridApi:any = ref(null);

const columnDefs = [
  { headerName: 'ID', field: 'id', sortable: true, filter: true },
  { headerName: 'Name', field: 'title', sortable: true, filter: true },
  { headerName: 'Brand', field: 'brand', sortable: true, filter: true },
  { headerName: 'Price', field: 'price', sortable: true, filter: true },
  { headerName: 'ASIN', field: 'asin', sortable: true, filter: true },
  { headerName: 'Sales', field: 'estimated_sales_30d', sortable: true, filter: true },
  { headerName: 'Sales Rank', field: 'sales_rank', sortable: true, filter: true },
  { headerName: 'Sellers', field: 'seller_count', sortable: true, filter: true },
  { headerName: 'Product Ratings', field: 'rating_count', sortable: true, filter: true },
  { headerName: 'Product Average Ratings', field: 'average_rating', sortable: true, filter: true },
  { headerName: 'Weight', field: 'weight', sortable: true, filter: true },
  { headerName: 'Time', field: 'time', sortable: true, filter: true },
];

const axiosConfig = {
  headers: {
    Authorization: `Bearer ${authToken}`,
    // Add other headers if needed
  },
};

const fetchData = async (type: 'LOAD' | 'FILTER') => {
  const url = 'http://localhost:55321/functions/v1/rhino-api/product/ag-grid/ssrm-filter/amazon';
  if (type === 'LOAD') {
    const response = await axios.post(url, {}, axiosConfig);
    rowData.value = response.data.data;
  } else if (type === 'FILTER') {
    const response = await axios.post(url, filterParams, axiosConfig);
    rowData.value = response.data.data;
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
