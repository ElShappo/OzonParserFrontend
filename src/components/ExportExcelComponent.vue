<template>
  <q-btn
    color="primary"
    icon="file_download"
    label="Export table (.xlsx)"
    @click="exportTable"
  />
</template>

<script setup>
import { defineProps } from "vue";
import * as XLSX from "xlsx";

const props = defineProps(["rows", "columns"]);

function exportTable() {
  let columnsSlice = props.columns.map((column) => column.name);
  let rowsSlice = props.rows.map((row) => Object.values(row));

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
