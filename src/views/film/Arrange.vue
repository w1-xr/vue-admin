<template>
  <div class="main">

    <el-dialog
        title="新增电影排片"
        :visible.sync="dialog"
        width="50%">

      <el-form ref="form" :model="arrangement" label-width="80px">

        <el-form-item label="电影名称">
          <el-input :disabled="true" v-model="arrangement.name"></el-input>
        </el-form-item>

        <el-form-item label="开放座位">
          <el-input-number v-model="arrangement.seatNumber" :min="10" :max="100"></el-input-number>
        </el-form-item>

        <el-form-item label="放映类型">
          <el-select v-model="arrangement.type" placeholder="请选择放映类型">
            <el-option label="2D 放映" value="2D放映"></el-option>
            <el-option label="3D 放映" value="3D放映"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item style="width: 500px" label="放映日期">
          <el-col :span="11">
            <el-date-picker type="date" placeholder="选择日期" v-model="arrangement.date"
            :picker-options="pickerOptions"
                            value-format="yyyy-MM-dd" style="width: 100%;"></el-date-picker>
          </el-col>
        </el-form-item>

        <el-form-item style="width: 500px" label="放映时间">
          <el-col :span="11">
            <el-time-picker placeholder="开始时间" value-format="HH:mm:ss" v-model="arrangement.startTime"
                            style="width: 100%;"></el-time-picker>
          </el-col>
          <el-col style="text-align: center" class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-time-picker placeholder="结束时间" value-format="HH:mm:ss" v-model="arrangement.endTime"
                            style="width: 100%;"></el-time-picker>
          </el-col>
        </el-form-item>

        <el-form-item label="电影票价">
          <el-input-number v-model="arrangement.price" :precision="2" :step="0.1" :max="999.99"></el-input-number>
        </el-form-item>
		
		<!-- <el-form-item label="添加折扣">
		  <el-radio-group v-model="arrangement.isDisCount" @change="handleRadioChange">
		      <el-radio label="是" value="Y"></el-radio>
		      <el-radio label="否" value="N"></el-radio>
		  </el-radio-group>
		</el-form-item>	
		<el-form-item label="折扣" v-show="showDisCount">
		  <el-input v-model="arrangement.disCountInfo" placeholder="请输入折扣比例(1-10)"></el-input>
		</el-form-item>
		
        <el-form-item label="排片人">
          <el-input type="textarea" v-model="arrangement.founder"></el-input>
        </el-form-item> -->
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialog = false">取 消</el-button>
        <el-button type="primary" @click="submitEditArrange">确 定</el-button>
      </span>
    </el-dialog>

    <el-table
        v-loading="loading"
        :data="List.filter(data => !search || data.name.toLowerCase().includes(search.toLowerCase()))"
        style="width: 100%">

      <el-table-column
          label="电影名"
          prop="name">
      </el-table-column>

      <el-table-column
          label="放映日期"
          prop="date">
      </el-table-column>

      <el-table-column
          label="开放座位"
          prop="seatNumber">
      </el-table-column>

      <el-table-column label="放映类型">
        <template slot-scope="scope">
          <el-tag effect="dark" v-if="scope.row.type === '2D放映'" type="success" disable-transitions>2D 放映</el-tag>
          <el-tag effect="dark" v-if="scope.row.type === '3D放映'" type="primary" disable-transitions>3D 放映</el-tag>
        </template>
      </el-table-column>

      <!-- <el-table-column
          label="票房统计"
          prop="boxOffice">
      </el-table-column> -->

      <el-table-column
          label="结束时间"
          prop="endTime">
      </el-table-column>

      <el-table-column label="本场票价">
        <template slot-scope="scope">
          <el-tag size="medium" effect="plain">{{ scope.row.price }} 元</el-tag>
        </template>
      </el-table-column>

      <el-table-column width="300" align="right">
        <template slot="header" slot-scope="scope">
          <el-input
              v-model="search"
              size="mini"
              placeholder="输入关键字搜索"/>
        </template>
        <template style="white-space: nowrap" slot-scope="scope">
          <!-- <el-button
              size="mini"
              icon="el-icon-s-ticket"
              type="success"
              @click="handleBoxOffice(scope.$index, scope.row)">统计票房
          </el-button> -->
          <el-button
              size="mini"
              icon="el-icon-edit"
              type="primary"
              @click="handleEdit(scope.$index, scope.row)">修改
          </el-button>

          <el-popconfirm
              confirm-button-text='确定'
              cancel-button-text='不用了'
              icon="el-icon-info"
              icon-color="red"
              @confirm="handleDelete(scope.$index, scope.row)"
              title=" 确定要删除此电影档期吗？ "
          >
            <el-button
                style="margin-left: 8px"
                size="mini"
                icon="el-icon-delete"
                type="danger"
                slot="reference">删除
            </el-button>
          </el-popconfirm>
        </template>
      </el-table-column>

    </el-table>

  </div>
