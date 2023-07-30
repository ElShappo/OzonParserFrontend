<template>
  <h4 class="text-center full-width text-weight-light">
    Choose the item from the list:
  </h4>
  <div class="q-pa-md">
    <q-table
      title="OZON marketplace"
      :rows="rows"
      :columns="columns"
      row-key="name"
      v-if="show"
    />
  </div>
  <!-- <q-select
    filled
    v-model="model"
    use-input
    input-debounce="0"
    label="Item"
    :options="productNamesRef"
    @filter="filterFn"
    @update:model-value="handleUpdate"
    style="min-width: 25vw"
    class="dropdown q-pb-xl"
  >
    <template v-slot:no-option>
      <q-item>
        <q-item-section class="text-grey"> No results </q-item-section>
      </q-item>
    </template>
  </q-select>

  <q-list bordered separator v-if="show">
    <q-item :href="linkToCheapest" target="_blank">
      <q-item-section side class="col-2">cheapest</q-item-section>
      <q-item-section>
        <q-item-label overline>{{ cheapestName }}</q-item-label>
        <q-item-label>With sale: {{ priceOfCheapestWithSale }}</q-item-label>
        <q-item-label>No sale: {{ priceOfCheapestWithoutSale }}</q-item-label>
      </q-item-section>
    </q-item>
    <q-item :href="linkToMostExpensive" target="_blank">
      <q-item-section side class="col-2">most expensive</q-item-section>
      <q-item-section>
        <q-item-label overline>{{ mostExpensiveName }}</q-item-label>
        <q-item-label
          >With sale: {{ priceOfMostExpensiveWithSale }}</q-item-label
        >
        <q-item-label
          >No sale: {{ priceOfMostExpensiveWithoutSale }}</q-item-label
        >
      </q-item-section>
    </q-item>
  </q-list> -->
</template>

<script setup>
import { ref, defineProps, onMounted } from "vue";
const props = defineProps([
  "productNames",
  "productNewMinPrices",
  "productNewMaxPrices",
]);

const model = ref(null);
const show = ref(false);

const productOldMinPricesRef = ref(props.productNewMinPrices); // new prices now become old
const productOldMaxPricesRef = ref(props.productNewMaxPrices); // new prices now become old

const productNewMinPricesRef = ref([]);
const productNewMaxPricesRef = ref([]);

// const cheapestName = ref("");
// const priceOfCheapestWithSale = ref("");
// const priceOfCheapestWithoutSale = ref("");
// const linkToCheapest = ref("");

// const mostExpensiveName = ref("");
// const priceOfMostExpensiveWithSale = ref("");
// const priceOfMostExpensiveWithoutSale = ref("");
// const linkToMostExpensive = ref("");

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

const productNamesRef = ref(props.productNames);
console.log(productNamesRef);

onMounted(() => {
  let i = 0;
  for (let productName of props.productNames) {
    findProductsWithMinMaxPrice(productName).then(
      ([productWithMinPrice, productWithMaxPrice]) => {
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

        // productNewMinPricesRef.value.push(priceOfCheapestWithSale.value);
        // productNewMaxPricesRef.value.push(priceOfMostExpensiveWithSale.value);

        rows.value.push({
          name: productName,
          "old min price": productOldMinPricesRef.value[i],
          // "old min price pn": productOldMinPricesRef.value[i],

          "old max price": productOldMaxPricesRef.value[i],
          // "old max price pn": productOldMaxPricesRef.value[i],

          "new min price": priceOfCheapestWithSale,
          "new min price pn": cheapestName,

          "new max price": priceOfMostExpensiveWithSale,
          "new max price pn": mostExpensiveName,
        });
        ++i;
      }
    );
  }
  show.value = true;
});

function filterFn(val, update) {
  if (val === "") {
    update(() => {
      productNamesRef.value = props.productNames;

      // here you have access to "ref" which
      // is the Vue reference of the QSelect
    });
    return;
  }

  update(() => {
    const needle = val.toLowerCase();
    productNamesRef.value = props.productNames.filter(
      (v) => v.toLowerCase().indexOf(needle) > -1
    );
  });
}

function fullDomain(partialPath) {
  console.log(partialPath);
  let improvedPartialPath = partialPath.match(/product\/.+/);
  let marketplaceDomain = "https://www.ozon.ru/";
  return marketplaceDomain + improvedPartialPath;
}

