<template>
  <div>
    <el-form :model="courseForm" label-width="80px" :rules="courseRules" ref="courseForm">
      <el-form-item label="课程名称" prop="name">
        <el-input v-model="courseForm.name" auto-complete="off"></el-input>
      </el-form-item>
      <el-form-item label="适用人群" prop="users">
        <el-input type="textarea" v-model="courseForm.users" auto-complete="off"></el-input>
      </el-form-item>
      <el-form-item label="课程分类" prop="categoryActive">
        <el-cascader
          expand-trigger="hover"
          :options="categoryList"
          v-model="categoryActive"
          :props="props">
        </el-cascader>
      </el-form-item>
      <el-form-item label="课程等级" prop="grade">
        <b v-for="grade in gradeList">
          <el-radio v-model="courseForm.grade" :label="grade.sdId">
            {{grade.sdName}}
          </el-radio> &nbsp; &nbsp;
        </b>
      </el-form-item>
      <el-form-item label="学习模式" prop="studymodel">
        <b v-for="studymodel_v in studymodelList">
          <el-radio v-model="courseForm.studymodel" :label="studymodel_v.sdId">
            {{studymodel_v.sdName}}
          </el-radio> &nbsp; &nbsp;
        </b>
      </el-form-item>
      <el-form-item label="课程介绍" prop="description">
        <el-input type="textarea" v-model="courseForm.description"></el-input>
      </el-form-item>
    </el-form>

    <div slot="footer" class="dialog-footer">
      <el-button type="primary" @click.native="save">提交</el-button>
    </div>

  </div>
</template>
<script>
  //查询数据字典数据字典为共用方法,写在了base/api/system.js中
  import * as systemApi from '../../../base/api/system';
  import utilApi from '../../../common/utils';
  import * as courseApi from '../api/course';
  export default {
    data() {
      return {
        studymodelList:[],
        gradeList:[],
        props: {
          value: 'id',
          label: 'label',
          children: 'children'
        },
        categoryList: [],
        //用户选择课程分类后，所选分类ID绑定到categoryActive（数组）中，选择了一级、二级分类，分别存储在 categoryActive数组的第一个、第二个元素中。
        categoryActive: [],
        courseForm: {
          id: '',
          name: '',
          users: '',
          grade: '',
          studymodel: '',
          mt: '',
          st: '',
          description: ''
        },
        courseRules: {
          name: [
            {required: true, message: '请输入课程名称',trigger: 'blur'}
          ],
          category: [
            {required: true, message: '请选择课程分类',trigger: 'blur'}
          ],
          grade: [
            {required: true, message: '请选择课程等级',trigger: 'blur'}
          ],
          studymodel: [
            {required: true, message: '请选择学习模式',trigger: 'blur'}
          ]
        }
      }
    },
    methods: {
      //定义save方法,
      save(){
        console.log("提交了课程");
        this.$refs.courseForm.validate((valid)=>{
          if(valid){
            this.$confirm('确认提交吗?','提示',{}).then(() =>{
              //当前选择的分类
              let mt = this.categoryActive[0];
              let st = this.categoryActive[1];
              this.courseForm.mt = mt;
              this.courseForm.st = st;

              //请求服务接口
              courseApi.addCourseBase(this.courseForm).then((res)=>{
                if(res.success){
                  this.$message.success('提交成功');
                  //跳转到课程图片
                  // this.$router.push({path: '/course/add/picture/1/'+this.courseId});
                }else{
                  if(res.message){
                    this.$message.error(res.message);
                  }else{
                    this.$message.error('提交失败');
                  }
                }
              });
            });
          }
        });
      }
    },
    created(){

    },
    mounted(){
      //查询数据字典
      //dType='201' 查询的结果为:学习模式
      systemApi.sys_getDictionary('201').then((res)=>{
        this.studymodelList = res.dvalue;
      });
      //dType='200' 查询的结果是:课程等级
      systemApi.sys_getDictionary('200').then((res)=>{
        this.gradeList = res.dvalue;
      });

      //查询课程分类
      courseApi.category_findlist({}).then((res) =>{
        this.categoryList = res.children;
      })

    }
  }
</script>
<style scoped>


</style>

