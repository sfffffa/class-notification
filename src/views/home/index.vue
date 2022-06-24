<template>
  <el-container>
    <!-- <el-header>Header</el-header> -->
    <el-main>
      <el-card style="margin-top:1%; margin-bottom:1%; ">
        <el-row>
          <el-col :span="2">
            <el-avatar 
              src="https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png"
              :size="80"></el-avatar>
          </el-col>
          <el-col :span="2">
            <h1 style="text-align: left; margin: 10% 0;">{{name}}</h1>
            <h3 style="text-align: left; margin: 10% 0;">5班</h3>
          </el-col>
        </el-row>
      </el-card>
      <el-card style="margin-top:1%; margin-bottom:1%; ">
        <el-row class="row" style="margin-bottom: 1%">
          <el-row style="margin-bottom:2%">
            <el-col :span="4" style="padding-left:5px">
              <h1 style="text-align: left; margin: 0% 0;">筛选条件</h1>
            </el-col>
            <el-col :span="2" :offset='18' style="align: right">
              <el-button style="width:70%" @click="reset">重置</el-button>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="9" :offset="0">
              <div>
                通告标题                
                <el-input v-model="filter.title" placeholder="请输入标题" style='max-width:70% '></el-input>
              </div>
            </el-col>
            <el-col :span="8">  
              <div>
                发布时间
                <el-date-picker
                  v-model="filter.time"
                  @change="change"
                  type="daterange"
                  align="right"
                  unlink-panels
                  range-separator="至"
                  start-placeholder="开始日期"
                  end-placeholder="结束日期">
                </el-date-picker>
              </div>
            </el-col>
            <el-col :span="7" :offset="0">
              <div>
                <el-checkbox-group v-model="filter.type" style='max-width:70%'>
                  <el-checkbox-button 
                    v-for="label in types.map((item)=>{return item.name})" 
                    :label="label" 
                    :key="label">
                    {{label}}
                  </el-checkbox-button>
                </el-checkbox-group>
              </div>
            </el-col>
          </el-row>
        </el-row>
        <el-row class="row">
          <el-table
            :data="searched"
            stripe
            style="width: 100%">
            <el-table-column
              label="公告标题"
              min-width="240">
              <template slot-scope="scope">
                <el-button type="text" style="font-size: large;" @click="notiClick(scope.row)">
                  {{scope.row.title}}
                </el-button>
              </template>
            </el-table-column>
            <el-table-column
              label="发布时间"
              min-width="180">
              <template slot-scope="scope">
                {{scope.row.time.replace(/T/,' ')}}
              </template>
            </el-table-column>
            <el-table-column
              label="标签"
              min-width="120">
              <template slot-scope="scope">
                <el-tag>{{scope.row.type}}</el-tag>
              </template>
            </el-table-column>
            <el-table-column
              align="right">
              <template slot="header" slot-scope="scope">
                <i class="el-icon-plus" @click="notiVisible = true"></i>
              </template>
            </el-table-column>
          </el-table>
        </el-row>
      </el-card>

      <el-dialog
        :title="curNoti[0].title"
        :visible.sync="dialogVisible"
        width="30%">
        <el-input  placeholder="请输入内容" v-show="isModify" v-model="curNoti[0].content"></el-input>
        <p style='font-family:"PingFang SC"' v-show="!isModify" >{{curNoti[0].content}}</p>
        <span slot="footer" class="dialog-footer">
          <span style="margin-right:27%">浏览{{curNoti[0].browseCount}}次</span>
          <span style="margin-right:27%">热度：{{curNoti[0].likeCount}}</span>
          <el-button type="primary" @click="dialogVisible = false" v-if="!canmodify" >确 认</el-button>
          <el-button type="primary" @click="isModify=!isModify" v-if="canmodify&&!isModify" >修 改</el-button>
          <el-button type="primary" @click="modify" v-if="canmodify&&isModify" >确 认</el-button>
        </span>
      </el-dialog>

      <el-dialog
        title="新增通告"
        :visible.sync="notiVisible"
        width="60%">
        <el-form ref="form" :model="form" label-width="80px">
          <el-form-item label="通告标题">
            <el-input v-model="form.title" placeholder="请输入标题"></el-input>
          </el-form-item>
          <el-form-item label="通告内容">
            <el-input type="textarea" v-model="form.content" placeholder="请输入内容"></el-input>
          </el-form-item>
          <el-form-item label="标签">
            <el-select v-model="form.type" style="width:100%" placeholder="请选择标签">
              <el-option v-for="label in types" :key="label.id" :label="label.name" :value="label.id"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button type="primary" @click="create">确 定</el-button>
          <el-button @click="notiVisible = false">取 消</el-button>
        </span>
      </el-dialog>

    </el-main>
  </el-container>
