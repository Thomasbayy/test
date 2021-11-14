<template>
  <v-data-table
    :headers="headers"
    :items="items"
    item-key="email"
    class="elevation-1 mt-10"
  >
    <!-- Above table -->
    <template
      #top
    >
      <div class="table-header">
        <v-text-field
          v-model="search"
          class="search-input mt-0 pt-0"
          placeholder="Search..."
        />

        <v-menu
          offset-y
          :close-on-content-click="false"
        >
          <template #activator="{ on, attrs }">
            <v-btn
              small
              v-bind="attrs"
              v-on="on"

            >
              Toggle columns
            </v-btn>
          </template>

          <v-list dense>
            <v-list-item
              v-for="header in headers"
              class="toggle-item"
            >
              <span class="toggle-item-name">
                {{ header.text }}
              </span>
              <v-checkbox v-model="headerToggledStatus[header.value]" />
            </v-list-item>
          </v-list>
        </v-menu>
      </div>
    </template>

    <!-- Year header -->
    <template #header.year="{ header }">
      <span>Year</span>
      <v-menu
        offset-y
        :close-on-content-click="false"
      >
        <template #activator="{ on, attrs }">
          <v-btn
            text
            icon
            :color="yearFilterIsModified ? 'primary' : 'grey'"
            small
            v-bind="attrs"
            v-on="on"
          >
            <v-icon>mdi-filter</v-icon>
          </v-btn>
        </template>
        <v-card class="pa-4 card year-slider">
          <v-range-slider
            :min="yearFilterMin"
            :max="yearFilterMax"
            v-model="yearFilter"
            :step="1"
            class="mt-6"
          >
            <template v-slot:thumb-label="props">
              {{ props.value }}
            </template>
          </v-range-slider>
        </v-card>
      </v-menu>
    </template>

    <!-- Sales header -->
    <template #header.sales="{ header }">
      <span>Sales</span>
      <v-menu
        offset-y
        :close-on-content-click="false"
      >
        <template #activator="{ on, attrs }">
          <v-btn
            text
            icon
            :color="salesFilter !== 0 ? 'primary' : 'grey'"
            small
            v-bind="attrs"
            v-on="on"
          >
            <v-icon>mdi-filter</v-icon>
          </v-btn>
        </template>
        <v-card class="pa-4 card year-slider">
          <v-list-item-group
            v-model="salesFilter"
            color="primary"
          >
            <v-list-item
              v-for="(option, i) in salesOptions"
              :key="i"
            >
              {{ option }}
            </v-list-item>
          </v-list-item-group>
        </v-card>
      </v-menu>
    </template>

    <!-- Country header -->
    <template #header.country="{ header }">
      <span>Country</span>
      <v-menu
        offset-y
        :close-on-content-click="false"
      >
        <template #activator="{ on, attrs }">
          <v-btn
            text
            icon
            :color="countryFilter ? 'primary' : 'grey'"
            small
            v-bind="attrs"
            v-on="on"
          >
            <v-icon>mdi-filter</v-icon>
          </v-btn>
        </template>
        <v-card class="pa-4 card">
          <v-autocomplete
            v-model="countryFilter"
            :items="countryOptions"
            clearable
          ></v-autocomplete>
        </v-card>
      </v-menu>
    </template>

    <!-- Name column -->
    <template
      #item.user="{ item }"
    >
      {{ `${item.user.title !== 'Honorable' ? `${item.user.title}.` : item.user.title} ${item.user.first_name} ${item.user.last_name}` }}
    </template>

    <!-- Gender column -->
    <template
      #item.gender="{ item }"
    >
      {{ item.gender }}
      <v-icon
        small
      >
        {{ item.gender === 'Male' ? 'mdi-gender-male' :
            item.gender === 'Female' ? 'mdi-gender-female' :
            item.gender === 'Non-binary' ? 'mdi-gender-non-binary' :
            'mdi-gender-male-female-variant'}}
      </v-icon>
    </template>

    <!-- Sales column -->
    <template
      #item.sales="{ item }"
    >
      <span class="sales-item">{{ Intl.NumberFormat('en-us', { style: 'currency', currency: 'USD', }).format(item.sales) }}</span>
    </template>

  </v-data-table>
</template>

<script>

import countries from '@/assets/countries.json';

export default {
  props: ['headers', 'items'],
  data() {
    return {
      search: '',
      headerToggledStatus: {},
      countryFilter: '',
      countryOptions: [],
      yearFilterMin: 1970,
      yearFilterMax: 2021,
      yearFilter: [1970, 2021],
      salesOptions: ['Unrestricted', '0 - 149.999', '150.000 - 299.999', '300.000 - 449.000', '450.000 - 599.999', '600.000 - 750.000', '750.000+'],
      salesFilter: 0
    }
  },
  computed: {
    yearFilterIsModified() {
      return this.yearFilter[0] !== 1970 || this.yearFilter[1] !== 2021;
    }
  },
  watch: {
    search() {
      this.emitFilters();
    }
  },
  mounted() {
    this.headerToggledStatus = this.headers.reduce((acc, cur) => {
      return { ...acc, [cur.value]: true }
    }, {});

    this.countryOptions = countries.map((country) => country.name);
  },
  methods: {
    emitFilters() {
      this.$emit('filter', {
        search,
        countryFilter,
        yearFilter,
        salesFilter
      })
    }
  }
}
</script>

<style lang="scss" scoped>
  .v-data-table {
    max-width: 100%;
  }

  .table-header {
    padding: 16px 16px 0 16px;
    display: flex;
    justify-content: space-between;
  }

  .search-input {
    max-width: 350px;
  }

  .header-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .toggle-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 50px;

    &-name {
      flex: 1;
    }
  }

  .sales-item {
    color: green;
  }

  .year-slider {
    width: 300px;
  }

  .sales-selected {
    background-color: #ada9fd;
    border-radius: 4px;
  }

</style>
