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
    <template v-slot:body-cell-name="props">
      <q-td :props="props">
        <div>
          <q-badge color="purple" :label="props.value" />
        </div>
        <div class="my-table-details">
          {{ props.row.details }}
        </div>
      </q-td>
    </template>
    <!-- <template v-slot:body="props">
      <q-tr :props="props">
        <q-td key="article number" :props="props">
          {{ props.row["article number"] }}
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['article number'] === null ||
              props.row['article number'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="name" :props="props">
          {{ props.row.name }}
          <span
            class="text-italic text-grey-5"
            v-if="props.row.name === null || props.row.name === undefined"
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="old min price" :props="props">
          {{ props.row["old min price"] }}
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['old min price'] === null ||
              props.row['old min price'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="old max price" :props="props">
          {{ props.row["old max price"] }}
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['old max price'] === null ||
              props.row['old max price'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="new min price" :props="props">
          {{ props.row["new min price"] }}
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['new min price'] === null ||
              props.row['new min price'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="new min price pn" :props="props">
          <a target="_blank" :href="props.row['new min price pl']">{{
            props.row["new min price pn"]
          }}</a>
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['new min price pn'] === null ||
              props.row['new min price pn'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="new max price" :props="props">
          {{ props.row["new max price"] }}
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['new max price'] === null ||
              props.row['new max price'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="new max price pn" :props="props">
          <a target="_blank" :href="props.row['new max price pl']">{{
            props.row["new max price pn"]
          }}</a>
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['new max price pn'] === null ||
              props.row['new max price pn'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="default price" :props="props">
          {{ props.row["default price"] }}
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['default price'] === null ||
              props.row['default price'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
        <q-td key="default price pn" :props="props">
          <a target="_blank" :href="props.row['default price pl']">{{
            props.row["default price pn"]
          }}</a>
          <span
            class="text-italic text-grey-5"
            v-if="
              props.row['default price pn'] === null ||
              props.row['default price pn'] === undefined
            "
            >{{ blankWord }}</span
          >
        </q-td>
      </q-tr>
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
