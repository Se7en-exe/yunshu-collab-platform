<template>
  <div class="login-page">
    <div class="paper-layer" aria-hidden="true">
      <span class="paper-grain"></span>
      <span class="paper-line"></span>
    </div>

    <main class="login-shell">
      <section class="brand-panel">
        <p class="brand-tag">COLLABORATION &amp; WORKFLOW PLATFORM</p>
        <h1>把人、事、单<br>放进同一条线</h1>
        <p class="brand-desc">
          从派任务到审单据，一条链走完；
          每一步都有记录，每个人都知道下一步做什么。
        </p>
        <ul class="brand-metrics">
          <li>
            <strong>任务派发</strong>
            <span>下发 · 接单 · 提交 · 复核</span>
          </li>
          <li>
            <strong>审批流转</strong>
            <span>逐级会签与轨迹回放</span>
          </li>
          <li>
            <strong>逾期提醒</strong>
            <span>到期自动标记并推送</span>
          </li>
          <li>
            <strong>数据看板</strong>
            <span>部门完成率与时效分析</span>
          </li>
        </ul>
      </section>

      <section class="form-panel">
        <div class="panel-header">
          <p class="kicker">SIGN IN</p>
          <h2>登录云枢</h2>
          <p class="subtitle">用你的工号账号进入业务工作台</p>
        </div>

        <el-form
          :model="form"
          :rules="rules"
          ref="formRef"
          label-width="0"
          class="login-form"
        >
          <el-form-item prop="username">
            <el-input
              v-model="form.username"
              placeholder="请输入账号"
              size="large"
              autocomplete="username"
            />
          </el-form-item>

          <el-form-item prop="password">
            <el-input
              v-model="form.password"
              type="password"
              placeholder="请输入密码"
              show-password
              size="large"
              autocomplete="current-password"
              @keyup.enter="handleLogin"
            />
          </el-form-item>

          <el-form-item>
            <el-button
              type="primary"
              size="large"
              class="submit-btn"
              :loading="submitting"
              @click="handleLogin"
            >
              进入系统
            </el-button>
          </el-form-item>

          <div class="action-row">
            <span>没有账号？</span>
            <el-link type="primary" @click="$router.push('/register')">立即注册</el-link>
          </div>
        </el-form>
      </section>
    </main>
  </div>
