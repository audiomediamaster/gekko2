<template>
  <q-card class="q-mr-md">
    <q-card-title class="bg-amber-8 text-white">Market</q-card-title>
    <q-card-separator/>
    <q-card-main>
      <exchange-picker
        v-on:exchange="setExchange($event)"
        :onlyTradable="onlyTradable"
        :onlyImportable="onlyImportable"
      />
    </q-card-main>
    <q-card-separator/>
    <q-card-main>
      <q-field
        v-if="currencies"
        label="Currency"
        helper="Pick a currency"
        :label-width="3"
      >
        <q-select
          filter
          autofocus-filter
          v-model="currency"
          :options="currencies"
          radio
        />
      </q-field>
    </q-card-main>
    <q-card-separator></q-card-separator>
    <q-card-main>
      <q-field
        v-if="currencies"
        label="Asset"
        helper="Pick an asset"
        :label-width="3"
      >
        <q-select
          filter
          autofocus-filter
          v-model="asset"
          :options="assets"
          radio
        />
      </q-field>
    </q-card-main>
  </q-card>
</template>

<script>

  import _ from 'lodash'
  import exchangePicker from './exchangepicker'

  export default {
    props: ['onlyTradable', 'onlyImportable'],
    components: {
      exchangePicker
    },
    data: function () {
      return {
        // defaults:
        exchange: 'binance',
        currency: 'BTC',
        asset: 'NEO',
      };
    },
    created: function () {
      this.emitConfig();
    },
    computed: {
      exchanges: function () {

        let exchanges = Object.assign({}, this.$store.state.config.exchanges);

        if (_.isEmpty(exchanges))
          return false;

        if (this.onlyTradable) {
          _.each(exchanges, (e, name) => {
            if (!e.tradable)
              delete exchanges[name];
          });
        }

        if (this.onlyImportable) {
          _.each(exchanges, (e, name) => {
            if (!e.importable)
              delete exchanges[name];
          });
        }

        return exchanges;
      },
      markets: function () {
        return this.exchanges ? this.exchanges[this.exchange] : null;
      },
      assets: function () {
        let list = this.exchanges ? this.exchanges[this.exchange].markets[this.currency] : [];
        if (list) {
          list = _.orderBy(_.map(list, function (item) {
            return {value: item, label: item}
          }),['label'],['asc'])
          if (list.length)
            this.asset = list[0].value
        }
        return list;
      },
      currencies: function () {
        let list = this.exchanges ? _.keys(this.exchanges[this.exchange].markets) : [];
        if (list) {
          list = _.orderBy(_.map(list, function (item) {
            return {value: item, label: item}
          }),['label'], ['asc']);
          if (list.length)
            this.currency = list[0].value;
        }
        return list;
      },
      watchConfig: function () {
        return {
          watch: {
            exchange: this.exchange,
            currency: this.currency,
            asset: this.asset,
          }
        }
      }
    },

    watch: {
      currency: function () {
        this.emitConfig()
      },
      asset: function () {
        this.emitConfig()
      },
      market: function () {
        this.emitConfig()
      },
      exchanges: function () {
        this.emitConfig()
      },
      exchange: function () {
        this.emitConfig()
      }
    },

    methods: {
      setExchange: function (exchange) {
        this.exchange = exchange;
      },
      emitConfig: function () {
        this.$emit('market', this.watchConfig);
      }
    }
  }
</script>
