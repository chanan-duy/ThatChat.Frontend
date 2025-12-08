<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { auth } from '@/services/authService'
import { toast } from 'vue-sonner'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import {
	Card,
	CardContent,
	CardDescription,
	CardFooter,
	CardHeader,
	CardTitle,
} from '@/components/ui/card'

const email = ref('')
const password = ref('')
const isLoading = ref(false)
const router = useRouter()

async function handleLogin() {
	if (!email.value || !password.value) {
		toast.warning('Пожалуйста, заполните все поля')
		return
	}

	isLoading.value = true
	try {
		await auth.login({
			email: email.value,
			password: password.value,
		})

		toast.success('Добро пожаловать!')
		router.push('/')
	} catch (e) {
		console.error(e)
		toast.error('Ошибка входа. Проверьте Email или пароль.')
	} finally {
		isLoading.value = false
	}
}
</script>

<template>
	<div class="flex items-center justify-center min-h-screen bg-zinc-50 dark:bg-zinc-950 p-4">
		<Card class="w-full max-w-md shadow-lg border-zinc-200 dark:border-zinc-800">
			<CardHeader>
				<CardTitle class="text-2xl text-center font-bold">ThatChat</CardTitle>
				<CardDescription class="text-center">Вход в систему</CardDescription>
			</CardHeader>
			<CardContent class="grid gap-4">
				<div class="grid gap-2">
					<Label for="email">Email</Label>
					<Input id="email" type="email" placeholder="m@example.com" v-model="email" />
				</div>
				<div class="grid gap-2">
					<Label for="password">Пароль</Label>
					<Input id="password" type="password" v-model="password" @keyup.enter="handleLogin" />
				</div>
			</CardContent>
			<CardFooter class="flex flex-col gap-2">
				<Button class="w-full" @click="handleLogin" :disabled="isLoading">
					{{ isLoading ? 'Входим...' : 'Войти' }}
				</Button>
				<div class="text-center text-sm text-muted-foreground">
					Нет аккаунта?
					<RouterLink to="/register" class="underline hover:text-primary transition-colors"
						>Регистрация</RouterLink
					>
				</div>
			</CardFooter>
		</Card>
	</div>
</template>
