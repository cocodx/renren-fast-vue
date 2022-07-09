<template>
    <el-row :gutter="24">
      <el-col :span="6">
        <category @tree-node-click="treenodeClick"></category>
      </el-col>
      <el-col :span="18">
        <div class="mod-config">
          <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
              <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
            </el-form-item>
            <el-form-item>
              <el-button @click="getDataList()">查询</el-button>
              <el-button type="success" @click="getAllDataList()">查询全部</el-button>
              <el-button
                type="primary"
                @click="addOrUpdateHandle()"
              >新增</el-button>
              <el-button
                type="danger"
                @click="deleteHandle()"
                :disabled="dataListSelections.length <= 0"
              >批量删除</el-button>
            </el-form-item>
          </el-form>
          <el-table
            :data="dataList"
            border
            v-loading="dataListLoading"
            @selection-change="selectionChangeHandle"
            style="width: 100%;"
          >
            <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
            <el-table-column prop="attrGroupId" header-align="center" align="center" label="分组id"></el-table-column>
            <el-table-column prop="attrGroupName" header-align="center" align="center" label="组名"></el-table-column>
            <el-table-column prop="sort" header-align="center" align="center" label="排序"></el-table-column>
            <el-table-column prop="descript" header-align="center" align="center" label="描述"></el-table-column>
            <el-table-column prop="icon" header-align="center" align="center" label="组图标"></el-table-column>
            <el-table-column prop="catelogId" header-align="center" align="center" label="所属分类id"></el-table-column>
            <el-table-column
              fixed="right"
              header-align="center"
              align="center"
              width="150"
              label="操作"
            >
              <template slot-scope="scope">
                <el-button type="text" size="small" @click="relationHandle(scope.row.attrGroupId)">关联</el-button>
                <el-button
                  type="text"
                  size="small"
                  @click="addOrUpdateHandle(scope.row.attrGroupId)"
                >修改</el-button>
                <el-button type="text" size="small" @click="deleteHandle(scope.row.attrGroupId)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-pagination
            @size-change="sizeChangeHandle"
            @current-change="currentChangeHandle"
            :current-page="pageIndex"
            :page-sizes="[10, 20, 50, 100]"
            :page-size="pageSize"
            :total="totalPage"
            layout="total, sizes, prev, pager, next, jumper"
          ></el-pagination>
          <!-- 弹窗, 新增 / 修改 -->
          <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>

          <!-- 修改关联关系 -->
          <relation-update v-if="relationVisible" ref="relationUpdate" @refreshData="getDataList"></relation-update>
        </div>
      </el-col>
    </el-row>
</template>

<script>
  /**
   * 父子组件，传递数据
   * 1）、子组件给attrgourp父组件，传递数据，使用事件机制；
   *  当子组件给父组件发送一个事件，携带上数据。
   *  this.$emit("事件名称",携带的数据......)
   *
   */
import Category from '../common/category'
import AddOrUpdate from "./attrgroup-add-or-update";
export default {
  components:{Category,AddOrUpdate},
  data(){
    return{
      catId: 0,
      dataForm: {
        key: ""
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      relationVisible: false
    }
  },
  methods:{
    //感知树节点被点击
    treenodeClick(data,node,component){
      console.log("感知到category的节点被点击：",data,node,component)
      console.log("刚才被点击的菜单id：",data.catId)
      if (node.level == 3){
        this.catId = data.catId;
        this.getDataList()
      }
    },
    getDataList(){
      this.dataListLoading=true;
      this.$http({
        url: this.$http.adornUrl(`/product/attrgroup/list/${this.catId}`),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
      }).then(({ data }) =>{
        if (data && data.code === 0) {
          this.dataList = data.page.list
          this.totalPage = data.page.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    addOrUpdateHandle(id){
      console.log("打开新增和修改的组件")
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    }
  }
}
</script>

<style scoped>

</style>
