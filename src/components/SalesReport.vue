<template>
  <div class="filters">
    <div class="menu">
      <select
        class="select"
        v-model="selectedCategory"
        @change="handleCategoryChange"
      >
        <option
          v-for="(value, index) in categories"
          :value="value"
          :key="'category-' + index"
        >
          {{ value }}
        </option>
      </select>

      <select class="select" v-model="selectedProduct" name="product">
        <option
          v-for="(product, index) in filteredProducts"
          :value="product"
          :key="'product-' + index"
        >
          {{ product.name }}
        </option>
      </select>

      <select class="select" v-model="selectedBrand" name="brand">
        <option
          v-for="(value, index) in brands"
          :value="value"
          :key="'brand-' + index"
        >
          {{ value }}
        </option>
      </select>
    </div>
  </div>

  <Chart :options="chartOptions" :highcharts="hcInstance" />
</template>

<script setup lang="ts">
import { onMounted, ref, watch, computed } from "vue";
import Highcharts from "highcharts";
import { Chart } from "highcharts-vue";

interface Product {
  id: number;
  category: number;
  brand: number;
  name: string;
  sales: number;
}

const salesData = {
  1: [
    // Calça
    { month: "Jan", sales: 101 },
    { month: "Feb", sales: 205 },
    { month: "Mar", sales: 150 },
    { month: "Apr", sales: 300 },
  ],
  2: [
    // Smart TV
    { month: "Jan", sales: 80 },
    { month: "Feb", sales: 120 },
    { month: "Mar", sales: 200 },
    { month: "Apr", sales: 180 },
  ],
  3: [
    // Carrinho
    { month: "Jan", sales: 50 },
    { month: "Feb", sales: 75 },
    { month: "Mar", sales: 100 },
    { month: "Apr", sales: 125 },
  ],
  4: [
    // Boneca
    { month: "Jan", sales: 70 },
    { month: "Feb", sales: 90 },
    { month: "Mar", sales: 110 },
    { month: "Apr", sales: 130 },
  ],
};

const categories = ref(["Vestuário", "Eletrônicos", "Brinquedos"]);
const selectedCategory = ref<string>("");
const selectedProduct = ref<Product | null>(null);
const selectedBrand = ref<string>("");

const filteredProducts = ref<Product[]>([]);

const allProducts = ref<Product[]>([
  { id: 1, category: 0, brand: 0, name: "Calça", sales: 61.41 },
  { id: 2, category: 1, brand: 1, name: "Smart tv", sales: 11.84 },
  { id: 3, category: 2, brand: 2, name: "Carrinho", sales: 10.85 },
  { id: 4, category: 2, brand: 2, name: "Boneca", sales: 10.85 },
]);

const brands = ref(["Nike", "Samsung", "HotWheels"]);

const hcInstance = Highcharts;

const chartOptions = computed(() => {
  let data = [];
  let subtitleText = "Todos os produtos";
  let seriesName = "Vendas";

  if (selectedProduct.value) {
    const productId = selectedProduct.value.id;
    const productSales = salesData[productId as keyof typeof salesData];

    if (productSales) {
      data = productSales.map((item) => item.sales);
      subtitleText = selectedProduct.value.name;
      seriesName = `Vendas - ${selectedProduct.value.name}`;
    }
  }

  return {
    chart: {
      type: "line",
      height: 600,
      width: 800,
    },
    title: {
      text: "Sales Report",
    },
    subtitle: {
      text: subtitleText,
    },
    xAxis: {
      categories: ["Jan", "Feb", "Mar", "Apr"],
    },
    yAxis: {
      title: {
        text: "Vendas (unidades)",
      },
    },
    plotOptions: {
      line: {
        dataLabels: {
          enabled: true,
        },
        enableMouseTracking: true,
      },
    },
    series: [
      {
        name: seriesName,
        data: data,
        color: "#007bff",
      },
    ],
    credits: {
      enabled: false,
    },
  };
});

onMounted(() => {
  selectedCategory.value = categories.value[0];
  handleCategoryChange();
});

const handleCategoryChange = () => {
  const filtered = allProducts.value.filter(
    (product) => categories.value[product.category] === selectedCategory.value
  );

  filteredProducts.value = filtered;

  if (filtered.length > 0) {
    selectedProduct.value = filtered[0];

    if (selectedProduct.value) {
      selectedBrand.value = brands.value[selectedProduct.value.brand];
    }
  } else {
    selectedProduct.value = null;
    selectedBrand.value = "";
  }
};

watch(selectedProduct, (newProduct) => {
  if (newProduct) {
    selectedBrand.value = brands.value[newProduct.brand];
  }
});

watch(selectedCategory, () => {
  handleCategoryChange();
});

watch(selectedBrand, (newBrand) => {
  if (newBrand && selectedCategory.value) {
    const brandIndex = brands.value.indexOf(newBrand);
    const filtered = allProducts.value.filter(
      (product) =>
        categories.value[product.category] === selectedCategory.value &&
        product.brand === brandIndex
    );

    if (filtered.length > 0) {
      filteredProducts.value = filtered;
      selectedProduct.value = filtered[0];
    }
  }
});
</script>

<style>
.navbar {
  display: flex;
  align-items: start;
  justify-content: space-around;
  color: #fff;
  font-size: 20px;
}
.navbar,
.menu,
.filters {
  background-color: rgb(44, 175, 254);
}

.menu {
  width: 100%;
  display: flex;
  align-items: start;
  justify-content: space-around;
}

.filters {
  padding-top: 20px;
  padding-bottom: 20px;
}

.select {
  font-size: 20px;
  padding: 8px;
  margin: 0 10px;
  border-radius: 4px;
  border: 1px solid #ddd;
}

.chart-container {
  margin: 20px;
  padding: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
</style>