async function findProductWithMinPrice(productName) {
  let queryAscPrice = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[0]}&text=${productName}`;

  return fetch(queryAscPrice)
    .then((response) => response.text())
    .then((html) => {
      const parser = new DOMParser();
      const htmlDocument = parser.parseFromString(html, "text/html");
      const cheapest = htmlDocument.documentElement.querySelector(
        ".widget-search-result-container > div > div"
      );

      let pricesSection = cheapest.querySelector(":scope > div > div");
      let link = cheapest.querySelector(":scope > div > a");
      let name = link.querySelector(":scope > div > span");

      let partialPath = link.href;
      let linkToCheapest = fullDomain(partialPath);

      let prices = pricesSection.querySelector(":scope > div");
      let priceOfCheapestWithSale =
        prices.querySelectorAll(":scope > span")[0].innerHTML;

      let priceOfCheapestWithoutSale = null;
      try {
        // sometimes there is no sale at all, if that happens, we trigger a warning
        priceOfCheapestWithoutSale =
          prices.querySelectorAll(":scope > span")[1].innerHTML;
      } catch (error) {
        console.warn(
          `Could not get a price without sale (cheapest) for ${productName}`
        );
      }

      let cheapestName = name.innerHTML;
      return [
        cheapestName,
        priceOfCheapestWithSale,
        priceOfCheapestWithoutSale,
        linkToCheapest,
      ];
    });
}

async function findProductWithMaxPrice(productName) {
  let queryDescPrice = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[1]}&text=${productName}`;
  return fetch(queryDescPrice)
    .then((response) => response.text())
    .then((html) => {
      console.log(html);
      const parser = new DOMParser();
      const htmlDocument = parser.parseFromString(html, "text/html");
      const mostExpensive = htmlDocument.documentElement.querySelector(
        ".widget-search-result-container > div > div"
      );

      let pricesSection = mostExpensive.querySelector(":scope > div > div");
      let link = mostExpensive.querySelector(":scope > div > a");
      let name = link.querySelector(":scope > div > span");

      let partialPath = link.href;
      let linkToMostExpensive = fullDomain(partialPath);

      let prices = pricesSection.querySelector(":scope > div");
      let priceOfMostExpensiveWithSale =
        prices.querySelectorAll(":scope > span")[0].innerHTML;

      let priceOfMostExpensiveWithoutSale = null;
      try {
        // sometimes there is no sale at all, if that happens, we trigger a warning
        priceOfMostExpensiveWithoutSale =
          prices.querySelectorAll(":scope > span")[1].innerHTML;
      } catch (error) {
        console.warn(
          `Could not get a price without sale (most expensive) for ${productName}`
        );
      }

      let mostExpensiveName = name.innerHTML;
      return [
        mostExpensiveName,
        priceOfMostExpensiveWithSale,
        priceOfMostExpensiveWithoutSale,
        linkToMostExpensive,
      ];
    });
}

async function findProductsWithMinMaxPrice(productName) {
  return Promise.all([
    findProductWithMinPrice(productName),
    findProductWithMaxPrice(productName),
  ]);
}

// function handleUpdate(value, i) {
//   let queryAscPrice = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[0]}&text=${value}`;
//   let queryDescPrice = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[1]}&text=${value}`;

//   let dataAscPrice = fetch(queryAscPrice)
//     .then((response) => response.text())
//     .then((html) => {
//       const parser = new DOMParser();
//       const htmlDocument = parser.parseFromString(html, "text/html");
//       const cheapest = htmlDocument.documentElement.querySelector(
//         ".widget-search-result-container > div > div"
//       );

//       let pricesSection = cheapest.querySelector(":scope > div > div");
//       let link = cheapest.querySelector(":scope > div > a");
//       let name = link.querySelector(":scope > div > span");

//       let partialPath = link.href;
//       linkToCheapest.value = fullDomain(partialPath);

//       let prices = pricesSection.querySelector(":scope > div");
//       priceOfCheapestWithSale.value =
//         prices.querySelectorAll(":scope > span")[0].innerHTML;
//       priceOfCheapestWithoutSale.value =
//         prices.querySelectorAll(":scope > span")[1].innerHTML;

//       cheapestName.value = name.innerHTML;
//     })
//     .then(() => fetch(queryDescPrice))
//     .then((response) => response.text())
//     .then((html) => {
//       console.log(html);
//       const parser = new DOMParser();
//       const htmlDocument = parser.parseFromString(html, "text/html");
//       const mostExpensive = htmlDocument.documentElement.querySelector(
//         ".widget-search-result-container > div > div"
//       );

//       let pricesSection = mostExpensive.querySelector(":scope > div > div");
//       let link = mostExpensive.querySelector(":scope > div > a");
//       let name = link.querySelector(":scope > div > span");

//       let partialPath = link.href;
//       linkToMostExpensive.value = fullDomain(partialPath);

//       let prices = pricesSection.querySelector(":scope > div");
//       priceOfMostExpensiveWithSale.value =
//         prices.querySelectorAll(":scope > span")[0].innerHTML;
//       priceOfMostExpensiveWithoutSale.value =
//         prices.querySelectorAll(":scope > span")[1].innerHTML;

//       mostExpensiveName.value = name.innerHTML;

//       productNewMinPricesRef.value.push(priceOfCheapestWithSale.value);
//       productNewMaxPricesRef.value.push(priceOfMostExpensiveWithSale.value);
//       rows.value.push({
//         name: value,
//         "old min price": productOldMinPricesRef.value[i],
//         "old max price": productOldMaxPricesRef.value[i],
//         "new min price": priceOfCheapestWithSale.value,
//         "new max price": priceOfMostExpensiveWithSale.value,
//       });
//       ++i;

//       show.value = true;

//       // console.log("test");
//       // console.log(props.productNames);
//     });
// }
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
</style>
