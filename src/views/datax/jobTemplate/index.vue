<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.jobDesc" placeholder="任务描述" style="width: 200px;" class="filter-item" />
      <el-input v-model="listQuery.author" placeholder="负责人" style="width: 200px;" class="filter-item" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="fetchData">
        搜索
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        添加
      </el-button>
      <!-- <el-checkbox v-model="showReviewer" class="filter-item" style="margin-left:15px;" @change="tableKey=tableKey+1">
        reviewer
      </el-checkbox> -->
    </div>
    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="任务ID" width="80">
        <template slot-scope="scope">{{ scope.row.id }}</template>
      </el-table-column>
      <el-table-column label="任务描述" align="center">
        <template slot-scope="scope">{{ scope.row.jobDesc }}</template>
      </el-table-column>
      <el-table-column label="所属项目" align="center">
        <template slot-scope="scope">{{ scope.row.jobProject }}</template>
      </el-table-column>
      <el-table-column label="Cron" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.jobCron }}</span>
        </template>
      </el-table-column>
      <el-table-column label="路由策略" align="center">
        <template slot-scope="scope"> {{ routeStrategies.find(t => t.value === scope.row.executorRouteStrategy).label }}</template>
      </el-table-column>
      <el-table-column label="负责人" align="center">
        <template slot-scope="scope">{{ scope.row.author }}</template>
      </el-table-column>
      <el-table-column label="注册节点" align="center">
        <template slot-scope="scope">
          <el-popover
            placement="bottom"
            width="500"
            @show="loadById(scope.row)"
          >
            <el-table :data="registerNode">
              <el-table-column width="150" property="title" label="执行器名称" />
              <el-table-column width="150" property="appName" label="appName" />
              <el-table-column width="150" property="registryList" label="机器地址" />
            </el-table>
            <el-button slot="reference">查看</el-button>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column label="下次触发时间" align="center">
        <template slot-scope="scope">
          <el-popover
            placement="bottom"
            width="300"
            @show="nextTriggerTime(scope.row)"
          >
            <h5 v-html="triggerNextTimes" />
            <el-button slot="reference">查看</el-button>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handlerUpdate(row)">
            编辑
          </el-button>
          <el-button size="mini" type="danger" @click="handlerDelete(row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination v-show="total>0" :total="total" :page.sync="listQuery.current" :limit.sync="listQuery.size" @pagination="fetchData" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" width="1000px">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="110px">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="执行器" prop="jobGroup">
              <el-select v-model="temp.jobGroup" placeholder="请选择执行器">
                <el-option v-for="item in executorList" :key="item.id" :label="item.title" :value="item.id" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="任务描述" prop="jobDesc">
              <el-input v-model="temp.jobDesc" size="medium" placeholder="请输入任务描述" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="路由策略" prop="executorRouteStrategy">
              <el-select v-model="temp.executorRouteStrategy" placeholder="请选择路由策略">
                <el-option v-for="item in routeStrategies" :key="item.value" :label="item.label" :value="item.value" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Cron" prop="jobCron">
              <el-input v-model="temp.jobCron" placeholder="请输入Cron表达式" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="阻塞处理" prop="executorBlockStrategy">
              <el-select v-model="temp.executorBlockStrategy" placeholder="请选择阻塞处理策略">
                <el-option v-for="item in blockStrategies" :key="item.value" :label="item.label" :value="item.value" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="负责人" prop="author">
              <el-input v-model="temp.author" placeholder="请输入负责人" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="任务类型" prop="glueType">
              <el-select v-model="temp.glueType" placeholder="任务脚本类型">
                <el-option v-for="item in glueTypes" :key="item.value" :label="item.label" :value="item.value" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="报警邮件">
              <el-input v-model="temp.alarmEmail" placeholder="请输入报警邮件，多个用逗号分隔" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="超时时间(分钟)">
              <el-input-number v-model="temp.executorTimeout" :min="0" :max="20" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="子任务ID">
              <el-select v-model="temp.childJobId" multiple placeholder="子任务ID" value-key="id">
                <el-option v-for="item in JobIdList" :key="item.id" :label="item.jobDesc" :value="item" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="失败重试次数">
              <el-input-number v-model="temp.executorFailRetryCount" :min="0" :max="20" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="所属项目" prop="jobProject">
              <el-input v-model="temp.jobProject" size="medium" placeholder="请输入所属项目" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="辅助参数" prop="incrementType">
              <el-select v-model="temp.incrementType" placeholder="请选择参数类型" value="">
                <el-option v-for="item in incrementTypes" :key="item.value" :label="item.label" :value="item.value" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row v-if="temp.incrementType === 2" :gutter="20">
          <el-col :span="12">
            <el-form-item label="增量开始时间" prop="incStartTime">
              <el-date-picker
                v-model="temp.incStartTime"
                type="datetime"
                placeholder="首次增量使用"
                format="yyyy-MM-dd HH:mm:ss"
                default-time="00:00:00"
                style="width: 57%"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="增量时间字段" prop="replaceParam">
              <el-input v-model="temp.replaceParam" placeholder="-DlastTime='%s' -DcurrentTime='%s'" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="增量时间格式" prop="replaceParamType">
              <el-select v-model="temp.replaceParamType" placeholder="增量时间格式">
                <el-option
                  v-for="item in replaceFormatTypes"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row v-if="temp.incrementType === 1" :gutter="20">
          <el-col :span="12">
            <el-form-item label="增量主键开始ID" prop="incStartId">
              <el-input v-model="temp.incStartId" placeholder="增量首次使用" style="width: 57%" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="ID增量参数" prop="replaceParam">
              <el-input v-model="temp.replaceParam" placeholder="-DstartId='%s' -DendId='%s'" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="主键字段名" prop="primaryKey">
              <el-input v-model="temp.primaryKey" placeholder="请填写主键字段名" style="width: 57%" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row v-if="temp.incrementType === 3" :gutter="20">
          <el-col :span="12">
            <el-form-item label="分区字段" prop="partitionField">
              <el-input v-model="partitionField" placeholder="请输入分区字段" style="width: 56%" />
            </el-form-item>
          </el-col>
          <el-col :span="7">
            <el-form-item label="分区时间">
              <el-select v-model="timeFormatType" placeholder="分区时间格式">
                <el-option
                  v-for="item in timeFormatTypes"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="5">
            <el-input-number v-model="timeOffset" :min="-20" :max="0" style="width: 65%" />
          </el-col>
        </el-row>
        <el-row :gutter="20">
          <el-col :span="24">
            <el-form-item label="JVM启动参数">
              <el-input
                v-model="temp.jvmParam"
                placeholder="-Xms1024m -Xmx1024m -XX:+HeapDumpOnOutOfMemoryError"
              />
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          确认
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import * as executor from '@/api/datax-executor'
import * as job from '@/api/datax-job-template'
import waves from '@/directive/waves' // waves directive
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

