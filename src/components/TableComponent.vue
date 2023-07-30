<template>
  <div class="q-pa-md q-pt-xl full-width">
    <q-table
      title="OZON marketplace"
      :rows="rows"
      :columns="columns"
      row-key="name"
      style="width: 100%; height: 600px"
      class="my-sticky-header-table"
      v-if="showTable"
    >
      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td key="name" :props="props">
            {{ props.row.name }}
          </q-td>
          <q-td key="old min price" :props="props">
            {{ props.row["old min price"] }}
          </q-td>
          <q-td key="old max price" :props="props">
            {{ props.row["old max price"] }}
          </q-td>
          <q-td key="new min price" :props="props">
            {{ props.row["new min price"] }}
          </q-td>
          <q-td key="new min price pn" :props="props">
            <a
              target="_blank"
              :href="
                cheapestProducts.find(
                  (product) => product.name === props.row['new min price pn']
                ).link
              "
              >{{ props.row["new min price pn"] }}</a
            >
          </q-td>
          <q-td key="new max price" :props="props">
            {{ props.row["new max price"] }}
          </q-td>
          <q-td key="new max price pn" :props="props">
            <a
              target="_blank"
              :href="
                mostExpensiveProducts.find(
                  (product) => product.name === props.row['new max price pn']
                ).link
              "
              >{{ props.row["new max price pn"] }}</a
            >
          </q-td>
        </q-tr>
      </template>
    </q-table>
    <TableSkeletonComponent v-else />
  </div>
  <q-btn
    color="primary"
    icon="file_download"
    label="Export table (.xlsx)"
    @click="exportTable"
    v-if="showTable"
  />
</template>

<script setup>
import * as XLSX from "xlsx";
import TableSkeletonComponent from "./TableSkeletonComponent.vue";
import { useQuasar } from "quasar";
import { ref, defineProps, onMounted } from "vue";
const props = defineProps([
  "productNames",
  "productNewMinPrices",
  "productNewMaxPrices",
]);
const $q = useQuasar();

const showTable = ref(false);

// after sending request to a marketplace for each of the names specified in .xlsx table
// you will end up having two lists: list of cheapest and a list of most expensive products
// each product from each of these lists will have the structure as follows: {name, priceWithSale, priceWithoutSale, link}
const cheapestProducts = ref([]);
const mostExpensiveProducts = ref([]);

const sortMethods = ["ozon_card_price", "price_desc"];

const columns = [
  {
    name: "name",
    required: true,
    label: "Item name",
    align: "left",
    field: (row) => row.name,
    format: (val) => `${val}`,
  },
  {
    name: "old min price",
    align: "center",
    label: "old min price",
    field: "old min price",
    sortable: true,
  },
  {
    name: "old max price",
    align: "center",
    label: "old max price",
    field: "old max price",
    sortable: true,
  },
  {
    name: "new min price",
    align: "center",
    label: "new min price",
    field: "new min price",
    sortable: true,
  },
  {
    name: "new min price pn",
    align: "center",
    label: "new min price pn",
    field: "new min price pn",
    sortable: true,
  },
  {
    name: "new max price",
    align: "center",
    label: "new max price",
    field: "new max price",
    sortable: true,
  },
  {
    name: "new max price pn",
    align: "center",
    label: "new max price pn",
    field: "new max price pn",
    sortable: true,
  },
];

const rows = ref([]);

onMounted(() => {
  let i = 0;
  let promises = [];

  const tableLoadingNotify = $q.notify({
    spinner: true,
    message: "Table is loading...",
    timeout: 0,
  });

  for (let productName of props.productNames) {
    promises.push(
      findProductsWithMinMaxPrice(productName).then(
        ([productWithMinPrice, productWithMaxPrice]) => {
          let [
            cheapestName,
            priceOfCheapestWithSale,
            priceOfCheapestWithoutSale,
            linkToCheapest,
          ] = productWithMinPrice;

          cheapestProducts.value.push({
            name: cheapestName,
            priceWithSale: priceOfCheapestWithSale,
            priceWithoutSale: priceOfCheapestWithoutSale,
            link: linkToCheapest,
          });

          let [
            mostExpensiveName,
            priceOfMostExpensiveWithSale,
            priceOfMostExpensiveWithoutSale,
            linkToMostExpensive,
          ] = productWithMaxPrice;

          mostExpensiveProducts.value.push({
            name: mostExpensiveName,
            priceWithSale: priceOfMostExpensiveWithSale,
            priceWithoutSale: priceOfMostExpensiveWithoutSale,
            link: linkToMostExpensive,
          });

          rows.value.push({
            name: productName,
            "old min price": props.productNewMinPrices[i], // new prices now become old
            "old max price": props.productNewMaxPrices[i], // new prices now become old

            "new min price": priceOfCheapestWithSale,
            "new min price pn": cheapestName, // 'pn' stands for product name

            "new max price": priceOfMostExpensiveWithSale,
            "new max price pn": mostExpensiveName, // 'pn' stands for product name
          });
          ++i;
        }
      )
    );
  }
  Promise.all(promises).then(() => {
    tableLoadingNotify();
    showTable.value = true;
  });
});

