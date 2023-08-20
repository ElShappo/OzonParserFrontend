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
import Excel from "exceljs/dist/exceljs";
const FileSaver = require("file-saver");

const props = defineProps(["rows", "columns", "visibleColumns"]);

async function exportTable() {
  let table = document.querySelector("#main-table table");
  const workbook = XLSX.utils.table_to_book(table);
  const buffer = XLSX.write(workbook, { bookType: "xlsx", type: "array" });

  const wb = new Excel.Workbook();
  await wb.xlsx.load(buffer);
  const ws = wb.getWorksheet("Sheet1");

  const newWb = new Excel.Workbook();
  const newWs = newWb.addWorksheet("Sheet1", {
    views: [{ state: "frozen", xSplit: 2, ySplit: 3 }],
  });

  ws.eachRow(function (row, i) {
    newWs.addRow(row.values);
  });

  newWs.eachRow(function (row, rowNumber) {
    row.alignment = { vertical: "middle", horizontal: "center" };
  });

  const autoWidth = (worksheet, minimalWidth = 5) => {
    worksheet.columns.forEach((column) => {
      let maxColumnLength = 0;
      column.eachCell({ includeEmpty: true }, (cell) => {
        if (typeof cell.value === "object") {
          const linkStyle = {
            underline: true,
            color: { argb: "FF0000FF" },
          };
          cell.font = linkStyle;
        }
        maxColumnLength = Math.max(
          maxColumnLength,
          minimalWidth,
          cell.value
            ? typeof cell.value === "object"
              ? cell.value.text.toString().length
              : cell.value.toString().length
            : 0
        );
      });
      column.width = maxColumnLength + 2;
    });
  };

  autoWidth(newWs, 5);

  newWs.mergeCells("A1:A3");
  newWs.mergeCells("B1:B3");

  const newBuffer = await newWb.xlsx.writeBuffer();
  FileSaver.saveAs(new Blob([newBuffer]), "result.xlsx");
}
</script>
