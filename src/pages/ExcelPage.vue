<template>
  <q-page class="column justify-start items-center">
    <q-file
      bottom-slots
      counter
      v-model="fileModel"
      label="Pick one file"
      filled
      style="max-width: 300px"
      @update:model-value="onFileAdd"
      class="q-mt-xl"
    >
      <template v-slot:append>
        <q-icon
          name="close"
          @click.stop.prevent="file = null"
          class="cursor-pointer"
        />
      </template>
    </q-file>
    <GetTableDataComponent
      :productArticleNumbers="productArticleNumbers"
      :productNames="productNames"
      :productNewMinPrices="productNewMinPrices"
      :productNewMaxPrices="productNewMaxPrices"
      v-if="showTableComponent"
    ></GetTableDataComponent>
  </q-page>
</template>

<script setup>
import { ref } from "vue";
import GetTableDataComponent from "src/components/GetTableDataComponent.vue";

const fileModel = ref(null);

const productArticleNumbers = ref([]); // product article numbers will be stored there after .xlsx file parse
const productNames = ref([]); // product names will be stored there after .xlsx file parse
const productNewMinPrices = ref([]); // product New min prices will be stored there after .xlsx file parse (if present)
const productNewMaxPrices = ref([]); // product New max prices will be stored there after .xlsx file parse (if present)

const showTableComponent = ref(false); // whether to show table with data (defaults to 'false' before .xlsx file parse)

function findCellAddressByContent(rawData, content) {
  // find the address (row index and column index) of the cell with a specific 'content'
  let rowIndex = 0;
  for (let row of rawData) {
    let columnIndex = 0;

    for (let cell of row) {
      if (cell === content) {
        return [rowIndex, columnIndex];
      }
      ++columnIndex;
    }
    ++rowIndex;
  }
  return [null, null];
}

function getAllBelowSpecified(rowIndex, columnIndex, rawData) {
  // get all cells below the specified address (excluding the topmost cell)
  let cells = [];
  for (let i = rowIndex + 1; i < rawData.length; i++) {
    cells.push(rawData[i][columnIndex]);
  }
  return cells;
}

function onFileAdd(file) {
  showTableComponent.value = false;
  console.log(file);
  const formData = new FormData();
  formData.append("file", file);
  fetch("http://localhost:5000/read", {
    method: "POST",
    body: formData,
  })
    .then((res) => fetch("http://localhost:5000/"))
    .then((res) => res.json())
    .then((rawData) => {
      console.log(rawData);

      const columnHeaders = [
        "Артикул",
        "Наименование товара",
        "Старая мин. цена",
        "Старая макс. цена",
      ];

      for (let columnHeader of columnHeaders) {
        let [headerRowIndex, headerColumnIndex] = findCellAddressByContent(
          rawData,
          columnHeader
        );

        console.error(headerRowIndex, headerColumnIndex);

        let productHeaderValues = getAllBelowSpecified(
          headerRowIndex,
          headerColumnIndex,
          rawData
        ); // get all values below the corresponding header

        productHeaderValues = productHeaderValues.filter(
          (item) => item !== null && item !== undefined && item !== ""
        ); // if there are empty cells, we get rid of them

        switch (columnHeader) {
          case "Артикул":
            productArticleNumbers.value = productHeaderValues;
            break;
          case "Наименование товара":
            productNames.value = productHeaderValues;
            break;
          case "Старая мин. цена":
            productNewMinPrices.value = productHeaderValues;
            break;
          case "Старая макс. цена":
            productNewMaxPrices.value = productHeaderValues;
            break;
        }
        console.error(productHeaderValues);
        showTableComponent.value = true;
      }
    });
}
</script>
