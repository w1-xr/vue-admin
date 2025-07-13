<template>
    <div class="film-list">
      <el-table
          v-loading="tableLoading"
          :data="filmList.filter(data => !search || data.name.toLowerCase().includes(search.toLowerCase()))"
          style="width: 100%">
  
        <el-table-column label="电影名">
          <template slot-scope="scope">
            <el-image
                style="width: 100px; height: 145px"
                :src="scope.row.cover">
            </el-image>
            <div style="font-size: 10px;padding-top: 5px;padding-left: 3px">{{scope.row.name}}</div>
          </template>
        </el-table-column>
  
        <el-table-column
            label="上映时间"
            prop="releaseTime">
        </el-table-column>
  
        <el-table-column
            label="类型"
            prop="type">
        </el-table-column>
  
        <el-table-column label="上架状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status === true" type="success" disable-transitions>上架</el-tag>
            <el-tag v-if="scope.row.status === false" type="warning" disable-transitions>下架</el-tag>
          </template>
        </el-table-column>
  
        <el-table-column
            label="地区"
            prop="region">
        </el-table-column>
  
        <el-table-column label="电影时长">
          <template slot-scope="scope">
            <el-tag size="medium" effect="plain">{{ scope.row.duration }} 分钟</el-tag>
          </template>
        </el-table-column>
  
        <el-table-column width="260" align="center">
          <template slot="header" slot-scope="scope">
            <el-input
                v-model="search"
                size="mini"
                placeholder="输入关键字搜索"/>
          </template>
          <template style="white-space: nowrap; " slot-scope="scope">
            <el-button
              size="medium"
              icon="el-icon-chat-dot-round"
              type="warning"
              @click="handleComment(scope.row)"
              style="margin-left: 8px"
            >评论</el-button>
          </template>
        </el-table-column>
  
      </el-table>
        <!-- 评论查看对话框 -->
        <el-dialog
            title="电影评论"
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
            <el-table-column align="center"
                prop="filmEvaluate.comment"
                label="评论内容"
            ></el-table-column>
    
            <el-table-column align="center"
                prop="user.nickname"
                label="用户"
                width="120"
            ></el-table-column>
    
            <el-table-column align="center"
                label="评分"
                width="220"
            >
                <template slot-scope="scope">
                <el-rate
                    v-model="scope.row.filmEvaluate.star"
                    disabled
                    show-score
                    :colors="['#99A9BF', '#F7BA2A', '#FF9900']"
                    :max="5"
                    :disabled-void-color="'#EFF2F7'"
                    >
                </el-rate>
                </template>
            </el-table-column>
    
            <el-table-column align="center"
                prop="filmEvaluate.createAt"
                label="评论时间"
                width="180"
            ></el-table-column>
    
            <el-table-column align="center"
                label="操作"
                fixed="right"
                width="120"
            >
                <template slot-scope="scope">
                <el-popconfirm
                    title="确定删除该评论吗？"
                    @confirm="handleDeleteComment(scope.row.filmEvaluate.id)"
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
    </div>
  </template>
  
  <script>
  import {ListAllFilm} from "@/api/film";
  import config from "@/config";
  import { DelFilmEvaluateByFId, FindFilmEvaluateByFId } from "@/api/film";

  export default {
    data() {
      return {
        header:{
          "Authorization": localStorage.getItem("token")
        },
        commentDialog: false,
        commentLoading: false,
        comments: [],
        currentFilmId: null,
        uploadAction: config.API_URL + '/upload',
        url: '',
        tableLoading: false,
        form: {
          cover: 'null',
          name: '',
          region: '',
          releaseTime: '',
          duration: 120,
          introduction: '',
          type: '',
          status: true,
        },
        arrangement: {
          name: '',
          fid: '',
          seatNumber: 40,
          price: 30.50,
          date: '',
          startTime: '',
          endTime: '',
          founder: '',
          type: '2D放映',
          isDisCount:'',
          disCountInfo:''
        },
        filmList: [],
        search: '',
      }
    },
  
    mounted() {
      this.tableLoading = true;
      this.showDisCount = false;
      ListAllFilm().then(res=>{
        setTimeout(()=>{
          this.filmList = res.data;
          this.tableLoading = false;
        },700)
      })
    },
    methods: {
      // 查看评论处理
      handleComment(row) {
        this.currentFilmId = row.id;
        this.commentDialog = true;
      },
  
      // 获取评论数据
      async fetchComments() {
        try {
          this.commentLoading = true;
          const res = await FindFilmEvaluateByFId(this.currentFilmId);
          this.comments = res.data;
        } catch (error) {
          console.error(error);
        } finally {
          this.commentLoading = false;
        }
      },
  
      // 删除评论
      async handleDeleteComment(commentId) {
        try {
          await DelFilmEvaluateByFId(commentId);
          this.$message.success("评论删除成功");
          this.fetchComments(); // 刷新列表
        } catch (error) {
          this.$message.error("评论删除失败");
        }
      }
    },
  }
  </script>
  
  <style scoped>
  .film-list {
    padding: 20px;
  }
  </style>
  