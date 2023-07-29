<template>
  <h4 class="text-center full-width text-weight-light">
    Choose the item from the list:
  </h4>
  <q-select
    filled
    v-model="model"
    use-input
    input-debounce="0"
    label="Item"
    :options="options"
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
  </q-list>
</template>

<script setup>
import { ref, defineProps, onMounted } from "vue";
const props = defineProps(["stringOptions"]);

const model = ref(null);
const show = ref(false);

const cheapestName = ref("");
const priceOfCheapestWithSale = ref("");
const priceOfCheapestWithoutSale = ref("");
const linkToCheapest = ref("");

const mostExpensiveName = ref("");
const priceOfMostExpensiveWithSale = ref("");
const priceOfMostExpensiveWithoutSale = ref("");
const linkToMostExpensive = ref("");

const sortMethods = ["ozon_card_price", "price_desc"];

// const stringOptions = [
//   "Dr. Beckmann Ловушка для цвета и грязи (одноразовая) 24 шт",
//   "Dr. Beckmann Очиститель для стиральных машин гигиенический 250 гр",
//   "Dr. Beckmann Спрей пятновыводитель Дезодорант и пот 250 мл",
//   "Dr.Beckmann Спрей пятновыводитель 250 мл",
//   "Dr. Beckmann Средство для чистки и блеска стеклокерамики спрей 250 мл",
//   "Aqualine Впитывающая губка, 3 шт.",
// ];

const options = ref(props.stringOptions);
console.log(options);

function filterFn(val, update) {
  if (val === "") {
    update(() => {
      options.value = props.stringOptions;

      // here you have access to "ref" which
      // is the Vue reference of the QSelect
    });
    return;
  }

  update(() => {
    const needle = val.toLowerCase();
    options.value = props.stringOptions.filter(
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

function handleUpdate(value) {
  let queryAscPrice = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[0]}&text=${value}`;
  let queryDescPrice = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[1]}&text=${value}`;

  let dataAscPrice = fetch(queryAscPrice)
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
      linkToCheapest.value = fullDomain(partialPath);

      let prices = pricesSection.querySelector(":scope > div");
      priceOfCheapestWithSale.value =
        prices.querySelectorAll(":scope > span")[0].innerHTML;
      priceOfCheapestWithoutSale.value =
        prices.querySelectorAll(":scope > span")[1].innerHTML;

      cheapestName.value = name.innerHTML;
    })
    .then(() => fetch(queryDescPrice))
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
      linkToMostExpensive.value = fullDomain(partialPath);

      let prices = pricesSection.querySelector(":scope > div");
      priceOfMostExpensiveWithSale.value =
        prices.querySelectorAll(":scope > span")[0].innerHTML;
      priceOfMostExpensiveWithoutSale.value =
        prices.querySelectorAll(":scope > span")[1].innerHTML;

      mostExpensiveName.value = name.innerHTML;
      show.value = true;

      console.log("test");
      console.log(props.stringOptions);
    });
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
</style>
