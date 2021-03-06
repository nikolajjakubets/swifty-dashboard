<!-- 

© 2018 SwiftyCloud OÜ. All rights reserved.
Contact: info@swifty.cloud

-->

<template>
  <div v-loading="loading">
    <p>Usage stats for your functions</p>

    <el-row>
      <el-col :span="24">
        <el-form label-width="170px" class="timeline-form">
          <el-form-item label="Timeline">
            <el-select placeholder="Timeline" v-model="periods">
              <el-option v-for="(value, label) in $store.getters.getPeriods" :label="label" :value="value" :key="value"></el-option>
            </el-select>
            <span class="period">from {{ periodFrom | moment('L') }} to {{ periodTill | moment('L') }}</span>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>

    <el-row v-if="!showFunctionDetails">
      <el-col :span="24">
        <el-table :data="funcStats" style="width: 100%">
          <el-table-column prop="name" label=""></el-table-column>
          <el-table-column prop="period" label="Chosen Period"></el-table-column>
          <el-table-column prop="limit" label="Tariff Plan Limit"></el-table-column>
        </el-table>
      </el-col>
    </el-row>

    <el-row v-if="showFunctionDetails">
      <el-col :span="24">
        <el-table
          :data="functions"
          show-summary
          sum-text="Summary"
          style="width: 100%">
          <el-table-column
            prop="name"
            label="Function Name"
            sortable>
          </el-table-column>
          <el-table-column
            prop="memory"
            label="RAM, MB"
            sortable
            show-overflow-tooltip>
          </el-table-column>
          <el-table-column
            property="called"
            label="Requests"
            sortable
            show-overflow-tooltip>
          </el-table-column>
          <el-table-column
            property="time"
            label="Execution Time, s"
            sortable
            show-overflow-tooltip>
          </el-table-column>
          <el-table-column
            property="gbs"
            label="GB-s"
            sortable
            show-overflow-tooltip>
          </el-table-column>
          <el-table-column
            property="traffic"
            label="Outbound traffic, MB"
            sortable
            show-overflow-tooltip>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>

    <div class="actions-block">
      <el-button size="small" type="text" v-if="!showFunctionDetails" @click="showFunctionDetails = true">Show Function Details</el-button>
      <el-button size="small" type="text" v-if="showFunctionDetails" @click="showFunctionDetails = false">Hide Function Details</el-button>
    </div>

    <hr>
    <el-button type="primary">Upgrade Plan</el-button>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loading: true,
      showFunctionDetails: false,

      periods: 0,
      periodFrom: null,
      periodTill: null,

      funcStats: [],
      functions: []
    }
  },

  created () {
    this.$store.dispatch('setStatsActiveTab', 'func')
    this.fetchFunctionStats()
    this.fetchStats()
  },

  watch: {
    'periods': function () {
      this.fetchFunctionStats()
      this.fetchStats()
    }
  },

  methods: {
    fetchStats () {
      this.loading = true
      this.$store.dispatch('getStats', { periods: this.periods }).then(response => {
        this.funcStats = [
          { name: 'Requests', period: response.data.stats[0].called, limit: 'no limit' },
          { name: 'GB-s', period: response.data.stats[0].gbs.toLocaleString(undefined, { minimumFractionDigits: 6 }), limit: '40.000' },
          { name: 'Outbound Traffic, MB', period: (response.data.stats[0].bytesout / 1048576).toLocaleString(undefined, { minimumFractionDigits: 6 }), limit: '5.000' }
        ]

        this.periodFrom = response.data.stats[0].from ? response.data.stats[0].from : this.$store.state.auth.user.created
        this.periodTill = response.data.stats[0].till ? response.data.stats[0].till : new Date()
      }).finally(() => {
        this.loading = false
      })
    },

    fetchFunctionStats () {
      this.loading = true
      this.$store.dispatch('fetchFunctions', {
        project: this.$store.getters.project,
        periods: this.periods
      }).then(functions => {
        let data = []

        functions.forEach((item) => {
          let called = 0
          let time = 0
          let bytesout = 0

          if (item.stats !== null) {
            item.stats.forEach((stats) => {
              called += stats.called
              time += stats.time
              bytesout += stats.bytesout
            })
          }

          data.push({
            name: item.name,
            memory: item.size.memory,
            called: called,
            time: (time / 1000000000).toLocaleString(undefined, { minimumFractionDigits: 6 }),
            gbs: (item.size.memory / 1000) * (time / 1024),
            traffic: (bytesout / 1048576).toLocaleString(undefined, { minimumFractionDigits: 6 })
          })
        })

        this.functions = data
      }).finally(() => {
        this.loading = false
      })
    }
  }
}
</script>

<style lang="scss" scoped>
  .timeline-form {
    .period {
      padding: 0 17px;
    }
  }

  .actions-block {
    margin-top: 10px;
  }
</style>
