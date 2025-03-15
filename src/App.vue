<template>
	<div class="webhook-deleter">
		<div class="card">
			<h1 class="title">Discord Webhook Deleter</h1>

			<div class="input-group">
				<label for="webhook-url">Enter Webhook URL:</label>
				<input
					id="webhook-url"
					v-model="webhookUrl"
					type="text"
					placeholder="discord.com/api/webhooks/..."
				>
			</div>

			<button @click="deleteWebhook" class="delete-btn" :disabled="isDeleting">
				<Trash2Icon v-if="!isDeleting" class="icon" />
				<LoaderIcon v-else class="icon animate-spin" />
				{{ isDeleting ? 'Deleting...' : 'Delete Webhook' }}
			</button>

			<TransitionGroup name="fade">
				<div v-if="status === 'success'" key="success" class="alert success" role="alert">
					<CheckCircleIcon class="icon" />
					<div>
						<strong>Success!</strong> The webhook has been successfully deleted.
					</div>
					<button @click="resetStatus" class="close-btn" aria-label="Close">
						<XIcon class="icon" />
					</button>
				</div>

				<div v-if="status === 'error'" key="error" class="alert error" role="alert">
					<AlertCircleIcon class="icon" />
					<div>
						<strong>Error:</strong> {{ errorMessage }}
					</div>
					<button @click="resetStatus" class="close-btn" aria-label="Close">
						<XIcon class="icon" />
					</button>
				</div>
			</TransitionGroup>

			<footer class="footer">
				<p>Made with ❤️ by spreehertz
				</p>
				<p class="contributor">Contributor:<a class="contributor_link" href="https://github.com/Fedox-die-Ente">Fedox-die-Ente</a></p>
				<a href="https://github.com/spreehertz/webhook-deleter" target="_blank" rel="noopener noreferrer">
					<GithubIcon class="icon" />
				</a>
			</footer>
		</div>
	</div>
</template>

<script setup>
import { ref } from 'vue'
import { Trash2Icon, LoaderIcon, CheckCircleIcon, AlertCircleIcon, XIcon, HeartIcon, GithubIcon } from 'lucide-vue-next'

const webhookUrl = ref('')
const status = ref('idle')
const errorMessage = ref('')
const isDeleting = ref(false)

const deleteWebhook = async () => {
	if (!webhookUrl.value) {
		status.value = 'error'
		errorMessage.value = 'Please enter a webhook URL.'
		return
	}
	const expression = "^https://discord\.com/api/.*"
	const regex = new RegExp(expression);
	if (!webhookUrl.value.match(regex)) {
		status.value = 'error'
		errorMessage.value = 'This is not a Discord webhook URL. Note: The webhook token is required.'
		return
	}
	isDeleting.value = true
	try {
		const response = await fetch(webhookUrl.value, { method: 'DELETE'})
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
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');

:root {
  --glass-bg: rgba(255, 255, 255, 0.1);
  --glass-border: rgba(255, 255, 255, 0.2);
  --glass-shadow: rgba(0, 0, 0, 0.25);
  --accent-color: rgba(150, 150, 150, 0.8);
}

.webhook-deleter {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #111;
  background-image: 
    linear-gradient(rgba(255, 255, 255, 0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 255, 255, 0.03) 1px, transparent 1px);
  background-size: 40px 40px;
  font-family: 'Inter', sans-serif;
  overflow-y: hidden;
  position: relative;
}

.webhook-deleter::before {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle at 50% 50%, rgba(255, 255, 255, 0.1), transparent 70%);
  animation: pulse 8s ease-in-out infinite;
}

.card {
  background: var(--glass-bg);
  backdrop-filter: blur(12px);
  border: 1px solid var(--glass-border);
  border-radius: 1rem;
  padding: 2.5rem;
  width: 100%;
  max-width: 420px;
  box-shadow: 
    0 10px 30px var(--glass-shadow),
    0 -1px 0 var(--glass-border) inset,
    0 1px 0 rgba(0, 0, 0, 0.1) inset;
  transform: perspective(1000px) rotateX(5deg);
  transform-style: preserve-3d;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  position: relative;
  z-index: 10;
}

.card:hover {
  transform: perspective(1000px) rotateX(0deg) translateY(-5px);
  box-shadow: 
    0 20px 40px var(--glass-shadow),
    0 -1px 0 var(--glass-border) inset,
    0 1px 0 rgba(0, 0, 0, 0.1) inset;
}

.card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  background: linear-gradient(to bottom, 
    rgba(255, 255, 255, 0.1) 0%, 
    rgba(255, 255, 255, 0.05) 40%, 
    rgba(255, 255, 255, 0) 100%);
  pointer-events: none;
  border-radius: 1rem;
}