</template>

<script>
import {FindAllArrangement, DeleteArrangement, UpdateArrangement} from "@/api/film";
import {UpdateUser} from "@/api/user";

export default {
  name: "Arrange",
  data() {
    return {
      pickerOptions: {
        disabledDate(time) {
          // 禁用今天之前的日期（包括今天的时间戳比较）
          return time.getTime() < Date.now();
        },
      },
      dialog: false,
      search: '',
      List: [],
      loading: false,
	    showDisCount:false,
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
		  isDisCount:'N',
		  disCountInfo:''
      },
    }
  },
   
  mounted() {
	this.showDisCount = false;
    this.load();
  },

  methods: {
    load() {
      this.loading = true;
	  this.handleRadioChange(event);
	  debugger;
      FindAllArrangement().then(res => {
        setTimeout(() => {
          this.loading = false;
          this.List = res.data
        }, 700)
      })
    },
    add30Minutes(timeStr) {
      // 1. 补全日期部分（任意日期均可，这里用 2000-01-01）
      const date = new Date(`2000-01-01 ${timeStr}`);

      // 2. 添加 30 分钟（30 * 60 * 1000 毫秒）
      date.setTime(date.getTime() - 30 * 60 * 1000);

      // 3. 提取时间部分并格式化为 HH:mm:ss
      const hours = String(date.getHours()).padStart(2, '0');
      const minutes = String(date.getMinutes()).padStart(2, '0');
      const seconds = String(date.getSeconds()).padStart(2, '0');

      return `${hours}:${minutes}:${seconds}`;
    },
    handleEdit(index, row) {
      this.dialog = true;
      this.arrangement = row;
	  this.handleRadioChange(this.arrangement.isDisCount);
    },

    submitEditArrange() {
      console.log(this.add30Minutes(this.arrangement.endTime))
      if (this.add30Minutes(this.arrangement.endTime) < this.arrangement.startTime) {
          this.$message({
            message: '结束时间必须大于开始时间30min',
            type: 'warning'
          });
          return;
      }
	  this.arrangement.disCountInfo=this.arrangement.isDisCount == "是"?this.arrangement.disCountInfo:'';
      UpdateArrangement(this.arrangement).then(res => {
        if (res.success) {
          this.dialog = false;
          this.$message({
            message: '排片操作成功！',
            type: 'success'
          });
        }
      })
    },

    handleDelete(index, row) {
      DeleteArrangement(row.id).then(res => {
        this.List.splice(index, 1)
        this.$message({
          message: '排片信息删除成功！',
          type: 'success'
        });
      })
    },

    handleBoxOffice(index, row) {
      this.$prompt('请输入此场次的票房', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
      }).then(({value}) => {
        if (!value) {
          this.$message({
            type: 'warning',
            message: '输入不能为空'
          });
          return
        }
        if (value < 0) {
          this.$message({
            type: 'warning',
            message: '请输入不少于0的数字'
          });
          return
        }
        this.List[index].boxOffice = value
        UpdateArrangement(this.List[index]).then(res => {
          this.$message({
            message: '票房统计成功',
            type: 'success',
          });
        })
      }).catch(err => {
        this.$message({
          type: 'warning',
          message: err
        });
      });
    },
	
	handleRadioChange(event) {
	    this.showDisCount = event == "是" ? true : false;
	}
  }

}
</script>

<style scoped>
.main {
  padding: 20px;
}
</style>
