<script setup lang="ts">
import { inject } from "vue";
import { RouterView } from "vue-router";
// 获取是否是移动端
const isMobile = inject<boolean>('isMobile');
// 响应式布局
import { useResponsiveLayout } from '@/hook/useResponsiveLayout';
useResponsiveLayout({ designWidth: 562.5 }); // 375 * 1.5 = 562.5    


// 初始化数据
if (!localStorage.length || (!localStorage.newsMenu && !localStorage.notesList && !localStorage.navMenu)) {
    fetch("localStorage.json")
        .then((response) => {
            // 检查响应状态，确保请求成功
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            // 解析JSON数据
            return response.json();
        })
        .then((data) => {
            // 遍历JSON对象的每个键值对并存储到localStorage
            for (const key in data) {
                if (data.hasOwnProperty(key)) {
                    localStorage.setItem(key, data[key]);
                }
            }
            console.log("Data stored in localStorage");
        })
        .catch((error) => {
            // 处理错误
            console.error("Error fetching or parsing data:", error);
        });
}
</script>

<template>
    <!-- 主出口 -->
    <div :class="{ 'isMobile': isMobile }">
        <RouterView />
    </div>
</template>

<style>
html,
body,
#app {
    width: 100%;
    height: 100%;
    overflow: hidden;
}

#app {
    overflow: hidden;
    background-color: var(--bgColor);
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
}
</style>
