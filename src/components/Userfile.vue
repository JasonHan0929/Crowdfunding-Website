<template>
  <div>
    <nav-bar></nav-bar>
    <div class="user-view">
      <div id="form">
        <div class="face-area">
          <h3>Profile</h3>
          <!-- <img v-if="imageUrl" :src="imageUrl" class="avatar"> -->
          <img :src="pic" class="avator">
          <button type="button" class="btn btn-info btn-lg" data-toggle="modal" data-target="#myModal">Upload photo</button>

          <div id="myModal" class="modal fade" role="dialog">
            <div class="modal-dialog">

              <!-- Modal content-->
              <div class="modal-content">
                <div class="modal-header">
                  <button type="button" class="close" data-dismiss="modal">&times;</button>
                  <h4 class="modal-title">Choose your photo</h4>
                </div>
                <div class="modal-body">
                  <!-- <VueImgInputer v-model="picValue" theme="light" size="middle"></VueImgInputer> -->

                    <!-- action="https://jsonplaceholder.typicode.com/posts/" -->

                  <el-upload
                    class="avatar-uploader"
                    action="/api/upload/postUpload"
                    :show-file-list="false"
                    :on-success="handleAvatarSuccess"
                    :before-upload="beforeAvatarUpload">
                    <img v-if="imageUrl" :src="imageUrl" class="avatar">
                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                  </el-upload>

                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-primary" data-dismiss="modal" @click="Save">Save</button>
                  <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
                </div>
              </div>

            </div>
          </div>

        </div>
        <div class="user-info">
          <h2>Profile Information</h2>
          <span class="info-title">Username</span><el-input class="info-detail" v-model="user.username"></el-input><br>
          <span class="info-title">Email&nbsp&nbsp&nbsp&nbsp&nbsp</span><el-input class="info-detail" v-model="user.email"></el-input><br>
          <span class="info-title">City&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp</span><el-input class="info-detail" v-model="user.city"></el-input><br>
          <span class="info-title">Occupation</span><el-input class="info-detail" v-model="user.occupation"></el-input><br>
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <el-row>
                <img style="width: 40%" src="/static/album/visa.png">
              </el-row>
              <span style="line-height: 36px;">Card Information</span>
             
            </div>
            <div class="cardbox" v-for="(card,index) in cards" :key="card">
                
                <div class="panel panel-default">
                  <div class="panel-body">
                    <div class="sponsor">
                      <h4>Card No.{{index+1}}</h4>
                      <div class="sponsor-content">
                        <strong>Card Number: </strong>
                        <span>{{card.cnum}}</span>
                      </div>
                      <div class="sponsor-content">
                        <strong>Exp date: </strong>
                        <span>{{new Date(card.exp_date).getMonth()}}/{{new Date(card.exp_date).getFullYear()}}</span>
                      </div>
                      <div class="sponsor-content">
                        <strong>Holder: </strong>
                        <span>{{card.holder}}</span>
                        <el-button style="float: right;" type="success" @click="removeCard(card)">Remove card</el-button>
                      </div>

                    </div>
                  </div>
                </div>                

            </div>
            <el-form ref="form" :model="form" label-width="80px">
              <el-form-item label="New Card Number">
                <el-input v-model="newCard.cnum" placeholder="Card Number"></el-input>
              </el-form-item>
              <el-form-item label="New Card Exp Date">
                <el-date-picker type="date" placeholder="Choose Date" v-model="newCard.exp_date" style="width: 100%;"></el-date-picker>
              </el-form-item>
              <el-form-item label="New Card Holder">
                <el-input v-model="newCard.holder" placeholder="Card Holder"></el-input>
              </el-form-item>            
            </el-form>
            <el-button style="float: left; margin-bottom: 3%" type="primary" @click="addCard">Add card</el-button>

          </el-card>
          <span class="info-title">Description</span><el-input class="info-detail bio" v-model="user.Description"></el-input><br>
        </div>
        <el-button type="primary" @click="updateUser">Update</el-button>

      <!-- <template v-if="user">
        <h1>User : {{ user.id }}</h1>
        <ul class="meta">
          <li><span class="label">Created:</span> {{ user.created | timeAgo }} ago</li>
          <li><span class="label">Karma:</span> {{user.karma}}</li>
          <li v-if="user.about" v-html="user.about" class="about"></li>
        </ul>
      </template> -->
      </div>
    </div>
  </div>
</template>

<script>
import VueImgInputer from 'vue-img-inputer'
import NavBar        from '../common/nav-bar'

