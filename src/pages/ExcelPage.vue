<template>
  <q-page class="column justify-start items-center">
    <q-uploader
      label="Upload your xlsx file"
      max-files="1"
      accept=".xlsx"
      @added="onFileAdd"
      class="q-mt-xl"
    />
    <TableComponent
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

function onFileAdd(event) {
  let filename = event[0].name;
  fetch(filename)
    .then((res) => {
      console.log("hey!");
      return res.arrayBuffer();
    })
    .then((res) => {
      console.log("file:", res);
      const workbook = XLSX.read(res);
      console.log(workbook);

      const worksheet = workbook.Sheets[workbook.SheetNames[0]];
      const rawData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
      console.log(rawData);

      let columnName = "Наименование товара";
      let [columnNameRowIndex, columnNameColumnIndex] =
        findCellAddressByContent(rawData, columnName);

      let columnNewMinPrice = "Старая мин. цена";
      let columnNewMaxPrice = "Старая макс. цена";

      let [columnNewMinPriceRowIndex, columnNewMinPriceColumnIndex] =
        findCellAddressByContent(rawData, columnNewMinPrice);

      let [columnNewMaxPriceRowIndex, columnNewMaxPriceColumnIndex] =
        findCellAddressByContent(rawData, columnNewMaxPrice);

      console.log(columnNameRowIndex, columnNameColumnIndex);
      console.log(columnNewMinPriceRowIndex, columnNewMinPriceColumnIndex);
      console.log(columnNewMaxPriceRowIndex, columnNewMaxPriceColumnIndex);

      productNames.value = getAllBelowSpecified(
        columnNameRowIndex,
        columnNameColumnIndex,
        rawData
      ); // get all product names below the corresponding title

      console.log(productNames.value);

      productNames.value = productNames.value.filter(
        (item) => item !== null && item !== undefined && item !== ""
      ); // if there are empty cells, we get rid of them

      console.log(productNames.value);

      productNewMinPrices.value = getAllBelowSpecified(
        columnNewMinPriceRowIndex,
        columnNewMinPriceColumnIndex,
        rawData
      ); // get all product New min prices below the corresponding title

      console.log(productNewMinPrices.value);

      productNewMinPrices.value = productNewMinPrices.value.filter(
        (item) => item !== null && item !== undefined && item !== ""
      ); // if there are empty cells, we get rid of them

      console.log(productNewMinPrices.value);

      productNewMaxPrices.value = getAllBelowSpecified(
        columnNewMaxPriceRowIndex,
        columnNewMaxPriceColumnIndex,
        rawData
      ); // get all product New max prices below the corresponding title

      console.log(productNewMaxPrices.value);

      productNewMaxPrices.value = productNewMaxPrices.value.filter(
        (item) => item !== null && item !== undefined && item !== ""
      ); // if there are empty cells, we get rid of them

      console.log(productNewMaxPrices.value);

      showTableComponent.value = true;
    });
}
</script>
