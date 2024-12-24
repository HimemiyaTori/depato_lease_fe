<script setup lang="ts">
import { getCode, getUserInfo, login } from '@/api/user'
import { HOME_URL } from '@/config/config'
import { useUserStore } from '@/store/modules/user'
import { getEnvByName } from '@/utils/getEnv'
import { timeFix } from '@/utils/index'
import { Lock, User } from '@element-plus/icons-vue'
import type { FormInstance } from 'element-plus'
import { ElNotification } from 'element-plus'
import { onMounted, reactive, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const router = useRouter()
const route = useRoute()
const ruleFormRef = ref<FormInstance>()
const userStore = useUserStore()

// 规则表单
const ruleForm = reactive({
  username: 'user',
  password: '123456',
  captchaKey: '',
  captchaCode: '',
})

const loading = ref(false)

// 验证规则
const validateUsername = (rule: any, value: string, callback: any) => {
  if (value === '') {
    callback(new Error('用户名不能为空'))
  } else if (value.length < 4) {
    callback(new Error('用户名长度不能小于4位'))
  } else {
    callback()
  }
}

const validatePassword = (rule: any, value: string, callback: any) => {
  if (value === '') {
    callback(new Error('密码不能为空'))
  } else if (value.length < 6) {
    callback(new Error('密码长度不能小于6位'))
  } else {
    callback()
  }
}

const validateCaptchaCode = (rule: any, value: string, callback: any) => {
  if (value === '') {
    callback(new Error('验证码不能为空'))
  } else {
    callback()
  }
}

const rules = reactive({
  username: [{ required: true, validator: validateUsername }],
  password: [{ required: true, validator: validatePassword }],
  captchaCode: [{ required: true, validator: validateCaptchaCode }],
})

// 验证码数据
const captcha = ref({
  image: '',
  key: '',
})

// 获取验证码
const getCaptcha = async () => {
  try {
    const { data } = await getCode()
    captcha.value = data
    ruleForm.captchaKey = data.key
  } catch (error) {
    console.log(error)
  }
}

// 提交表单
const submitForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return
  formEl.validate(async (valid) => {
    if (!valid) return

    // 在提交表单之前，强制将 captchaCode 转换为小写， 避免大小写问题
    const captchaCodeLower = ruleForm.captchaCode.toLowerCase()

    const loginData = { 
      ...ruleForm,
      captchaCode: captchaCodeLower,
    }

    try {
      loading.value = true
      const { data } = await login(loginData)
      userStore.setToken(data)
      router.replace((route.query.redirect as string) || HOME_URL)

      const userInfo = await getUserInfo()
      userStore.setUserInfo(userInfo.data)

      ElNotification({
        title: `hi, ${timeFix()}!`,
        message: '欢迎回来',
        type: 'success',
      })
    } catch (error) {
      getCaptcha()
    } finally {
      loading.value = false
    }
  })
}

// 在 mounted 时获取验证码
onMounted(() => {
  getCaptcha()
})
</script>


<style scoped lang="scss">
@import './index';
</style>