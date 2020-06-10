<template>
  <div>
    <!-- 课程图片修改页面 -->
    <!--
        el-upload参数说明：
        action：必选参数，上传的地址
        list-type：文件列表的类型（text/picture/picture-card）
        before-upload：上传前执行钩子方法 ，function(ﬁle)
        on-success：上传成功 执行的钩子方法 ，function(response, ﬁle, ﬁleList)
        on-error：上传失败的钩子方法，function(err, ﬁle, ﬁleList)
        on-remove：文件删除的钩子方法，function(ﬁle, ﬁleList)
        ﬁle-list：文件列表，此列表为上传成功 的文件
        limit：最大允许上传个数
        on-exceed：文件超出个数限制时的钩子，方法为：function(ﬁles, ﬁleList)
        data：提交上传的额外参数，需要封装为json对象，最终提交给服务端为key/value串
      -->
    <el-upload
      action="/api/filesystem/upload"
      list-type="picture-card"
      :before-upload="setuploaddata"
      :on-success="handleSuccess"
      :file-list="fileList"
      :limit="picmax"
      :on-exceed="rejectupload"
      :before-remove="handleRemove"
      :data="uploadval"
      name="file"
    >
      <i class="el-icon-plus"></i>
    </el-upload>
  </div>
</template>
<script>
    import * as sysConfig from '@/../config/sysConfig';
    import * as courseApi from '../../api/course';
    import utilApi from '../../../../common/utils';
    import * as systemApi from '../../../../base/api/system';
    export default {
      data() {
        return {
          picmax:1,
          courseid:'',
          dialogImageUrl:'',
          dialogVisible:false,
          fileList:[],
          uploadval:{filetag:"course"},
          imgUrl:sysConfig.imgUrl
        }
      },
      methods: {
        //超出文件上传个数提示信息
        rejectupload(){
          this.$message.error("最多上传"+this.picmax+"个图片");
        },
        //在上传之前设置请求的数据
        setuploaddata(){

        },
        //删除图片
        handleRemove(file,fileList){
          console.log("删除图片======>>>>>>");
          console.log(file);
          // alert('删除');
          /**
           * Promise对象在处理过程中有三种状态：
           * pending：进行中
           * resolved：操作成功
           * rejected: 操作失败
           * 1）方法执行一些业务逻辑。
           * 2）如果操作成功将Promise的状态由pending变为resolved，并将操作结果传出去
           * 3）如果操作失败会将promise的状态由pending变为rejected，并将失败结果传出去。
           */
          return new Promise((resolve,reject)=>{
            //调用删除课程图片的方法
            courseApi.deleteCoursePic(this.courseid).then((res)=>{
              if(res.success){
                this.$message.success("删除成功!");
                resolve(); //通过
              }else{
                this.$message.error(res.message);
                reject(); //拒绝
              }
            });
          });
        },
        //上传成功的钩子方法
        handleSuccess(response,file,fileList){
          console.log(response);
          if(response.success){
            alert('上传成功!');
            //课程图片上传成功,则将课程图片信息保存到数据库中
            //获取课程路径,也就是课程的pic
            let pic = response.fileSystem.filePath;
            courseApi.addCoursePic(this.courseid,pic).then((res)=>{
              console.log("courseid:====="+this.courseid);
              if(res.success){
                this.$message.success("图片信息保存成功!");
              }else {
                this.$message.error("图片信息保存失败!");
              }
            });
          }else {
            //上传失败
            this.handleError();
          }
        },
        //上传失败执行的钩子方法
        handleError(err,file,fileList){
          this.$message.error('上传失败');
          //清空文件队列
          this.fileList = [];
        }
      },
      mounted() {
        //查询课程id
        this.courseid = this.$route.params.courseid;
        //在课程图片页面的mounted钩子方法 中查询课程图片信息，并将图片地址赋值给数据对象
        //查询课程图片
        courseApi.findCoursePicList(this.courseid).then((res)=>{
          console.log(res);
          if(res.pic){
            let name = "图片";
            //imgUrl : http://img.xuecheng.com/
            //url: http://img.xuecheng.com/group1/M00/00/00/wKjTj16CMACAQDcjAABJ2JkHzc8102.jpg
            let url = this.imgUrl +res.pic;
            let fileId = res.courseid;
            console.log("name: "+name+"url: "+url+"fileId: "+fileId);
            //先清空fileList，然后再将数据放入集合之中
            this.fileList = [];
            this.fileList.push({name:name,url:url,fileId:fileId});
          }
          console.log(this.fileList);
        });
      }
    }
</script>
<style>

</style>
