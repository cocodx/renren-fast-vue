<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽">
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button @click="batchDelete" type="danger">批量删除</el-button>
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
             show-checkbox
             node-key="catId" :default-expanded-keys="expandedKey"
             :draggable="draggable"
             :allow-drop="allowDrop"
             :allow-drag="allowDrag"
             @node-drop="handleDrop"
             ref="menuTree">

    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level<=2" type="text" size="mini" @click="() => append(data)">Append</el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">edit</el-button>
          <el-button v-if="node.childNodes.length==0" type="text" size="mini"
                     @click="() => remove(node, data)">Delete</el-button>
        </span>
      </span>

    </el-tree>


    <el-dialog
      :title=title
      :visible.sync="dialogVisible"
      width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <el-button type="primary" @click="submitData">确 定</el-button>
      <el-button @click="dialogVisible = false">取 消</el-button>
    </el-dialog>
  </div>

</template>

<script>
  export default {
    data() {
      return {
        draggable:false,
        updateNodes:[],
        title:"",
        dialogType:"",
        category:{
          name:"",
          parentCid:0,
          catLevel:0,
          showStatus:1,
          sort:0,
          catId:null
        },
        dialogVisible:false,
        menus: [],
        expandedKey: [],
        defaultProps: {
          children: "children",
          label: "name"
        },
        maxLevel:1,
        pCid:[]
      }
    },
    methods: {
      getMenus() {
        this.$http({
          url: this.$http.adornUrl("/product/category/list/tree"),
          method: "get"
        }).then(({data}) => {
          console.log("成功获取到菜单数据", data.data)
          this.menus = data.data
        })
      },
      allowDrag(draggingNode){
        console.log("判断节点能够被拖动：",draggingNode)
        // return draggingNode.data.label.indexOf('三级 3-2-2') === -1;
        return true
      },
      batchDelete(){
        let catIds = []
        let checkedNodes = this.$refs.menuTree.getCheckedNodes()
        console.log("被选中的元素：",checkedNodes)
        for (let i = 0; i < checkedNodes.length; i++) {
          catIds.push(checkedNodes[i].catId)
        }
        this.$confirm(`是否批量删除【${catIds}】菜单`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(this.catIds, false)
          }).then(({data}) => {
            console.log("成功获取到菜单数据", data.data)
            this.$message({
              type: 'success',
              message: '菜单批量删除成功!'
            });
            this.getMenus()
          })

        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },
      batchSave(){
        this.$http({
          url: this.$http.adornUrl("/product/category/update/sort"),
          method: "post",
          data: this.$http.adornData(this.updateNodes, false)
        }).then(({data}) => {
          this.$message({
            message: '菜单顺序修改成功',
            type: 'success'
          });
          //刷新菜单
          this.getMenus()
          //要展开的菜单
          this.expandedKey=this.pCid
          this.updateNodes=[]
          this.maxLevel=0
        })
      },
      handleDrop(draggingNode, dropNode, dropType, ev) {
        console.log('tree drop: ', draggingNode, dropNode, dropType);
        //当前节点，最新的父节点id，获取dropNode的parentId
        //当前节点，拖拽节点的最新顺序，
        //当前拖拽节点的最新层级
        let pCid=0
        let siblings = null
        if(dropType=='before' || dropType=='after'){
          pCid = dropNode.parent.data.catId
          siblings = dropNode.data.parent.childNodes
        }else{
          pCid = dropNode.data.catId
          siblings = dropNode.childNodes
        }
        this.pCid.push(pCid)
        for(let i=0;i<siblings.length;i++){
          if(siblings[i].data.catId == draggingNode.data.catId){
            //如果遍历的是正在的节点
            let catLevel = draggingNode.data.catLevel
            if(siblings[i].level != draggingNode.level){
              //如果当前层级发生变化
              //修改他子节点的层级
              catLevel = siblings[i].level
              this.updateChildNodeLevel(siblings[i])

            }
            this.updateNodes.push({catId:siblings[i].data.catId.catId,sort:i,parentCid:pCid,catLevel:catLevel})
          }else{
            this.updateNodes.push({catId:siblings[i].data.catId.catId,sort:i})
          }

        }
        console.log("updateNodes",this.updateNodes)
      },
      updateChildNodeLevel(node){
        if(node.childNodes.length>0){
          for(let i=0;i<node.childNodes.length;i++){
            var cNode = node.childNodes[i].data
            this.updateNodes.push({catId:cNode.catId,catLevel:node.childNodes[i].level})
            this.updateChildNodeLevel(node.childNodes[i])
          }
        }
      },
      allowDrop(draggingNode, dropNode, type) {
        //1、被拖动的当前节点以及所在的父节点总层数不能大于3
        //1)、被拖动的当前节点的总层数
        console.log("allowDrop:",draggingNode,dropNode,type)
        return true
        // this.countNodeLevel(draggingNode);
        // //当前正在拖动的节点+父节点所在的深度不大于3
        // console.log("深度 this maxLevel:",this.maxLevel);
        // let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
        // if(type == "inner"){
        //   console.log(`this.maxLevel:${this.maxLevel};draggingNode.data.catLevel:${draggingNode.data.catLevel};dropNode.level:${dropNode.level}`)
        //   return deep + deep+dropNode.level <=3
        // }else{
        //   return deep+dropNode.parent.level <=3
        // }
      },
      countNodeLevel(node){
        //找到所有的子节点，求出最大深度
        if(node.childNodes!=null && node.childNodes.length>0){
          for (let i = 0; i < node.children.length ; i++) {
            if(node.childNodes[i].level>this.maxLevel){
              this.maxLevel = node.childNodes[i].catLevel;
            }
            this.countNodeLevel(node.childNodes[i]);
          }
        }
      },
      handleNodeClick(data) {
        console.log(data)
      },
      edit(data){
        console.log(data)
        this.dialogType="edit"
        this.title="修改分类"
        this.dialogVisible = true

        //发送请求获取当前节点最新的数据
        this.$http({
          url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
          method: "get",
        }).then(({data}) => {
          //请求成功
          console.log("返回实体：",data)
          this.category.name=data.data.name
          this.category.catId=data.data.catId
          this.category.icon = data.data.icon
          this.category.productUnit=data.data.productUnit
          this.category.parentCid=data.data.parentCid
          this.category.catLevel=data.data.catLevel
          this.category.sort=data.data.sort
          this.category.showStatus=data.data.showStatus
        })
      },
      append(data) {
        console.log(data)
        this.dialogType="add"
        this.title="添加分类"
        this.dialogVisible=true
        this.category.parentCid=data.catId;
        this.category.catLevel=data.catLevel*1+1;

        this.category.name=""
        this.category.catId=null
        this.category.icon = ""
        this.category.productUnit= ""
        this.category.parentCid=data.catId
        this.category.catLevel=data.catLevel*1+1
        this.category.sort=0
        this.category.showStatus=1

      },
      submitData(){
        if(this.dialogType=="add"){
          this.addCategory()
        }
        if(this.dialogType="edit"){
          this.editCategory()
        }
      },
      editCategory(){
        var { catId,name,icon,productUnit} = this.category
        this.$http({
          url: this.$http.adornUrl("/product/category/update"),
          method: "post",
          data: this.$http.adornData({catId,name,icon,productUnit}, false)
        }).then(({data}) => {
          this.$message({
            message: '菜单修改成功',
            type: 'success'
          });
          //关闭对话框
          this.dialogVisible=false
          this.getMenus()
          this.expandedKey=[this.category.parentCid]
        })
      },

      remove(node, data) {
        console.log(node, data)

        this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(() => {
          var ids = [data.catId]
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            this.$message({
              message: '菜单删除成功',
              type: 'success'
            });
            //刷新新的菜单
            this.getMenus()
            this.expandedKey=[node.parent.data.catId]
          })
        }).catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          })
        })
      },
      //三级分类的方法
      addCategory(){
        console.log("提交的三级分类的数据",this.category)
        this.$http({
          url: this.$http.adornUrl("/product/category/save"),
          method:"post",
          data: this.$http.adornData(this.category,false)
        }).then(({data}) => {
          this.$message({
            type: "success",
            message: "菜单保存成功"
          })
          //关闭对话框
          this.dialogVisible=false
          this.getMenus()
          this.expandedKey=[this.category.parentCid]
        })
      }

    },
    created() {
      this.getMenus()
    }
  }
</script>
