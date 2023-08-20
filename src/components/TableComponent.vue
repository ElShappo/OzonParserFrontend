<template>
  <q-table
    title="Marketplaces"
    :rows="props.rows"
    row-key="name"
    separator="cell"
    style="width: 100%; height: 600px"
    class="my-sticky-header-table"
    id="main-table"
    :pagination="{
      rowsPerPage: props.rows.length,
    }"
    :hide-pagination="true"
  >
    <template v-slot:header>
      <tr class="headers-1 text-center">
        <th rowspan="3" style="font-size: 1.5em">ID</th>
        <th rowspan="3" style="font-size: 1.5em">name</th>
        <th colspan="6" style="font-size: 1.5em">OZON</th>
        <th colspan="6" style="font-size: 1.5em">WB</th>
      </tr>
      <tr class="headers-2 text-center">
        <td colspan="2" style="font-size: 1.5em">min</td>
        <td colspan="2" style="font-size: 1.5em">max</td>
        <td colspan="2" style="font-size: 1.5em">default</td>

        <td colspan="2" style="font-size: 1.5em">min</td>
        <td colspan="2" style="font-size: 1.5em">max</td>
        <td colspan="2" style="font-size: 1.5em">default</td>
      </tr>
      <tr class="headers-3 text-center">
        <td>price</td>
        <td>name</td>

        <td>price</td>
        <td>name</td>

        <td>price</td>
        <td>name</td>

        <td>price</td>
        <td>name</td>

        <td>price</td>
        <td>name</td>

        <td>price</td>
        <td>name</td>
      </tr>
    </template>

    <template v-slot:body="props">
      <tr :props="props">
        <td :props="props" key="articleNumber" class="text-center">
          {{ props.row["articleNumber"] }}
        </td>
        <td :props="props" key="name">{{ props.row["name"] }}</td>

        <td :props="props" key="minPriceOzon" class="text-center">
          {{ props.row["minPriceOzon"] }}
        </td>
        <td :props="props" key="minPricePnOzon">
          <a
            target="_blank"
            :href="props.row['minPricePlOzon']"
            v-if="
              props.row['minPricePnOzon'] !== null &&
              props.row['minPricePnOzon'] !== undefined
            "
            >{{ props.row["minPricePnOzon"] }}</a
          >
          <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
        </td>

        <td :props="props" key="maxPriceOzon" class="text-center">
          {{ props.row["maxPriceOzon"] }}
        </td>
        <td :props="props" key="maxPricePlOzon">
          <a
            target="_blank"
            :href="props.row['maxPricePlOzon']"
            v-if="
              props.row['maxPricePnOzon'] !== null &&
              props.row['maxPricePnOzon'] !== undefined
            "
            >{{ props.row["maxPricePnOzon"] }}</a
          >
          <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
        </td>

        <td :props="props" key="defaultPriceOzon" class="text-center">
          {{ props.row["defaultPriceOzon"] }}
        </td>
        <td :props="props" key="defaultPricePlOzon">
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
        </td>

        <td :props="props" key="minPriceWb" class="text-center">
          {{ props.row["minPriceWb"] }}
        </td>
        <td :props="props" key="minPricePlWb">
          <a
            target="_blank"
            :href="props.row['minPricePlWb']"
            v-if="
              props.row['minPricePnWb'] !== null &&
              props.row['minPricePnWb'] !== undefined
            "
            >{{ props.row["minPricePnWb"] }}</a
          >
          <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
        </td>

        <td :props="props" key="maxPriceWb" class="text-center">
          {{ props.row["maxPriceWb"] }}
        </td>
        <td :props="props" key="maxPricePlWb">
          <a
            target="_blank"
            :href="props.row['maxPricePlWb']"
            v-if="
              props.row['maxPricePnWb'] !== null &&
              props.row['maxPricePnWb'] !== undefined
            "
            >{{ props.row["maxPricePnWb"] }}</a
          >
          <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
        </td>

        <td :props="props" key="defaultPriceWb" class="text-center">
          {{ props.row["defaultPriceWb"] }}
        </td>
        <td :props="props" key="defaultPricePlWb">
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
        </td>
      </tr>
    </template>

    <!-- <template v-slot:body-cell="props">
      <q-td :props="props">
        <div v-if="props.value !== null && props.value !== undefined">
          {{ props.value }}
        </div>
        <div class="text-italic text-grey-5" v-else>
          {{ blankWord }}
        </div>
      </q-td>
    </template>

    <template v-slot:body-cell-minPricePnOzon="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['minPricePlOzon']"
          v-if="
            props.row['minPricePnOzon'] !== null &&
            props.row['minPricePnOzon'] !== undefined
          "
          >{{ props.row["minPricePnOzon"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>

    <template v-slot:body-cell-minPricePnWb="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['minPricePlWb']"
          v-if="
            props.row['minPricePnWb'] !== null &&
            props.row['minPricePnWb'] !== undefined
          "
          >{{ props.row["minPricePnWb"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>

    <template v-slot:body-cell-maxPricePnOzon="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['maxPricePlOzon']"
          v-if="
            props.row['maxPricePnOzon'] !== null &&
            props.row['maxPricePnOzon'] !== undefined
          "
          >{{ props.row["maxPricePnOzon"] }}</a
        >
        <span class="text-italic text-grey-5" v-else>{{ blankWord }}</span>
      </q-td>
    </template>

    <template v-slot:body-cell-maxPricePnWb="props">
      <q-td :props="props">
        <a
          target="_blank"
          :href="props.row['maxPricePlWb']"
          v-if="
            props.row['maxPricePnWb'] !== null &&
            props.row['maxPricePnWb'] !== undefined
          "
          >{{ props.row["maxPricePnWb"] }}</a
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
    </template> -->
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
.headers-1 td,
.headers-2 td,
.headers-3 td,
.headers-1 th,
.headers-2 th,
.headers-3 th {
  border: 1px solid rgba(0, 0, 0, 0.12) !important;
}

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
