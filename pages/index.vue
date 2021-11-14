<template lang="pug">
  v-container
    v-row
      v-col(cols)
        DataTable(
          v-if="items.length"
          :headers="headers"
          :items="items"
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
      items: [],
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
  async created() {
    this.items = await this.fetchData(0, 50)
  },
  methods: {
    async fetchData(page, size) {
      const start = page * size
      await this.delay(200)
      return await sales.results.slice(start, start + size)
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    }
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%
</style>
