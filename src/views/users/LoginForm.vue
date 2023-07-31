<template>
    <div class="login-form-container">
        <p class="form-title"><span>欢迎回到 全文档</span></p>
        <p class="form-subtitle"><span>好久不见，来继续文档畅游之旅吧</span></p>
        <div style="margin-top: 40px;">
            <div class="search-input-top">
                <input type="text" v-model="username" name="username" placeholder="用户名"></input>
            </div>
            <div class="search-input-top">
                <input  type="password" v-model="pwd" name="password" placeholder="密码"
                       @keyup.enter="login"></input>
            </div>
        </div>
        <submit-button
            :src="buttonSrc"
            label="点我确定登录"
            @click="login"
        ></submit-button>
        <p>还没有账号，现在<span class="focus-word">
            <router-link to="/registry">注册一个</router-link>
        </span></p>
    </div>
</template>

<script>

import UserRequest from '@/api/user'
import SubmitButton from '@/components/SubmitButton'
import * as CryptoJS from 'crypto-js';
// import other from '../../utils/other'
// import other from '@/utils/other'
export default {
    name: "LoginForm.vue",
    data() {
        return {
            buttonSrc: require("@/assets/source/upload.png"),
            username: "",
            pwd: "",
            fromRouteName: ""
        }
    },
    components: {
        SubmitButton
    },
    beforeRouteEnter(to, from, next) {
        next(vm => {
            vm.fromRouteName = from.name
        })
    },
    methods: {
        async login() {
            let params = {
                "username": this.username,
                "password": this.pwd
            }
            let data = {
                code: "+xNQOWEnv+XE7F9qbdgAvk6s3wVrXSni5PkipDAHnUthy2Tciq9QEbFL+wpxuWtWc9jgx7eFAPq7//MK0hhlPp3JnvjqgLpHLWolvy5dEL0=",
                grant_type:"password",
                password:this.pwd,
                randomStr: "blockPuzzle",
                scope:"server",
                username:this.username
            }
             // 密码加密
			const user = this.encryption({
				data: data,
				key: 'pigxpigxpigxpigx',
				param: ['password'],
			});
            params.password=user.password
            await UserRequest.postUserLogin(params).then(
                response => {
                    if (response.data == null) {
                        this.$Message.error(response.message);
                    } else {
                        localStorage.setItem("token", response.data.token)
                        localStorage.setItem("id", response.data.userId)
                        localStorage.setItem("username", response.data.username)
                        localStorage.setItem("avatar", response.data.avatar)
                        localStorage.setItem("type", response.data['type'] || '普通用户');
                        if (this.fromRouteName === "Registry") {
                            this.$router.push({
                                path: '/',
                                query: {
                                    userName: this.userName
                                }
                            })
                        } else {
                            this.$router.back()
                        }
                    }
                }
            )
        },
        encryption(params) {
                    let { data, type, param, key } = params;
                    const result = JSON.parse(JSON.stringify(data));
                    if (type === 'Base64') {
                        param.forEach((ele) => {
                            result[ele] = btoa(result[ele]);
                        });
                    } else {
                        param.forEach((ele) => {
                            var data = result[ele];
                            key = CryptoJS.enc.Latin1.parse(key);
                            var iv = key;
                            // 加密
                            var encrypted = CryptoJS.AES.encrypt(data, key, {
                                iv: iv,
                                mode: CryptoJS.mode.CFB,
                                padding: CryptoJS.pad.NoPadding,
                            });
                            result[ele] = encrypted.toString();
                        });
                    }
                return result;
        }
    }
}
</script>

<style scoped lang="scss">
.login-form-container {
    text-align: left;
    font-family: PingFangSC-Medium, PingFang SC, sans-serif;
    font-weight: 500;
    color: #000000;
    padding: 37px 86px 0;

    .form-title {
        font-size: 24px;
        line-height: 33px;
    }
    .form-subtitle {
        margin-top: 11px;
        margin-bottom: 20px;
        font-size: 16px;
        line-height: 22px;
    }

    .search-input-top {
        margin-top: 20px;
        width: 100%;
        height: 45px;
        background: #FFFFFF;
        border-radius: 8px;
        border: 1px solid #000000;
        padding-left: 10px;

        display: flex;
        justify-content: flex-start;
        align-content: center;

        input {
            height: 41px;
            width: calc(100% - 6px);
            text-decoration: none;
            outline: none;
            border: none;
        }
    }

    .focus-word {
        color: #FBD642;
        &:hover {
            cursor: pointer;
        }
    }
}
</style>