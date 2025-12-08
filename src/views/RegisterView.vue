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

async function handleRegister() {
	if (!email.value || !password.value) {
		toast.warning('Заполните все поля')
		return
	}

	if (password.value.length < 4) {
		toast.warning('Пароль слишком короткий (минимум 4 символа)')
		return
	}

	isLoading.value = true
	try {
		await auth.register({
			email: email.value,
			password: password.value,
		})

		toast.success('Аккаунт создан! Входим...')

		await auth.login({
			email: email.value,
			password: password.value,
		})

		router.push('/')
	} catch (e: unknown) {
		console.error(e)
		toast.error('Ошибка регистрации. Возможно, email уже занят.')
	} finally {
		isLoading.value = false
	}
}
</script>

<template>
	<div class="flex items-center justify-center min-h-screen bg-zinc-50 dark:bg-zinc-950 p-4">
		<Card class="w-full max-w-md shadow-lg border-zinc-200 dark:border-zinc-800">
			<CardHeader>
				<CardTitle class="text-2xl text-center font-bold">Регистрация</CardTitle>
				<CardDescription class="text-center">Создайте аккаунт в ThatChat</CardDescription>
			</CardHeader>
			<CardContent class="grid gap-4">
				<div class="grid gap-2">
					<Label for="email">Email</Label>
					<Input id="email" type="email" placeholder="name@example.com" v-model="email" />
				</div>
				<div class="grid gap-2">
					<Label for="password">Пароль</Label>
					<Input id="password" type="password" v-model="password" />
				</div>
			</CardContent>
			<CardFooter class="flex flex-col gap-2">
				<Button class="w-full" @click="handleRegister" :disabled="isLoading">
					{{ isLoading ? 'Создаем...' : 'Зарегистрироваться' }}
				</Button>
				<div class="text-center text-sm text-muted-foreground">
					Уже есть аккаунт?
					<RouterLink to="/login" class="underline hover:text-primary transition-colors"
						>Войти</RouterLink
					>
				</div>
			</CardFooter>
		</Card>
	</div>
</template>
