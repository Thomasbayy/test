<template lang="pug">
  v-container
    v-row
      v-col(cols)
        DataTable(
          v-if="items.length || (!items.length && !loading)"
          :headers="headers"
          :items="items"
          @filter="fetchData"
          :totalItems="totalItems"
          :loading="loading"
        )
        v-progress-circular(
          v-else
          width="2"
          color="rs__primary"
          indeterminate
        ).mx-auto
</template>

<script>
import DataTable from '~/components/DataTable.vue'
import sales from '~/api/sales.js'

export default {
  components: {
    DataTable
  },
  data() {
    return {
      sales,
      totalItems: 0,
      items: [],
      loading: false,
      headers: [
        { text: 'Name', value: 'user', align: 'start' },
        { text: 'Email', value: 'email' },
        { text: 'Gender', value: 'gender' },
        { text: 'Year', value: 'year' },
        { text: 'Sales', value: 'sales' },
        { text: 'Country', value: 'country' },
      ],
    }
  },
  created() {
    this.fetchData({ pagination: { itemsPerPage: 10, page: 1, sortBy: [], sortDesc: [false]}})
  },
  methods: {
    async fetchData({ pagination, search = '', countryFilter = null, yearFilter = [1900, 2021], salesFilter = null }) {
      this.loading = true;
      let results = sales.results;
      if (search) {
        results = results.filter((item) =>
          item.user.first_name.toLowerCase().includes(search.toLowerCase()) ||
          item.user.last_name.toLowerCase().includes(search.toLowerCase()) ||
          item.email.toLowerCase().includes(search.toLowerCase())
        );
      }

      if (countryFilter) {
        results = results.filter((item) => item.country.toLowerCase() === countryFilter.toLowerCase())
      }

      if (yearFilter) {
        results = results.filter((item) => {
          return item.year >= yearFilter[0] && item.year <= yearFilter[1];
        })
      }

      if (salesFilter) {
        results = results.filter((item) => {
          return item.sales > salesFilter[0] && item.sales < salesFilter[1];
        })
      }

      if (pagination.sortBy.length) {
        results.sort((a, b) => {
          if (pagination.sortBy[0] === 'user') {
            if (pagination.sortDesc[0]) {
              return b.user.last_name < a.user.last_name ? -1 : 1;
            } else {
              return a.user.last_name < b.user.last_name ? -1 : 1;
            }
          } else {
            if (pagination.sortDesc[0]) {
              return b[pagination.sortBy[0]] < a[pagination.sortBy[0]] ? -1 : 1;
            } else {
              return a[pagination.sortBy[0]] < b[pagination.sortBy[0]] ? -1 : 1;
            }
          }
        })
      }

      this.totalItems = results.length;
      const start = (pagination.page - 1) * pagination.itemsPerPage
      await this.delay(200)
      this.items = await results.slice(start, start + pagination.itemsPerPage < results.length ? start + pagination.itemsPerPage : results.length);
      this.loading = false;
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%
</style>
