<template>
  <div>
    <div class="searchBox">
      <el-input v-model="keywordTxt" placeholder="RMA# , Customer , Tracking "></el-input>
      <el-select v-model="value" placeholder="select">
        <el-option v-for="item in status" :key="item.value" :label="item.label" :value="item.value">
        </el-option>
      </el-select>
      <el-date-picker type="date" v-model="startDate" placeholder="Start Date" format="MM/dd/yyyy" value-format="MM/dd/yyyy"></el-date-picker>
      <el-date-picker type="date" v-model="endDate" :picker-options="endDateOpt" placeholder="End Date" class="endDate" format="MM/dd/yyyy" value-format="MM/dd/yyyy"></el-date-picker>
      <el-button type="warning" @click="selectRmas()">Search</el-button>
    </div>
    <div class="wrapBtn">
      <el-button type="primary" round v-on:click="addRma(0)">Add RMA</el-button>
      <el-button type="primary" round v-on:click="addRma(1)">Whole Order RMA</el-button>
    </div>
    <div class="rmaList">
      <el-table :data="rmaList" stripe border :row-class-name="tableRowClassName" @current-change="handleCurrentChange">
        <el-table-column label="RMA #">
          <template slot-scope="scope">
            <div v-show="scope.row.rmaType==1">{{scope.row.rmaId}}<a href=""><i class="el-icon-document"></i></a></div>
            <div v-show="scope.row.rmaType==0">{{scope.row.rmaId}}</div>
          </template>
        </el-table-column>
        <el-table-column prop="createDate" label="Date">
        </el-table-column>
        <el-table-column prop="trackingNo" label="Tracking #">
        </el-table-column>
        <el-table-column prop="remarks" label="Remarks">
        </el-table-column>
        <el-table-column label="Customer">
          <template slot-scope="scope">
            <div v-show="scope.row.userId!=null"><a href="###">{{scope.row.userName}}</a></div>
            <div v-show="scope.row.userId==null" class="search" @click="showUserList(scope.row.rmaId)">
              <el-input v-model="user.userName" placeholder="select or search customer" readonly></el-input>
              <i class="el-icon-search"></i>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="status" label="Status">
        </el-table-column>
        <el-table-column prop="" label="Operations">
          <template slot-scope="scope">
            <i class="el-icon-delete" @click="deleteRma(scope.row.rmaId)"></i>
            <i class="el-icon-tickets" @click="showOrderItems(scope.row)"></i>
            <i class="el-icon-view"></i>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="currentPage" :page-sizes="[10, 20, 30, 40]" :page-size="100" layout="sizes, prev, pager, next" :total="total">
      </el-pagination>
    </div>
    <div>
      <el-dialog title="User List" :visible.sync="userListVisible">
        <user-list :rmaId="user.rmaId">
        </user-list>
      </el-dialog>
    </div>
    <div>
      <el-dialog title="Rma Items" :visible.sync="orderItemsVisible">
        <rma-items :rmaId="user.rmaId" :userName="user.userName" :userId="user.userId" :rmaType="user.rmaType">
        </rma-items>
      </el-dialog>
    </div>
  </div>
</template>
<script type="text/javascript">
import UserList from '@/components/rma/rma-manager/UserList'
import RmaItems from '@/components/rma/rma-manager/RmaItems'
import ItemsAdd from '@/components/rma/rma-manager/ItemsAdd'

