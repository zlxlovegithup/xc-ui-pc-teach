<template>
  <!-- 课程计划页面 -->
  <div>
    <!-- 通过变量teachplayFormVisible控制弹出窗口是否显示 -->
    <el-button type="primary" @click="teachplayFormVisible = true">添加课程计划</el-button>
    <el-tree
      :data="teachplanList"
      :props="defaultProps"
      node-key="id"
      default-expand-all
      :expand-on-click-node="false"
      :render-content="renderContent"> <!--树节点的内容区的渲染 Function-->
    </el-tree>

    <!--<el-dialog title="添加课程计划" :visible.sync="teachplayFormVisible" >

      <el-form ref="teachplanForm"  :model="teachplanActive" label-width="140px" style="width:600px;" :rules="teachplanRules" >
        <el-form-item label="上级结点" >
          <el-select v-model="teachplanActive.parentid" placeholder="不填表示根结点">
            <el-option
              v-for="item in teachplanList"
              :key="item.id"
              :label="item.pname"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="章节/课时名称" prop="pname">
          <el-input v-model="teachplanActive.pname" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="课程类型" >
          <el-radio-group v-model="teachplanActive.ptype">
            <el-radio class="radio" label='1'>视频</el-radio>
            <el-radio class="radio" label='2'>文档</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="学习时长（分钟）  请输入数字" >
          <el-input type="number" v-model="teachplanActive.timelength" auto-complete="off" ></el-input>
        </el-form-item>
        <el-form-item label="排序字段" >
          <el-input v-model="teachplanActive.orderby" auto-complete="off" ></el-input>
        </el-form-item>
        <el-form-item label="章节/课时介绍" prop="description">
          <el-input type="textarea" v-model="teachplanActive.description" ></el-input>
        </el-form-item>

        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="teachplanActive.status" >
            <el-radio class="radio" label="0" >未发布</el-radio>
            <el-radio class="radio" label='1'>已发布</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item  >
          <el-button type="primary" v-on:click="addTeachplan">提交</el-button>
          <el-button type="primary" v-on:click="resetForm">重置</el-button>
        </el-form-item>

      </el-form>
    </el-dialog>-->
    <!-- title: 属性用于定义标题，它是可选的，默认值为空
         visible: 需要设置visible属性，它接收Boolean，当为true时显示 Dialog-->
    <el-dialog title="添加课程计划" :visible.sync="teachplayFormVisible">
      <!-- v-model: 指令在表单 <input>、<textarea> 及 <select> 元素上创建双向数据绑定
           v-rules: 表单验证规则-->
      <el-form ref="teachplanForm" :model="teachplanActive" label-width="140px" style="width: 600px;" :rules="teachplanRules">
        <!-- 在 Form 组件中，每一个表单域由一个 Form-Item 组件构成，表单域中可以放置各种类型的表单控件，
            包括 Input、Select、Checkbox、Radio、Switch、DatePicker、TimePicker -->
        <!-- label: 标签文本 -->
        <el-form-item label="上级节点">
          <el-select v-model="teachplanActive.parentid" placeholder="不填表示根结点">
            <el-option
              v-for="item in teachplanList"
              :key="item.id"
              :label="item.pname"
              :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <!-- prop: 表单域 model字段，在使用 validate、resetFields 方法的情况下，该属性是必填的 -->
        <el-form-item label="章节/课时名称" prop="pname">
          <!-- auto-complete: 原生属性，自动补全 on/off -->
          <el-input v-model="teachplanActive.pname" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="课程类型">
          <el-radio-group v-model="teachplanActive.ptype">
            <el-radio class="radio" label="1">视频</el-radio>
            <el-radio class="radio" label="2">文档</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="学习时长(分钟) 请输入数字">
          <el-input type="number" v-model="teachplanActive.timelength" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="排序字段">
          <el-input v-model="teachplanActive.orderby" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="章节/课时介绍" prop="description">
          <el-input type="textarea" v-model="teachplanActive.description"></el-input>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="teachplanActive.status">
            <el-radio class="radio" label="0">未发布</el-radio>
            <el-radio class="radio" label="1">已发布</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" v-on:click="addTeachplan">提交</el-button>
          <el-button type="primary" v-on:click="resetForm">重置</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!-- 在父组件的视图中使用子组件，同时传入变量ischoose，并指定父组件的方法名为choosemedia
          这里使用el-dialog 实现弹出窗口。
    -->
    <el-dialog title="选择媒资文件" :visible.sync="mediaFormVisible">
      <media-list v-bind:ischoose="true" @choosemedia="choosemedia"></media-list>
    </el-dialog>
  </div>
