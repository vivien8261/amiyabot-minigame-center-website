<template>
    <div class="app">
        <NotificationBanner />
        <header>
            <h1>兔兔小游戏中心 - 创建房间</h1>
        </header>
        <div class="create-game-page">
            <div class="game-list" :disabled="isGameCreating">
                <button v-for="game in gameList" :key="game.id" @click="selectGame(game.id)" class="game-button"
                    :disabled="game.notAvailable">
                    <img :src="game.image" :alt="game.name" class="game-image">
                    <div class="game-name">{{ game.name }}</div>
                    <div v-if="game.notAvailable">(敬请期待)</div>
                </button>
            </div>
            <div class="operation-group">
                <div class="private-room-switch-group">
                    <label for="private-room-switch">私人房间</label>
                    <el-switch class="private-room-switch" v-model="isPrivateRoom" id="private-room-switch"></el-switch>
                    <el-tooltip content="私人房间只能通过房间号或邀请链接加入，不能从游戏大厅搜到。" placement="top">
                        <el-button circle size="small">
                            <i class="fa-solid fa-question"></i>
                        </el-button>
                    </el-tooltip>

                </div>

                <button @click="goBack" class="back-button">返回首页</button>
            </div>
        </div>
    </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';
import { games } from '@src/api/GamesData.ts';
import { invokeGameHub, addGameHubListener, removeGameHubListener, isConnected } from '@src/api/SignalR.ts';
import NotificationBanner from '@src/components/SystemNotificationCarousel.vue';

const router = useRouter();

// 示例游戏数据，应从后端或其他数据源获取
const gameList = ref(games);
const isGameCreating = ref(false);
const isPrivateRoom = ref(false);

var check_connection = () => {
    if (!isConnected()) {
        router.push('/regular-home');
        return;
    }
    setTimeout(check_connection, 500);
}

check_connection()

function selectGame(gameId: number) {
    console.log(`选择游戏 ${gameId}`);
    const game = gameList.value.find(g => g.id === gameId);
    if (!game) {
        console.error(`未找到游戏 ${gameId}`);
        return;
    }

    // 创建房间并跳转到房间等待页面
    invokeGameHub('CreateGame', game.type, JSON.stringify({ IsPrivate: isPrivateRoom.value }));
    isGameCreating.value = true
}

var gameCreateListener = (response: any) => {
    isGameCreating.value = false

    var gameId = response.Game.Id;
    localStorage.setItem('current-game-id', gameId);

    // 跳转到房间等待页面
    router.push('/regular-home/room-waiting/' + gameId);
}

onMounted(() => {
    addGameHubListener('GameCreated', gameCreateListener);
});

onUnmounted(() => {
    removeGameHubListener('GameCreated', gameCreateListener);
});

function goBack() {
    console.log('返回首页');
    localStorage.removeItem('current-game-id');
    router.push('/regular-home');
}
</script>

<style scoped>
.app {
    text-align: center;
}

.create-game-page {
    text-align: center;
    padding: 20px;
}

.game-list {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: flex-start;
    gap: 20px;
}

.game-button {
    border: none;
    background-color: #ffffff;
}

.game-image {
    width: 100px;
    height: 100px;
    border: none;
    background-color: #f0f0f0;
    border-radius: 10px;
    overflow: hidden;
    position: relative;
    object-fit: cover;
}

.game-name {
    font-size: 20px;
    font-weight: bold;
}

.operation-group {
    margin-top: 20px;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
}

.private-room-switch-group {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin-right: 20px;
}

.private-room-switch-group label {
    margin-right: 10px;
}

.private-room-switch {
    margin-right: 10px;
}

.back-button {
    padding: 10px 20px;
    background-color: #D32F2F;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;
}

.back-button:hover {
    background-color: #C62828;
}
</style>