<template>
  <view class="todo-container">
    <!-- 顶部标题 -->
    <view class="header">
      <text class="title">我的任务清单</text>
      <text class="subtitle">今日事，今日毕</text>
    </view>

    <!-- 添加任务区域 -->
    <view class="add-task">
      <input 
        class="task-input" 
        v-model="newTask" 
        placeholder="添加新任务..." 
        placeholder-class="placeholder"
        @confirm="addTask"
      />
      <button class="add-btn" @click="addTask">
        <text class="add-icon">+</text>
      </button>
    </view>

    <!-- 任务统计 -->
    <view class="stats">
      <view class="stat-item">
        <text class="stat-number">{{ totalTasks }}</text>
        <text class="stat-label">总任务</text>
      </view>
      <view class="stat-item">
        <text class="stat-number">{{ completedTasks }}</text>
        <text class="stat-label">已完成</text>
      </view>
      <view class="stat-item">
        <text class="stat-number">{{ pendingTasks }}</text>
        <text class="stat-label">待完成</text>
      </view>
    </view>

    <!-- 任务列表 -->
    <view class="task-list">
      <view 
        v-for="task in filteredTasks" 
        :key="task.id" 
        class="task-item"
        :class="{ completed: task.completed }"
      >
        <view class="task-checkbox" @click="toggleTask(task.id)">
          <view class="checkbox" :class="{ checked: task.completed }">
            <text v-if="task.completed" class="checkmark">✓</text>
          </view>
        </view>
        
        <view class="task-content">
          <text class="task-text">{{ task.text }}</text>
          <text class="task-time">{{ task.time }}</text>
        </view>
        
        <view class="task-actions">
          <button class="action-btn delete-btn" @click="deleteTask(task.id)">
            <text class="delete-icon">×</text>
          </button>
        </view>
      </view>
    </view>

    <!-- 过滤器 -->
    <view class="filters">
      <button 
        class="filter-btn" 
        :class="{ active: filter === 'active' }"
        @click="filter = 'active'"
      >
        待完成
      </button>
      <button 
        class="filter-btn" 
        :class="{ active: filter === 'completed' }"
        @click="filter = 'completed'"
      >
        已完成
      </button>
      <button 
        class="filter-btn" 
        :class="{ active: filter === 'all' }"
        @click="filter = 'all'"
      >
        全部
      </button>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { onLoad } from '@dcloudio/uni-app';
// 任务接口定义
interface Task {
  id: number;
  text: string;
  completed: boolean;
  time: string;
}

// 响应式数据
const newTask = ref('');
const filter = ref<'all' | 'active' | 'completed'>('all');
const db = uniCloud.databaseForJQL()

// 写死的任务数据 - 您可以在这里修改
// const tasks = ref<Task[]>([
//   { id: 1, text: '学习Vue3框架', completed: true, time: '09:00' },
//   { id: 2, text: '完成项目原型设计', completed: false, time: '10:30' },
//   { id: 3, text: '团队会议讨论需求', completed: false, time: '14:00' },
//   { id: 4, text: '编写技术文档', completed: true, time: '16:00' },
//   { id: 5, text: '健身锻炼', completed: false, time: '18:30' }
// ]);

const tasks = ref<Task[]>([]);

onLoad(() => {
  db.collection('todolist').where(
    '"user_id" == $env.uid'
  ).get().then(res => {
    console.log(res.data);
    tasks.value = res.data;
  });
});

// 计算属性
const totalTasks = computed(() => tasks.value.length);
const completedTasks = computed(() => tasks.value.filter(task => task.completed).length);
const pendingTasks = computed(() => tasks.value.filter(task => !task.completed).length);

const filteredTasks = computed(() => {
    switch (filter.value) {
        case 'all':
            return tasks.value;
        case 'completed':
            return tasks.value.filter(task => task.completed);
        default:
            return tasks.value.filter(task => !task.completed);
    }
});

