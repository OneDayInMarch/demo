<template>
  <div>
    <div style="width=300px;float: left; margin-bottom: 10px;">
        <el-button type="primary"  @click="openAll">全部展开</el-button>
        <el-button type="primary"  @click="closeAll">全部收起</el-button>
        <el-button type="primary" @click="dialogVisible = true">字段配置</el-button>
    </div>
    <div>
        <el-table :data="tableData" style="width:100%" row-key="id" border 
        :tree-props="{children: 'children', hasChildren: 'hasChildren'}" 
        :expand-row-keys="expandRowKeys"
        stripe sortable="true">
                <el-table-column :prop="col.prop" :label="col.label" v-for="(col,index) in activeFields" :key="index" icon="el-icon-search" ></el-table-column>
                <el-table-column label="操作"  fixed="right">
                    <template slot-scope="scope">
                        <el-dropdown  @command="handleCommand">
                            <span>...</span>
                            <el-dropdown-menu>
                                <el-dropdown-item icon="el-icon-search" :command="beforeHandleCommand(scope.$index, scope.row,'search')" >查看</el-dropdown-item>
                                <el-dropdown-item icon="el-icon-edit-outline" :command="beforeHandleCommand(scope.$index, scope.row,'edit')">修改</el-dropdown-item>
                                <el-dropdown-item icon="el-icon-delete" :command="beforeHandleCommand(scope.$index, scope.row,'delete')">删除</el-dropdown-item>
                                <el-dropdown-item :command="beforeHandleCommand(scope.$index, scope.row,'moveTop')">移至顶部</el-dropdown-item>
                                <el-dropdown-item icon="el-icon-arrow-up" :command="beforeHandleCommand(scope.$index, scope.row,'moveUp')">上移</el-dropdown-item>
                                <el-dropdown-item icon="el-icon-arrow-down" :command="beforeHandleCommand(scope.$index, scope.row,'moveDown')">下移</el-dropdown-item>
                                <el-dropdown-item :command="beforeHandleCommand(scope.$index, scope.row,'moveBottom')">移至底部</el-dropdown-item>
                            </el-dropdown-menu>
                        </el-dropdown>
                    </template>
                </el-table-column>
        </el-table>
    </div>
    <el-dialog  :visible.sync="dialogVisible" width="30%"  center >
    <span slot="title" class="title">字段配置</span>
        <el-tabs v-model="activeName">
            <el-tab-pane label="字段选择" name="fieldsChosen">
                <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全部</el-checkbox>
                <div style="margin:15px 0;"></div>                
                <el-checkbox-group  v-model="checkedColumns" @change="handleCheckedList">
                    <div>
                        <el-checkbox style="width:40%;" v-for="item in columns" :label="item" :key="item">{{item}}</el-checkbox>
                    </div>
                </el-checkbox-group>

            </el-tab-pane>
            <el-tab-pane label="字段顺序" name="fieldSorted">
                <vuedraggable v-model="fieldList">
                    <transition-group>
                        <div v-for="field in fieldList" :key="field" style="background-color:gainsboro;margin:10px; text-align: center;">
                            {{field}}
                        </div>
                    </transition-group>
                </vuedraggable>

            </el-tab-pane>
        </el-tabs>
        <span slot="footer" class="dialog-footer">
            <el-button class="buttonLeft" type="primary" plain @click="recoverChecked">恢复默认</el-button>
            <el-button @click="dialogVisible = false" class="buttonRight">取 消</el-button>
            <el-button type="primary" @click="submit" class="buttonRight" >确 定</el-button>
        </span>
    </el-dialog>

  </div>
</template>

