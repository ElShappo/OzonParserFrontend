<template>
  <div class="q-pa-md q-pt-xl full-width">
    <TableComponent
      :rows="rows"
      :columns="columns"
      v-if="showTable"
    ></TableComponent>

    <TableSkeletonComponent v-else></TableSkeletonComponent>
  </div>

  <ExportExcelComponent
    :rows="rows"
    :columns="columns"
    v-if="showTable"
  ></ExportExcelComponent>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useQuasar } from "quasar";
import TableComponent from "./TableComponent.vue";
import TableSkeletonComponent from "./TableSkeletonComponent.vue";
import ExportExcelComponent from "./ExportExcelComponent.vue";

const props = defineProps([
  "productArticleNumbers",
  "productNames",
  "productNewMinPrices",
  "productNewMaxPrices",
]);

const $q = useQuasar();
const showTable = ref(false);

const columns = ref([
  {
    name: "article number",
    required: true,
    label: "Article number",
    align: "left",
    field: "article number",
    sortable: true,
  },
  {
    name: "name",
    required: true,
    label: "item name",
    align: "left",
    field: (row) => row.name,
    format: (val) => `${val}`,
    sortable: true,
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
    name: "new min price pl",
    align: "center",
    label: "new min price pl",
    field: "new min price pl",
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
  {
    name: "new max price pl",
    align: "center",
    label: "new max price pl",
    field: "new max price pl",
    sortable: true,
  },
  {
    name: "default price",
    align: "center",
    label: "default price",
    field: "default price",
    sortable: true,
  },
  {
    name: "default price pn",
    align: "center",
    label: "default price pn",
    field: "default price pn",
    sortable: true,
  },
  {
    name: "default price pl",
    align: "center",
    label: "default price pl",
    field: "default price pl",
    sortable: true,
  },
]);

const rows = ref([]);

onMounted(() => {
  $q.loading.show({
    message: "Table is being loaded, please wait...",
    boxClass: "text-h5 text-weight-light",
  });

  // const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

  Promise.all(
    props.productNames.map(async (productName, index) => {
      return findProductsWithMinMaxPrice(productName).then(
        ([productWithMinPrice, productWithMaxPrice, productDefault]) => {
          let [
            cheapestName,
            priceOfCheapestWithSale,
            priceOfCheapestWithoutSale,
            linkToCheapest,
          ] = productWithMinPrice;

          let [
            mostExpensiveName,
            priceOfMostExpensiveWithSale,
            priceOfMostExpensiveWithoutSale,
            linkToMostExpensive,
          ] = productWithMaxPrice;

          let [
            defaultName,
            priceOfDefaultWithSale,
            priceOfDefaultWithoutSale,
            linkToDefault,
          ] = productDefault;

          rows.value.push({
            "article number": String(props.productArticleNumbers[index]),
            name: props.productNames[index],
            "old min price": props.productNewMinPrices[index], // new prices now become old
            "old max price": props.productNewMaxPrices[index], // new prices now become old

            "new min price": priceOfCheapestWithSale,
            "new min price pn": cheapestName, // 'pn' stands for product name
            "new min price pl": linkToCheapest, // 'pl' stands for product link

            "new max price": priceOfMostExpensiveWithSale,
            "new max price pn": mostExpensiveName, // 'pn' stands for product name
            "new max price pl": linkToMostExpensive, // 'pl' stands for product link

            "default price": priceOfDefaultWithSale,
            "default price pn": defaultName, // 'pn' stands for product name
            "default price pl": linkToDefault, // 'pl' stands for product link
          });
        }
      );
    })
  ).then(() => {
    $q.loading.hide();
    showTable.value = true;
  });
});

function fullDomain(partialPath) {
  console.log(partialPath);
  let improvedPartialPath = partialPath.match(/product\/.+/);
  let marketplaceDomain = "https://www.ozon.ru/";
  return marketplaceDomain + improvedPartialPath;
}

async function findProductWithMinOrMaxPriceWb(productName, mode = "default") {
  // wb stands for wildberries
  const sortMethods = ["priceup", "pricedown"];

  let url = new URL("https://search.wb.ru/exactmatch/ru/common/v4/search");

  // url.searchParams.set("TestGroup", "no_test");
  // url.searchParams.set("TestID", "no_test");
  // url.searchParams.set("appType", "1");
  url.searchParams.set("curr", "rub");
  url.searchParams.set("dest", "-1257786");
  url.searchParams.set("query", productName);
  url.searchParams.set(
    "regions",
    "80,38,83,4,64,33,68,70,30,40,86,75,69,22,1,31,66,110,48,71,114"
  );
  url.searchParams.set("resultset", "catalog");
  url.searchParams.set("spp", "0");
  url.searchParams.set("suppressSpellcheck", "false");
  url.searchParams.set("uclusters", "0");

  // let query;
  mode === "min"
    ? url.searchParams.set("sort", sortMethods[0])
    : url.searchParams.set("sort", sortMethods[1]) && (mode = "max");

  return fetch(url)
    .then((response) => response.json())
    .then((json) => {
      let priceWithSale = json.data.products[0].salePriceU;
      let priceWithoutSale = json.data.products[0].priceU;
      let name = json.data.products[0].name;

      return [name, priceWithSale, priceWithoutSale];
    })
    .catch((e) => {
      console.error(
        `Error occurred on product = ${productName}, mode = ${mode} `
      );
      console.error(e);
      console.error(url.href);
    });
}

async function findProductWithMinOrMaxPriceOzon(productName, mode = "default") {
  const sortMethods = ["ozon_card_price", "price_desc", ""];

  let url = new URL("https://www.ozon.ru/search");

  url.searchParams.set("from_global", "true");
  url.searchParams.set("text", `${productName}`);

  switch (mode) {
    case "min":
      url.searchParams.set("sorting", sortMethods[0]);
      break;
    case "max":
      url.searchParams.set("sorting", sortMethods[1]);
      break;
    default: // i.e. sort by popularity
      mode = "default";
      break;
  }

  return fetch(url)
    .then((response) => response.text())
    .catch((e) => {
      console.error(`Error occurred when trying to fetch ${url}`);
      console.error(e);
    })
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
        console.error(url);
        console.error(html);
        console.error(htmlDocument);
        return [null, null, null, null];
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
    findProductWithMinOrMaxPriceOzon(productName, "min"),
    findProductWithMinOrMaxPriceOzon(productName, "max"),
    findProductWithMinOrMaxPriceOzon(productName, "default"),

    // findProductWithMinOrMaxPriceWb(productName, "min"),
    // findProductWithMinOrMaxPriceWb(productName, "max"),
  ]);
}
</script>
