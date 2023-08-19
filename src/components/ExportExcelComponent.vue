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

function getMaxColumnWidthByColumnName(
  worksheet,
  columnName,
  startRow,
  endRow,
  method
) {
  let widths = [];
  for (let i = startRow; i < endRow; ++i) {
    let cell = worksheet[`${columnName}${i}`]; // access current cell
    console.log(cell);
    if (cell === undefined) {
      console.log(`Row ${i}`);
      continue;
    }
    widths.push(String(cell.v).length); // string conversion is required because we might need to find the length of a number or the length of some other type
  }
  console.log("Now at " + columnName);
  console.log(method(widths));
  return method(widths);
}

function getColumnStyles(worksheet, method) {
  const range = worksheet["!fullref"]; // get the cell range in the table (e.g. it can be A1:BC75)
  const [startAddress, endAddress] = range.match(/[a-z]+\d+/gi); // from comment above 'startAddress' may contain A1, 'endAddress' - BC75
  const [startColumn, endColumn] = range.match(/[a-z]+/gi); // from comment above 'startColumn' may contain A, 'endColumn' - BC
  const [startRow, endRow] = range.match(/\d+/gi); // from comment above 'startRow' may contain 1, 'endRow' - 75

  let columnStyles = []; // stores styles for each column

  const startColumnDecoded = XLSX.utils.decode_col(startColumn);
  const endColumnDecoded = XLSX.utils.decode_col(endColumn);

  for (let i = startColumnDecoded; i <= endColumnDecoded; ++i) {
    const columnName = XLSX.utils.encode_col(i); // from '0-format' to 'A1' format
    columnStyles.push({
      wch: getMaxColumnWidthByColumnName(
        worksheet,
        columnName,
        startRow,
        endRow,
        method
      ),
    });
  }
  return columnStyles;
}

function exportTable() {
  let table = document.querySelector("#main-table table");
  const workbook = XLSX.utils.table_to_book(table);
  let worksheet = workbook.Sheets["Sheet1"];

  console.log(workbook.SheetNames);
  console.log(worksheet);
  console.log(worksheet["A1"]);
  console.log(worksheet["A2"]);

  let range = worksheet["!fullref"];

  console.log(range);
  console.log(XLSX.utils.decode_range(range));
  console.log(XLSX.utils.encode_col(26));
  console.log(XLSX.utils.decode_col("B"));

  worksheet["!cols"] = getColumnStyles(worksheet, function (widths) {
    return Math.max(...widths);
  });
  XLSX.writeFile(workbook, "test1.xlsx");
}
</script>
