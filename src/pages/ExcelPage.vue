<template>
  <q-uploader
    label="Upload your xlsx file"
    max-files="1"
    accept=".xlsx"
    @added="onFileAdd"
  />
</template>

<script setup>
import * as XLSX from "xlsx";

function findCellAddressByContent(rawData, content) {
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
}

function getAllBelowSpecified(rowIndex, columnIndex, rawData) {
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
      let [rowIndex, columnIndex] = findCellAddressByContent(
        rawData,
        columnName
      );

      console.log(rowIndex, columnIndex);

      let items = getAllBelowSpecified(rowIndex, columnIndex, rawData);
      console.log(items);
    });
}
</script>
