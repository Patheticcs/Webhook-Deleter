<template>
  <div class="webhook-deleter">
    <div class="animated-background">
      <div class="grid-overlay"></div>
      <div class="particles-container">
        <div v-for="n in 30" :key="n" class="particle"></div>
      </div>
      <div class="floating-orbs">
        <div class="orb orb-1"></div>
        <div class="orb orb-2"></div>
        <div class="orb orb-3"></div>
        <div class="orb orb-4"></div>
      </div>
    </div>
    <div class="card" ref="card">
      <div class="glowing-border"></div>
      <div class="card-content">
        <h1 class="title">Discord Webhook Deleter</h1>
        <div class="animated-underline">
          <div class="underline-glow"></div>
        </div>
        <div class="input-group">
          <label for="webhook-url">Enter Webhook URL:</label>
          <div class="input-container">
            <LinkIcon class="input-icon" :class="{ 'icon-active': inputFocus }" />
            <input 
              id="webhook-url" 
              v-model="webhookUrl" 
              type="text" 
              placeholder="discord.com/api/webhooks/..." 
              @focus="inputFocus = true" 
              @blur="inputFocus = false" 
              :class="{ 'input-active': inputFocus }"
            >
            <span v-if="webhookUrl" @click="webhookUrl = ''" class="clear-input">
              <XIcon class="clear-icon" />
            </span>
          </div>
        </div>
        <button 
          @click="deleteWebhook" 
          class="delete-btn" 
          :disabled="isDeleting || !webhookUrl" 
          :class="{ 'btn-active': !isDeleting && webhookUrl }"
        >
          <div class="btn-background"></div>
          <div class="btn-glow"></div>
          <div class="btn-content">
            <Trash2Icon v-if="!isDeleting" class="icon" />
            <LoaderIcon v-else class="icon animate-spin" />
            {{ isDeleting ? 'Deleting...' : 'Delete Webhook' }}
          </div>
        </button>
        <TransitionGroup name="slide-fade">
          <div v-if="status === 'success'" key="success" class="alert success" role="alert">
            <div class="alert-glow success-glow"></div>
            <CheckCircleIcon class="icon alert-icon" />
            <div class="alert-content">
              <strong>Success!</strong> The webhook has been successfully deleted.
            </div>
            <button @click="resetStatus" class="close-btn" aria-label="Close">
              <XIcon class="icon" />
            </button>
          </div>
          <div v-if="status === 'error'" key="error" class="alert error" role="alert">
            <div class="alert-glow error-glow"></div>
            <AlertCircleIcon class="icon alert-icon" />
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
        <GithubIcon class="footer-icon github" @click="openGithub" />
      </div>
    </div>
    <div class="credits">Designed with <span class="heart-text">❤</span> by DevTeam</div>
  </div>
</template>

