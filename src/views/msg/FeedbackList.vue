<template>
  <div>
    <el-button v-if="showclearbtn" @click="clearItems" id="clearFeedbackButton" round>处理完毕</el-button>
    <el-switch v-if="showclearbtn" v-model="showMsgTypeSwitch" class="feedbackSwitch"
               active-color="#13ce66" active-text="超时信息" active-value="timeout"
               inactive-color="#ff4949" inactive-text="异常状态" inactive-value="exception">
    </el-switch>
    <el-table :data="tableData">
      <el-table-column min-width="35">
      </el-table-column>
      <el-table-column
        prop="id"
        label="序号"
        min-width="80"
        sortable>
      </el-table-column>
      <el-table-column
        prop="currentDate"
        label="日期"
        min-width="100"
        sortable>
      </el-table-column>
      <!--width="180"-->
      <el-table-column
        prop="currentTime"
        label="时间"
        min-width="100"
        sortable>
      </el-table-column>
      <!--width="180"-->
      <el-table-column
        prop="groupId"
        label="组号"
        min-width="80"
        sortable>
      </el-table-column>
      <!--width="180"-->
      <el-table-column
        prop="deviceId"
        label="设备号"
        min-width="90"
        sortable>
      </el-table-column>
      <!--width="200"-->
      <el-table-column
        prop="type"
        label="消息类型"
        min-width="150"
        :filters="[{
          text: '消息重发次数超出限制',value: '消息重发次数超出限制'}, {
          text: '通道连接已断开', value: '通道连接已断开' }, {
          text: '设备工作状态异常', value: '设备工作状态异常'}, {
           text: '设备组无响应', value: '设备组无响应'}]"
        :filter-method="filterTag"
        filter-placement="bottom-end">
        <template slot-scope="scope">
          <el-tag
            :type="scope.row.tag"
            close-transition>{{scope.row.type}}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="extra"
        label="描述"
        min-width="180"
        sortable>
      </el-table-column>
      <el-table-column min-width="1">
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
  import { getFeedbackItemsException, getFeedbackItemsTimeout, clearFeedbackItems } from '@/api/feedbackMsg'
  import { setTimer, touchError } from '@/utils/timer'

  export default {
    data: function() {
      return {
        // 消息类型设定
        showMsgTypeSwitch: 'exception',
        showclearbtn: true,
        tableData: null
      }
    },
    name: 'MsgFeedbacklist',
    methods: {
      getData() {
        const vm = this
        let call = null
        switch (this.showMsgTypeSwitch) {
          case 'exception':
            call = getFeedbackItemsException()
            break
          case 'timeout':
            call = getFeedbackItemsTimeout()
            break
        }

        if (call !== null) {
          call.then(response => {
            vm.tableData = vm.calculateTableData(response.data)
          }).catch(error => {
            touchError(this, this.getData, error)
          })
        }
      },
      filterTag(value, row) {
        return row.type === value
      },
      calculateTableData(rawItems) {
        const tableData = []
        for (let i = 0; i < rawItems.length; i++) {
          const item = rawItems[i]
          let extra = ''
          switch (item.type) {
            case 'RESEND_OUT_BOUND':
              extra = '基础消息「' + item.baseMsg.msgCodec.detail + '」'
              break
            case 'WORK_STATUS_EXCEPTION':
              extra = '异常类型「' + item.baseMsg.statusDescription + '」'
          }
          tableData.push({
            id: i + 1,
            currentDate: item.currentDate,
            currentTime: item.currentTime,
            groupId: item.groupId,
            deviceId: item.deviceId,
            type: item.description,
            extra: extra
          })
        }
        return tableData
      },
      clearItems() {
        clearFeedbackItems()
      }
    },
    watch: {
      showMsgTypeSwitch: function(val, oldVal) {
        console.log('状态类型转换', oldVal + '->' + val)
        setTimer(this.getData, 3000)
      }
    },
    mounted: function() {
      if (this.$route.query.showclearbtn === 'false') {
        this.showclearbtn = false
      }
      if (this.$route.query.type) {
        this.showMsgTypeSwitch = this.$route.query.type
        console.log('type被设置：', this.showMsgTypeSwitch)
      }
      setTimer(this.getData, 3000)
    }
  }
</script>

<style scoped>

  #clearFeedbackButton {
    z-index: 100;
    position: fixed;
    right: 100px;
    top: 5px;
  }

  .feedbackSwitch {
    z-index: 100;
    position: fixed;
    right: 250px;
    top: 16px;
  }
</style>