export default {
  data() {
    return {
      startDate: null,
      endDate: null,
      endDateOpt: {
        disabledDate: (time) => {
          return time.format("MM/dd/yyyy") < this.startDate;
        }
      },
      userListVisible: false,
      orderItemsVisible: false,
      innerVisible: false,
      user: {
        rmaId: '',
        userName: '',
        userId: '',
        rmaType: ''
      },
      form: {
        rmaAddVo: {
          orderItems: [],
          userId: 0,
          remarks: '',
        }
      },
      rmaList: [],
      total: 1,
      pageSize: 10,
      currentPage: 1,
      createRmaUrl: "/admin/rma/createRma",
      selectRmasUrl: '/admin/rma/selectRmas',
      deleteRmaUrl: '/admin/rma/deleteRma',
      keywordTxt: '',
      status: [{
        value: '0',
        label: 'Pending'
      }, {
        value: '1',
        label: 'Shipping'
      }, {
        value: '2',
        label: 'Receipt'
      }, {
        value: '3',
        label: 'Inspect'
      }, {
        value: '4',
        label: 'Finish'
      }],
      value: '',

    }
  },
  mounted() {
    this.selectRmas(1, this.pageSize);
  },

  methods: {
    showOrderItems(row) {
      this.orderItemsVisible = true;
      this.user.rmaId = row.rmaId;
      this.user.userName = row.userName;
      this.user.userId = row.userId
      this.user.rmaType = row.rmaType
    },
    showUserList(rmaId) {
      this.userListVisible = true;
      this.user.rmaId = rmaId
    },
    tableRowClassName({ row, rowIndex }) {
      if (rowIndex === 1) {
        return 'warning-row';
      } else if (rowIndex === 3) {
        return 'success-row';
      }
      return '';
    },
    handleSizeChange(pageSize) {
      this.selectRmas(1, pageSize);
      this.pageSize = pageSize;
    },
    handleCurrentChange(currentPage) {
      this.selectRmas(currentPage, this.pageSize);
    },
    addRma: async function(rmaType) {
      let length = this.rmaList.filter(v => v.rmaType == rmaType && v.userId == null).length
      if (length > 0) {
        this.$alert("Already have an empty RMA", {
          confirmButtonText: 'sure',
        });
      } else {
        var rmaTypeInfo = {
          "rmaType": rmaType,
        }
        const res = await this.$request.post(this.createRmaUrl, rmaTypeInfo);
        if (res.data.code == 200) {
          var _data = {
            rmaId: res.data.data.rmaId,
            rmaType: rmaType,
            userName: null,
          }
          this.rmaList.unshift(_data);
        }
      }

    },
    selectRmas: async function(currentPage, pageSize) {
      var selectRmasInfo = {
        "pageSize": this.pageSize,
        "pageNum": this.currentPage,
        "keyword": this.keywordTxt,
        "status": this.value,
        "startDate": this.startDate,
        "endDate": this.endDate
      }
      const res = await this.$request.post(this.selectRmasUrl, selectRmasInfo);
      if (res.data.code == 200) {
        this.rmaList = res.data.data.list;
        this.total = res.data.data.total
      }
    },
    deleteRma: async function(rmaId) {
      const res = await this.$request.post(this.deleteRmaUrl + "?rmaId=" + rmaId);
    }
  },
  components: {
    UserList,
    RmaItems,
    ItemsAdd
  }
}

</script>
<style type="text/css">
.searchBox {
  background: #363D42;
  border-radius: 5px;
  padding: 20px;
  margin: 20px 0 0 0;
  overflow: hidden;
}

.searchBox .el-input {
  width: 220px;
  float: left;
}

.searchBox .el-select {
  float: left;
  margin: 0 10px
}

.endDate {
  margin: 0 10px
}

.searchBox .el-button {
  float: left;
}

.wrapBtn {
  border-bottom: 1px solid #ccc;
  padding: 10px;
  overflow: hidden;
}

.wrapBtn .el-button {
  float: left;
  outline: none;
}

.el-table {

  margin-top: 20px;
}

.el-table th {
  text-align: center;
}

.el-table_1_column_7 i {
  font-size: 20px;
  margin: 0 5px
}

.search {
  border: 1px solid #ccc;
  border-radius: 15px;
  width: 230px;
  height: 30px;
  margin: 0 auto;
  padding: 0 10px 0 15px
}

.search .el-input {
  border: 0;
  width: 160px;
  float: left;
  height: 28px;
}

.rmaList .el-input__inner {
  height: 28px;
  line-height: 28px;
  border: 0;
  background: transparent;
  padding: 0
}

.search .el-icon-search {
  margin-top: 7px;
  border-left: 1px solid #ccc;
  padding-left: 10px
}
.el-table--border td:last-child,
.el-table--border th:last-child {
  border-right: 0
}

</style>
