<template>
  <div class="q-pa-md q-pt-xl full-width">
    <TableComponent
      :rows="rows"
      :columns="columns"
      :visibleColumns="visibleColumns"
      v-if="showTable"
    ></TableComponent>

    <TableSkeletonComponent v-else></TableSkeletonComponent>
  </div>

  <ExportExcelComponent
    :rows="rows"
    :columns="columns"
    :visibleColumns="visibleColumns"
    v-if="showTable"
  ></ExportExcelComponent>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import { useQuasar } from "quasar";
import TableComponent from "./TableComponent.vue";
import TableSkeletonComponent from "./TableSkeletonComponent.vue";
import ExportExcelComponent from "./ExportExcelComponent.vue";

const props = defineProps(["productArticleNumbers", "productNames"]);

const $q = useQuasar();
const showTable = ref(false);

const rows = ref([]);

// here we deduce 'columns' from 'rows' ('rows' will be filled later with objects, whose keys will become columns)
const columns = computed(() => {
  if (rows.value.length === 0) {
    return null;
  }
  return Object.keys(rows.value[0]).map((item) => {
    return { name: item, label: item, field: item, align: "center" };
  });
});

// here we deduce which columns we want to show
// typically we decide to show all columns except for those containing 'pl' in their names
// (i.e. we do not store a link to a product as a separate column)
const visibleColumns = computed(() => {
  return columns.value
    .filter((column) => !column.name.includes("Pl"))
    .map((item) => item.name);
});

onMounted(() => {
  $q.loading.show({
    message: "Table is being loaded, please wait...",
    boxClass: "text-h5 text-weight-light",
  });

  // const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

  Promise.all(
    props.productNames.slice(0, 4).map(async (productName, index) => {
      return findProductsWithMinMaxPrice(productName).then(
        ([
          productWithMinPriceOzon,
          productWithMaxPriceOzon,
          productDefaultOzon,

          productWithMinPriceWb,
          productWithMaxPriceWb,
          productDefaultWb,
        ]) => {
          let [
            cheapestNameOzon,
            priceOfCheapestWithSaleOzon,
            priceOfCheapestWithoutSaleOzon,
            linkToCheapestOzon,
          ] = productWithMinPriceOzon;

          let [
            mostExpensiveNameOzon,
            priceOfMostExpensiveWithSaleOzon,
            priceOfMostExpensiveWithoutSaleOzon,
            linkToMostExpensiveOzon,
          ] = productWithMaxPriceOzon;

          let [
            defaultNameOzon,
            priceOfDefaultWithSaleOzon,
            priceOfDefaultWithoutSaleOzon,
            linkToDefaultOzon,
          ] = productDefaultOzon;

          let [
            cheapestNameWb,
            priceOfCheapestWithSaleWb,
            priceOfCheapestWithoutSaleWb,
            linkToCheapestWb,
          ] = productWithMinPriceWb;

          let [
            mostExpensiveNameWb,
            priceOfMostExpensiveWithSaleWb,
            priceOfMostExpensiveWithoutSaleWb,
            linkToMostExpensiveWb,
          ] = productWithMaxPriceWb;

          let [
            defaultNameWb,
            priceOfDefaultWithSaleWb,
            priceOfDefaultWithoutSaleWb,
            linkToDefaultWb,
          ] = productDefaultWb;

          rows.value.push({
            articleNumber: String(props.productArticleNumbers[index]),
            name: props.productNames[index],

            newMinPriceOzon: priceOfCheapestWithSaleOzon,
            newMinPricePnOzon: cheapestNameOzon, // 'pn' stands for product name
            newMinPricePlOzon: linkToCheapestOzon, // 'pl' stands for product link

            newMinPriceWb: priceOfCheapestWithSaleWb,
            newMinPricePnWb: cheapestNameWb, // 'pn' stands for product name
            newMinPricePlWb: linkToCheapestWb, // 'pl' stands for product link

            newMaxPriceOzon: priceOfMostExpensiveWithSaleOzon,
            newMaxPricePnOzon: mostExpensiveNameOzon, // 'pn' stands for product name
            newMaxPricePlOzon: linkToMostExpensiveOzon, // 'pl' stands for product link

            newMaxPriceWb: priceOfMostExpensiveWithSaleWb,
            newMaxPricePnWb: mostExpensiveNameWb, // 'pn' stands for product name
            newMaxPricePlWb: linkToMostExpensiveWb, // 'pl' stands for product link

            defaultPriceOzon: priceOfDefaultWithSaleOzon,
            defaultPricePnOzon: defaultNameOzon, // 'pn' stands for product name
            defaultPricePlOzon: linkToDefaultOzon, // 'pl' stands for product link

            defaultPriceWb: priceOfDefaultWithSaleWb,
            defaultPricePnWb: defaultNameWb, // 'pn' stands for product name
            defaultPricePlWb: linkToDefaultWb, // 'pl' stands for product link
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

// async function findProductWithMinOrMaxPriceWb(productName, mode = "default") {
//   // wb stands for wildberries
//   const sortMethods = ["priceup", "pricedown"];

//   let url = new URL("https://search.wb.ru/exactmatch/ru/common/v4/search");

//   // url.searchParams.set("TestGroup", "no_test");
//   // url.searchParams.set("TestID", "no_test");
//   // url.searchParams.set("appType", "1");
//   url.searchParams.set("curr", "rub");
//   url.searchParams.set("dest", "-1257786");
//   url.searchParams.set("query", productName);
//   url.searchParams.set(
//     "regions",
//     "80,38,83,4,64,33,68,70,30,40,86,75,69,22,1,31,66,110,48,71,114"
//   );
//   url.searchParams.set("resultset", "catalog");
//   url.searchParams.set("spp", "0");
//   url.searchParams.set("suppressSpellcheck", "false");
//   url.searchParams.set("uclusters", "0");

//   // let query;
//   mode === "min"
//     ? url.searchParams.set("sort", sortMethods[0])
//     : url.searchParams.set("sort", sortMethods[1]) && (mode = "max");

//   return fetch(url)
//     .then((response) => response.json())
//     .then((json) => {
//       let priceWithSale = json.data.products[0].salePriceU;
//       let priceWithoutSale = json.data.products[0].priceU;
//       let name = json.data.products[0].name;

//       return [name, priceWithSale, priceWithoutSale];
//     })
//     .catch((e) => {
//       console.error(
//         `Error occurred on product = ${productName}, mode = ${mode} `
//       );
//       console.error(e);
//       console.error(url.href);
//     });
// }

async function findProductWithMinOrMaxPriceWb(productName, mode = "min") {
  const req = await fetch(`http://localhost:5000/wb/${productName}/${mode}`);
  const res = await req.json();
  console.log(res);
  return res;
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

    findProductWithMinOrMaxPriceWb(productName, "min"),
    findProductWithMinOrMaxPriceWb(productName, "max"),
    findProductWithMinOrMaxPriceWb(productName, "default"),
  ]);
}
</script>
