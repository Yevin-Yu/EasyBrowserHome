<template>
    <div class="user-main" @click="openUserInfo">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 1024 1024">
            <path fill="currentColor"
                d="M628.736 528.896A416 416 0 0 1 928 928H96a415.872 415.872 0 0 1 299.264-399.104L512 704zM720 304a208 208 0 1 1-416 0 208 208 0 0 1 416 0">
            </path>
        </svg>
        <span>{{ isLogin ? userInfo.username : '未登录' }}</span>
    </div>

    <!-- 用户弹窗 -->
    <el-dialog v-model="isShowUserInfo" :show-close="false" :title="isLogin ? '个人信息' : '登录'" width="500">
        <!-- 登录 -->
        <div v-if="!isLogin">
            <div class="login-content">
                <h3>邮箱</h3>
                <input type="text" v-model="loginParams.email" />
                <h3>密码</h3>
                <input type="password" v-model="loginParams.password" />
            </div>
            <div class="dialog-footer">
                <el-button @click="openRegister">注册</el-button>
                <el-button type="primary" :class="{ 'is-loading-btn': isLoginLoading }" :loading="isLoginLoading"
                    @click="subLogin"> 登录 </el-button>
            </div>
        </div>
        <!-- 个人信息 -->
        <div v-else class="user-content">
            <ul>
                <li><span>用户名：</span><span>{{ userInfo.username }}</span></li>
                <li><span>邮箱：</span><span>{{ userInfo.email }}</span></li>
                <li>
                    <span>最后登录：</span>
                    <span>
                        {{ userInfo.last_login ? formatDate(userInfo.last_login, 'YYYY-MM-DD HH: mm: ss') : '' }}
                    </span>
                </li>
            </ul>
            <div class="dialog-footer">
                <el-button @click="openChangePassowrd">修改密码</el-button>
                <el-button type="primary" @click="loginOut">登出</el-button>
            </div>
        </div>
    </el-dialog>
    <!-- 修改密码 -->
    <el-dialog v-model="isShowChangePassword" :show-close="false" title="修改密码" width="500">
        <div>
            <div class="login-content">
                <h3>旧密码</h3>
                <input type="password" v-model="resetPasswordParams.old_password" />
                <h3>新密码</h3>
                <input type="password" v-model="resetPasswordParams.new_password" />
                <h3>确认新密码</h3>
                <input type="password" v-model="resetPasswordParams.confirm_password" />
            </div>
            <div class="dialog-footer">
                <el-button @click="isShowChangePassword = false">取消</el-button>
                <el-button type="primary" :class="{ 'is-loading-btn': isChangeLoading }" :loading="isChangeLoading"
                    @click="changePassword">确认</el-button>
            </div>
        </div>
    </el-dialog>
    <!-- 注册 -->
    <el-dialog v-model="isShowRegister" :show-close="false" title="注册" width="500">
        <div>
            <div class="login-content">
                <h3>用户名</h3>
                <input type="text" v-model="registerParams.username" />
                <h3>邮箱</h3>
                <input type="text" v-model="registerParams.email" />
                <h3>密码</h3>
                <input type="password" v-model="registerParams.password" />
                <h3>确认密码</h3>
                <input type="password" v-model="registerParams.confirm_password" />
                <h3>注册码</h3>
                <input type="password" v-model="registerParams.code" />
            </div>
            <div class="dialog-footer">
                <el-button @click="isShowRegister = false">取消</el-button>
                <el-button type="primary" :class="{ 'is-loading-btn': isRegisterLoading }" :loading="isRegisterLoading" @click="subRegister">确认</el-button>
                <p class="tips">注册码可以联系作者获取：iyuwb0521@outlook.com</p>
            </div>
        </div>
    </el-dialog>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from 'element-plus'
import { storeToRefs } from 'pinia'
import api from '@/api/api'; // 导入封装的 API
// 时间格式化
import { formatDate } from '@/hook/useFormatDate'
// 获取用户信息
import { useUserStore } from '@/stores/useAuthStore'
let { updateIsLogin, updateUserInfo } = useUserStore()
let { isLogin, userInfo } = storeToRefs(useUserStore());
// 弹窗登陆以及信息相关
let isShowUserInfo = ref(false);
let loginParams = ref({ email: '', password: '' })
function openUserInfo() {
    isShowUserInfo.value = true
}
// 登录
const isLoginLoading = ref(false)
async function subLogin() {
    if (loginParams.value.email == '' || loginParams.value.password == '') {
        return ElMessage({
            message: '邮箱或密码不能为空',
            type: 'warning',
        })
    }
    isLoginLoading.value = true
    let data = await api.post('/login', loginParams.value).catch(() => {
        isLoginLoading.value = false
    })
    if (data && data.code == 200) {
        ElMessage({
            message: '登录成功',
            type: 'success',
        })
        isShowUserInfo.value = false
        localStorage.setItem('token', data.data.token);
        updateIsLogin(true)
        updateUserInfo()
        isLoginLoading.value = false
    } else {
        ElMessage({
            message: data && data.message || '系统错误，请稍后再试',
            type: 'error',
        })
        isLoginLoading.value = false
    }
}
// 登出
function loginOut() {
    localStorage.removeItem('token');
    updateIsLogin(false)
}