function fullDomain(partialPath) {
  console.log(partialPath);
  let improvedPartialPath = partialPath.match(/product\/.+/);
  let marketplaceDomain = "https://www.ozon.ru/";
  return marketplaceDomain + improvedPartialPath;
}

async function findProductWithMinOrMaxPrice(productName, mode = "min") {
  let query;
  mode === "min"
    ? (query = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[0]}&text=${productName}`)
    : (query = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[1]}&text=${productName}`) &&
      (mode = "max");

  return fetch(query)
    .then((response) => response.text())
    .then((html) => {
      const parser = new DOMParser();
      const htmlDocument = parser.parseFromString(html, "text/html");
      const extremum = htmlDocument.documentElement.querySelector(
        ".widget-search-result-container > div > div"
      );
      let pricesSection;
      let link;
      try {
        pricesSection = extremum.querySelector(":scope > div > div");
        link = extremum.querySelector(":scope > div > a");
      } catch (error) {
        console.error(
          `Could not get pricesSection & link (${mode}) for ${productName}`
        );
        console.error(productName);
        console.error(query);
        console.error(html);
        console.error(htmlDocument);
      }
      let name = link.querySelector(":scope > div > span");

      let partialPath = link.href;
      let linkToExtremum = fullDomain(partialPath);

      let prices = pricesSection.querySelector(":scope > div");
      let priceOfExtremumWithSale =
        prices.querySelectorAll(":scope > span")[0].innerHTML;

      let priceOfExtremumWithoutSale = null;
      try {
        // sometimes there is no sale at all, if that happens, we trigger a warning
        priceOfExtremumWithoutSale =
          prices.querySelectorAll(":scope > span")[1].innerHTML;
      } catch (error) {
        console.warn(
          `Could not get a price without sale (${mode}) for ${productName}`
        );
      }

      let extremumName = name.innerHTML;
      return [
        extremumName,
        priceOfExtremumWithSale,
        priceOfExtremumWithoutSale,
        linkToExtremum,
      ];
    });
}

async function findProductsWithMinMaxPrice(productName) {
  return Promise.all([
    findProductWithMinOrMaxPrice(productName, "min"),
    findProductWithMinOrMaxPrice(productName, "max"),
  ]);
}

function exportTable() {
  let columnsSlice = columns.map((column) => column.name);
  let rowsSlice = rows.value.map((row) => Object.values(row));

  let aoa = rowsSlice.slice();
  aoa.unshift(columnsSlice);

  console.log("hey!");
  console.warn(aoa);

  const workbook = XLSX.utils.book_new();
  let worksheet = XLSX.utils.aoa_to_sheet(aoa);
  XLSX.utils.book_append_sheet(workbook, worksheet, "Sheet1");
  XLSX.writeFile(workbook, "test1.xlsx");
}
</script>

<style>
.name {
  display: inline-block;
}
.price {
  display: inline-block;
}
.q-item__section.column.q-item__section--side {
  text-align: center;
}

.my-sticky-header-table {
  /* height or max-height is important */
  height: 310px;
}

.q-table__top,
.q-table__bottom,
thead tr:first-child th {
  /* bg color is important for th; just specify one */
  background-color: white;
}

thead tr th {
  position: sticky;
  z-index: 1;
}
thead tr:first-child th {
  top: 0;
}

/* this is when the loading indicator appears */
&.q-table--loading thead tr:last-child th {
  /* height of all previous header rows */
  top: 48px;
}

/* prevent scrolling behind sticky top row on focus */
tbody {
  /* height of all previous header rows */
  scroll-margin-top: 48px;
}
</style>