.title {
  font-size: 1.75rem;
  font-weight: 700;
  text-align: center;
  margin-bottom: 2rem;
  color: white;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  position: relative;
}

.title::after {
  content: '';
  position: absolute;
  bottom: -8px;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 2px;
  background: var(--accent-color);
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.9);
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

input,
.delete-btn {
  width: 100%;
  padding: 0.85rem;
  box-sizing: border-box;
  font-family: 'Inter', sans-serif;
  border-radius: 0.5rem;
  transition: all 0.3s ease;
}

input {
  background: rgba(255, 255, 255, 0.07);
  border: 1px solid rgba(255, 255, 255, 0.15);
  color: white;
  caret-color: white;
}

input:focus {
  outline: none;
  border-color: rgba(255, 255, 255, 0.3);
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.1);
}

.delete-btn {
  background: rgba(200, 0, 0, 0.8);
  color: white;
  border: none;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  transform: translateY(0);
  transition: all 0.2s ease;
}

.delete-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.2),
    transparent
  );
  transition: all 0.5s ease;
}

.delete-btn:hover:not(:disabled) {
  background: rgba(220, 0, 0, 0.9);
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.3);
}

.delete-btn:hover:not(:disabled)::before {
  left: 100%;
}

.delete-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.alert {
  margin-top: 1.25rem;
  padding: 1rem;
  border-radius: 0.5rem;
  display: flex;
  align-items: flex-start;
  backdrop-filter: blur(5px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  animation: fadeIn 0.3s ease;
}

.success {
  background-color: rgba(0, 0, 0, 0.3);
  color: rgba(255, 255, 255, 0.9);
  border-left: 3px solid rgba(200, 200, 200, 0.8);
}

.error {
  background-color: rgba(0, 0, 0, 0.3);
  color: rgba(255, 255, 255, 0.9);
  border-left: 3px solid rgba(200, 0, 0, 0.8);
}

.close-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
  color: currentColor;
  margin-left: auto;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.close-btn:hover {
  opacity: 1;
}

.footer {
  margin-top: 2.5rem;
  text-align: center;
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.6);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.contributor {
  margin-top: -0.5rem;
}

.contributor_link {
  transition: color 0.2s;
  color: rgba(255, 255, 255, 0.6);
  text-decoration: none;
}

.contributor_link:hover {
  color: white;
}

.footer a {
  color: rgba(255, 255, 255, 0.6);
  margin-left: 0.5rem;
  transition: color 0.2s;
}

.footer a:hover {
  color: white;
}

.icon {
  width: 1.25rem;
  height: 1.25rem;
  margin-right: 0.5rem;
  filter: invert(1);
}

@media (max-width: 640px) {
  .card {
    padding: 1.75rem;
    max-width: 90%;
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes pulse {
  0%, 100% {
    opacity: 0.3;
  }
  50% {
    opacity: 0.6;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-spin {
  animation: spin 1s linear infinite;
}

#webhook-url {
  font-family: 'Inter', sans-serif;
}

/* 3D effect for card elements */
input, .delete-btn, .alert {
  transform: translateZ(20px);
  transition: transform 0.3s ease;
}

/* Floating glass orbs in background */
.webhook-deleter::after {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  background-image: 
    radial-gradient(circle at 20% 30%, rgba(255, 255, 255, 0.03) 0%, transparent 50%),
    radial-gradient(circle at 80% 70%, rgba(255, 255, 255, 0.03) 0%, transparent 50%);
  animation: moveBubbles 20s infinite alternate ease-in-out;
}

@keyframes moveBubbles {
  0% {
    background-position: 0% 0%, 100% 100%;
  }
  100% {
    background-position: 100% 0%, 0% 100%;
  }
}
</style>
