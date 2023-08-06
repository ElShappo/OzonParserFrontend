<template>
  <q-btn
    color="primary"
    icon="file_download"
    label="Export table (.xlsx)"
    @click="exportTable"
  />
</template>

<script setup>
import { defineProps, onMounted, ref } from "vue";
import * as XLSX from "xlsx";

const props = defineProps(["rows", "columns"]);

console.error(props.columns);
console.error(props.rows);

let columnsWithoutLinks = ref(
  props.columns.filter((column) => !column.name.includes("pl"))
);
let rowsWithoutLinks = ref(
  props.rows.map((row) => {
    for (let key in row) {
      if (key.includes("pl")) {
        delete row[key];
      }
    }
    return row;
  })
);

let columnsWithLinks = ref(
  props.columns.filter((column) => column.name.includes("pl"))
);
let rowsWithLinks = ref(
  props.rows.map((row) => {
    for (let key in row) {
      if (!key.includes("pl")) {
        delete row[key];
      }
    }
    return row;
  })
);
console.error(props.rows);
console.error(rowsWithLinks.value);

function exportTable() {
  let columnsSlice = columnsWithoutLinks.value.map((column) => column.name); // get array of column names
  let rowsSlice = rowsWithoutLinks.value.map((row) => Object.values(row)); // get array of arrays (each subarray is the row in excel)

  let aoa = rowsSlice.slice(); // copy the rowsSlice
  aoa.unshift(columnsSlice); // insert row of column names at the beginning

  // console.log("hey!");
  // console.warn(aoa);

  const workbook = XLSX.utils.book_new();
  let worksheet = XLSX.utils.aoa_to_sheet(aoa);
  XLSX.utils.book_append_sheet(workbook, worksheet, "Sheet1");
  XLSX.writeFile(workbook, "test1.xlsx");
}
</script>
