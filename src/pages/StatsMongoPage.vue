<!-- 

© 2018 SwiftyCloud OÜ. All rights reserved.
Contact: info@swifty.cloud

-->

<template>
  <div v-loading="loading">
    <p>Usage stats for MongoDB</p>

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

    <el-row>
      <el-col :span="24">
        <el-table :data="mongoStats" style="width: 100%">
          <el-table-column prop="name" label=""></el-table-column>
          <el-table-column prop="period" label="All Period"></el-table-column>
          <el-table-column prop="limit" label="Tariff Plan Limit"></el-table-column>
        </el-table>
      </el-col>
    </el-row>

    <div class="actions-block">
      <el-button type="primary">Upgrade Plan</el-button>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loading: false,
      mongoStats: [
        { name: 'Used Storage, MB', period: 0, limit: null },
        { name: 'Number of databases', period: 0, limit: null }
      ],

      periods: 0,
      periodFrom: null,
      periodTill: null
    }
  },

  created () {
    this.$store.dispatch('setStatsActiveTab', 'mongodb')
    this.fetchStats()
  },

  methods: {
    fetchStats () {
      this.loading = true
      this.$store.dispatch('getStats', { periods: this.periods }).then(response => {
        if ('mongo' in response.data.mware) {
          this.mongoStats[0].period = response.data.mware.mongo.disk_usage / 1024
          this.mongoStats[1].period = response.data.mware.mongo.count
        }

        this.periodFrom = response.data.stats[0].from ? response.data.stats[0].from : this.$store.state.auth.user.created
        this.periodTill = response.data.stats[0].till ? response.data.stats[0].till : new Date()
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

  .text-red {
    color: #d0021b;
  }

  .actions-block {
    margin-top: 40px;
  }
</style>
