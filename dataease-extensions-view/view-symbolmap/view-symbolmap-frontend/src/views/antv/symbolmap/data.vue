<template>
  <div>
    <!-- 经度 -->
    <el-row class="padding-lr">
      <span style="width: 80px;text-align: right;">
        <span>{{ $t('plugin_view_symbol_map.longitude')}}</span>/<span >{{ $t('chart.dimension') }}</span>
      </span>
      <draggable v-model="longitudes" group="drag" animation="300" :move="onMove" class="drag-block-style"
        @add="addLongitudes" @update="calcData(true)">
        <transition-group class="draggable-group">
          <location-x-item v-for="(item,index) in longitudes" :key="item.id" :param="param" :index="index" :item="item" :dimension-data="dimensionData" :quota-data="quotaData"
            @onLocationXItemRemove="locationItemRemove" @onNameEdit="showRename" />
        </transition-group>
      </draggable>
      <div v-if="!longitudes || longitudes.length === 0" class="drag-placeholder-style">
        <span class="drag-placeholder-style-span">{{ $t('chart.placeholder_field') }}</span>
      </div>
    </el-row>

    <!-- 纬度 -->
    <el-row class="padding-lr" style="margin-top: 6px;">
      <span style="width: 80px;text-align: right;">
        <span>{{ $t('plugin_view_symbol_map.latitude') }}</span>/<span >{{ $t('chart.dimension') }}</span>
      </span>
      <draggable v-model="latitudes" group="drag" animation="300" :move="onMove" class="drag-block-style"
        @add="addLatitudes" @update="calcData(true)">
        <transition-group class="draggable-group">
          <location-y-item v-for="(item,index) in latitudes" :key="item.id" :param="param" :index="index" :item="item" :chart="chart" :dimension-data="dimensionData"
            :quota-data="quotaData" @onLocationYItemRemove="locationItemRemove" @onNameEdit="showRename" />
        </transition-group>
      </draggable>
      <div v-if="!latitudes || latitudes.length === 0" class="drag-placeholder-style">
        <span class="drag-placeholder-style-span">{{ $t('chart.placeholder_field') }}</span>
      </div>
    </el-row>

    <!-- 符号大小 -->
    <el-row class="padding-lr" style="margin-top: 6px;">
      <span style="width: 80px;text-align: right;">
        <span>{{ $t('plugin_view_symbol_map.mark_size') }}</span>/<span>{{ $t('chart.quota') }}</span>
        <el-tooltip class="item" effect="dark" placement="bottom">
            <div slot="content">
            该指标生效时，样式大小中的气泡大小属性将失效
            </div>
            <i class="el-icon-info" style="cursor: pointer;color: #606266;" />
        </el-tooltip>
      </span>
      <draggable v-model="view.yaxis" group="drag" animation="300" :move="onMove" class="drag-block-style"
        @add="addYaxis" @update="calcData(true)">
        <transition-group class="draggable-group">
          <quota-item v-for="(item,index) in view.yaxis" :key="item.id" :param="param" :index="index" :item="item"
            :chart="chart" :dimension-data="dimensionData" :quota-data="quotaData" @onQuotaItemChange="quotaItemChange"
            @onQuotaItemRemove="quotaItemRemove" @editItemFilter="showQuotaEditFilter" @onNameEdit="showRename"
            @editItemCompare="showQuotaEditCompare" />
        </transition-group>
      </draggable>
      <div v-if="!view.yaxis || view.yaxis.length === 0" class="drag-placeholder-style">
        <span class="drag-placeholder-style-span">{{ $t('chart.placeholder_field') }}</span>
      </div>
    </el-row>

    <!-- 结果过滤器 -->
    <el-row class="padding-lr" style="margin-top: 6px;">
      <span>{{ $t('chart.result_filter') }}</span>

      <draggable v-model="view.customFilter" group="drag" animation="300" :move="onMove" class="theme-item-class"
        style="padding:2px 0 0 0;width:100%;min-height: 32px;border-radius: 4px;border: 1px solid #DCDFE6;overflow-x: auto;display: flex;align-items: center;background-color: white;"
        @add="addCustomFilter" @update="calcData(true)">
        <transition-group class="draggable-group">
          <filter-item v-for="(item,index) in view.customFilter" :key="item.id" :param="param" :index="index"
            :item="item" :dimension-data="dimensionData" :quota-data="quotaData" @onFilterItemRemove="filterItemRemove"
            @editItemFilter="showEditFilter" />
        </transition-group>
      </draggable>
      <div v-if="!view.customFilter || view.customFilter.length === 0" class="drag-placeholder-style">
        <span class="drag-placeholder-style-span">{{ $t('chart.placeholder_field') }}</span>
      </div>
    </el-row>

  </div>
