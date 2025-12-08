<script setup lang="ts">
import { ref, onMounted, watch, nextTick } from 'vue'
import { useRouter } from 'vue-router'
import { useChatStore } from '@/stores/chatStore'
import { auth } from '@/services/authService'
import { toast } from 'vue-sonner'

import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { ScrollArea } from '@/components/ui/scroll-area'
import { Avatar, AvatarFallback } from '@/components/ui/avatar'

import { SendHorizontal, Paperclip, LogOut, FileIcon, X } from 'lucide-vue-next'

const router = useRouter()
const store = useChatStore()

const apiUrl = import.meta.env.VITE_API_URL

const messageText = ref('')
const fileInput = ref<HTMLInputElement | null>(null)
const selectedFile = ref<File | null>(null)
const messagesEndRef = ref<HTMLDivElement | null>(null)

const myEmail = localStorage.getItem('userEmail') || ''

onMounted(async () => {
	if (!auth.isLoggedIn.value) {
		router.push('/login')
		return
	}
	await store.init()
})

watch(
	() => store.messages.length,
	async () => {
		await nextTick()
		scrollToBottom()
	},
)

watch(
	() => store.activeChatId,
	async () => {
		selectedFile.value = null
		messageText.value = ''
		await nextTick()
		scrollToBottom()
	},
)

function scrollToBottom() {
	messagesEndRef.value?.scrollIntoView({ behavior: 'smooth' })
}

async function handleSendMessage() {
	if (!messageText.value.trim() && !selectedFile.value) return

	try {
		await store.sendMessage(messageText.value, selectedFile.value)

		messageText.value = ''
		selectedFile.value = null
		if (fileInput.value) fileInput.value.value = ''

		scrollToBottom()
	} catch (e) {
		console.error(e) // Логируем ошибку, чтобы ESLint не ругался
		toast.error('Не удалось отправить сообщение')
	}
}

function handleFileSelect(event: Event) {
	const target = event.target as HTMLInputElement
	if (target.files && target.files.length > 0) {
		// TS Fix: явно приводим к File или null, так как индекс может вернуть undefined
		selectedFile.value = target.files[0] || null
	}
}

function removeFile() {
	selectedFile.value = null
	if (fileInput.value) fileInput.value.value = ''
}

function handleLogout() {
	auth.logout()
	router.push('/login')
}

// Открытие файла в новой вкладке (fix для шаблона)
function openFile(url: string) {
	window.open(url, '_blank')
}

function getInitials(name?: string) {
	if (!name) return '??'
	return name.substring(0, 2).toUpperCase()
}

function formatTime(dateStr: string) {
	return new Date(dateStr).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
}
</script>

