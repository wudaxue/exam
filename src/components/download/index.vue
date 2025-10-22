<template>
  <div class="p-6 max-w-xl mx-auto">
    <h2 class="text-lg font-bold mb-4">下载管理器</h2>

    <!-- 添加任务 -->
    <div class="mb-4 space-x-2">
      <button @click="addSingleTask" class="btn">添加单个下载</button>
      <button @click="addBatchTask" class="btn">添加批量下载</button>
    </div>

    <!-- 任务列表 -->
    <div v-for="task in tasks" :key="task.id" class="border p-3 mb-2 rounded">
      <div class="flex justify-between items-center">
        <div>
          <p class="font-semibold">{{ task.name }}</p>
          <p class="text-sm text-gray-600">
            {{ task.status === 'completed' ? '已完成 ✅' : task.progressText }}
          </p>
        </div>
        <div>
          <button
            v-if="task.status === 'pending'"
            @click="startTask(task)"
            class="btn-small"
          >
            开始
          </button>
          <button
            v-if="task.status === 'downloading'"
            disabled
            class="btn-small"
          >
            进行中
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import { downloadSingleFile, downloadFilesWithLimit } from './downloadHelper';

// 下载任务类型
type TaskType = 'single' | 'batch';
type TaskStatus = 'pending' | 'downloading' | 'completed';

interface DownloadTask {
  id: number;
  name: string;
  type: TaskType;
  urls: string[];
  status: TaskStatus;
  progressText: string;
}

const tasks = ref<DownloadTask[]>([]);

let taskId = 1;

// 添加单个下载任务
function addSingleTask(): void {
  tasks.value.push({
    id: taskId,
    name: `单个下载任务 ${taskId}`,
    type: 'single',
    urls: [
      'https://global.a.com/saasbox/resources/png/big-data__59fd7bc4e2ca22ad37d93e0cbe0429c3.png',
    ],
    status: 'pending',
    progressText: '等待中',
  });
  taskId++;
}

// 添加批量下载任务
function addBatchTask(): void {
  tasks.value.push({
    id: taskId,
    name: `批量下载任务 ${taskId}`,
    type: 'batch',
    urls: [
      'https://global.a.com/saasbox/resources/png/3d-story__85c97d075e3d18389f9ccdbe21b9501e.png',
      'https://global.a.com/saasbox/resources/png/big-data__59fd7bc4e2ca22ad37d93e0cbe0429c3.png',
      'https://global.a.com/saasbox/resources/png/3D__519281adecc43b53545308482b0a37cb.png',
      'https://global.a.com/saasbox/resources/png/2d-rigging__76abb69ee06922736b106a1e177a5992.png',
      'https://global.a.com/saasbox/resources/png/video-2-animation__4ae33e71ffa80cf80724fa7e784d97ae.png',
      'https://global.a.com/saasbox/resources/png/yjtp__32bbcd999456dddff16fd1dfa7eadf67.png',
    ],
    status: 'pending',
    progressText: '等待中',
  });
  taskId++;
}

// 开始任务
async function startTask(task: DownloadTask): Promise<void> {
  // 暂停其他下载中任务
  for (const t of tasks.value) {
    if (t.id !== task.id && t.status === 'downloading') {
      t.status = 'pending';
      t.progressText = '等待中';
    }
  }

  task.status = 'downloading';

  if (task.type === 'single') {
    if (typeof task.urls[0] === 'string') {
      await downloadSingleFile(task.urls[0]);
      task.status = 'completed';
      task.progressText = '下载完成 ✅';
    } else {
      task.status = 'failed' as TaskStatus;
      task.progressText = '下载地址无效 ❌';
    }
  } else {
    await downloadFilesWithLimit(task.urls, 5, (completed: number, total: number) => {
      task.progressText = `进度：${completed}/${total}`;
      if (completed === total) {
        task.status = 'completed';
        task.progressText = '批量下载完成 ✅';
      }
    });
  }
}
</script>

<style scoped>
.btn {
  background-color: #3b82f6;
  color: white;
  padding: 6px 12px;
  border-radius: 6px;
}
.btn-small {
  background-color: #10b981;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 13px;
}
</style>