<script setup>
import {
  ref,
  onMounted,
  onUnmounted,
  watchEffect
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

// 3D Card Effect
const handleMouseMove = (e) => {
  if (!card.value) return
  const cardRect = card.value.getBoundingClientRect()
  const cardCenterX = cardRect.left + cardRect.width / 2
  const cardCenterY = cardRect.top + cardRect.height / 2
  const mouseX = e.clientX - cardCenterX
  const mouseY = e.clientY - cardCenterY
  const maxRotate = 8 // Maximum rotation in degrees
  const rotateY = (mouseX / cardRect.width) * maxRotate
  const rotateX = -(mouseY / cardRect.height) * maxRotate
  
  card.value.style.transform = `perspective(1500px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) translateZ(10px)`
  
  // Update shadow and glow effect
  const distance = Math.sqrt(mouseX * mouseX + mouseY * mouseY)
  const maxDistance = Math.sqrt((cardRect.width / 2) * (cardRect.width / 2) + (cardRect.height / 2) * (cardRect.height / 2))
  const intensity = 0.3 + (0.7 * distance / maxDistance)
  
  if (card.value.querySelector('.glowing-border')) {
    card.value.querySelector('.glowing-border').style.opacity = intensity.toString()
  }
}

const resetCardPosition = () => {
  if (card.value) {
    card.value.style.transform = 'perspective(1500px) rotateX(0) rotateY(0) translateZ(0)'
    if (card.value.querySelector('.glowing-border')) {
      card.value.querySelector('.glowing-border').style.opacity = '0.3'
    }
  }
}

// Initialize particles
const initializeParticles = () => {
  const particles = document.querySelectorAll('.particle')
  particles.forEach(particle => {
    setParticleProperties(particle)
  })
}

const setParticleProperties = (particle) => {
  const size = Math.random() * 3 + 1
  const duration = Math.random() * 40 + 20
  const x = Math.random() * 100
  const y = Math.random() * 100
  const delay = Math.random() * 15
  
  particle.style.width = `${size}px`
  particle.style.height = `${size}px`
  particle.style.left = `${x}vw`
  particle.style.top = `${y}vh`
  particle.style.animationDuration = `${duration}s`
  particle.style.animationDelay = `${delay}s`
  particle.style.opacity = Math.random() * 0.6 + 0.2
}

onMounted(() => {
  document.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('resize', resetCardPosition)
  initializeParticles()
  
  // Periodically respawn particles
  const particleInterval = setInterval(() => {
    const particles = document.querySelectorAll('.particle')
    particles.forEach(particle => {
      if (Math.random() > 0.7) {
        setParticleProperties(particle)
      }
    })
  }, 10000)
  
  // Clean up interval on unmount
  onUnmounted(() => {
    clearInterval(particleInterval)
  })
})

onUnmounted(() => {
  document.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('resize', resetCardPosition)
})

// Delete webhook functionality
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
    errorMessage.value = 'This is not a Discord webhook URL. Please use a valid URL starting with https://discord.com/api/'
    return
  }
  
  isDeleting.value = true
  try {
    const response = await fetch(webhookUrl.value, {
      method: 'DELETE'
    })
    
    console.log(response)
    if (response.status == 404) throw new Error("This webhook does not exist (maybe you've already deleted it).")
    if (!response.ok) throw new Error(`Failed to delete webhook. Server responded with ${response.status}.`)
    
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

const openGithub = () => {
  window.open('https://github.com', '_blank')
}

// Automatic reset of notifications after 5 seconds
watchEffect(() => {
  if (status.value !== 'idle') {
    const timer = setTimeout(() => {
      resetStatus()
    }, 5000)
    
    return () => clearTimeout(timer)
  }
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&family=Syne:wght@600;700;800&display=swap');

:root {
  --glass-bg: rgba(10, 10, 15, 0.5);
  --glass-border: rgba(255, 255, 255, 0.1);
  --glass-highlight: rgba(255, 255, 255, 0.15);
  --glass-shadow: rgba(0, 0, 0, 0.45);
  --accent-color: rgba(100, 120, 240, 0.8);
  --delete-color: rgba(240, 40, 60, 0.8);
  --text-color: rgba(240, 240, 250, 0.95);
  --success-color: rgba(30, 200, 120, 0.8);
  --error-color: rgba(240, 60, 60, 0.8);
  --neon-purple: rgba(140, 80, 255, 0.8);
  --neon-blue: rgba(0, 170, 255, 0.8);
}

.webhook-deleter {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 0;
  padding: 20px;
  box-sizing: border-box;
  font-family: 'Inter', sans-serif;
  color: rgba(255, 255, 255, 0.95);
  position: relative;
  overflow: hidden;
}

/* Enhanced Background */
.animated-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -10;
  background: linear-gradient(135deg, #000000 0%, #0a0a0a 30%, #101010 70%, #151515 100%);
  overflow: hidden;
}

.grid-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 200%;
  height: 200%;
  background-image:
    linear-gradient(rgba(255, 255, 255, 0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 255, 255, 0.03) 1px, transparent 1px);
  background-size: 40px 40px;
  transform: rotate(5deg) scale(1.5);
  animation: gridMove 180s linear infinite;
}

@keyframes gridMove {
  0% {
    transform: translateY(0) translateX(0) rotate(5deg) scale(1.5);
  }
  100% {
    transform: translateY(-50%) translateX(-50%) rotate(5deg) scale(1.5);
  }
}

/* Particles */
.particles-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.particle {
  position: absolute;
  background: rgba(255, 255, 255, 0.4);
  border-radius: 50%;
  pointer-events: none;
  animation: floatParticle linear infinite;
}

@keyframes floatParticle {
  0% {
    transform: translateY(0) translateX(0);
  }
  25% {
    transform: translateY(-20vh) translateX(10vw);
  }
  50% {
    transform: translateY(-40vh) translateX(0);
  }
  75% {
    transform: translateY(-60vh) translateX(-10vw);
  }
  100% {
    transform: translateY(-100vh) translateX(0);
    opacity: 0;
  }
}

/* Enhanced Orbs */
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
  filter: blur(60px);
  opacity: 0.15;
  mix-blend-mode: screen;
}

.orb-1 {
  top: 20%;
  left: 10%;
  width: 400px;
  height: 400px;
  background: radial-gradient(circle, rgba(140, 80, 255, 0.6) 0%, rgba(140, 80, 255, 0) 70%);
  animation: floatOrb1 25s ease-in-out infinite alternate;
}

.orb-2 {
  bottom: 10%;
  right: 15%;
  width: 350px;
  height: 350px;
  background: radial-gradient(circle, rgba(0, 170, 255, 0.6) 0%, rgba(0, 170, 255, 0) 70%);
  animation: floatOrb2 28s ease-in-out infinite alternate;
}

.orb-3 {
  top: 50%;
  right: 10%;
  width: 300px;
  height: 300px;
  background: radial-gradient(circle, rgba(80, 200, 255, 0.6) 0%, rgba(80, 200, 255, 0) 70%);
  animation: floatOrb3 30s ease-in-out infinite alternate;
}

.orb-4 {
  bottom: 40%;
  left: 20%;
  width: 250px;
  height: 250px;
  background: radial-gradient(circle, rgba(200, 120, 255, 0.6) 0%, rgba(200, 120, 255, 0) 70%);
  animation: floatOrb4 32s ease-in-out infinite alternate;
}

@keyframes floatOrb1 {
  0% {
    transform: translate(0, 0) scale(1);
  }
  50% {
    transform: translate(100px, 50px) scale(1.1);
  }
  100% {
    transform: translate(50px, 100px) scale(0.9);
  }
}

@keyframes floatOrb2 {
  0% {
    transform: translate(0, 0) scale(1);
  }
  50% {
    transform: translate(-80px, -40px) scale(1.2);
  }
  100% {
    transform: translate(-40px, -80px) scale(0.95);
  }
}

@keyframes floatOrb3 {
  0% {
    transform: translate(0, 0) scale(1);
  }
  33% {
    transform: translate(-60px, 80px) scale(1.15);
  }
  66% {
    transform: translate(40px, -60px) scale(0.9);
  }
  100% {
    transform: translate(-30px, 30px) scale(1.05);
  }
}

@keyframes floatOrb4 {
  0% {
    transform: translate(0, 0) scale(1);
  }
  33% {
    transform: translate(70px, -50px) scale(1.1);
  }
  66% {
    transform: translate(-50px, 70px) scale(0.95);
  }
  100% {
    transform: translate(40px, 40px) scale(1.05);
  }
}

/* Enhanced Card */
.card {
  position: relative;
  width: 100%;
  max-width: 460px;
  background: rgba(5, 5, 10, 0.7);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 1.5rem;
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow:
    0 20px 40px rgba(0, 0, 0, 0.6),
    0 0 0 1px rgba(255, 255, 255, 0.1) inset;
  overflow: hidden;
  transition: all 0.6s cubic-bezier(0.23, 1, 0.32, 1);
  z-index: 10;
  will-change: transform;
}

.glowing-border {
  position: absolute;
  inset: 0;
  border-radius: 1.5rem;
  border: 1px solid transparent;
  background: linear-gradient(90deg, 
    transparent 0%, 
    rgba(140, 80, 255, 0.3) 25%, 
    rgba(0, 170, 255, 0.3) 50%, 
    rgba(140, 80, 255, 0.3) 75%, 
    transparent 100%
  ) border-box;
  -webkit-mask: linear-gradient(#fff 0 0) padding-box, linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  opacity: 0.3;
  transition: opacity 0.3s ease;
  pointer-events: none;
  z-index: 0;
  filter: blur(1px);
  animation: borderRotate 6s linear infinite;
}

@keyframes borderRotate {
  0% {
    background-position: 0% 0%;
  }
  100% {
    background-position: 200% 0%;
  }
}

.card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  background: linear-gradient(180deg,
      rgba(255, 255, 255, 0.03) 0%,
      rgba(255, 255, 255, 0.01) 10%,
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
  background: radial-gradient(ellipse at center, rgba(255, 255, 255, 0.1), transparent 70%);
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
  font-family: 'Syne', sans-serif;
  font-size: 1.85rem;
  font-weight: 800;
  text-align: center;
  margin-bottom: 0.5rem;
  background: linear-gradient(90deg, #ffffff 0%, rgba(200, 200, 255, 0.9) 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 15px rgba(255, 255, 255, 0.3);
  letter-spacing: 0.05em;
  animation: titlePulse 4s ease-in-out infinite;
}

@keyframes titlePulse {
  0%, 100% {
    filter: brightness(1);
  }
  50% {
    filter: brightness(1.3);
  }
}

.animated-underline {
  position: relative;
  height: 2px;
  width: 120px;
  margin: 0 auto 2rem;
  background: linear-gradient(90deg, transparent, rgba(200, 200, 255, 0.7), transparent);
  overflow: hidden;
}

.underline-glow {
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.9), transparent);
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
  letter-spacing: 0.05em;
  color: rgba(255, 255, 255, 0.9);
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

.input-container {
  position: relative;
  display: flex;
  align-items: center;
  transition: all 0.3s ease;
}

.input-icon {
  position: absolute;
  left: 1rem;
  color: rgba(255, 255, 255, 0.5);
  width: 18px;
  height: 18px;
  transition: all 0.3s ease;
}

.icon-active {
  color: var(--neon-blue);
  filter: drop-shadow(0 0 5px rgba(0, 170, 255, 0.5));
}

input {
  width: 100%;
  padding: 1rem 1rem 1rem 2.5rem;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 0.75rem;
  font-family: 'Inter', sans-serif;
  color: white;
  font-size: 0.95rem;
  letter-spacing: 0.025em;
  transition: all 0.3s ease;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2) inset;
}

input:focus {
  outline: none;
  border-color: rgba(100, 120, 240, 0.5);
  background: rgba(255, 255, 255, 0.05);
  box-shadow:
    0 2px 10px rgba(0, 0, 0, 0.1) inset,
    0 0 0 1px rgba(100, 120, 240, 0.3);
}

.clear-input {
  position: absolute;
  right: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: rgba(255, 255, 255, 0.5);
  transition: all 0.2s ease;
}

.clear-input:hover {
  color: rgba(255, 255, 255, 0.9);
}

.clear-icon {
  width: 16px;
  height: 16px;
}

/* Enhanced Button */
.delete-btn {
  position: relative;
  width: 100%;
  padding: 0;
  height: 50px;
  border: none;
  border-radius: 0.75rem;
  font-family: 'Inter', sans-serif;
  font-size: 1rem;
  font-weight: 600;
  letter-spacing: 0.05em;
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
  background: linear-gradient(45deg, rgba(50, 50, 50, 0.8), rgba(70, 70, 70, 0.9));
  opacity: 0.9;
  transition: all 0.3s ease;
  z-index: 1;
}

.btn-glow {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, 
    rgba(100, 120, 240, 0.5), 
    rgba(0, 170, 255, 0.5)
  );
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 2;
  filter: blur(1px);
}