</template>

<script>
  import LocationXItem from '@/components/views/LocationXItem'
  import LocationYItem from '@/components/views/LocationYItem'
  import QuotaItem from '@/components/views/QuotaItem'
  import FilterItem from '@/components/views/FilterItem'
  import messages from '@/de-base/lang/messages'

  export default {
    props: {

      obj: {
        type: Object,
        default: () => {}
      }
    },
    components: {
      LocationXItem,
      LocationYItem,
      QuotaItem,
      FilterItem
    },
    data() {
      return {
        widgets: [],
        places: [],
        moveId: -1,
        showDrill: false,
        options: [{
            value: 'point',
            label: '点'
          },
          {
            value: 'line',
            label: '线'
          }
        ],
        longitudes: [],
        latitudes: []
      }
    },
    computed: {
      param() {
        return this.obj.param
      },
      view() {
        return this.obj.view
      },
      chart() {
        return this.obj.chart
      },
      dimensionData() {
        return this.obj.dimensionData
      },
      quotaData() {
        return this.obj.quotaData
      }
    },
    created() {
      this.longitudes = this.view.xaxis && this.view.xaxis.length && [this.view.xaxis[0]] || []
      this.latitudes = this.view.xaxis && this.view.xaxis.length > 1 && [this.view.xaxis[1]] || []
      this.$emit('on-add-languanges', messages)
    },
    watch: {
      longitudes(val) {
          this.view.xaxis = [...this.longitudes, ...this.latitudes]   
      },
      latitudes(val) {
          this.view.xaxis = [...this.longitudes, ...this.latitudes]   
      }
    },
    methods: {
      executeAxios(url, type, data, callBack) {
        const param = {
          url: url,
          type: type,
          data: data,
          callBack: callBack
        }
        this.$emit('execute-axios', param)
        if (process.env.NODE_ENV === 'development') {
          execute(param).then(res => {
            if (param.callBack) {
              param.callBack(res)
            }
          }).catch(e => {
            if (param.error) {
              param.error(e)
            }
          })
        }
      },


      onMove(e, originalEvent) {
        this.moveId = e.draggedContext.element.id
        return true
      },

      addLongitudes(e) {
        this.dragCheckType(this.longitudes, 'd')
        this.dragMoveDuplicate(this.longitudes, e)
        if (this.longitudes.length > 1) {
          this.longitudes = [this.longitudes[0]]
        }
        this.calcData(true)
      },
      addLatitudes(e) {
        this.dragCheckType(this.latitudes, 'd')
        this.dragMoveDuplicate(this.latitudes, e)
        if (this.latitudes.length > 1) {
          this.latitudes = [this.latitudes[0]]
        }
        this.calcData(true)
      },
      /* addLabelItems(e) {
        this.dragMoveDuplicate(this.labelItems, e)
        if (this.labelItems.length > 1) {
          this.labelItems = [this.labelItems[0]]
        }
        this.calcData(true)
      },
      addTooltipItems(e) {
        this.dragMoveDuplicate(this.tooltipItems, e)
        if (this.tooltipItems.length > 1) {
          this.tooltipItems = [this.tooltipItems[0]]
        }
        this.calcData(true)
      }, */
      addYaxis(e) {
        this.dragCheckType(this.view.yaxis, 'q')
        this.dragMoveDuplicate(this.view.yaxis, e)
        if (this.view.yaxis.length > 1) {
          this.view.yaxis = [this.view.yaxis[0]]
        }
        this.calcData(true)
      },
      calcData(cache) {
        this.view.xaxis = [...this.longitudes, ...this.latitudes]       
       
        this.$emit('plugin-call-back', {
          eventName: 'calc-data',
          eventParam: {
            cache
          }
        })
      },

      locationItemRemove(item) {
        if (item.removeType === 'locationX') {
          this.longitudes.splice(item.index, 1)
        } else if (item.removeType === 'locationY') {
          this.latitudes.splice(item.index, 1)
        }
        this.calcData(true)
      },
      showDimensionEditFilter(item) {

        this.$emit('plugin-call-back', {
          eventName: 'show-dimension-edit-filter',
          eventParam: item
        })
      },
      showRename(item) {

        this.$emit('plugin-call-back', {
          eventName: 'show-rename',
          eventParam: item
        })
      },

      quotaItemChange(item) {
        this.calcData(true)
      },
      quotaItemRemove(item) {
        if (item.removeType === 'quota') {
          this.view.yaxis.splice(item.index, 1)
        } else if (item.removeType === 'quotaExt') {
          this.view.yaxisExt.splice(item.index, 1)
        }
        this.calcData(true)
      },
      showQuotaEditFilter(item) {

        this.$emit('plugin-call-back', {
          eventName: 'show-quota-edit-filter',
          eventParam: item
        })
      },
      showQuotaEditCompare(item) {

        this.$emit('plugin-call-back', {
          eventName: 'show-quota-edit-compare',
          eventParam: item
        })
      },
      dragCheckType(list, type) {
        if (list && list.length > 0) {
          for (let i = 0; i < list.length; i++) {
            if (list[i].groupType !== type) {
              list.splice(i, 1)
            }
          }
        }
      },
      dragMoveDuplicate(list, e) {
        const that = this
        const dup = list.filter(function (m) {
          return m.id === that.moveId
        })
        if (dup && dup.length > 1) {
          list.splice(e.newDraggableIndex, 1)
        }
      },
      addCustomFilter(e) {
        // 记录数等自动生成字段不做为过滤条件
        if (this.view.customFilter && this.view.customFilter.length > 0) {
          for (let i = 0; i < this.view.customFilter.length; i++) {
            if (this.view.customFilter[i].id === 'count') {
              this.view.customFilter.splice(i, 1)
            }
          }
        }
        this.dragMoveDuplicate(this.view.customFilter, e)
        this.calcData(true)
      },
      filterItemRemove(item) {
        this.view.customFilter.splice(item.index, 1)
        this.calcData(true)
      },
      showEditFilter(item) {
        this.$emit('plugin-call-back', {
          eventName: 'show-edit-filter',
          eventParam: item
        })
      }
    }
  }

