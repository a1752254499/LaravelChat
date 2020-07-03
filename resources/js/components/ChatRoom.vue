<template>
    <div class="container">
        <a href="?room_id=1" class="btn btn-danger">房间1</a>
        <a href="?room_id=2" class="btn btn-primary">房间2</a>
        <hr class="divider">
        <div class="row m-0">
            <div class="col-md-8 border-1 bg-white overflow-hidden">
                <div class="panel panel-default">
                    <div class="panel-heading py-2 title">聊天室</div>
                    <div class="con">
                        <div class="panel-body">
                            <div class="messages">
                                <div class="media" v-for="(message,index) in messages" :key="index">
                                    <div class="media-left">
                                        <a href="#" class="mr-2">
                                            <img class="media-object avatar-circle" :src="message.avatar" alt="">
                                        </a>
                                    </div>
                                    <div class="media-body">
                                        <p class="time">{{message.time}}</p>
                                        <h4 class="media-heading">{{message.name}}</h4>
                                        {{message.content}}
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-md-4 border-1 bg-white overflow-hidden">
                <div class="panel panel-default">
                    <div class="panel-heading py-2 bg-white title">在线用户</div>
                    <div class="panel-body py-3">
                        <ul class="list-group">
                            <li class="list-group-item" v-for="user in users" :key="user.id">
                                <img class="img-circle" :src="user.avatar" alt="">
                                {{user.name}}
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
        <form @submit.prevent="onSubmit">
            <div class="form-group pt-2">
                <label for="user_id">私聊</label>
                <select id="user_id" class="form-control" v-model="user_id">
                    <option :value="user.id" v-for="user in users" :key="user.id">{{user.name}}</option>
                </select>
            </div>
            <div class="form-group">
                <label for="content">内容</label>
                <textarea id="content" class="form-control" rows="2" v-model="content"></textarea>
            </div>
            <button type="submit" class="btn btn-primary">发送</button>
        </form>
    </div>
</template>

<script>
let ws = new WebSocket("ws://127.0.0.1:7272");
export default {
    data(){
        return{
            messages:[],
            content:'',
            users:[],
            user_id:''
        }
    },
    created:function(){
        ws.onmessage = (e) => {
            let data = JSON.parse(e.data)
            let type = data.type || ''
            switch(type){
                case 'ping':
                    ws.send('peng');
                    break;
                case 'init':
                    axios.post('/init',{client_id:data.client_id})
                    break;
                case 'say':
                    this.messages.push(data.data);
                    this.$nextTick(function(){
                        $('.panel-body').animate({scrollTop: $('.messages').height()});
                    })
                    break;
                case 'users':
                    this.users = data.data;
                    break;
                case 'history':
                    this.messages = data.data;
                    break;
                case 'logout':
                    this.$delete(this.clients,data.client_id);
                    break;
                default:
                    console.log(data)
            }
        }
    },
    methods:{
        onSubmit(){
            axios.post('/say',{content:this.content,user_id:this.user_id})
            this.content = ""
        }
    }
}
</script>

<style scoped>
    .panel-body{
        height:480px;
        overflow:auto;
        -ms-overflow-style: none;
        scrollbar-width: none;
    }
    .panel-body::-webkit-scrollbar{
        display: none;
    }
    .avatar-circle{
        width:64px;
        height:64px;
        border-radius:100%;
    }
    .img-circle{
        width:48px;
        height:48px;
    }
    .time{
        float: right;
    }
    .media{
        margin-top:24px;
    }

    .border-1{
        border: 1px solid rgba(0, 0, 0, 0.125);
        border-radius:0.25rem;
    }
    .title{
        margin:0 -15px;
        padding:0 15px;
        border-bottom:1px solid rgba(0, 0, 0, 0.125);
    }
</style>