</template>
<script>
  let id = 1000;
  import * as courseApi from '../../api/course';
  import utilApi from '../../../../common/utils';
  import * as systemApi from '../../../../base/api/system';
  import mediaList from '@/module/media/page/media_list.vue'; //引入子组件

  export default {
    //引入子组件
    components:{
      mediaList
    },
    data() {
      return {
        mediaFormVisible:false,  //媒资窗口是否显示
        teachplayFormVisible:false,//控制添加窗口是否显示
        teachplanList : [{
          id: 1,
          pname: '一级 1',
          children: [{
            id: 4,
            pname: '二级 1-1',
            children: [{
              id: 9,
              pname: '三级 1-1-1'
            }, {
              id: 10,
              pname: '三级 1-1-2'
            }]
          }]
        }],
        defaultProps:{
          children: 'children',
          label: 'pname'
        },
        teachplanRules: {
          pname: [
            //required: 是否必填，如不设置，则会根据校验规则自动生成
            //message: 提示内容
            //trigger:'blur'  表示“当失去焦点时（光标不显示的时候），触发此提示"请输入课程计划名称",此处应该是有一个校验，若失去焦点，则触发trigger进行校验，若校验不成功，则进行提示
            {required: true, message: '请输入课程计划名称', trigger: 'blur'}
          ],
          status: [
            {required: true, message: '请选择状态', trigger: 'blur'}
          ]
        },
        teachplanActive:{},
        teachplanId:''
      }
    },
    methods: {
      //选择视频，打开窗口
      choosevideo(data){
          //得到当前的课程计划
          this.teachplanId = data.id
          alert(this.teachplanId)
          console.log("========this.teachplanId:"+this.teachplanId);
          console.log("========data.mediaFileOriginalName:"+data.mediaFileOriginalName);
          console.log("========data:"+data);
          this.mediaFormVisible = true;//打开媒资窗口
      },
      //保存选择的视频
      //参数包括：媒资文件id、媒资文件的原始名称、媒资文件 url
      choosemedia(mediaId,fileOriginalName,mediaUrl){
        //保存视频到课程计划表中
        let teachplanMedia ={}
        teachplanMedia.mediaId =mediaId;
        teachplanMedia.mediaFileOriginalName =fileOriginalName;
        teachplanMedia.mediaUrl =mediaUrl;
        //课程id
        teachplanMedia.courseId =this.courseid;
        //课程计划
        teachplanMedia.teachplanId=this.teachplanId

        //保存媒资信息
        courseApi.savemedia(teachplanMedia).then(res=>{
            if(res.success){
                this.$message.success("选择视频成功")
              //查询课程计划
              this.findTeachplan()
            }else{
              this.$message.error(res.message)
            }
        })
      },
      //提交课程计划
      addTeachplan(){
        //校验表单
        this.$refs.teachplanForm.validate((valid) =>{
          if(valid){
            //添加课程计划时带上课程id
            this.teachplanActive.courseid = this.courseid;
            courseApi.addTeachplan(this.teachplanActive).then((res) => {
              if(res.success){
                this.$message.success('提交成功!');
                //清空表单
                this.teachplanActive = {};
                //刷新整个数
                this.findTeachplan();
              }else{
                this.$message.error('提交失败!');
              }
            });
          }
        });
      },
      //重置表单
      resetForm(){
        this.teachplanActive = {}
      },

      append(data) {
        const newChild = { id: id++, label: 'testtest', children: [] };
        if (!data.children) {
          this.$set(data, 'children', []);
        }
        data.children.push(newChild);

      },
      edit(data){
        //alert(data.id);
      },
      remove(node, data) {
        const parent = node.parent;
        const children = parent.data.children || parent.data;
        const index = children.findIndex(d => d.id === data.id);
        children.splice(index, 1);

      },
      //使用render-content指定渲染函数，该函数返回需要的节点区内容即可
      renderContent(h, { node, data, store }) {
        return (
          <span style="flex: 1; display: flex; align-items: center; justify-content: space-between; font-size: 14px; padding-right: 8px;">
            <span>
              <span>{node.label}</span>
            </span>
            <span>
              <el-button style="font-size: 12px;" type="text" on-click={ () => this.choosevideo(data) }>{data.mediaFileoriginalname}&nbsp;&nbsp;&nbsp;&nbsp; 选择视频</el-button>
              <el-button style="font-size: 12px;" type="text" on-click={ () => this.edit(data) }>修改</el-button>
              <el-button style="font-size: 12px;" type="text" on-click={ () => this.remove(node, data) }>删除</el-button>
            </span>
          </span>);
      },
      findTeachplan(){
        this.teachplanList = [];
        //查询课程计划
        courseApi.findTeachplanList(this.courseid).then(res=>{
            // if(res && res.children){
            //   this.teachplanList = res.children;
            // }
            //     this.teachplanList = [];
                if(res && res.children){
                  this.teachplanList = res.children;
                }
        });
      }
    },
    mounted(){
      //课程id
      this.courseid = this.$route.params.courseid;
      //查询课程计划
      this.findTeachplan();

    }
  }
</script>
<style>

</style>
