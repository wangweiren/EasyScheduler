<template>
  <div class="datasource-model">
    <div class="select-listpp">
      <x-select v-model="type"
                style="width: 160px;"
                @on-change="_handleTypeChanged"
                :disabled="isDetails">
        <x-option
                v-for="city in typeList"
                :key="city.code"
                :value="city.code"
                :label="city.code">
        </x-option>
      </x-select>
      <x-select :placeholder="$t('请选择数据源')"
                v-model="datasource"
                style="width: 288px;"
                :disabled="isDetails">
        <x-option
                v-for="city in datasourceList"
                :key="city.id"
                :value="city"
                :label="city.code">
        </x-option>
      </x-select>
    </div>
  </div>
</template>
<script>
  import _ from 'lodash'
  import i18n from '@/module/i18n'
  import disabledState from '@/module/mixin/disabledState'

  export default {
    name: 'datasource',
    data () {
      return {
        // Data source type
        type: '',
        // Data source type(List)
        typeList: [],
        // data source
        datasource: {},
        // data source(List)
        datasourceList: []
      }
    },
    mixins: [disabledState],
    props: {
      data: Object,
      supportType: Array
    },
    methods: {
      /**
       * Verify data source
       */
      _verifDatasource () {
        if (!this.datasource) {
          this.$message.warning(`${i18n.$t('请选择数据源')}`)
          return false
        }
        this.$emit('on-dsData', {
          type: this.type,
          datasource: this.datasource.id
        })
        return true
      },
      /**
       * Get the corresponding datasource data according to type
       */
      _getDatasourceData () {
        return new Promise((resolve, reject) => {
          this.store.dispatch('dag/getDatasourceList', this.type).then(res => {
            this.datasourceList = _.map(res.data, v => {
              return {
                id: v.id,
                code: v.name,
                disabled: false
              }
            })
            resolve()
          })
        })
      },
      /**
       * Brush type
       */
      _handleTypeChanged ({ value }) {
        this.type = value
        this._getDatasourceData().then(res => {
          this.datasource = this.datasourceList.length && this.datasourceList[0] || {}
          this.$emit('on-dsData', {
            type: this.type,
            datasource: this.datasource.id
          })
        })
      }
    },
    watch: {},
    created () {
      let supportType = this.supportType || []
      this.typeList = _.cloneDeep(this.store.state.dag.dsTypeListS)
      // Have a specified data source
      if (supportType.length) {
        let is = (type) => {
          return !!_.filter(supportType, v => v === type).length
        }
        this.typeList = _.filter(this.typeList, v => is(v.code))
      }

      this.type = _.cloneDeep(this.data.type) || this.typeList[0].code
      // init data
      this._getDatasourceData().then(res => {
        if (_.isEmpty(this.data)) {
          this.$nextTick(() => {
            this.datasource = this.datasourceList[0]
          })
        } else {
          this.$nextTick(() => {
            this.datasource = _.filter(this.datasourceList, v => v.id === this.data.datasource)[0]
          })
        }
        this.$emit('on-dsData', {
          type: this.type
        })
      })
    },
    mounted () {

    },
    components: { }
  }
</script>