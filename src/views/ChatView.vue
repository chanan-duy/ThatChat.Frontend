<script setup lang="ts">
import { Avatar, AvatarFallback } from '@/components/ui/avatar'
import { Button } from '@/components/ui/button'
import {
	Dialog,
	DialogContent,
	DialogDescription,
	DialogFooter,
	DialogHeader,
	DialogTitle,
	DialogTrigger,
} from '@/components/ui/dialog'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { ScrollArea } from '@/components/ui/scroll-area'
import { auth } from '@/services/authService'
import { useChatStore } from '@/stores/chatStore'
import { isAxiosError } from 'axios'
import {
	AudioWaveformIcon,
	CircleDot,
	Download,
	FileIcon,
	LogOut,
	Paperclip,
	PlusIcon,
	SendHorizontal,
	X,
} from 'lucide-vue-next'
import { nextTick, onMounted, ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { toast } from 'vue-sonner'

const router = useRouter()
const store = useChatStore()
const apiUrl = import.meta.env.VITE_API_URL

const messageText = ref('')
const fileInput = ref<HTMLInputElement | null>(null)
const selectedFile = ref<File | null>(null)

const messagesEndRef = ref<HTMLDivElement | null>(null)
const textareaRef = ref<HTMLTextAreaElement | null>(null)

const myEmail = localStorage.getItem('__userEmail') || '' // I know, I know
const isCreateChatOpen = ref(false)
const newChatEmail = ref('')

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
		resizeTextarea()
		await nextTick()
		scrollToBottom()
	},
)

function scrollToBottom() {
	if (!messagesEndRef.value) return

	messagesEndRef.value.scrollIntoView({
		behavior: 'instant',
		block: 'end',
	})
}

function handleImageLoad() {
	scrollToBottom()
}

function resizeTextarea() {
	if (textareaRef.value) {
		textareaRef.value.style.height = 'auto'
		textareaRef.value.style.height = `${Math.min(textareaRef.value.scrollHeight, 150)}px`
	}
}

function handleInput() {
	resizeTextarea()
}

async function handleSendMessage() {
	if (!messageText.value.trim() && !selectedFile.value) return

	try {
		await store.sendMessage(messageText.value, selectedFile.value)

		messageText.value = ''
		selectedFile.value = null
		if (fileInput.value) fileInput.value.value = ''

		resizeTextarea()
		scrollToBottom()
	} catch (e) {
		console.error(e)
		toast.error('Не удалось отправить сообщение', {
			description: `${e}\n${(isAxiosError(e) && e.response?.data) || ''}`,
		})
	}
}

function handleKeydown(e: KeyboardEvent) {
	if (e.key === 'Enter' && !e.shiftKey) {
		e.preventDefault()
		handleSendMessage()
	}
}

function handleFileSelect(event: Event) {
	const target = event.target as HTMLInputElement
	if (target.files && target.files.length > 0) {
		selectedFile.value = target.files[0] || null
	}
}

function removeFile() {
	selectedFile.value = null
	if (fileInput.value) fileInput.value.value = ''
}

async function handleCreateChat() {
	if (!newChatEmail.value) return
	try {
		await store.createPrivateChat(newChatEmail.value)
		isCreateChatOpen.value = false
		newChatEmail.value = ''
		toast.success('Чат создан')
	} catch (e) {
		console.error(e)
		toast.error('Ошибка. Пользователь не найден?')
	}
}

function handleLogout() {
	auth.logout()
	router.push('/login')
}

function openFile(url: string) {
	window.open(url, '_blank')
}

function getInitials(name?: string) {
	if (!name) return '??'
	return name.substring(0, 2).toUpperCase()
}

function formatTime(dateStr: string) {
	return new Date(dateStr).toLocaleTimeString([], {
		hour: '2-digit',
		minute: '2-digit',
		hour12: false,
	})
}