.btn-content {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  color: white;
  z-index: 3;
}

.delete-btn:hover:not(:disabled) .btn-background {
  filter: brightness(1.2);
}

.delete-btn:hover:not(:disabled) .btn-glow {
  opacity: 0.7;
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
  background: radial-gradient(circle, rgba(255, 255, 255, 0.2) 0%, transparent 60%);
  opacity: 0;
  transition: opacity 0.6s ease;
  z-index: 2;
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
  width: 50%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  z-index: 3;
  animation: btnShine 3s infinite;
}

@keyframes btnShine {
  0% {
    left: -100%;
  }
  20%, 100% {
    left: 200%;
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

/* Enhanced Alerts */
.alert {
  position: relative;
  margin-top: 1.5rem;
  padding: 1rem;
  border-radius: 0.75rem;
  display: flex;
  align-items: flex-start;
  background: rgba(10, 10, 15, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.15);
  backdrop-filter: blur(10px);
  overflow: hidden;
}

.alert-glow {
  position: absolute;
  top: 0;
  left: 0;
  width: 3px;
  height: 100%;
  filter: blur(2px);
}

.success-glow {
  background: var(--success-color);
  box-shadow: 0 0 15px 2px rgba(30, 200, 120, 0.5);
}

.error-glow {
  background: var(--error-color);
  box-shadow: 0 0 15px 2px rgba(240, 60, 60, 0.5);
}

.alert-icon {
  position: relative;
  z-index: 1;
}

.success .alert-icon {
  color: var(--success-color);
}

.error .alert-icon {
  color: var(--error-color);
}

.alert-content {
  margin: 0 0.5rem;
  flex: 1;
  position: relative;
  z-index: 1;
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
  z-index: 1;
}

.close-btn:hover {
  opacity: 1;
}

/* Footer */
.card-footer {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
  font-size: 0.8rem;
  color: rgba(255, 255, 255, 0.7);
  background: rgba(0, 0, 0, 0.3);
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
