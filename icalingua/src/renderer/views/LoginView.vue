<template>
    <div id="login">
        <el-form
            ref="loginForm"
            :model="form"
            :rules="rules"
            :hide-required-asterisk="true"
            :disabled="disabled"
            label-position="left"
        >
            <h1 class="title">
                <span>
                    {{ $route.query.bridge === 'true' ? '配置 Bridge 服务器' : '登录' }}
                </span>
                <span>Version {{ ver }}</span>
            </h1>
            <el-form-item prop="username" v-if="$route.query.disableIdLogin === 'false'">
                <el-input type="text" placeholder="QQ ID" v-model.number="form.username" />
            </el-form-item>
            <el-form-item prop="password" v-if="$route.query.disableIdLogin === 'false'">
                <el-input type="password" placeholder="密码" v-model="form.password" />
            </el-form-item>
            <el-form-item prop="signAPIAddress" v-if="$route.query.disableIdLogin === 'false'">
                <el-input type="text" placeholder="头部签名API地址" v-model="form.signAPIAddress" />
            </el-form-item>
            <el-form-item prop="signAPIKey" v-if="$route.query.disableIdLogin === 'false'">
                <el-input type="text" placeholder="头部签名 API Key" v-model="form.signAPIKey" />
            </el-form-item>
            <el-form-item prop="protocol" label="协议" v-if="$route.query.disableIdLogin === 'false'">
                <!-- shit code -->
                <div class="protocols">
                    <span>Android Phone</span>
                    <el-radio-group v-model="form.protocol" size="mini">
                        <el-radio-button label="10001">8.2.11</el-radio-button>
                        <el-radio-button label="6">8.8.88</el-radio-button>
                        <el-radio-button label="7">8.9.33</el-radio-button>
                        <el-radio-button label="1">8.9.50</el-radio-button>
                        <el-radio-button label="11">8.9.58</el-radio-button>
                        <el-radio-button label="13">8.9.63</el-radio-button>
                        <el-radio-button label="15">8.9.68</el-radio-button>
                        <el-radio-button label="17">8.9.70</el-radio-button>
                        <el-radio-button label="23">8.9.71</el-radio-button>
                        <el-radio-button label="19">8.9.73</el-radio-button>
                        <el-radio-button label="21">8.9.75</el-radio-button>
                        <el-radio-button label="25">8.9.76</el-radio-button>
                        <el-radio-button label="27">8.9.78</el-radio-button>
                        <el-radio-button label="29">8.9.80</el-radio-button>
                        <el-radio-button label="31">8.9.83</el-radio-button>
                        <el-radio-button label="33">8.9.85</el-radio-button>
                        <el-radio-button label="35">8.9.88</el-radio-button>
                        <el-radio-button label="37">8.9.93</el-radio-button>
                        <el-radio-button label="39">9.0.0</el-radio-button>
                        <el-radio-button label="41">9.0.8</el-radio-button>
                        <el-radio-button label="43">9.0.17</el-radio-button>
                        <el-radio-button label="45">9.0.25</el-radio-button>
                        <el-radio-button label="47">9.0.35</el-radio-button>
                        <el-radio-button label="49">9.0.50</el-radio-button>
                        <el-radio-button label="51">9.0.56</el-radio-button>
                        <el-radio-button label="53">9.0.70</el-radio-button>
                        <el-radio-button label="55">9.0.95</el-radio-button>
                        <el-radio-button label="57">9.1.0</el-radio-button>
                        <el-radio-button label="59">9.1.20</el-radio-button>
                    </el-radio-group>
                    <span>Android Pad</span>
                    <el-radio-group v-model="form.protocol" size="mini">
                        <el-radio-button label="8">8.9.33</el-radio-button>
                        <el-radio-button label="2">8.9.50</el-radio-button>
                        <el-radio-button label="12">8.9.58</el-radio-button>
                        <el-radio-button label="14">8.9.63</el-radio-button>
                        <el-radio-button label="16">8.9.68</el-radio-button>
                        <el-radio-button label="18">8.9.70</el-radio-button>
                        <el-radio-button label="24">8.9.71</el-radio-button>
                        <el-radio-button label="20">8.9.73</el-radio-button>
                        <el-radio-button label="22">8.9.75</el-radio-button>
                        <el-radio-button label="26">8.9.76</el-radio-button>
                        <el-radio-button label="28">8.9.78</el-radio-button>
                        <el-radio-button label="30">8.9.80</el-radio-button>
                        <el-radio-button label="32">8.9.83</el-radio-button>
                        <el-radio-button label="34">8.9.85</el-radio-button>
                        <el-radio-button label="36">8.9.88</el-radio-button>
                        <el-radio-button label="38">8.9.93</el-radio-button>
                        <el-radio-button label="40">9.0.0</el-radio-button>
                        <el-radio-button label="42">9.0.8</el-radio-button>
                        <el-radio-button label="44">9.0.17</el-radio-button>
                        <el-radio-button label="46">9.0.25</el-radio-button>
                        <el-radio-button label="48">9.0.35</el-radio-button>
                        <el-radio-button label="50">9.0.50</el-radio-button>
                        <el-radio-button label="52">9.0.56</el-radio-button>
                        <el-radio-button label="54">9.0.70</el-radio-button>
                        <el-radio-button label="56">9.0.95</el-radio-button>
                        <el-radio-button label="58">9.1.0</el-radio-button>
                        <el-radio-button label="60">9.1.20</el-radio-button>
                    </el-radio-group>
                    <span>Watch</span>
                    <el-radio-group v-model="form.protocol" size="mini">
                        <el-radio-button label="30002">2.0.5</el-radio-button>
                        <el-radio-button label="30001">2.0.8</el-radio-button>
                        <el-radio-button label="3">9.0.1</el-radio-button>
                        <el-radio-button label="30003">9.0.3</el-radio-button>
                    </el-radio-group>
                    <span>iPad</span>
                    <el-radio-group v-model="form.protocol" size="mini">
                        <el-radio-button label="9">8.9.33</el-radio-button>
                        <el-radio-button label="5">8.9.50</el-radio-button>
                    </el-radio-group>
                    <span>Other</span>
                    <el-radio-group v-model="form.protocol" size="mini">
                        <el-radio-button label="4">macOS 6.8.2</el-radio-button>
                        <el-radio-button label="10">TIM 3.5.1</el-radio-button>
                    </el-radio-group>
                </div>
            </el-form-item>
            <el-form-item label="状态" v-if="$route.query.disableIdLogin === 'false'">
                <el-radio-group v-model="form.onlineStatus" size="small">
                    <el-radio-button label="11">在线</el-radio-button>
                    <el-radio-button label="31">离开</el-radio-button>
                    <el-radio-button label="41">隐身</el-radio-button>
                    <el-radio-button label="50">忙碌</el-radio-button>
                    <el-radio-button label="60">Q我吧</el-radio-button>
                    <el-radio-button label="70">请勿打扰</el-radio-button>
                </el-radio-group>
            </el-form-item>
            <el-form-item prop="autologin">
                <span class="el-form-item__label">自动登录</span>
                <el-switch v-model="form.autologin" />
            </el-form-item>
            <el-form-item prop="forceAlgoT544">
                <span class="el-form-item__label">使用8.9.50的Tlv544算法</span>
                <el-switch v-model="form.forceAlgoT544" />
            </el-form-item>
            <el-form-item prop="useNT">
                <span class="el-form-item__label">Use NT's register</span>
                <el-switch v-model="form.useNT" />
            </el-form-item>
            <el-form-item label="存储引擎">
                <el-select v-model="form.storageType" size="small">
                    <el-option label="MongoDB" value="mdb">MongoDB</el-option>
                    <el-option label="Redis" value="redis">Redis</el-option>
                    <el-option label="SQLite (内置)" value="sqlite">SQLite (内置)</el-option>
                    <el-option label="MySQL / MariaDB" value="mysql">MySQL / MariaDB</el-option>
                    <el-option label="PostgreSQL" value="pg">PostgreSQL</el-option>
                </el-select>
            </el-form-item>
            <el-form-item prop="connStr" v-show="form.storageType === 'mdb'">
                <el-input
                    :show-password="form.mdbConnStr && form.mdbConnStr.split(':').length > 2"
                    placeholder="MongoDB connect string"
                    v-model="form.mdbConnStr"
                />
            </el-form-item>
            <el-form-item prop="rdsHost" v-show="form.storageType === 'redis'">
                <el-input placeholder="Redis Host" v-model="form.rdsHost" />
            </el-form-item>
            <el-form-item prop="sqlHost" v-show="form.storageType === 'mysql' || form.storageType === 'pg'">
                <el-input placeholder="Host" v-model="form.sqlHost" />
            </el-form-item>
            <el-form-item prop="sqlUsername" v-show="form.storageType === 'mysql' || form.storageType === 'pg'">
                <el-input placeholder="username" v-model="form.sqlUsername" />
            </el-form-item>
            <el-form-item prop="sqlPassword" v-show="form.storageType === 'mysql' || form.storageType === 'pg'">
                <el-input placeholder="password" type="password" v-model="form.sqlPassword" />
            </el-form-item>
            <el-form-item prop="sqlDatabase" v-show="form.storageType === 'mysql' || form.storageType === 'pg'">
                <el-input placeholder="database" v-model="form.sqlDatabase" />
            </el-form-item>
            <p v-if="errmsg" class="error">
                {{ errmsg }}
            </p>
            <el-form-item class="buttons">
                <el-button type="primary" v-on:click="onSubmit('loginForm')">
                    <span v-show="!form.password">二维码</span>
                    登录
                </el-button>
                <el-button type="warning" v-on:click="cannotLogin">更换设备信息</el-button>
            </el-form-item>
        </el-form>
        <QrcodeDrawer @login="onSubmit('loginForm')" />
        <el-drawer
            class="sms-drawer"
            title="短信验证"
            :visible="shouldSubmitSmsCode"
            direction="btt"
            :close-on-press-escape="false"
            :show-close="false"
            :wrapper-closable="false"
            size="100%"
        >
            <p v-if="phone">{{ sendTime !== -1 ? '已' : '' }}向 {{ phone }} 发送验证码</p>
            <el-input
                placeholder="短信验证码"
                v-model="smsCode"
                @input="smsCode = smsCode.slice(0, 6)"
                @keydown.enter.native="submitSmsCode"
            />
            <div class="buttons">
                <el-button @click="submitSmsCode" type="primary" v-if="sendTime !== -1">提交</el-button>
                <el-button @click="sendSmsCode" type="primary" v-if="sendTime === -1">发送验证码</el-button>
                <el-button @click="sendSmsCode" :disabled="sendTime !== 0" v-else>
                    重发{{ sendTime !== 0 ? ` (${sendTime}s)` : '' }}
                </el-button>
                <el-button v-if="verifyUrl" @click="QRCodeVerify">扫码验证</el-button>
            </div>
        </el-drawer>
    </div>