export default {
  name: 'JobTemplate',
  components: { Pagination },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    const validateIncParam = (rule, value, callback) => {
      if (!value) {
        callback(new Error('Increment parameters is required'))
      }
      callback()
    }
    const validatePartitionParam = (rule, value, callback) => {
      if (!this.partitionField) {
        callback(new Error('Partition parameters is required'))
      }
      callback()
    }
    return {
      list: null,
      listLoading: true,
      total: 0,
      listQuery: {
        current: 1,
        size: 10,
        jobGroup: 0,
        triggerStatus: -1,
        jobDesc: '',
        executorHandler: '',
        author: ''
      },
      dialogPluginVisible: false,
      pluginData: [],
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      rules: {
        jobGroup: [{ required: true, message: 'jobGroup is required', trigger: 'change' }],
        executorRouteStrategy: [{ required: true, message: 'executorRouteStrategy is required', trigger: 'change' }],
        executorBlockStrategy: [{ required: true, message: 'executorBlockStrategy is required', trigger: 'change' }],
        jobDesc: [{ required: true, message: 'jobDesc is required', trigger: 'blur' }],
        jobProject: [{ required: true, message: 'jobProject is required', trigger: 'blur' }],
        jobCron: [{ required: true, message: 'jobCron is required', trigger: 'blur' }],
        author: [{ required: true, message: 'author is required', trigger: 'blur' }],
        incStartId: [{ trigger: 'blur', validator: validateIncParam }],
        replaceParam: [{ trigger: 'blur', validator: validateIncParam }],
        primaryKey: [{ trigger: 'blur', validator: validateIncParam }],
        incStartTime: [{ trigger: 'change', validator: validateIncParam }],
        replaceParamType: [{ trigger: 'change', validator: validateIncParam }],
        partitionField: [{ trigger: 'blur', validator: validatePartitionParam }]
      },
      temp: {
        id: undefined,
        jobGroup: '',
        jobCron: '',
        jobDesc: '',
        executorRouteStrategy: 'RANDOM',
        executorBlockStrategy: 'DISCARD_LATER',
        childJobId: '',
        executorFailRetryCount: '',
        alarmEmail: '',
        executorTimeout: '',
        author: '',
        jobConfigId: '',
        executorHandler: 'executorJobHandler',
        glueType: 'BEAN',
        executorParam: '',
        replaceParam: '',
        jvmParam: '',
        incStartTime: '',
        partitionInfo: '',
        incrementType: 0,
        incStartId: '',
        primaryKey: ''
      },
      resetTemp() {
        this.temp = this.$options.data().temp
        this.timeOffset = 0
        this.timeFormatType = 'yyyy-MM-dd'
        this.partitionField = ''
      },
      executorList: '',
      JobIdList: '',
      blockStrategies: [
        { value: 'SERIAL_EXECUTION', label: '单机串行' },
        { value: 'DISCARD_LATER', label: '丢弃后续调度' },
        { value: 'COVER_EARLY', label: '覆盖之前调度' }
      ],
      routeStrategies: [
        { value: 'FIRST', label: '第一个' },
        { value: 'LAST', label: '最后一个' },
        { value: 'ROUND', label: '轮询' },
        { value: 'RANDOM', label: '随机' },
        { value: 'CONSISTENT_HASH', label: '一致性HASH' },
        { value: 'LEAST_FREQUENTLY_USED', label: '最不经常使用' },
        { value: 'LEAST_RECENTLY_USED', label: '最近最久未使用' },
        { value: 'FAILOVER', label: '故障转移' },
        { value: 'BUSYOVER', label: '忙碌转移' }
        // { value: 'SHARDING_BROADCAST', label: '分片广播' }
      ],
      replaceFormatTypes: [
        { value: 'yyyy/MM/dd', label: 'yyyy/MM/dd' },
        { value: 'HH:mm:ss', label: 'HH:mm:ss' },
        { value: 'yyyy/MM/dd HH:mm:ss', label: 'yyyy/MM/dd HH:mm:ss' },
        { value: 'Timestamp', label: '时间戳' }
      ],
      glueTypes: [
        { value: 'BEAN', label: 'DataX任务' }
      ],
      incrementTypes: [
        { value: 0, label: '无' },
        { value: 1, label: '主键自增' },
        { value: 2, label: '时间自增' },
        { value: 3, label: 'HIVE分区' }
      ],
      triggerNextTimes: '',
      registerNode: [],
      timeOffset: 0,
      timeFormatType: 'yyyy-MM-dd',
      partitionField: '',
      timeFormatTypes: [
        { value: 'yyyy-MM-dd', label: 'yyyy-MM-dd' },
        { value: 'yyyyMMdd', label: 'yyyyMMdd' },
        { value: 'yyyy/MM/dd', label: 'yyyy/MM/dd' }
      ]
    }
  },
  created() {
    this.fetchData()
    this.getExecutor()
    this.getJobIdList()
  },

  methods: {
    getExecutor() {
      job.getExecutorList().then(response => {
        const { content } = response
        this.executorList = content
      })
    },
    getJobIdList() {
      job.getJobIdList().then(response => {
        const { content } = response
        this.JobIdList = content
      })
    },
    fetchData() {
      this.listLoading = true
      job.getList(this.listQuery).then(response => {
        const { content } = response
        this.total = content.recordsTotal
        this.list = content.data
        this.listLoading = false
      })
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.temp.jobGroup = this.executorList[0]['id']
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.temp.childJobId) {
            const childJobs = []
            for (const i in this.temp.childJobId) {
              childJobs.push(this.temp.childJobId[i].id)
            }
            this.temp.childJobId = childJobs.toString()
          }
          if (this.partitionField) this.temp.partitionInfo = this.partitionField + ',' + this.timeOffset + ',' + this.timeFormatType
          job.createJob(this.temp).then(() => {
            this.fetchData()
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handlerUpdate(row) {
      this.resetTemp()
      this.temp = Object.assign({}, row) // copy obj
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      const arrchildSet = []
      const arrJobIdList = []
      if (this.JobIdList) {
        for (const n in this.JobIdList) {
          if (this.JobIdList[n].id !== this.temp.id) {
            arrJobIdList.push(this.JobIdList[n])
          }
        }
        this.JobIdList = arrJobIdList
      }

      if (this.temp.childJobId) {
        const arrString = this.temp.childJobId.split(',')
        for (const i in arrString) {
          for (const n in this.JobIdList) {
            if (this.JobIdList[n].id === arrString[i]) {
              arrchildSet.push(this.JobIdList[n])
            }
          }
        }
        this.temp.childJobId = arrchildSet
      }

      if (this.temp.partitionInfo) {
        const partition = this.temp.partitionInfo.split(',')
        this.partitionField = partition[0]
        this.timeOffset = partition[1]
        this.timeFormatType = partition[2]
      }
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.temp.childJobId) {
            const childJobs = []
            for (const i in this.temp.childJobId) {
              childJobs.push(this.temp.childJobId[i].id)
            }
            this.temp.childJobId = childJobs.toString()
          }
          if (this.partitionField) this.temp.partitionInfo = this.partitionField + ',' + this.timeOffset + ',' + this.timeFormatType
          job.updateJob(this.temp).then(() => {
            this.fetchData()
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handlerDelete(row) {
      this.$confirm('确定删除吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        job.removeJob(row.id).then(response => {
          this.fetchData()
          this.$notify({
            title: 'Success',
            message: 'Delete Successfully',
            type: 'success',
            duration: 2000
          })
        })
      })

      // const index = this.list.indexOf(row)
    },
    nextTriggerTime(row) {
      job.nextTriggerTime(row.jobCron).then(response => {
        const { content } = response
        this.triggerNextTimes = content.join('<br>')
      })
    },
    loadById(row) {
      executor.loadById(row.jobGroup).then(response => {
        this.registerNode = []
        const { content } = response
        this.registerNode.push(content)
      })
    }
  }
}
</script>

<style>
  .el-dropdown + .el-dropdown {
    margin-left: 15px;
  }
</style>
