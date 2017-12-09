<template>
  <div>
    <div class="wrapBtn">
      <el-button type="primary" round v-on:click="addInventory(0)">Add Inventory</el-button>
    </div>
  <div class="productList">
    <el-table border
      :data="productList"
      style="width: 100%">
      <el-table-column
        prop="date"
        label="Date"
        width="180">
      </el-table-column>
      <el-table-column
        prop="shortName"
        label="shortName"
        width="260">
      </el-table-column>
      <el-table-column
        label="ProductName">
        <template slot-scope="scope">
          <div v-if="scope.row.ProductName!=null">{{ scope.row.ProductName }}</div>
          <div v-if="scope.row.ProductName==null">
            <el-input
              placeholder="productName,shortName"
              suffix-icon="el-icon-search" @click="searchProduct()">
            </el-input></div>
        </template>
      </el-table-column>
      <el-table-column
        prop="qty"
        label="qty"
       width="180">
      </el-table-column>
      <el-table-column
        prop="operation"
        label="Operations"
       width="180">
        <template slot-scope="scope">
          <!--<el-button @click="handleClick(scope.row)" type="text" size="small">delete</el-button>-->
          <i class="el-icon-edit" ></i>
          <i class="el-icon-delete @click=delete()"></i>
        </template>
      </el-table-column>
    </el-table>
  </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        productList: [{
          date: '2016-05-02',
          shortName: 'iPhone 8 Plus Front Screen',
          ProductName: 'iPhone 8 Plus A1864 A1897 A1898 Front Screen',
          qty:3
        }, {
          date: '2016-05-04',
          shortName: 'Product_1019',
          ProductName: 'Product_1019,B Stock',
          qty:2
        }, {
          date: '2016-05-01',
          shortName: 'Asus Memo Pad Touch Panel',
          ProductName: 'Asus Memo Pad 10 ME103 LCD Screen Display',
          qty:1
        }, {
          date: '2016-05-03',
          shortName: 'Lenovo LCD Screen',
          ProductName: 'Lenovo IdeaTab S5000 LCD Screen Display',
          qty:5
        }]
      }
    },
    methods: {
        addInventory() {
          var _inventory = {
              date: "2017-12-09",
              shortName:null,
              productName:null,
              qty:null
          }
          this.productList.push(_inventory)
        },
      searchProduct(){

      }
    }
  }
</script>

<style type="text/css">
  .el-table th{
    text-align: center;
  }
  .el-table_1_column_4 i{
    font-size: 20px;
    margin: 0 5px
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

  #app {
    margin-top: 0px ;
  }
</style>
