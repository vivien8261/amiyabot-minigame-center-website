<template>
    <div class="join-room-page">
        <NotificationBanner/>
        <h2>加入房间</h2>
        <div class="input-group">
            <input v-model="roomNumber" type="text" placeholder="请输入房间号或邀请链接" 
                @input="handleInput"/>
        </div>
        <button @click="joinRoom" class="join-room-button">加入房间</button>
        <button @click="goBack" class="back-button">返回首页</button>
    </div>
</template>

<script lang="ts" setup>
import { ref ,onMounted,onUnmounted} from 'vue';
import { useRouter } from 'vue-router';
import {ElMessage} from 'element-plus';
import { invokeGameHub,addGameHubListener,removeGameHubListener,isConnected } from '@src/api/SignalR.ts';
import { getGame } from '@src/api/Game.ts';
import NotificationBanner from '@src/components/SystemNotificationCarousel.vue';

const router = useRouter();

const roomNumber = ref('');

var check_connection = () => {
  if (!isConnected()) {
    router.push('/regular-home');
    return;
  }
  setTimeout(check_connection, 500);
}

check_connection()

function handleInput(event:any) {
    const pastedText = event.target.value;
    const urlPattern = /\/#\/regular-home\/.*\/.*\?joinCode=(\d+)/;
    const match = pastedText.match(urlPattern);
    if (match && match[1]) {
        // 提取 joinCode 的值并将输入框内容改写为该值
        roomNumber.value = match[1];
      }
}

function joinRoom() {
    console.log(`加入房间号为 ${roomNumber.value} 的房间`);
    invokeGameHub('JoinGame', roomNumber.value);
}

var gameJoinListener = (response:any) => {
    
    var playerJoined = response.JoinedPlayer.Id;
    if(playerJoined != localStorage.getItem('user-id')){
        //意外收到别人的消息,不处理
        return;
    }

    var gameId = response.Game.Id;
    localStorage.setItem('current-game-id', gameId);    
    // 跳转到房间等待页面
    router.push('/regular-home/room-waiting/'+gameId);
}

var alertListner = (response:any) => {
    //Toast
    ElMessage({
        message: response.Message,
        type: 'error',
    });
}

onMounted(async () => {
    addGameHubListener('PlayerJoined', gameJoinListener);
    addGameHubListener('Alert', alertListner);

    if(localStorage.getItem('current-game-id')&&localStorage.getItem('current-game-id')!==""){
    const gameId = localStorage.getItem('current-game-id')??"";
    const game = await getGame( gameId );
    roomNumber.value=game.joinCode;
    invokeGameHub('JoinGame', roomNumber.value);
}
});

onUnmounted(() => {
    removeGameHubListener('PlayerJoined', gameJoinListener);
    removeGameHubListener('Alert', alertListner);
});

function goBack() {
    console.log('返回首页');
    localStorage.removeItem('current-game-id');
    router.push('/regular-home');
}
</script>

<style scoped>
.join-room-page {
    text-align: center;
    padding: 20px;
}

.input-group {
    margin: 20px;
}

input {
    padding: 10px;
    width: 200px;
    margin-bottom: 20px;
    font-size: 16px;
}

button {
    padding: 10px 20px;
    margin: 5px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;
}

.back-button {
    background-color: #D32F2F;
}

button:hover {
    background-color: #45a049;
}

.back-button:hover {
    background-color: #C62828;
}
</style>