</template>
<script setup>
import { reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { ElMessage } from 'element-plus'
import { loginAPI } from '@/api/user'

const router = useRouter()
const userStore = useUserStore()
const formRef = ref(null)
const submitting = ref(false)

const form = reactive({ username: '', password: '' })
const rules = {
  username: [{ required: true, message: '账号不可为空', trigger: 'blur' }],
  password: [{ required: true, message: '密码不可为空', trigger: 'blur' }]
}

const normalizeRole = (role) => {
  return String(role || '').trim().toLowerCase()
}

const handleLogin = () => {
  formRef.value.validate(async (valid) => {
    if (valid) {
      submitting.value = true
      try {
        const res = await loginAPI(form)
        if (res.code !== 200 || !res.data?.token) {
          throw new Error(res.msg || '登录失败')
        }

        userStore.setToken(res.data.token)
        userStore.setUserInfo(res.data.userInfo || {})
        const currentRole = normalizeRole(res.data.userInfo?.role)

        const roleText = currentRole === 'superadmin'
          ? '超级管理员'
          : (currentRole === 'admin' ? '管理员' : '普通员工')
        ElMessage.success(`登录成功，当前角色：${roleText}`)
        router.push('/')
      } catch (error) {
        ElMessage.error(error.message || '登录失败')
      } finally {
        submitting.value = false
      }
    }
  })
}
</script>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@500;700&family=Manrope:wght@400;500;700&display=swap');

:global(body) {
  margin: 0;
}

.login-page {
  --paper: #eef4f3;
  --paper-soft: #f6faf9;
  --ink: #12211f;
  --ink-soft: #5a6b68;
  --line: rgba(18, 44, 42, 0.18);
  --accent: #17706a;
  --accent-soft: #d5ece7;

  position: relative;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  padding: 28px;
  font-family: 'Manrope', 'Segoe UI', sans-serif;
  background:
    radial-gradient(circle at 8% 14%, rgba(23, 112, 106, 0.16), transparent 40%),
    radial-gradient(circle at 88% 12%, rgba(127, 227, 207, 0.2), transparent 38%),
    linear-gradient(135deg, #f2f8f7 0%, #e9f2f0 56%, #dfecea 100%);
}

.paper-layer {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.paper-grain {
  position: absolute;
  inset: 0;
  opacity: 0.26;
  background-image: radial-gradient(rgba(0, 0, 0, 0.12) 0.45px, transparent 0.45px);
  background-size: 4px 4px;
}

.paper-line {
  position: absolute;
  inset: 20px;
  border: 1px solid rgba(51, 44, 35, 0.12);
}

.login-shell {
  width: min(1080px, 100%);
  display: grid;
  grid-template-columns: 1.08fr 0.92fr;
  border: 1px solid rgba(51, 44, 35, 0.2);
  border-radius: 6px;
  overflow: hidden;
  background: linear-gradient(145deg, var(--paper) 0%, var(--paper-soft) 100%);
  box-shadow:
    0 14px 34px rgba(63, 49, 38, 0.12),
    0 2px 0 rgba(255, 255, 255, 0.45) inset;
}

.brand-panel {
  position: relative;
  padding: clamp(30px, 5vw, 64px);
  color: var(--ink);
  border-right: 1px solid rgba(51, 44, 35, 0.18);
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.brand-tag {
  font-size: 11px;
  letter-spacing: 0.32em;
  color: var(--accent);
  margin: 0 0 16px;
}

.brand-panel h1 {
  margin: 0;
  font-family: 'Cinzel', 'Times New Roman', serif;
  font-weight: 700;
  line-height: 1.04;
  letter-spacing: 0.01em;
  font-size: clamp(2.2rem, 4.6vw, 4rem);
  text-wrap: balance;
}

.brand-desc {
  margin: 26px 0 34px;
  max-width: 44ch;
  line-height: 1.9;
  color: var(--ink-soft);
}

.brand-metrics {
  margin: 0;
  padding: 0;
  list-style: none;
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
}

.brand-metrics li {
  min-width: 120px;
  padding: 14px 16px;
  border: 1px solid rgba(44, 37, 31, 0.16);
  border-radius: 2px;
  background: rgba(255, 255, 255, 0.3);
}

.brand-metrics strong {
  display: block;
  color: var(--ink);
  font-size: 1.02rem;
  margin-bottom: 6px;
}

.brand-metrics span {
  font-size: 0.82rem;
  color: var(--ink-soft);
}

.form-panel {
  padding: clamp(24px, 5vw, 58px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  color: var(--ink);
}

.panel-header {
  margin-bottom: 28px;
}

.kicker {
  margin: 0 0 12px;
  font-size: 11px;
  letter-spacing: 0.3em;
  color: var(--accent);
}

.panel-header h2 {
  margin: 0 0 8px;
  font-family: 'Cinzel', 'Times New Roman', serif;
  font-size: clamp(1.7rem, 3vw, 2.4rem);
  font-weight: 700;
}

.subtitle {
  margin: 0;
  color: var(--ink-soft);
}

.login-form :deep(.el-form-item) {
  margin-bottom: 18px;
}

.login-form :deep(.el-input__wrapper) {
  border-radius: 2px;
  border: 1px solid rgba(34, 31, 27, 0.18);
  box-shadow: none;
  background: rgba(255, 255, 255, 0.58);
  transition: border-color 0.28s ease, box-shadow 0.28s ease, transform 0.28s ease;
}

.login-form :deep(.el-input__wrapper.is-focus) {
  border-color: rgba(46, 78, 102, 0.58);
  box-shadow: 0 0 0 3px rgba(46, 78, 102, 0.13);
  transform: translateY(-1px);
}

.login-form :deep(.el-input__inner) {
  color: #1f1f1d;
  height: 46px;
}

.login-form :deep(.el-input__inner::placeholder) {
  color: #8a847c;
}

.submit-btn {
  width: 100%;
  border: 1px solid #0f4b47;
  height: 46px;
  border-radius: 2px;
  font-weight: 700;
  letter-spacing: 0.08em;
  background: linear-gradient(90deg, #0f4b47 0%, #17706a 100%);
  color: #f9f5ee;
  box-shadow: 0 8px 16px rgba(39, 60, 77, 0.18);
  transition: transform 0.28s ease, box-shadow 0.28s ease, filter 0.28s ease;
}

.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 22px rgba(39, 60, 77, 0.22);
  filter: brightness(1.03);
}

.submit-btn:active {
  transform: translateY(0);
}

.action-row {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 10px;
  font-size: 0.92rem;
  color: var(--ink-soft);
}

.action-row :deep(.el-link__inner) {
  font-weight: 700;
}

@keyframes drift {
  0% {
    transform: translate3d(0, 0, 0) scale(1);
  }
  100% {
    transform: translate3d(18px, -14px, 0) scale(1.06);
  }
}

@media (max-width: 980px) {
  .login-shell {
    grid-template-columns: 1fr;
    max-width: 680px;
  }

  .brand-panel {
    border-right: none;
    border-bottom: 1px solid rgba(51, 44, 35, 0.18);
  }
}

@media (max-width: 640px) {
  .login-page {
    padding: 14px;
  }

  .brand-metrics {
    gap: 10px;
  }

  .brand-metrics li {
    flex: 1 1 100%;
  }

  .action-row {
    justify-content: space-between;
  }
}

@media (prefers-reduced-motion: reduce) {
  .submit-btn,
  .login-form :deep(.el-input__wrapper) {
    animation: none;
    transition: none;
  }
}
</style>