function getFileUrl(path: string) {
	if (!path) return ''
	if (path.startsWith('http')) return path
	const base = apiUrl.replace(/\/$/, '')
	const relative = path.replace(/^\//, '')
	return `${base}/${relative}`
}
</script>

<template>
	<div class="flex h-screen w-full bg-background overflow-hidden">
		<aside class="w-80 border-r flex flex-col bg-card/30 shrink-0">
			<div class="p-4 border-b flex items-center justify-between h-16 shrink-0">
				<h2 class="font-semibold text-lg tracking-tight">Чаты</h2>
				<div class="flex gap-1">
					<Dialog v-model:open="isCreateChatOpen">
						<DialogTrigger as-child>
							<Button variant="ghost" size="icon" title="Создать чат" class="h-8 w-8">
								<PlusIcon class="w-5 h-5" />
							</Button>
						</DialogTrigger>
						<DialogContent class="sm:max-w-[425px]">
							<DialogHeader>
								<DialogTitle>Новый чат</DialogTitle>
								<DialogDescription>
									Введите Email пользователя для начала общения.
								</DialogDescription>
							</DialogHeader>
							<div class="grid gap-4 py-4">
								<div class="grid gap-2">
									<Label for="email">Email собеседника</Label>
									<Input
										id="email"
										v-model="newChatEmail"
										placeholder="friend@example.com"
										@keyup.enter="handleCreateChat"
									/>
								</div>
							</div>
							<DialogFooter>
								<Button type="submit" @click="handleCreateChat">Начать общение</Button>
							</DialogFooter>
						</DialogContent>
					</Dialog>
					<Button variant="ghost" size="icon" @click="handleLogout" title="Выйти" class="h-8 w-8">
						<LogOut class="w-4 h-4 text-muted-foreground hover:text-destructive" />
					</Button>
				</div>
			</div>

			<ScrollArea class="flex-1">
				<div class="flex flex-col gap-1 p-2">
					<button
						v-for="chat in store.chats"
						:key="chat.id"
						@click="store.selectChat(chat.id)"
						:class="[
							'flex items-center gap-3 p-3 rounded-lg text-left transition-all duration-200',
							store.activeChatId === chat.id
								? 'bg-secondary text-secondary-foreground shadow-sm'
								: 'hover:bg-muted/50 text-muted-foreground hover:text-foreground',
						]"
					>
						<Avatar class="w-10 h-10 border shadow-sm">
							<AvatarFallback class="bg-background text-foreground/80 text-xs font-medium">
								{{ getInitials(chat.name) }}
							</AvatarFallback>
						</Avatar>
						<div class="flex-1 min-w-0">
							<div class="font-medium text-sm truncate leading-none mb-1">
								{{ chat.name || 'Без названия' }}
							</div>
							<div class="text-[11px] opacity-70 font-normal flex items-center gap-1">
								<CircleDot
									:class="chat.isGlobal ? 'text-blue-500' : 'text-green-500'"
									fill="currentColor"
									class="w-2 h-2"
								/>
								{{ chat.isGlobal ? 'Глобальный' : 'Личный' }}
							</div>
						</div>
					</button>
				</div>
			</ScrollArea>
		</aside>

		<main class="flex-1 flex flex-col h-full overflow-hidden relative min-w-0 bg-background">
			<header
				v-if="store.activeChat"
				class="h-16 border-b shrink-0 flex items-center px-6 bg-background/80 backdrop-blur z-20"
			>
				<div class="flex items-center gap-3">
					<Avatar class="h-9 w-9 border">
						<AvatarFallback class="bg-primary/10 text-primary text-xs">
							{{ getInitials(store.activeChat.name) }}
						</AvatarFallback>
					</Avatar>
					<div>
						<h3 class="font-semibold text-sm leading-none mb-1">{{ store.activeChat.name }}</h3>
					</div>
				</div>
			</header>

			<div
				v-if="!store.activeChat"
				class="flex-1 flex items-center justify-center text-muted-foreground flex-col gap-4 p-8 text-center"
			>
				<div class="w-16 h-16 rounded-2xl bg-muted flex items-center justify-center">
					<AudioWaveformIcon :size="34" />
				</div>
				<div class="flex flex-col gap-2">
					<h3 class="font-semibold text-lg text-foreground">Добро пожаловать</h3>
					<p class="text-sm">Выберите чат из списка или создайте новый для начала общения</p>
				</div>
			</div>

			<div
				v-if="store.activeChat"
				ref="scrollContainerRef"
				class="flex-1 overflow-y-auto min-h-0 scroll-smooth"
			>
				<div class="flex flex-col gap-6 max-w-4xl mx-auto pb-2">
					<div
						v-for="msg in store.messages"
						:key="msg.id"
						:class="[
							'flex gap-3 max-w-[85%]',
							msg.senderEmail === myEmail ? 'self-end flex-row-reverse' : 'self-start',
						]"
					>
						<Avatar v-if="msg.senderEmail !== myEmail" class="w-8 h-8 mt-auto shrink-0 border">
							<AvatarFallback class="text-[10px] bg-muted">
								{{ getInitials(msg.senderEmail) }}
							</AvatarFallback>
						</Avatar>

						<div
							:class="[
								'flex flex-col shadow-sm border relative overflow-hidden min-w-[60px]',
								msg.senderEmail === myEmail
									? 'bg-primary text-primary-foreground rounded-2xl rounded-br-none border-primary/20'
									: 'bg-card text-card-foreground rounded-2xl rounded-bl-none border-border',
							]"
						>
							<div
								v-if="msg.senderEmail !== myEmail && store.activeChat.isGlobal"
								class="px-3 pt-2 text-[10px] text-primary/70 font-bold"
							>
								{{ msg.senderEmail || 'Unknown' }}
							</div>

							<div class="p-3">
								<div v-if="msg.fileUrl" class="mb-2">
									<div
										v-if="msg.fileUrl.match(/\.(jpeg|jpg|gif|png|webp)$/i)"
										class="relative group"
									>
										<img
											:src="getFileUrl(msg.fileUrl)"
											class="rounded-lg object-contain bg-black/5 dark:bg-white/5 cursor-zoom-in block"
											style="max-height: 350px; width: auto; max-width: 100%"
											alt="Attachment"
											loading="lazy"
											@click="openFile(getFileUrl(msg.fileUrl))"
											@load="handleImageLoad"
										/>
									</div>

									<a
										v-else
										:href="getFileUrl(msg.fileUrl)"
										target="_blank"
										class="flex items-center gap-3 p-3 rounded-lg transition-colors border group"
										:class="
											msg.senderEmail === myEmail
												? 'bg-primary-foreground/10 hover:bg-primary-foreground/20 border-primary-foreground/20'
												: 'bg-background/50 hover:bg-background/80 border-border/50'
										"
									>
										<div
											class="p-2 rounded-full transition-colors"
											:class="
												msg.senderEmail === myEmail
													? 'bg-primary-foreground/10 text-primary-foreground'
													: 'bg-primary/10 text-primary group-hover:bg-primary/20'
											"
										>
											<FileIcon class="w-4 h-4" />
										</div>
										<div class="flex-1 overflow-hidden">
											<div class="text-xs font-medium truncate">Файл вложения</div>
											<div class="text-[10px] opacity-70">Нажмите чтобы скачать</div>
										</div>
										<Download class="w-4 h-4 opacity-70" />
									</a>
								</div>

								<p
									v-if="msg.text"
									class="text-sm whitespace-pre-wrap wrap-break-word leading-relaxed"
								>
									{{ msg.text }}
								</p>
							</div>

							<div class="px-3 pb-1.5 text-[10px] text-right opacity-60 select-none font-medium">
								{{ formatTime(msg.createdAt) }}
							</div>
						</div>
					</div>

					<div ref="messagesEndRef" class="h-px w-full"></div>
				</div>
			</div>

			<div v-if="store.activeChat" class="shrink-0 p-4 bg-background border-t z-20">
				<div class="max-w-4xl mx-auto">
					<div
						v-if="selectedFile"
						class="flex items-center gap-3 mb-3 p-2 bg-muted/40 rounded-lg w-fit border shadow-sm animate-in slide-in-from-bottom-2 fade-in duration-200"
					>
						<div class="w-8 h-8 rounded bg-background flex items-center justify-center border">
							<FileIcon class="w-4 h-4 text-blue-500" />
						</div>
						<div class="flex flex-col">
							<span class="text-xs font-medium max-w-[200px] truncate">{{
								selectedFile.name
							}}</span>
							<span class="text-[10px] text-muted-foreground"
								>{{ (selectedFile.size / 1024).toFixed(1) }} KB</span
							>
						</div>
						<Button
							variant="ghost"
							size="icon"
							class="h-6 w-6 ml-1 hover:bg-destructive/10 hover:text-destructive rounded-full"
							@click="removeFile"
						>
							<X class="w-3 h-3" />
						</Button>
					</div>

					<div class="flex gap-2 items-end">
						<input type="file" ref="fileInput" class="hidden" @change="handleFileSelect" />

						<Button
							type="button"
							variant="outline"
							size="icon"
							class="shrink-0 h-[42px] w-[42px] rounded-xl"
							@click="fileInput?.click()"
							title="Прикрепить файл"
						>
							<Paperclip class="w-5 h-5 text-muted-foreground" />
						</Button>

						<div class="flex-1 relative">
							<textarea
								ref="textareaRef"
								v-model="messageText"
								rows="1"
								placeholder="Напишите сообщение..."
								class="flex w-full rounded-xl border border-input bg-transparent px-4 py-2.5 text-sm shadow-sm placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:cursor-not-allowed disabled:opacity-50 min-h-[42px] max-h-[150px] resize-none overflow-y-auto leading-relaxed"
								:disabled="!store.isConnected"
								@input="handleInput"
								@keydown="handleKeydown"
							></textarea>
						</div>

						<Button
							type="submit"
							size="icon"
							class="shrink-0 h-[42px] w-[42px] rounded-xl transition-all active:scale-95"
							:disabled="(!messageText.trim() && !selectedFile) || !store.isConnected"
							@click="handleSendMessage"
						>
							<SendHorizontal class="w-5 h-5 ml-0.5" />
						</Button>
					</div>
				</div>
			</div>
		</main>
	</div>
</template>
