<template lang="pug">
  section.section.container
    h1.title
      input.input.title.is-paddingless.is-borderless(v-model="fleet.name")
    h3.subtitle {{ total }} / {{ fleet.max }}

    .columns(v-for="(ship,i) in fleet.ships" :key="ship.id")
      .column.is-one-quarter
        figure.image
          img(:src="ship.ship.image" :alt="ship.ship.name" @click="deleteShip(i)")
        div.is-size-7 Add
          button.button.is-small(v-for="spot in ship.ship.slots" @click="openUpgradeSelect(spot, ship)") {{ spot }}

      .column
        .columns.is-mobile
          .column.is-half-mobile.is-2-tablet(v-for="(upgrade, u) in ship.upgrades" :key="upgrade.id")
            figure.image
              img(:src="upgrade.upgrade.image" :alt="upgrade.upgrade.name" @click="deleteUpgrade(i, u)")

    button.button.is-primary(@click="openShipSelect") Add Ship

    b-modal(:active.sync="showUpgradeSelect")
      .card
        .card-content
          .columns.content
            .column.is-one-quarter(v-for="upgrade in upgradeList" :key="upgrade.id")
              figure.image(@click="selectedShip.upgrades.push({ upgrade: upgrade }); showUpgradeSelect = false")
                img(:src="upgrade.image")

    b-modal(:active.sync="showShipSelect")
      .card
        .card-content
          .columns.content
            .column.is-one-quarter(v-for="ship in shipList" :key="ship.id")
              figure.image(@click="fleet.ships.push({ ship: ship, upgrades: [] }); showShipSelect = false")
                img(:src="ship.image")
</template>

<script>
import _ from 'lodash'
export default {
  data() {
    return {
      showUpgradeSelect: false,
      upgradeList: [],
      selectedShip: {},
      showShipSelect: false,
      shipList: []
    }
  },

  computed: {
    payload() {
      return {
        name: this.fleet.name,
        total: this.total,
        ships: _.map(this.fleet.ships, (ship) => {
          return {
            'ship': ship.ship._id,
            upgrades: _.map(ship.upgrades, (upgrade) => {
              return { upgrade: upgrade.upgrade._id }
            })
          }
        })
      }
    },
    total() {
      return _.sumBy(this.fleet.ships, 'ship.cost') +
        _.sum(
          _.map(this.fleet.ships, (ship) => {
            return _.sumBy(ship.upgrades, 'upgrade.cost')
          })
        )
    }
  },

  watch: {
    fleet: {
      handler: _.debounce(function (data) {
        this.save()
      }, 300),
      deep: true
    }
  },

  async asyncData({ params, payload, $axios }) {
    return {
      fleet: params.payload || await $axios.$get(`fleets/${params.id}`)
    }
  },

  methods: {
    deleteShip(i) {
      this.fleet.ships.splice(i, 1)
    },

    deleteUpgrade(ship, upgrade) {
      ship = this.fleet.ships[ship]
      ship.upgrades.splice(upgrade, 1)
    },

    async save() {
      await this.$axios.put(`fleets/${this.fleet.id}`, this.payload)
    },

    async openUpgradeSelect(type, ship) {
      this.showUpgradeSelect = true
      this.selectedShip = ship
      this.upgradeList = await this.$axios.$get('upgrades', { params: { type: type } })
    },

    async openShipSelect(faction) {
      this.showShipSelect = true
      this.shipList = await this.$axios.$get('ships', { params: { faction: this.fleet.faction } })
    }
  }
}
</script>

<style scoped>
  .is-borderless {
    border: none;
    box-shadow: none;
    border-color: transparent;
  }
</style>
