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

const props = defineProps(["rows", "columns", "visibleColumns"]);

function getMaxColumnWidth(rows, columnName, defaultValue = 10) {
  return rows.reduce(
    (w, r) => Math.max(w, String(r[columnName]).length, columnName.length),
    defaultValue
  );
}

function exportTable() {
  console.warn(props.rows);
  console.warn(props.columns);

  let columnsWithoutLinks = props.columns.filter(
    (column) => !column.name.includes("pl")
  ); // get all column names that do not describe product links

  let rowsWithoutLinks = props.rows.map((row) => {
    let newRow = {};
    for (let key in row) {
      if (!key.includes("pl")) {
        newRow[key] = row[key];
      }
    }
    return newRow;
  }); // now every subrow of all rows describes anything but product links

  let columnsWithLinks = props.columns.filter((column) =>
    column.name.includes("pl")
  ); // get all column names that ONLY describe product links

  let rowsWithLinks = props.rows.map((row) => {
    let newRow = {};
    for (let key in row) {
      if (key.includes("pl")) {
        newRow[key] = row[key];
      }
    }
    return newRow;
  }); // now every subrow of all rows describes ONLY product links

  console.warn(columnsWithoutLinks);
  console.warn(rowsWithoutLinks);

  console.warn(columnsWithLinks);
  console.warn(rowsWithLinks);

  let columnsSlice = columnsWithoutLinks.map((column) => column.name); // get a list of column names
  let rowsSlice = rowsWithoutLinks.map((row) => Object.values(row)); // get aoa without column names at the beginning

  let aoa = rowsSlice.slice(); // 'aoa' stands for array of arrays
  aoa.unshift(columnsSlice); // prepend column names

  console.log("hey!");
  console.warn(aoa);

  const workbook = XLSX.utils.book_new();
  let worksheet = XLSX.utils.aoa_to_sheet(aoa); // 'aoa' stands for array of arrays
  XLSX.utils.book_append_sheet(workbook, worksheet, "Sheet1");

  console.warn(worksheet);

  let columnStyles = [];
  for (let columnName of columnsSlice) {
    columnStyles.push({
      wch: getMaxColumnWidth(rowsWithoutLinks, columnName, 10),
    });
  }
  worksheet["!cols"] = columnStyles;

  let index = 2;
  for (let row of props.rows) {
    let linkToCheapest = row["new min price pl"];
    let linkToMostExpensive = row["new max price pl"];
    let linkToDefault = row["default price pl"];

    if (worksheet[`F${index}`]) {
      worksheet[`F${index}`].l = { Target: linkToCheapest };
    }
    if (worksheet[`H${index}`]) {
      worksheet[`H${index}`].l = { Target: linkToMostExpensive };
    }
    if (worksheet[`J${index}`]) {
      worksheet[`J${index}`].l = { Target: linkToDefault };
    }

    ++index;
  }

  XLSX.writeFile(workbook, "test1.xlsx");
}
</script>