// 修改密码
const isShowChangePassword = ref(false)
const resetPasswordParams = ref({ old_password: '', new_password: '', confirm_password: '' })
function openChangePassowrd() {
    isShowUserInfo.value = false
    isShowChangePassword.value = true
}
const isChangeLoading = ref(false)
function changePassword() {
    if (resetPasswordParams.value.old_password == '' || resetPasswordParams.value.new_password == '' || resetPasswordParams.value.confirm_password == '') {
        return ElMessage({
            message: '请输入完整信息',
            type: 'warning',
        })
    }
    if (resetPasswordParams.value.new_password != resetPasswordParams.value.confirm_password) {
        return ElMessage({
            message: '两次密码不一致',
            type: 'warning',
        })
    }
    isChangeLoading.value = true
    api.post('/login/reset', resetPasswordParams.value).then(res => {
        if (res.code == 200) {
            ElMessage({
                message: '修改成功',
                type: 'success',
            })
            isShowChangePassword.value = false
        } else {
            ElMessage({
                message: res.message,
                type: 'error',
            })
        }
        isChangeLoading.value = false
    }).catch(() => {
        ElMessage({
            message: '系统错误，请稍后再试',
            type: 'error',
        })
        isChangeLoading.value = false
    })
}
// 注册
const isShowRegister = ref(false)
const registerParams = ref({ username: '', email: '', password: '', confirm_password: '', code: '' })
function openRegister() {
    isShowUserInfo.value = false
    isShowRegister.value = true
}
const isRegisterLoading = ref(false)
function subRegister() {
    if (registerParams.value.username == '' || registerParams.value.email == '' || registerParams.value.password == '' || registerParams.value.confirm_password == '' || registerParams.value.code == '') {
        return ElMessage({
            message: '请输入完整信息',
            type: 'warning',
        })
    }
    let emailReg = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/;
    if (!emailReg.test(registerParams.value.email)) {
        return ElMessage({
            message: '邮箱格式不正确',
            type: 'warning',
        })
    }
    if (registerParams.value.password != registerParams.value.confirm_password) {
        return ElMessage({
            message: '两次密码不一致',
        })
    }
    isRegisterLoading.value = true
    api.post('/login/register', registerParams.value).then(res => {
        if (res.code == 200) {
            ElMessage({
                message: '注册成功',
                type: 'success',
            })
            isShowRegister.value = false
            isShowUserInfo.value = true
        } else {
            ElMessage({
                message: res.message,
                type: 'warning',
            })
        }
        isRegisterLoading.value = false
    }).catch(() => {
        ElMessage({
            message: '系统错误，请稍后再试',
            type: 'error',
        })
        isRegisterLoading.value = false
    })
}
</script>

<style lang="less" scoped>
.user-main:hover,
.user-main:active {
    box-shadow: var(--sideShadowActive);
}

.user-main {
    cursor: pointer;
    height: 40px;
    line-height: 40px;
    position: absolute;
    right: 24px;
    top: 24px;
    border-radius: 55px;
    background: var(--bgColorDefaut);
    box-shadow: var(--shadow);
    border: var(--border);
    color: var(--fontColor);
    padding: 0 12px;

    svg {
        position: relative;
        top: 7px;
    }

    span {
        text-align: center;
        font-size: 14px;
        display: inline-block;
        padding: 0 10px 0 10px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        vertical-align: top;
    }
}

.login-content {
    h3 {
        color: var(--fontColor);
        padding: 12px;
    }

    input {
        outline: none;
        border: none;
        color: #333;
        background-color: #fff;
        height: 38px;
        width: 320px;
        border-radius: 16px;
        padding: 0 24px;
    }

}

.dialog-footer {
    text-align: center;
}

.tips {
    color: #999;
    font-size: 12px;
    margin-top: 20px;
}

.user-content {
    ul {
        list-style: none;
        color: var(--fontColor);
        font-size: 16px;

        li {
            text-align: left;
            padding: 6px;
            line-height: 24px;

            span:first-child {
                text-align: right;
                display: inline-block;
                width: 120px;
            }
        }
    }
}
</style>