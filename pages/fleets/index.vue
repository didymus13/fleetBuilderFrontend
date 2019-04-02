<template lang="pug">
  section.section.container
    h1.title Your fleets
      button.button.is-primary(@click="createFleet('rebel')")
        b-icon(pack="fab" icon="rebel")
        span New
      button.button.is-primary(@click="createFleet('imperial')")
        b-icon(pack="fab" icon="empire")
        span New
    ul
      li(v-for="fleet in fleets" :key="fleet.id")
        nuxt-link(:to="{ name: 'fleets-id', params: { id: fleet.id, payload: fleet } }")
          | ( {{ fleet.total || 0 }} / {{ fleet.max || 0 }} )
          | {{ fleet.name || fleet.id }}
        button.button.is-danger.is-small(@click="deleteFleet(fleet)")
          b-icon(icon="delete")
</template>

<script>
export default {
  async asyncData({ $axios }) {
    return {
      fleets: await $axios.$get('/fleets')
    }
  },

  methods: {
    async createFleet(faction) {
      const fleet = await this.$axios.$post('fleets', { name: `Your new ${faction} fleet`, faction: faction })
      this.$router.push({ name: 'fleets-id', params: { id: fleet.id } })
    },
    async deleteFleet(fleet) {
      await this.$axios.$delete(`fleets/${fleet.id}`)
      this.fleets = await this.$axios.$get('fleets')
    }
  }
}
</script>
