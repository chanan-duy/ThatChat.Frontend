<script setup lang="ts">
import { Toaster } from '@/components/ui/sonner'
import { RouterView } from 'vue-router'
import 'vue-sonner/style.css'
</script>

<template>
	<div>
		<RouterView />

		<Toaster position="top-center" />
	</div>
</template>
