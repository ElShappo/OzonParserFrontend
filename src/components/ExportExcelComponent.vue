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

// function getMaxColumnWidthByColumnName(rows, columnName, defaultValue = 10) {
//   return rows.reduce(
//     (w, r) => Math.max(w, String(r[columnName]).length, columnName.length),
//     defaultValue
//   );
// }

function getMaxColumnWidthByColumnName(
  worksheet,
  columnName,
  startRow,
  endRow
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
  console.log(Math.max(...widths));
  return Math.max(...widths);
}

function getColumnStyles(worksheet) {
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
        endRow
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

  worksheet["!cols"] = getColumnStyles(worksheet);
  XLSX.writeFile(workbook, "test1.xlsx");
}

// function exportTable() {
//   console.warn(props.rows);
//   console.warn(props.columns);

//   let columnsWithoutLinks = props.columns.filter(
//     (column) => !column.name.includes("pl")
//   ); // get all column names that do not describe product links

//   let rowsWithoutLinks = props.rows.map((row) => {
//     let newRow = {};
//     for (let key in row) {
//       if (!key.includes("pl")) {
//         newRow[key] = row[key];
//       }
//     }
//     return newRow;
//   }); // now every subrow of all rows describes anything but product links

//   let columnsWithLinks = props.columns.filter((column) =>
//     column.name.includes("pl")
//   ); // get all column names that ONLY describe product links

//   let rowsWithLinks = props.rows.map((row) => {
//     let newRow = {};
//     for (let key in row) {
//       if (key.includes("pl")) {
//         newRow[key] = row[key];
//       }
//     }
//     return newRow;
//   }); // now every subrow of all rows describes ONLY product links

//   console.warn(columnsWithoutLinks);
//   console.warn(rowsWithoutLinks);

//   console.warn(columnsWithLinks);
//   console.warn(rowsWithLinks);

//   let columnsSlice = columnsWithoutLinks.map((column) => column.name); // get a list of column names
//   let rowsSlice = rowsWithoutLinks.map((row) => Object.values(row)); // get aoa without column names at the beginning

//   let aoa = rowsSlice.slice(); // 'aoa' stands for array of arrays
//   aoa.unshift(columnsSlice); // prepend column names

//   console.log("hey!");
//   console.warn(aoa);

//   const workbook = XLSX.utils.book_new();
//   let worksheet = XLSX.utils.aoa_to_sheet(aoa); // 'aoa' stands for array of arrays
//   XLSX.utils.book_append_sheet(workbook, worksheet, "Sheet1");

//   console.warn(worksheet);

//   let columnStyles = [];
//   for (let columnName of columnsSlice) {
//     columnStyles.push({
//       wch: getMaxColumnWidthByColumnName(rowsWithoutLinks, columnName, 10),
//     });
//   }
//   worksheet["!cols"] = columnStyles;

//   let index = 2;
//   for (let row of props.rows) {
//     let linkToCheapest = row["new min price pl"];
//     let linkToMostExpensive = row["new max price pl"];
//     let linkToDefault = row["default price pl"];

//     if (worksheet[`F${index}`]) {
//       worksheet[`F${index}`].l = { Target: linkToCheapest };
//     }
//     if (worksheet[`H${index}`]) {
//       worksheet[`H${index}`].l = { Target: linkToMostExpensive };
//     }
//     if (worksheet[`J${index}`]) {
//       worksheet[`J${index}`].l = { Target: linkToDefault };
//     }

//     ++index;
//   }

//   XLSX.writeFile(workbook, "test1.xlsx");
// }
</script>
