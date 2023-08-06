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
  console.warn(props.rows);
  console.warn(props.columns);

  let columnsWithoutLinks = props.columns.filter(
    (column) => !column.name.includes("pl")
  );

  let rowsWithoutLinks = props.rows.map((row) => {
    let newRow = {};
    for (let key in row) {
      if (!key.includes("pl")) {
        newRow[key] = row[key];
      }
    }
    return newRow;
  });

  let columnsWithLinks = props.columns.filter((column) =>
    column.name.includes("pl")
  );

  let rowsWithLinks = props.rows.map((row) => {
    let newRow = {};
    for (let key in row) {
      if (key.includes("pl")) {
        newRow[key] = row[key];
      }
    }
    return newRow;
  });

  console.warn(columnsWithoutLinks);
  console.warn(rowsWithoutLinks);

  console.warn(columnsWithLinks);
  console.warn(rowsWithLinks);

  let columnsSlice = columnsWithoutLinks.map((column) => column.name);
  let rowsSlice = rowsWithoutLinks.map((row) => Object.values(row));

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