export default {
  name: 'user-view',
  created(){ // 组件创建时调用
    const userInfo = this.getUserInfo(); // 新增一个获取用户信息的方法
    if(userInfo != null){
      this.user.username = userInfo.name;
      this.user.email = userInfo.email;
      this.user.city = userInfo.city;
      this.user.occupation = userInfo.occupation;
      this.pic = userInfo.pic=="null"?"/static/album/avatar.png":userInfo.pic
      console.log(this.pic)
    }else{
      this.id = '';
      this.name = ''
    }
  },
  data () {
    return {
      user: {
        // paramId: 'user id from params:' + this.$route.params.id,
        username: '',
        email: '',
        city: '',
        occupation: '',
        bio: '*',
        pic:'',
        Description:''
      },
      newCard:{
        cnum:'',
        exp_date:'',
        holder:''
      },
      cards: [],
      pic:'',
      focused: false,
      picValue: '',

      imageUrl: ''


    }
  },
  watch: {
  },
  methods: {

    Save () {
      this.pic=this.imageUrl
    },
    focus () {

    },
    getUserInfo(){ // 获取用户信息
        this.$http.post('/api/card/cardlist', {uid: localStorage.getItem('id')}) // get ownername using id
            .then((res) => {
              console.log(res.data);

              this.cards=res.data     
              }, (err) => {
                  this.$message.error('Oops, try again later！')

            })
        const name = localStorage.getItem('name');
        const city = localStorage.getItem('city');
        const occupation = localStorage.getItem('occupation');
        const email = localStorage.getItem('email');
        const pic = localStorage.getItem('picurl')
        if(name != null && name != 'null'){
          // let decode = jwt.verify(token,'vue-koa-demo'); // 解析token
          return {name: name,city: city,occupation: occupation, email: email, pic: pic} // decode解析出来实际上就是{name: XXX,id: XXX}
        }else {
          return null
        }
      },

    handleAvatarSuccess(res, file) {
      // this.imageUrl = URL.createObjectURL(file.raw);
      this.imageUrl=res.data.pictureUrl;
      console.log(res.data.pictureUrl);
      let urlobj = {
        "thisname": localStorage.getItem('name'),
        "upic": res.data.pictureUrl
      }
      this.$http.post('/api/user/update', urlobj)
        .then((res) => {
                console.log(res);
        
              })
    },
    beforeAvatarUpload(file) {
      // const isJPG = file.type === 'image/jpeg';
      const isLt2M = file.size / 1024 / 1024 < 2;

      // if (!isJPG) {
      //   this.$message.error('JPG only');
      // }
      if (!isLt2M) {
        this.$message.error('Avator must be smaller than 2MB!');
      }
      return isLt2M;
    },
    updateUser(){
      let userobj = {
        thisname: localStorage.getItem('name'),
        uname: this.user.username,
        uemail: this.user.email,
        ucity: this.user.city,
        uoccupation: this.user.occupation,
        upic: this.pic
      }
      this.$http.post('/api/user/update', userobj)
        .then((res) => {
          console.log(res);
          if(res.data.success){ // 如果成功
             
            localStorage.setItem('name',this.user.username);
            localStorage.setItem('email',this.user.email);
            localStorage.setItem('city',this.user.city);
            localStorage.setItem('occupation',this.user.occupation);
            localStorage.setItem('picurl',this.pic);
            this.$message({ // 登录成功，显示提示语
              type: 'success',
              message: 'Update successfully！'
            });
            alert('submit!');

            this.$router.go('/userfile') 
          }else{
              this.$message.error(res.data.info); // 
            }
          }, (err) => {
              this.$message.error('Oops, try again later！')

        })
    },
    addCard() {
        let temp=new Date(this.newCard.exp_date)
        console.log(temp.getFullYear()+'-'+temp.getMonth()+'-'+temp.getDate())
        let obj={
          uid: localStorage.getItem('id'),
          cnum: this.newCard.cnum,
          exp_date: temp.getFullYear()+'-'+temp.getMonth()+'-'+temp.getDate(),
          holder: this.newCard.holder
        }
        this.$http.post('/api/card/addCard', obj)
          .then((res) => {
            if(res.data.success){ // 如果成功
                this.$message({ // 登录成功，显示提示语
                type: 'success',
                message: 'Add successfully！'
              });

              this.$router.go('/userfile') 
            }else{
                this.$message.error('Card existed'); // 
              }
            }, (err) => {
                this.$message.error('Oops, try again later！')

        })
    },
    removeCard(card){
      let obj={
        uid: card.uid,
        cnum: card.cnum
      }
      this.$http.post('/api/card/deleteCard', obj)
          .then((res) => {
            if(res.data.success){ // 如果成功
                this.$message({ // 登录成功，显示提示语
                type: 'success',
                message: 'Remove successfully！'
              });

              this.$router.go('/userfile') 
            }else{
                this.$message.error(res.data.info); // 
              }
            }, (err) => {
                this.$message.error('Oops, try again later！')

        })

    }

  },

  components:{
    VueImgInputer,
    NavBar
  }
  // preFetch: fetchUser,
  // beforeMount () {
  //   fetchUser(this.$store)
  // }
}
</script>

<style lang="stylus" scoped>
.user-view
  width 720px
  margin-top 50px
  margin-left auto
  margin-right auto
  padding 0
  padding-bottom 60px
  #form
    .face-area
      width 100px
      margin auto
      text-align center
      font 12px/1.3 'Arial','Microsoft YaHei'
      img
        width 100px
        height 100px
        display block
        margin-bottom 20px
        border-radius 100%
      .focus
        width 92px
        height 32px
        display inline-block
        text-align center
        font 12px/1.3 'Arial','Microsoft YaHei'
        font-size 14px
        word-spacing 2px
        color #fff
        box-shadow 0px 1px 2px rgba(0, 0, 0, 0.3)
        border-radius 3px
        background-color #f4722e
        border 1px solid #f4722e
        cursor pointer
    .user-info
      margin-top 50px
      margin-left 110px
      padding-right 60px
      font 12px/1.3 'Arial','Microsoft YaHei'
      h2
        padding-bottom 8px
        border-bottom 1px solid #d1d1d1
        font 18px/1.3 'Arial','Microsoft YaHei'
        font-weight 600
      span
        line-height 36px
      .info-title
        width 80px
        float left
        text-align right
        font-size 20px
        color #808080
      .info-detail
        display block
        
        font-size 18px
        color #333

  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #20a0ff;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
  .cardbox{
    text-align: left
  }
  .box-card {
    font-size: 15px
    text-align: left;
    width: 480px;
  }

</style>
