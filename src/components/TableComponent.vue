<template>
  <q-table
    title="OZON marketplace"
    :rows="props.rows"
    :columns="props.columns"
    :visible-columns="
      props.columns
        .filter((column) => !column.name.includes('pl'))
        .map((item) => item.name)
    "
    row-key="name"
    style="width: 100%; height: 600px"
    class="my-sticky-header-table"
  >
    <!-- <template v-slot:body-cell-["new min price pn ozon"]="props">
      <q-td :props="props">
        <div>
          <q-badge color="purple" :label="props.value" />
        </div>
        <div class="my-table-details">
          {{ props.row.details }}
        </div>
      </q-td>
    </template> -->
  </q-table>
</template>

<script setup>
import { ref, defineProps } from "vue";
const props = defineProps(["rows", "columns"]);
const blankWord = ref("empty");
console.error(props.columns);
console.error(props.columns.filter((column) => !column.name.includes("pl")));
console.error(props.rows);
</script>

<style>
.q-item__section.column.q-item__section--side {
  text-align: center;
}

.my-sticky-header-table {
  /* height or max-height is important */
  height: 310px;
}

.q-table__top,
.q-table__bottom,
thead tr:first-child th {
  /* bg color is important for th; just specify one */
  background-color: white;
}

thead tr th {
  position: sticky;
  z-index: 1;
}
thead tr:first-child th {
  top: 0;
}

/* this is when the loading indicator appears */
&.q-table--loading thead tr:last-child th {
  /* height of all previous header rows */
  top: 48px;
}

/* prevent scrolling behind sticky top row on focus */
tbody {
  /* height of all previous header rows */
  scroll-margin-top: 48px;
}
</style>