</template>

<script>

export default ({
  data() {
    return {
      notifications:[
        {
          title: "关于近期教学工作安排的通知",
          time: "2022-6-22 1:41:23",
          type: ['教学','疫情']
        },
        {
          title: "关于近期哈哈工作安排的通知",
          time: "2022-6-22 1:41:23",
          type: ['教学']
        }
      ],
      types:[],
      filter:{
        title:'',
        type:[],
        time:'',
        status:''
      },
      dialogVisible: false,
      curNotiId:'',
      checkboxGroup:[],
      isModify: false,
      id:'',
      name:'cyf',
      notiVisible:false,
      form:{
        title:'',
        content:'',
        type:''
      }
    }
  },
  computed:{
    searched(){
      return this.notifications.filter((item)=>{
        if(this.filter.time==''){
          return item.title.includes(this.filter.title)&&this.filter.type.includes(item.type);
        }
        else{
          return item.title.includes(this.filter.title) &&
            this.filter.type.includes(item.type) &&
            this.filter.time[0].getTime() <= (new Date(item.time)).getTime() &&
            this.filter.time[1].getTime() >= (new Date(item.time)).getTime();
        }
      })
    },
    curNoti(){
      if(this.curNotiId==''){
        return [{
          title: '',
          content: '',
          hot: 0,
        }];
      }
      let hot=0;
      this.$axios
        .post("/api/Notification/LikeCount?blogid="+this.curNotiId)
        .then((response)=>{
          console.log("huai")
          console.log(response)
          hot=response.data.data.likeCount
        })
      return this.notifications.filter((item)=>{
        // console.log(item);
        if(item.id==this.curNotiId){
          item.hot=hot;
        }
        return item.id==this.curNotiId;
      })
    },
    canmodify(){
      // console.log(this.curNoti[0])
      return this.id==this.curNoti[0].writerId;
    }
  },
  methods: {
    getNotifications(){
      this.$axios
        .get("/api/Type/Types")
        .then((response)=>{
          this.types=response.data.data;

          this.$axios
            .get("/api/Notification/GetNotificationPage",{
              params:{
                page:1,
                size:10
              }
            })
            .then((response)=>{
              // console.log(response.data.data);
              this.notifications=response.data.data
              .map((item)=>{
                // item.time=item.time.replace(/T/,' ');
                item.type=this.types[item.typeId-1].name;
                return item;
              })
              this.filter.type=this.types.map((item)=>{
                return item.name;
              })
              // console.log("1")
              // console.log(this.filter.type)
            })
        })
      
      
    },
    change(){
      console.log(this.filter.time[0])
      console.log(this.filter.time[1])
    },
    notiClick(row){
      // console.log(row.id)
      this.$axios
        .post("/api/Notification/BrowseCount?blogid="+row.id)
        .then((response)=>{
          // console.log("hao")
          // console.log(response)
        })

      this.dialogVisible=true;
      for(let i=0;i<this.notifications.length;++i){
        if(this.notifications[i].id==row.id){
          ++this.notifications[i].browseCount;
        }
      }
      
      this.curNotiId=row.id;
      // console.log(this.curNoti);

    },
    reset(){
      this.filter.title='';
      this.filter.type=this.types.map((item)=>{ return item.name; });
      this.filter.time='';
      this.filter.status='';
    },
    modify(){
      this.isModify=!this.isModify;
      this.$axios
        .put("/api/Notification/Edit?id="+
          this.curNotiId+"&title="+
          this.curNoti[0].title+"&content="+
          this.curNoti[0].content+"&typeid="+
          this.curNoti[0].typeId)
        .then((response)=>{
          // console.log(response);
        })
      this.getNotifications();
    },
    create(){
      this.notiVisible = false;
      console.log("/api/Notification/Create?writerid="+this.id+
        "&title="+this.form.title+
        "&content="+this.form.content+
        "&typeid="+this.form.type)
      this.$axios
        .post("/api/Notification/Create?writerid="+this.id+
        "&title="+this.form.title+
        "&content="+this.form.content+
        "&typeid="+this.form.type)
        .then((response)=>{
          console.log(response)
        })
      this.getNotifications();
    }
  },
  mounted() {
    this.getNotifications();
    this.id=this.$route.query.id;
    this.name=this.$route.query.name;
    // console.log("12345")
    // console.log(this.$route.query.name)
  }
})
</script>


<style scoped>
.el-card{
  padding: 1%;
  margin: 2%
}

.row{
  padding:1.5%;
  border:1px solid rgba(0, 0, 0, .12);
  border-radius: 3px;
  
}

.el-table{
  font-size: larger;
}

</style>