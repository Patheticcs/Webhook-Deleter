<template>
<div class="webhook-deleter">
  <div class="animated-background">
    <div class="grid-overlay"></div>
    <div class="floating-orbs">
      <div class="orb orb-1"></div>
      <div class="orb orb-2"></div>
      <div class="orb orb-3"></div>
    </div>
  </div>
  <div class="card" ref="card">
    <div class="card-content">
      <h1 class="title">Discord Webhook Deleter</h1>
      <div class="animated-underline"></div>
      <div class="input-group">
        <label for="webhook-url">Enter Webhook URL:</label>
        <div class="input-container">
          <LinkIcon class="input-icon" />
          <input id="webhook-url" v-model="webhookUrl" type="text" placeholder="discord.com/api/webhooks/..." @focus="inputFocus = true" @blur="inputFocus = false" :class="{ 'input-active': inputFocus }">
        </div>
      </div>
      <button @click="deleteWebhook" class="delete-btn" :disabled="isDeleting" :class="{ 'btn-active': !isDeleting && webhookUrl }">
        <div class="btn-background"></div>
        <div class="btn-content">
          <Trash2Icon v-if="!isDeleting" class="icon" />
          <LoaderIcon v-else class="icon animate-spin" />
          {{ isDeleting ? 'Deleting...' : 'Delete Webhook' }}
        </div>
      </button>
      <TransitionGroup name="slide-fade">
        <div v-if="status === 'success'" key="success" class="alert success" role="alert">
          <CheckCircleIcon class="icon" />
          <div class="alert-content">
            <strong>Success!</strong> The webhook has been successfully deleted.
          </div>
          <button @click="resetStatus" class="close-btn" aria-label="Close">
            <XIcon class="icon" />
          </button>
        </div>
        <div v-if="status === 'error'" key="error" class="alert error" role="alert">
          <AlertCircleIcon class="icon" />
          <div class="alert-content">
            <strong>Error:</strong> {{ errorMessage }}
          </div>
          <button @click="resetStatus" class="close-btn" aria-label="Close">
            <XIcon class="icon" />
          </button>
        </div>
      </TransitionGroup>
    </div>
    <div class="card-footer">
      <div class="pulse-ring"></div>
      <HeartIcon class="footer-icon heart" />
      <span>Secure • Fast • Reliable</span>
      <GithubIcon class="footer-icon github" />
    </div>
  </div>