</template>

<script>
import { ipcRenderer } from 'electron'
import ipc from '../utils/ipc'
import md5 from 'md5'
import QrcodeDrawer from '../components/QrcodeDrawer'

let loginTimeout = null
let qrLoginInterval = null

export default {
    name: 'LoginView',
    components: { QrcodeDrawer },
    data() {
        return {
            ver: '',
            /**
             * @type LoginForm
             */
            form: {},
            rules: {
                username: [{ required: true, trigger: 'blur' }],
            },
            disabled: false,
            errmsg: '',
            shouldSubmitSmsCode: false,
            smsCode: '',
            verifyUrl: '',
            phone: '',
            sendTime: -1,
        }
    },
    async created() {
        this.ver = await ipc.getVersion()
        const _form = await ipc.getAccount()
        if (!_form.signAPIAddress) _form.signAPIAddress = ''
        this.form = _form
        ipcRenderer.on('error', (_, msg) => {
            if (loginTimeout) clearTimeout(loginTimeout) && (loginTimeout = null)
            if (qrLoginInterval) clearInterval(qrLoginInterval) && (qrLoginInterval = null)
            this.errmsg = msg
            this.disabled = false
            this.shouldSubmitSmsCode = false

            const tmp = String(msg).split(' ')
            const code = tmp[tmp.length - 1]
            switch (code) {
                case '(235)':
                    this.$alert('设备信息可能被封禁, 请点击 更换设备信息 按钮后重试')
                    break
                case '(237)':
                    this.$alert('账号登录过于频繁，请稍后再试')
                    break
                case '(45)':
                    if (this.form.protocol === 3) break
                    if (String(msg).includes('QQ版本过低'))
                        this.$alert('账号被限制使用内置的 QQ 版本登录，请更换更高版本协议后重试')
                    else
                        this.$alert(
                            this.form.signAPIAddress
                                ? '可能是 API 配置有误或版本不匹配，请检查 API 配置'
                                : '账号被风控需要头部签名，请根据 README 配置头部签名 API 地址',
                        )
                    break
                default:
                    break
            }
        })
        ipcRenderer.on('smsCodeVerify', (_, data) => {
            if (loginTimeout) clearTimeout(loginTimeout)
            const parsed = JSON.parse(data)
            console.log(parsed.url)
            this.shouldSubmitSmsCode = true
            this.verifyUrl = parsed.url
            this.phone = parsed.phone
        })
        ipcRenderer.on('qrcodeLogin', async (_, url) => {
            if (qrLoginInterval) clearInterval(qrLoginInterval)
            qrLoginInterval = setInterval(() => {
                ipcRenderer.send('createBot', this.form)
            }, 5 * 1000)
        })
    },
    methods: {
        onSubmit(formName) {
            this.$refs[formName].validate(async (valid) => {
                if (valid || this.$route.query.disableIdLogin === 'true') {
                    this.disabled = true
                    if (this.form.password && !/^([a-f\d]{32}|[A-F\d]{32})$/.test(this.form.password))
                        this.form.password = md5(this.form.password)
                    if (!this.form.signAPIAddress) {
                        this.$message.warning('未配置签名 API，可能禁止登录或无法发送消息')
                    }
                    if (this.form.useNT) {
                        this.$message('使用实验性 NT 上线，可能不支持部分功能')
                    }
                    if (this.form.password) {
                        if (loginTimeout) clearTimeout(loginTimeout)
                        loginTimeout = setTimeout(() => {
                            this.$alert(
                                '登录时间似乎过长了，请检查网络是否正常，切换非同类协议请先删除 token，若仍无法登录请携带日志反馈',
                            )
                        }, 60 * 1000)
                    }
                    await ipcRenderer.send('createBot', this.form)
                } else {
                    return false
                }
            })
        },
        submitSmsCode() {
            ipcRenderer.send('submitSmsCode', this.smsCode)
        },
        sendSmsCode() {
            ipcRenderer.send('submitSmsCode', 'sendSmsCode')
            this.sendTime = 60
            const timer = setInterval(() => {
                if (this.sendTime === 0) {
                    clearInterval(timer)
                    return
                }
                this.sendTime--
            }, 1000)
        },
        QRCodeVerify() {
            if (loginTimeout) clearTimeout(loginTimeout)
            ipcRenderer.send('QRCodeVerify', this.verifyUrl)
        },
        cannotLogin() {
            this.$confirm(
                '无法登录/发送消息有可能由风控造成，随机生成不同的设备消息或许可以解决，但也有可能造成更严重的风控，是否尝试随机生成?',
                '提示',
                {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                },
            ).then(() => {
                ipcRenderer.send('randomDevice', this.form.username)
                this.$message({
                    type: 'success',
                    message: `已尝试随机生成 ${this.form.username} 的设备消息`,
                })
            })
        },
    },
}
</script>

<style scoped>
#login {
    padding: 15px;
    font-family: 'CircularSpotifyTxT Light Web', sans-serif;
}

#login::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -1;
    background-position: bottom;
    background-repeat: no-repeat;
    background-size: contain;
    background-image: url('../assets/loginbg.jpg');
}

.title {
    font-size: 16px;
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    margin: 0 0 15px;
}

.error {
    color: red;
    margin: 0 0 22px;
}

.protocols {
    clear: left;
    display: grid;
    grid-template-columns: max-content 1fr;
    align-items: center;
    gap: 4px;
}

.protocols span {
    color: #606266;
    font-size: 12px;
    line-height: normal;
}

.buttons {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    margin-bottom: 0;
}

.sms-drawer {
    text-align: center;
}

.sms-drawer :deep(.el-drawer__body) {
    padding: 0 20px 20px;
}

.sms-drawer p,
.sms-drawer .el-input {
    margin: 0 0 15px;
}
</style>