<script>
import vuedraggable from 'vuedraggable';
const columnOptions=['编号', '计划开始日期','计划完成日期','实际开始日期','实际完成日期','参考提前期',
'计划提前期','实际提前期','前置任务','产出物', '负责人', '负责人部门', '经办人', '经办人部门',
'重要程度','状态', '逾期状态 ','承诺交期','预测交期','备注'];
export default {
  name: 'Test',
    components: {
    vuedraggable
  },
  data () {
    return {
        expandRowKeys: [],
        activeName:'fieldsChosen',
        dialogVisible:false,
              // 列信息
        checkedColumns:['编号','计划开始日期','计划完成日期','实际开始日期','实际完成日期'],
        checkAll:false,
        columns:columnOptions,
        isIndeterminate:true,
        fieldsTemp:[],//存放按指定顺序排列的fields数组
        fields:[
            {label:"计划名称", prop:"planName", visible: true},
            {label:"编号", prop:"id", visible: true },
            {label:"计划开始日期", prop:"beginTimeP", visible: true },
            {label:"计划完成日期", prop:"endTimeP", visible: true },
            {label:"实际开始日期", prop:"beginTimeS", visible: true },
            {label:"实际完成日期", prop:"endTimeS" , visible: true },
            {label:"参考提前期", prop:"referTime", visible: false },
            {label:"计划提前期", prop:"planTime", visible: false },
            {label:"实际提前期", prop:"practicalTime", visible: false },
            {label:"前置任务", prop:"frontTask", visible: false },
            {label:"产出物", prop:"product", visible: false },
            {label:"负责人", prop:"principal", visible: false },
            {label:"负责人部门", prop:"principalDep", visible: false },
            {label:"经办人", prop:"operator", visible: false },
            {label:"经办人部门", prop:"operatorDep", visible: false },
            {label:"重要程度", prop:"important", visible: false },
            {label:"状态", prop:"status", visible: false },
            {label:"逾期状态", prop:"delayStatus" , visible: false },
            {label:"承诺交期", prop:"promiseTime" , visible: false },
            {label:"预测交期", prop:"predictTime" , visible: false },
            {label:"备注", prop:"remark", visible: false }
        ],
        // fieldList:['编号', '计划开始日期','计划完成日期','实际开始日期','实际完成日期','参考提前期',
        // '计划提前期','实际提前期','前置任务','产出物', '负责人', '负责人部门', '经办人', '经办人部门',
        // '重要程度','状态', '逾期状态 ','承诺交期','预测交期','备注'],
        fieldList:columnOptions,
        tableData: [
            {
                id: "001",
                planName: "ffff",
                beginTimeP: "2022-02-03",
                endTimeP: "2022-02-03",
                beginTimeS: "2022-02-03",
                endTimeS: "2022-02-03",
                children: [
                    { id: "00101", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"},
                    { id: "00102", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"},
                    { id: "00103", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"},
                    { id: "00104", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"}
                ]
            },
            {
                id: "002",
                planName: "uuuu",
                beginTimeP: "2022-02-03",
                endTimeP: "2022-02-03",
                beginTimeS: "2022-02-03",
                endTimeS: "2022-02-03",
                children: [
                    { id: "00201", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"},
                    { id: "00202", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"},
                ]
            },
            {
                id: "003",
                planName: "yyyy",
                beginTimeP: "2022-02-03",
                endTimeP: "2022-02-03",
                beginTimeS: "2022-02-03",
                endTimeS: "2022-02-03",
                children: [
                    { id: "00301", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"},
                    { id: "00302", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"}
                ]
            },
            {
                id: "004",
                planName: "xxxx",
                beginTimeP: "2022-02-03",
                endTimeP: "2022-02-03",
                beginTimeS: "2022-02-03",
                endTimeS: "2022-02-03",
                children: [
                    { id: "00401", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"},
                    { id: "00402", planName: "ffff",beginTimeP: "2022-02-03",endTimeP: "2022-02-03",beginTimeS: "2022-02-03",endTimeS: "2022-02-03"}
                ]
            }
        ]
    }
  },
  computed:{
    activeFields: function(){
        return this.fields.filter((item)=>{
            return item.visible;
        })
    }
  },
  methods:{
    // 全部关闭
    closeAll() {
        const arr=[]
        this.expandRowKeys = arr
        console.log(this.expandRowKeys)
        //全部关闭数组置空失败
    },
    // 全部打开
    openAll() {
      this.expandRowKeys = this.tableData.map((item) => {
        return item.id
      })
    },
    beforeHandleCommand(index,row,command) {
        return {
            'index':index,
            'row':row,
            'command':command
        }
    },
    handleCommand(command) {
        switch(command.command) {
            case "search":
                this.search(command.index);
                break;
            case "edit":
                this.edit(command.index);
                break;
            case "delete":
                this.delete(command.index);
                break;
            case "moveTop":
                this.moveTop(command.index);
                break;
            case "moveUp":
                this.moveUp(command.index);
                break;
            case "moveDown":
                this.moveDown(command.index);
                break;
            case "moveBottom":
                this.moveBottom(command.index);
                break;
        }
    },
    handleCheckAllChange(val){
        this.checkedColumns = val ? columnOptions : [];
        this.isIndeterminate = false;    
    },
    handleCheckedList(val){
        this.checkedColumns=val;
        let checkedCount = val.length;
        this.checkAll = checkedCount === this.columns.length;
        this.isIndeterminate = checkedCount > 0 && checkedCount < this.columns.length;
    },
    recoverChecked(){
        this.checkedColumns=['编号','计划开始日期','计划完成日期','实际开始日期','实际完成日期'];
        this.fieldList=columnOptions;
    },
    submit(){
        this.dialogVisible=false; 
        //更新表格展示列
        this.fields.forEach((item)=>{
            item.visible=false;
        })
        this.checkedColumns.forEach((item)=>{
            for(let i=0;i<this.fields.length;i++){
                if(this.fields[i].label==item){
                    this.fields[i].visible=true;
                }
            }
        })
        //更新表格列顺序
        //更改后列顺序this.fieldList
        this.fieldsTemp=[];//清空存放按指定顺序排列的fields数组
        let n=0;
        this.fieldList.forEach((item)=>{
            for(let i=0;i<this.fields.length;i++){
                if(this.fields[i].label==item){
                    this.fieldsTemp[n++]=this.fields[i];
                }
            }
        })
        this.fields=this.fieldsTemp;
    },

    edit(index,row){
        
    },
    delete(index) {
       this.tableData.splice(index,1);
    },
    // 置顶
    moveTop(index) {
       // 将要置顶的元素存储后删除
       const temp = this.tableData.splice(index, 1)[0];
       // 将元素unshift到数组第一位
       this.tableData.unshift(temp);
    },
    // 置底
    moveBottom(index) {
      // 将要置底的元素存储后删除
      const temp = this.tableData.splice(index, 1)[0];
      // 将元素push到数组最后一位
      this.tableData.push(temp);
    },
    // 调整顺序，arr 数组，indexAdd 添加元素的位置，indexDel 删除元素的位置（indexAdd与indexDel交换位置）
    handleChangeOrder(arr, indexAdd, indexDel) {
      arr[indexAdd] = arr.splice(indexDel, 1, arr[indexAdd])[0];
      return arr;
    },
    // 上移，与前一个元素交换顺序
    moveUp(index) {
      this.handleChangeOrder(this.tableData, index, index - 1);
    },
    // 下移，与后一个元素交换顺序
    moveDown(index) {
      this.handleChangeOrder(this.tableData, index, index + 1);
    },

  }
}
</script>

<style scoped>
/* #PointerIcon {
    transform:rotate(90deg);
} */
.title{
    float: left;
}
.buttonLeft{
    width:20%;
    margin-right: 40%;
}
.buttonRight{
    width:15%;
}
</style>
