<template>
  <q-table
    title="OZON marketplace"
    :rows="props.rows"
    :columns="props.columns"
    :visible-columns="props.visibleColumns"
    row-key="name"
    separator="cell"
    style="width: 100%; height: 600px"
    class="my-sticky-header-table"
  >
    <template v-slot:body-cell="props">
      <q-td :props="props">
        <div v-if="props.value !== null && props.value !== undefined">
          {{ props.value }}
        </div>
        <div class="text-italic text-grey-5" v-else>
          {{ blankWord }}
        </div>
      </q-td>
    </template>

    <template v-slot:body-cell-newMinPricePnOzon="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['newMinPricePlOzon']"
          v-if="
            props.row['newMinPricePnOzon'] !== null &&
            props.row['newMinPricePnOzon'] !== undefined
          "
          >{{ props.row["newMinPricePnOzon"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>

    <template v-slot:body-cell-newMinPricePnWb="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['newMinPricePlWb']"
          v-if="
            props.row['newMinPricePnWb'] !== null &&
            props.row['newMinPricePnWb'] !== undefined
          "
          >{{ props.row["newMinPricePnWb"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>

    <template v-slot:body-cell-newMaxPricePnOzon="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['newMaxPricePlOzon']"
          v-if="
            props.row['newMaxPricePnOzon'] !== null &&
            props.row['newMaxPricePnOzon'] !== undefined
          "
          >{{ props.row["newMaxPricePnOzon"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>

    <template v-slot:body-cell-newMaxPricePnWb="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['newMaxPricePlWb']"
          v-if="
            props.row['newMaxPricePnWb'] !== null &&
            props.row['newMaxPricePnWb'] !== undefined
          "
          >{{ props.row["newMaxPricePnWb"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>

    <template v-slot:body-cell-defaultPricePnOzon="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['defaultPricePlOzon']"
          v-if="
            props.row['defaultPricePnOzon'] !== null &&
            props.row['defaultPricePnOzon'] !== undefined
          "
          >{{ props.row["defaultPricePnOzon"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>

    <template v-slot:body-cell-defaultPricePnWb="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['defaultPricePlWb']"
          v-if="
            props.row['defaultPricePnWb'] !== null &&
            props.row['defaultPricePnWb'] !== undefined
          "
          >{{ props.row["defaultPricePnWb"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>
  </q-table>
</template>

<script setup>
import { ref, defineProps } from "vue";
const props = defineProps(["rows", "columns", "visibleColumns"]);
const blankWord = ref("empty");

console.error(props.rows);
console.error(props.columns);
console.error(props.visibleColumns);
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