</div>
</template>
<script setup>
  import {
    ref,
    onMounted,
    onUnmounted
  } from 'vue'
  import {
    Trash2Icon,
    LoaderIcon,
    CheckCircleIcon,
    AlertCircleIcon,
    XIcon,
    HeartIcon,
    GithubIcon,
    LinkIcon
  } from 'lucide-vue-next'
  const webhookUrl = ref('')
  const status = ref('idle')
  const errorMessage = ref('')
  const isDeleting = ref(false)
  const inputFocus = ref(false)
  const card = ref(null)
  const handleMouseMove = (e) => {
    if (!card.value) return
    const cardRect = card.value.getBoundingClientRect()
    const cardCenterX = cardRect.left + cardRect.width / 2
    const cardCenterY = cardRect.top + cardRect.height / 2
    const mouseX = e.clientX - cardCenterX
    const mouseY = e.clientY - cardCenterY
    const rotateY = mouseX * 0.03
    const rotateX = -mouseY * 0.03
    card.value.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) translateZ(10px)`
  }
  const resetCardPosition = () => {
    if (card.value) {
      card.value.style.transform = 'perspective(1000px) rotateX(0) rotateY(0) translateZ(0)'
    }
  }
  onMounted(() => {
    document.addEventListener('mousemove', handleMouseMove)
    window.addEventListener('resize', resetCardPosition)
  })
  onUnmounted(() => {
    document.removeEventListener('mousemove', handleMouseMove)
    window.removeEventListener('resize', resetCardPosition)
  })
  const deleteWebhook = async () => {
    if (!webhookUrl.value) {
      status.value = 'error'
      errorMessage.value = 'Please enter a webhook URL.'
      return
    }
    const expression = "^https://discord\\.com/api/.*"
    const regex = new RegExp(expression)
    if (!webhookUrl.value.match(regex)) {
      status.value = 'error'
      errorMessage.value = 'This is not a Discord webhook URL. Note: The webhook token is required.'
      return
    }
    isDeleting.value = true
    try {
      const response = await fetch(webhookUrl.value, {
        method: 'DELETE'
      })
      console.log(response)
      if (response.status == 404) throw new Error("This webhook does not exist (maybe you've already deleted it).")
      if (!response.ok) throw new Error(`Failed to delete webhook.`)
      status.value = 'success'
      webhookUrl.value = ''
    } catch (error) {
      status.value = 'error'
      errorMessage.value = error instanceof Error ? error.message : 'An unknown error occurred'
    } finally {
      isDeleting.value = false
    }
  }
  const resetStatus = () => {
    status.value = 'idle'
    errorMessage.value = ''
  }
</script>
<style scoped>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');

  :root {
    --glass-bg: rgba(10, 10, 15, 0.5);
    --glass-border: rgba(255, 255, 255, 0.1);
    --glass-highlight: rgba(255, 255, 255, 0.15);
    --glass-shadow: rgba(0, 0, 0, 0.45);
    --accent-color: rgba(120, 130, 240, 0.8);
    --delete-color: rgba(240, 40, 60, 0.8);
    --text-color: rgba(240, 240, 250, 0.95);
    --success-color: rgba(30, 200, 120, 0.8);
  }

  .webhook-deleter {
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0;
    padding: 20px;
    box-sizing: border-box;
    font-family: 'Inter', sans-serif;
    color: rgba(100, 100, 100, 0.95);
    position: relative;
    overflow: hidden;
  }

  .animated-background {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -10;
    background: linear-gradient(135deg, #090909 0%, #1a1a1a 100%);
    overflow: hidden;
  }

  .grid-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-image:
      linear-gradient(rgba(255, 255, 255, 0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(255, 255, 255, 0.03) 1px, transparent 1px);
    background-size: 30px 30px;
    animation: gridMove 150s linear infinite;
  }

  .floating-orbs {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  .orb {
    position: absolute;
    border-radius: 50%;
    filter: blur(40px);
    opacity: 0.25;
  }

  .orb-1 {
    top: 20%;
    left: 10%;
    width: 300px;
    height: 300px;
    background: radial-gradient(circle, rgba(100, 100, 100, 0.5) 0%, rgba(100, 100, 100, 0) 70%);
    animation: floatOrb1 15s ease-in-out infinite alternate;
  }

  .orb-2 {
    bottom: 10%;
    right: 15%;
    width: 250px;
    height: 250px;
    background: radial-gradient(circle, rgba(80, 80, 80, 0.5) 0%, rgba(80, 80, 80, 0) 70%);
    animation: floatOrb2 18s ease-in-out infinite alternate;
  }

  .orb-3 {
    top: 50%;
    right: 10%;
    width: 200px;
    height: 200px;
    background: radial-gradient(circle, rgba(120, 120, 120, 0.5) 0%, rgba(120, 120, 120, 0) 70%);
    animation: floatOrb3 20s ease-in-out infinite alternate;
  }

  @keyframes floatOrb1 {
    0% {
      transform: translate(0, 0);
    }

    100% {
      transform: translate(100px, 50px);
    }
  }

  @keyframes floatOrb2 {
    0% {
      transform: translate(0, 0);
    }

    100% {
      transform: translate(-80px, -40px);
    }
  }

  @keyframes floatOrb3 {
    0% {
      transform: translate(0, 0);
    }

    100% {
      transform: translate(-60px, 80px);
    }
  }

  @keyframes gridMove {
    0% {
      background-position: 0 0;
    }

    100% {
      background-position: 1000px 1000px;
    }
  }

  .card {
    position: relative;
    width: 100%;
    max-width: 440px;
    background: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border-radius: 1.5rem;
    border: 1px solid rgba(255, 255, 255, 0.1);
    box-shadow:
      0 20px 40px rgba(0, 0, 0, 0.45),
      0 0 0 1px rgba(255, 255, 255, 0.15) inset;
    overflow: hidden;
    transition: all 0.6s cubic-bezier(0.23, 1, 0.32, 1);
    z-index: 10;
  }

  .card::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 100%;
    background: linear-gradient(180deg,
        rgba(255, 255, 255, 0.05) 0%,
        rgba(255, 255, 255, 0.02) 10%,
        rgba(255, 255, 255, 0) 100%);
    pointer-events: none;
    z-index: 1;
  }

  .card::after {
    content: '';
    position: absolute;
    top: -100%;
    left: -100%;
    right: -100%;
    bottom: -100%;
    background: radial-gradient(circle at 50% 50%, rgba(255, 255, 255, 0.15), transparent 40%);
    opacity: 0;
    z-index: 0;
    transition: opacity 0.6s ease;
    pointer-events: none;
  }

  .card:hover::after {
    opacity: 0.4;
  }

  .card-content {
    position: relative;
    z-index: 10;
    padding: 2.5rem;
  }

  .title {
    font-size: 1.75rem;
    font-weight: 700;
    text-align: center;
    margin-bottom: 0.5rem;
    background: linear-gradient(90deg, #fff 0%, #e0e0e0 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
    letter-spacing: 0.5px;
  }

  .animated-underline {
    position: relative;
    height: 2px;
    width: 100px;
    margin: 0 auto 2rem;
    background: linear-gradient(90deg, transparent, rgba(200, 200, 200, 0.8), transparent);
    overflow: hidden;
  }

  .animated-underline::after {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.8), transparent);
    animation: shimmer 3s infinite;
  }

  @keyframes shimmer {
    0% {
      left: -100%;
    }

    100% {
      left: 100%;
    }
  }

  .input-group {
    display: flex;
    flex-direction: column;
    margin-bottom: 1.75rem;
  }

  label {
    display: block;
    margin-bottom: 0.75rem;
    font-weight: 600;
    font-size: 0.9rem;
    letter-spacing: 0.5px;
    color: rgba(255, 255, 255, 0.8);
    text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
  }

  .input-container {
    position: relative;
    display: flex;
    align-items: center;
  }

  .input-icon {
    position: absolute;
    left: 1rem;
    color: rgba(255, 255, 255, 0.5);
    width: 18px;
    height: 18px;
    transition: all 0.3s ease;
  }

  input {
    width: 100%;
    padding: 0.9rem 1rem 0.9rem 2.5rem;
    background: rgba(255, 255, 255, 0.05);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 0.75rem;
    font-family: 'Inter', sans-serif;
    color: white;
    font-size: 0.95rem;
    letter-spacing: 0.25px;
    transition: all 0.3s ease;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1) inset;
  }

  input:focus {
    outline: none;
    border-color: rgba(150, 150, 150, 0.5);
    background: rgba(255, 255, 255, 0.08);
    box-shadow:
      0 2px 10px rgba(0, 0, 0, 0.1) inset,
      0 0 0 3px rgba(150, 150, 150, 0.15);
  }

  .input-active+.input-icon {
    color: rgba(200, 200, 200, 0.8);
  }

  .delete-btn {
    position: relative;
    width: 100%;
    padding: 0.9rem;
    border: none;
    border-radius: 0.75rem;
    font-family: 'Inter', sans-serif;
    font-size: 1rem;
    font-weight: 600;
    letter-spacing: 0.5px;
    cursor: pointer;
    overflow: hidden;
    background: transparent;
    transition: all 0.3s ease;
    transform: translateZ(0);
  }

  .btn-background {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(45deg, rgba(80, 80, 80, 0.8), rgba(120, 120, 120, 0.9));
    opacity: 0.9;
    transition: all 0.3s ease;
    z-index: 1;
  }

  .btn-content {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    z-index: 2;
  }

  .delete-btn:hover:not(:disabled) .btn-background {
    filter: brightness(1.1);
  }

  .delete-btn:active:not(:disabled) .btn-background {
    filter: brightness(0.9);
  }

  .delete-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  .delete-btn:not(:disabled)::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 60%);
    opacity: 0;
    transition: opacity 0.6s ease;
    z-index: 1;
    pointer-events: none;
  }

  .delete-btn:hover:not(:disabled)::before {
    opacity: 0.4;
  }

  .btn-active:not(:disabled)::after {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
    z-index: 2;
    animation: btnShine 3s infinite;
  }

  @keyframes btnShine {
    0% {
      left: -100%;
    }

    20%,
    100% {
      left: 100%;
    }
  }

  .icon {
    margin-right: 0.5rem;
  }

  .animate-spin {
    animation: spin 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }

    100% {
      transform: rotate(360deg);
    }
  }

  .alert {
    margin-top: 1.5rem;
    padding: 1rem;
    border-radius: 0.75rem;
    display: flex;
    align-items: flex-start;
    background: rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(255, 255, 255, 0.1);
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.15);
    backdrop-filter: blur(4px);
  }

  .success {
    border-left: 3px solid rgba(180, 180, 180, 0.8);
  }

  .error {
    border-left: 3px solid rgba(100, 100, 100, 0.8);
  }

  .alert-content {
    margin: 0 0.5rem;
    flex: 1;
  }

  .alert strong {
    font-weight: 600;
    margin-right: 4px;
  }

  .close-btn {
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    color: currentColor;
    opacity: 0.7;
    transition: opacity 0.2s;
  }

  .close-btn:hover {
    opacity: 1;
  }

  .card-footer {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 1rem;
    font-size: 0.8rem;
    color: rgba(255, 255, 255, 0.6);
    background: rgba(0, 0, 0, 0.2);
    border-top: 1px solid rgba(255, 255, 255, 0.05);
  }

  .footer-icon {
    width: 14px;
    height: 14px;
    margin-right: 0.5rem;
    filter: brightness(0.8);
  }

  .heart {
    color: rgba(240, 60, 80, 0.8);
    margin-right: 8px;
    animation: pulse 2s infinite;
  }

  .github {
    margin-left: 8px;
  }

  .footer-icon:hover {
    opacity: 1;
    transform: scale(1.2);
  }

  .pulse-ring {
    position: absolute;
    left: calc(50% - 60px);
    top: 50%;
    transform: translateY(-50%);
    width: 14px;
    height: 14px;
    border-radius: 50%;
    box-shadow: 0 0 0 rgba(240, 60, 80, 0.4);
    animation: pulse-ring 2s infinite;
  }

  @keyframes pulse-ring {
    0% {
      box-shadow: 0 0 0 0 rgba(240, 60, 80, 0.4);
    }

    70% {
      box-shadow: 0 0 0 10px rgba(240, 60, 80, 0);
    }

    100% {
      box-shadow: 0 0 0 0 rgba(240, 60, 80, 0);
    }
  }

  @keyframes pulse {
    0% {
      transform: scale(0.95);
    }

    50% {
      transform: scale(1.05);
    }

    100% {
      transform: scale(0.95);
    }
  }

  .slide-fade-enter-active {
    transition: all 0.4s cubic-bezier(0.23, 1, 0.32, 1);
  }

  .slide-fade-leave-active {
    transition: all 0.2s cubic-bezier(0.23, 1, 0.32, 1);
  }

  .slide-fade-enter-from,
  .slide-fade-leave-to {
    transform: translateY(20px);
    opacity: 0;
  }

  @media (max-width: 480px) {
    .card-content {
      padding: 1.75rem 1.25rem;
    }

    .title {
      font-size: 1.5rem;
    }

    input,
    .delete-btn {
      padding: 0.8rem;
      font-size: 0.9rem;
    }

    .alert {
      padding: 0.8rem;
      font-size: 0.85rem;
    }
  }

  @media (max-width: 350px) {
    .card-content {
      padding: 1.5rem 1rem;
    }

    .title {
      font-size: 1.3rem;
    }
  }

  @media (hover: hover) {
    .card:hover {
      transform: perspective(1000px) scale(1.02);
      box-shadow:
        0 30px 60px var(--glass-shadow),
        0 0 0 1px var(--glass-highlight) inset;
    }
  }

  @media (hover: none) {
    .card {
      transform: none !important;
    }
  }
</style>
