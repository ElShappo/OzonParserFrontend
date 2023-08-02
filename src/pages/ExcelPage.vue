<template>
  <q-page class="column justify-start items-center">
    <q-file
      v-model="file"
      label="Pick one file"
      filled
      style="max-width: 300px"
      @update:model-value="onFileAdd"
    />
    <!-- <q-uploader
      label="Upload your xlsx file"
      max-files="1"
      accept=".xlsx"
      url="http://localhost:5000/read"
      class="q-mt-xl"
      field-name="file"
    /> -->
    <TableComponent
      :productArticleNumbers="productArticleNumbers"
      :productNames="productNames"
      :productNewMinPrices="productNewMinPrices"
      :productNewMaxPrices="productNewMaxPrices"
      v-if="showTableComponent"
    ></TableComponent>
  </q-page>
</template>

<script setup>
import * as XLSX from "xlsx";
import { ref } from "vue";
import TableComponent from "src/components/TableComponent.vue";

const file = ref(null);

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

        console.log(headerRowIndex, headerColumnIndex);

        let productHeaderValues = getAllBelowSpecified(
          headerRowIndex,
          headerColumnIndex,
          rawData
        ); // get all values below the corresponding header

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