<template>
	<div class="flex h-screen w-full bg-background overflow-hidden">
		<aside class="w-80 border-r flex flex-col bg-muted/10">
			<div class="p-4 border-b flex items-center justify-between">
				<h2 class="font-semibold text-lg">Чаты</h2>
				<Button variant="ghost" size="icon" @click="handleLogout" title="Выйти">
					<LogOut class="w-5 h-5 text-muted-foreground hover:text-destructive" />
				</Button>
			</div>

			<ScrollArea class="flex-1">
				<div class="flex flex-col gap-1 p-2">
					<button
						v-for="chat in store.chats"
						:key="chat.id"
						@click="store.selectChat(chat.id)"
						:class="[
							'flex items-center gap-3 p-3 rounded-lg text-left transition-colors text-sm font-medium',
							store.activeChatId === chat.id
								? 'bg-primary text-primary-foreground'
								: 'hover:bg-muted text-muted-foreground hover:text-foreground',
						]"
					>
						<Avatar class="w-10 h-10 border bg-background">
							<AvatarFallback class="text-foreground">{{ getInitials(chat.name) }}</AvatarFallback>
						</Avatar>
						<div class="truncate">
							{{ chat.name || 'Без названия' }}
							<div class="text-xs opacity-70 font-normal">
								{{ chat.isGlobal ? 'Глобальный' : 'Личный' }}
							</div>
						</div>
					</button>
				</div>
			</ScrollArea>
		</aside>

		<main class="flex-1 flex flex-col relative">
			<header
				v-if="store.activeChat"
				class="h-16 border-b flex items-center px-6 bg-background/50 backdrop-blur sticky top-0 z-10"
			>
				<div class="flex items-center gap-3">
					<Avatar>
						<AvatarFallback>{{ getInitials(store.activeChat.name) }}</AvatarFallback>
					</Avatar>
					<div>
						<h3 class="font-semibold leading-none">{{ store.activeChat.name }}</h3>
						<span v-if="!store.isConnected" class="text-xs text-destructive animate-pulse"
							>Переподключение...</span
						>
						<span v-else class="text-xs text-muted-foreground">Online</span>
					</div>
				</div>
			</header>

			<div
				v-else
				class="flex-1 flex items-center justify-center text-muted-foreground flex-col gap-2"
			>
				<div class="text-4xl">👋</div>
				<p>Выберите чат, чтобы начать общение</p>
			</div>

			<ScrollArea v-if="store.activeChat" class="flex-1 p-4">
				<div class="flex flex-col gap-4 max-w-3xl mx-auto pb-4">
					<div
						v-for="msg in store.messages"
						:key="msg.id"
						:class="[
							'flex gap-3 max-w-[80%]',
							msg.senderEmail === myEmail ? 'self-end flex-row-reverse' : 'self-start',
						]"
					>
						<Avatar v-if="msg.senderEmail !== myEmail" class="w-8 h-8 mt-1">
							<AvatarFallback class="text-xs">{{ getInitials(msg.senderEmail) }}</AvatarFallback>
						</Avatar>

						<div
							:class="[
								'rounded-2xl px-4 py-2 text-sm shadow-sm relative group',
								msg.senderEmail === myEmail
									? 'bg-primary text-primary-foreground rounded-tr-sm'
									: 'bg-muted text-foreground rounded-tl-sm',
							]"
						>
							<div
								v-if="msg.senderEmail !== myEmail && store.activeChat.isGlobal"
								class="text-[10px] opacity-50 mb-1 font-bold"
							>
								{{ msg.senderEmail }}
							</div>

							<div v-if="msg.fileUrl" class="mb-2 mt-1">
								<img
									v-if="msg.fileUrl.match(/\.(jpeg|jpg|gif|png)$/i)"
									:src="`${apiUrl}${msg.fileUrl}`"
									class="rounded-md max-w-[200px] max-h-[200px] object-cover border cursor-pointer hover:opacity-90"
									@click="openFile(`${apiUrl}${msg.fileUrl}`)"
								/>
								<a
									v-else
									:href="`${apiUrl}${msg.fileUrl}`"
									target="_blank"
									class="flex items-center gap-2 bg-background/20 p-2 rounded hover:bg-background/30 transition-colors border border-transparent hover:border-border"
								>
									<FileIcon class="w-4 h-4" />
									<span class="underline underline-offset-2">Скачать файл</span>
								</a>
							</div>

							<p class="whitespace-pre-wrap wrap-break-word leading-relaxed">{{ msg.text }}</p>

							<div
								:class="[
									'text-[10px] text-right mt-1 opacity-60',
									msg.senderEmail === myEmail ? 'text-primary-foreground' : 'text-muted-foreground',
								]"
							>
								{{ formatTime(msg.createdAt) }}
							</div>
						</div>
					</div>

					<div ref="messagesEndRef" class="h-1"></div>
				</div>
			</ScrollArea>

			<div v-if="store.activeChat" class="p-4 bg-background border-t">
				<div class="max-w-3xl mx-auto">
					<div
						v-if="selectedFile"
						class="flex items-center gap-2 mb-2 p-2 bg-muted/50 rounded-md w-fit border text-xs"
					>
						<FileIcon class="w-4 h-4 text-blue-500" />
						<span class="max-w-[200px] truncate font-medium">{{ selectedFile.name }}</span>
						<Button
							variant="ghost"
							size="icon"
							class="h-5 w-5 ml-2 hover:bg-destructive/10 hover:text-destructive"
							@click="removeFile"
						>
							<X class="w-3 h-3" />
						</Button>
					</div>

					<form @submit.prevent="handleSendMessage" class="flex gap-2 items-end">
						<input type="file" ref="fileInput" class="hidden" @change="handleFileSelect" />

						<Button
							type="button"
							variant="outline"
							size="icon"
							class="shrink-0"
							@click="fileInput?.click()"
							title="Прикрепить файл"
						>
							<Paperclip class="w-5 h-5 text-muted-foreground" />
						</Button>

						<Input
							v-model="messageText"
							placeholder="Напишите сообщение..."
							class="flex-1 min-h-10"
							:disabled="!store.isConnected"
						/>

						<Button
							type="submit"
							size="icon"
							:disabled="(!messageText && !selectedFile) || !store.isConnected"
						>
							<SendHorizontal class="w-5 h-5" />
						</Button>
					</form>
				</div>
			</div>
		</main>
	</div>
</template>