</script>

<style lang="scss" scoped>
  .padding-lr {
    padding: 0 6px;
  }

  .itxst {
    margin: 10px;
    text-align: left;
  }

  .col {
    width: 40%;
    flex: 1;
    padding: 10px;
    border: solid 1px #eee;
    border-radius: 5px;
    float: left;
  }

  .col+.col {
    margin-left: 10px;
  }

  .view-panel {
    display: flex;
    height: calc(100% - 40px);
    background-color: #f7f8fa;
  }

  .blackTheme .view-panel {
    background-color: var(--MainBG);
  }

  .drag-list {
    height: calc(100% - 26px);
    overflow: auto;
  }

  .item-dimension {
    padding: 2px 10px;
    margin: 2px 2px 0 2px;
    border: solid 1px #eee;
    text-align: left;
    color: #606266;
    /*background-color: rgba(35,46,64,.05);*/
    background-color: white;
    display: block;
    word-break: break-all;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .blackTheme .item-dimension {
    border: solid 1px;
    border-color: var(--TableBorderColor);
    color: var(--TextPrimary);
    background-color: var(--MainBG);
  }

  .item-dimension+.item-dimension {
    margin-top: 2px;
  }

  .item-dimension:hover {
    color: #1890ff;
    background: #e8f4ff;
    border-color: #a3d3ff;
    cursor: pointer;
  }

  .blackTheme .item-dimension:hover {
    color: var(--Main);
    background: var(--ContentBG);
    cursor: pointer;
  }

  .item-quota {
    padding: 2px 10px;
    margin: 2px 2px 0 2px;
    border: solid 1px #eee;
    text-align: left;
    color: #606266;
    background-color: white;
    display: block;
    word-break: break-all;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .blackTheme .item-quota {
    border: solid 1px;
    border-color: var(--TableBorderColor);
    color: var(--TextPrimary);
    background-color: var(--MainBG);
  }

  .item-quota+.item-quota {
    margin-top: 2px;
  }

  .item-quota:hover {
    color: #67c23a;
    background: #f0f9eb;
    border-color: #b2d3a3;
    cursor: pointer;
  }

  .blackTheme .item-quota:hover {
    background: var(--ContentBG);
  }

  .el-form-item {
    margin-bottom: 0;
  }

  span {
    font-size: 12px;
  }

  .tab-header>>>.el-tabs__header {
    border-top: solid 1px #eee;
    border-right: solid 1px #eee;
  }

  .tab-header>>>.el-tabs__item {
    font-size: 12px;
    padding: 0 60px !important;
  }

  .blackTheme .tab-header>>>.el-tabs__item {
    background-color: var(--MainBG);
  }

  .tab-header>>>.el-tabs__nav-scroll {
    padding-left: 0 !important;
  }

  .tab-header>>>.el-tabs__header {
    margin: 0 !important;
  }

  .tab-header>>>.el-tabs__content {
    height: 100%;
  }

  .draggable-group {
    display: block;
    width: 100%;
    height: calc(100% - 6px);
  }

  .chart-icon {
    width: 20px;
    height: 20px;
  }

  .el-radio {
    margin: 5px;
  }

  .el-radio>>>.el-radio__label {
    padding-left: 0;
  }

  .attr-style {
    height: calc(100vh - 56px - 60px - 40px - 40px);
  }

  .blackTheme .attr-style {
    color: var(--TextPrimary);
  }

  .attr-selector {
    width: 100%;
    height: 100%;
    margin: 6px 0;
    padding: 0 4px;
    display: flex;
    align-items: center;
    background-color: white
  }

  .blackTheme .attr-selector {

    background-color: var(--MainBG)
  }

  .disabled-none-cursor {
    cursor: not-allowed;
    pointer-events: none;
  }

  .chart-class {
    height: 100%;
    padding: 10px;
  }

  .table-class {
    height: calc(100% - 20px);
  }

  .dialog-css>>>.el-dialog__title {
    font-size: 14px;
  }

  .dialog-css>>>.el-dialog__header {
    padding: 20px 20px 0;
  }

  .dialog-css>>>.el-dialog__body {
    padding: 10px 20px 20px;
  }

  .filter-btn-class {
    padding: 6px;
    border: none;
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .chart-error-class {
    text-align: center;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #ece7e7;
  }

  .blackTheme .chart-error-class {

    background-color: var(--MainBG)
  }

  .field-height {
    height: calc(50% - 20px);
    border-top: 1px solid #E6E6E6;
  }

  .blackTheme .field-height {

    border-top: 1px solid;
    border-color: var(--TableBorderColor) !important;
  }

  .padding-tab {
    padding: 0;
    height: 100%;
  }

  .tree-select-span {
    >>>div.vue-treeselect__control {
      height: 32px !important;
      font-weight: normal !important;
    }
  }

  .drag-block-style {
    padding: 2px 0 0 0;
    width: 100%;
    min-height: 32px;
    border-radius: 4px;
    border: 1px solid #DCDFE6;
    overflow-x: hidden;
    display: flex;
    align-items: center;
    background-color: white;
  }

  .blackTheme .drag-block-style {
    border: 1px solid;
    border-color: var(--TableBorderColor);
    background-color: var(--ContentBG);
  }

  .drag-placeholder-style {
    position: absolute;
    top: calc(50% - 2px);
    left: 0;
    width: 100%;
    color: #CCCCCC;
  }

  .blackTheme .drag-placeholder-style {
    color: var(--TextPrimary);
  }

  .drag-placeholder-style-span {
    padding-left: 16px;
  }

  .blackTheme .theme-border-class {
    color: var(--TextPrimary) !important;
    background-color: var(--ContentBG);
  }

  .blackTheme .padding-lr {
    border-color: var(--TableBorderColor) !important;
  }

  .blackTheme .theme-item-class {
    background-color: var(--MainBG) !important;
    border-color: var(--TableBorderColor) !important;
  }

  .icon-class {
    color: #6c6c6c;
  }

  .blackTheme .icon-class {
    color: #cccccc;
  }

  .result-count {
    width: 80px;
  }

  .radio-span>>>.el-radio__label {
    margin-left: 4px;
  }

</style>
