<template>
    <div class="worker-comments">
      <!-- 客服评价对话框 -->
      <el-dialog
        title="客服评价"
        :visible.sync="commentDialog"
        width="70%"
        @opened="fetchComments"
      >
        <el-table
          v-loading="commentLoading"
          :data="comments"
          style="width: 100%"
          max-height="500"
        >
          <el-table-column  align="center"
            prop="workerEvaluate.content"
            label="评价内容"
          ></el-table-column>
  
          <el-table-column  align="center"
            prop="user.nickname"
            label="用户"
            width="120"
          ></el-table-column>
  
          <el-table-column  align="center"
            label="评价类型"
            width="150"
            prop="workerEvaluate.type"
          >
          </el-table-column>
  
          <el-table-column  align="center"
            prop="workerEvaluate.createAt"
            label="评价时间"
            width="180"
          ></el-table-column>
  
          <el-table-column  align="center"
            label="操作"
            fixed="right"
            width="120"
          >
            <template slot-scope="scope">
              <el-popconfirm
                title="确定删除该评价吗？"
                @confirm="handleDeleteComment(scope.row.id)"
              >
                <el-button
                  slot="reference"
                  type="danger"
                  size="mini"
                  icon="el-icon-delete"
                >删除</el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
      </el-dialog>
  
      <!-- 员工列表 -->
      <el-table
        v-loading="tableLoading"
        :data="workerList"
        style="width: 100%"
      >
        <el-table-column align="center"
            prop="avatar"
            label="头像"
            width="120"
            >
            <template slot-scope="scope">
                <el-avatar :size="50" :src="scope.row.avatar"></el-avatar>
            </template>
        </el-table-column>
        <el-table-column label="员工姓名" width="140" align="center" prop="nickname">
        </el-table-column>
        <el-table-column label="部门" width="140" align="center" prop="department">
        </el-table-column>
        <el-table-column align="center"
          prop="username"
          label="工号"
          width="120"
        ></el-table-column>
  
        <el-table-column label="在职状态" width="120" align="center">
          <template slot-scope="scope">
            <el-tag :type="scope.row.entry ? 'success' : 'danger'">
              {{ scope.row.entry ? '在职' : '离职' }}
            </el-tag>
          </template>
        </el-table-column>
  
        <el-table-column align="center"
          label="联系电话"
          width="200"
        >
            <template slot-scope="scope">
                <div v-if="scope.row.phone == ''">无</div>
                <div v-else>{{ scope.row.phone }}</div>
            </template>
        </el-table-column>
  
        <el-table-column align="center"
          prop="createAt"
          label="入职时间"
          width="180"
        ></el-table-column>
  
        <el-table-column label="操作" fixed="right" align="center">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="info"
              icon="el-icon-chat-dot-round"
              @click="handleComment(scope.row)"
            >评价</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </template>
  
  <script>
  import { FindAllWorker, GetWorkerComments, DeleteComment } from '@/api/worker'
  
  export default {
    data() {
      return {
        tableLoading: false,
        commentDialog: false,
        commentLoading: false,
        workerList: [],
        comments: [],
        currentWorkerId: null,
        searchParams: {
          department: '',
          entryStatus: 'all'
        }
      }
    },
  
    mounted() {
      this.loadWorkers()
    },
  
    methods: {
      async loadWorkers() {
        try {
          this.tableLoading = true
          const res = await FindAllWorker()
          this.workerList = res.data
        } catch (error) {
          console.error('获取员工列表失败:', error)
        } finally {
          this.tableLoading = false
        }
      },
  
      handleComment(worker) {
        this.currentWorkerId = worker.id
        this.commentDialog = true
      },
  
      async fetchComments() {
        try {
          this.commentLoading = true
          console.log(this.currentWorkerId);
          const res = await GetWorkerComments(this.currentWorkerId)
          this.comments = res.data
        } catch (error) {
          console.error('获取评价失败:', error)
          this.$message.error('获取评价信息失败')
        } finally {
          this.commentLoading = false
        }
      },
  
      async handleDeleteComment(commentId) {
        try {
          await DeleteComment(commentId)
          this.$message.success('评价删除成功')
          this.fetchComments()
        } catch (error) {
          this.$message.error('评价删除失败')
        }
      }
    }
  }
  </script>
  
  <style scoped>
  .worker-comments {
    padding: 20px;
  }
  
  .worker-info {
    display: flex;
    align-items: center;
  }
  
  .worker-detail {
    margin-left: 15px;
  }
  
  .worker-detail .name {
    font-weight: 600;
    margin-bottom: 5px;
  }
  
  .worker-detail .dept {
    font-size: 12px;
    color: #909399;
  }
  
  .el-avatar {
    flex-shrink: 0;
  }
</style>