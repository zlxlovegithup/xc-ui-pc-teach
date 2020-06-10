<template>
  <session>
    <el-row>
      <el-col :span="4" :offset="2">
        <el-card :body-style="{ padding: '10px' }">
          <img src="/static/images/add.jpg" class="image" height="150px">
          <div style="padding: 10px;">
            <span>课程名称</span>
            <div class="bottom clearfix">
              <time class="time"></time>
              <router-link class="mui‐tab‐item" :to="{path:'/course/add/base'}">
                <el-button type="text" class="button">新增课程</el-button>
              </router-link>
            </div>
          </div>
        </el-card>
      </el-col>

      <el-col :span="4" v-for="(course,index) in courses" :key="course.id" :offset="index > 0 ? 2:2">
        <el-card :body-style="{ padding: '10px' }">
          <!-- 使用了文件系统存储图片 -->
          <img :src="course.pic!=null?imgUrl+course.pic:'/static/images/nonepic.jpg'" class="image" height="150px">
          <div style="padding: 10px;">
            <span>{{course.name}}</span>
            <div class="bottom clearfix">
              <time class="time"></time>
              <el-button type="text" class="button" @click="handleManage(course.id)">管理课程</el-button>
            </div>
          </div>
        </el-card>
      </el-col>

      <!-- 实现分页  "-->
      <el-col :span="24" class="toolbar">
        <el-pagination
          background
          @current-change="handleCurrentChange"
          style="float: right;"
          :page-size="size"
          layout="prev, pager, next"
          :total="total"
          :current-page="page">
        </el-pagination>
      </el-col>
    </el-row>
  </session>
</template>
<script>
  import * as courseApi from '../api/course';
  import utilApi from '../../../common/utils';
  let sysConfig = require('@/../config/sysConfig'); //@表示src包
  export default {
    data() {
      return {
        page:1,
        size:9,
        total:0,
        courses:[],
        sels:[],//列表中选中列
        imgUrl:sysConfig.imgUrl //http://img.xuecheng.com/
      };
    },
    methods: {
      //管理课程
      handleManage(id){
        console.log(id);
        //使用 this.$router.push(location) 来修改 url，完成跳转。
        //push 后面可以是对象，也可以是字符串：
        this.$router.push({path: '/course/manager/'+id})
      },
      //获取课程列表
      getCourse(){
        courseApi.findCourseList(this.page,this.size,{}).then((res)=>{
          console.log(res);
          if(res.success){
            this.total = res.queryResult.total;
            this.courses = res.queryResult.list;
          }
        });
      },
      //分页方法
      handleCurrentChange(val){
        this.page = val;
        this.getCourse();
      }
    },
    created() {
    },
    mounted() {
      //查询我的课程
      this.getCourse();
    }
  }
</script>
<style scoped>
   .el-col-8 {
      width: 20%;
   }
   .el-col-offset-2 {
     margin-left:2%;
   }
   .time {
      font-size: 13px;
      color: #999;
   }
   .bottom {
     margin-top: 13px;
     line-height: 12px;
   }
   .button {
     padding: 0;
     float: right;
   }
   .image {
     width: 100%;
     display: block;
   }
   .clearfix:before,
   .clearfix:after {
     display: table;
     content: "";
   }
   .clearfix:after {
     clear: both;
   }

</style>