// 方法
const addTask = () => {
  if (newTask.value.trim()) {
    const now = new Date();
    const timeString = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
    
    tasks.value.push({
      id: Date.now(),
      text: newTask.value,
      completed: false,
      time: timeString
    });

    db.collection('todolist').add({
      text: newTask.value,
      user_id: uniCloud.getCurrentUserInfo().uid,
    });

    newTask.value = '';
  }
};

const toggleTask = (id: number) => {
  const task = tasks.value.find(task => task.id === id);
  if (task) {
    task.completed = !task.completed;
  }
};

const deleteTask = (id: number) => {
  tasks.value = tasks.value.filter(task => task.id !== id);
};
</script>

<style scoped>
.todo-container {
  padding: 30rpx;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  min-height: 100vh;
}

/* 头部样式 */
.header {
  text-align: center;
  margin-bottom: 40rpx;
}

.title {
  display: block;
  font-size: 48rpx;
  font-weight: bold;
  color: #2c3e50;
  margin-bottom: 10rpx;
}

.subtitle {
  display: block;
  font-size: 28rpx;
  color: #7f8c8d;
}

/* 添加任务区域 */
.add-task {
  display: flex;
  margin-bottom: 30rpx;
  background: white;
  border-radius: 20rpx;
  padding: 10rpx;
  box-shadow: 0 4rpx 15rpx rgba(0, 0, 0, 0.1);
}

.task-input {
  flex: 1;
  padding: 20rpx;
  font-size: 32rpx;
  border: none;
  outline: none;
}

.placeholder {
  color: #bdc3c7;
}

.add-btn {
  width: 80rpx;
  height: 80rpx;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
}

.add-icon {
  color: white;
  font-size: 36rpx;
  font-weight: bold;
}

/* 统计区域 */
.stats {
  display: flex;
  justify-content: space-around;
  background: white;
  border-radius: 20rpx;
  padding: 30rpx 20rpx;
  margin-bottom: 30rpx;
  box-shadow: 0 4rpx 15rpx rgba(0, 0, 0, 0.1);
}

.stat-item {
  text-align: center;
}

.stat-number {
  display: block;
  font-size: 36rpx;
  font-weight: bold;
  color: #2c3e50;
}

.stat-label {
  display: block;
  font-size: 24rpx;
  color: #7f8c8d;
  margin-top: 8rpx;
}

/* 任务列表 */
.task-list {
  margin-bottom: 30rpx;
}

.task-item {
  display: flex;
  align-items: center;
  background: white;
  border-radius: 20rpx;
  padding: 30rpx;
  margin-bottom: 20rpx;
  box-shadow: 0 4rpx 15rpx rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.task-item.completed {
  opacity: 0.7;
  background: #f8f9fa;
}

.task-checkbox {
  margin-right: 20rpx;
}

.checkbox {
  width: 40rpx;
  height: 40rpx;
  border: 2rpx solid #bdc3c7;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.checkbox.checked {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: #667eea;
}

.checkmark {
  color: white;
  font-size: 24rpx;
  font-weight: bold;
}

.task-content {
  flex: 1;
}

.task-text {
  display: block;
  font-size: 32rpx;
  color: #2c3e50;
  margin-bottom: 8rpx;
}

.task-item.completed .task-text {
  text-decoration: line-through;
  color: #95a5a6;
}

.task-time {
  display: block;
  font-size: 24rpx;
  color: #7f8c8d;
}

.task-actions {
  margin-left: 20rpx;
}

.action-btn {
  width: 60rpx;
  height: 60rpx;
  border-radius: 50%;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
}

.delete-btn {
  background: #e74c3c;
}

.delete-icon {
  color: white;
  font-size: 28rpx;
  font-weight: bold;
}

/* 过滤器 */
.filters {
  display: flex;
  justify-content: space-around;
  background: white;
  border-radius: 20rpx;
  padding: 20rpx;
  box-shadow: 0 4rpx 15rpx rgba(0, 0, 0, 0.1);
}

.filter-btn {
  padding: 16rpx 32rpx;
  border: none;
  background: transparent;
  border-radius: 25rpx;
  font-size: 28rpx;
  color: #7f8c8d;
  transition: all 0.3s ease;
}

.filter-btn.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}
</style>