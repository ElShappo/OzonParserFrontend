<template>
  <q-page class="column justify-start items-center">
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

    <!-- <div
      class="item full-width row justify-center text-center text-h5"
      v-show="show"
    >
      <div class="full-width q-pb-md">
        <div class="">Name:</div>
        <div class="name col-7"></div>
      </div>

      <div class="full-width">
        <div class="">Price:</div>
        <div class="price col-7"></div>
      </div>
    </div> -->

    <q-list bordered separator v-if="show">
      <q-item clickable v-ripple>
        <q-item-section>
          <q-item-label overline>{{ cheapestName }}</q-item-label>
          <q-item-label>Price with sale: {{ priceWithSale }}</q-item-label>
          <q-item-label>Price with sale: {{ priceWithoutSale }}</q-item-label>
        </q-item-section>
      </q-item>
    </q-list>
  </q-page>
</template>

<script setup>
import { ref } from "vue";

const model = ref(null);
const show = ref(false);
const cheapestName = ref("");
const priceWithSale = ref("");
const priceWithoutSale = ref("");

const sortMethods = ["ozon_card_price", "price_desc"];

const stringOptions = [
  "Aqualine Впитывающая губка, 3 шт.",
  "Google",
  "Facebook",
  "Twitter",
  "Apple",
  "Oracle",
];
const options = ref(stringOptions);

function filterFn(val, update) {
  if (val === "") {
    update(() => {
      options.value = stringOptions;

      // here you have access to "ref" which
      // is the Vue reference of the QSelect
    });
    return;
  }

  update(() => {
    const needle = val.toLowerCase();
    options.value = stringOptions.filter(
      (v) => v.toLowerCase().indexOf(needle) > -1
    );
  });
}

function handleUpdate(value) {
  let queryAscPrice = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[0]}&text=${value}`;
  let queryDescPrice = `https://www.ozon.ru/search/?from_global=true&sorting=${sortMethods[1]}&text=${value}`;

  let dataAscPrice = fetch(queryAscPrice)
    .then((response) => response.text())
    .then((html) => {
      console.log(html);
      const parser = new DOMParser();
      const htmlDocument = parser.parseFromString(html, "text/html");
      const cheapest = htmlDocument.documentElement.querySelector(".i5j.ij6");

      console.log(cheapest);

      let pricesSection = cheapest.querySelector(":scope > div > div");
      let name = cheapest.querySelector(":scope > div > a > div > span");

      let prices = pricesSection.querySelector(":scope > div");
      priceWithSale.value =
        prices.querySelectorAll(":scope > span")[0].innerHTML;
      priceWithoutSale.value =
        prices.querySelectorAll(":scope > span")[1].innerHTML;

      cheapestName.value = name.innerHTML;
      show.value = true;
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
</style>
