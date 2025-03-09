<template>
	<footer class="pb-6 mt-0">
		<div class="max-w-6xl mx-auto px-4">
			<div class="flex flex-col items-center justify-center space-y-4">
				<!-- Logo -->
				<img
					src="../assets/La-loggia-completo.svg"
					alt="La Loggia"
					class="h-20 w-auto opacity-70" />

				<!-- Copyright -->
				<div class="text-xs text-gray-400 pt-2">
					© {{ new Date().getFullYear() }} La Loggia. Tutti i diritti riservati.
				</div>
			</div>
		</div>
	</footer>
</template>

<script setup>
// No complex logic needed for this minimal footer
</script>

<style scoped>
/* Match the styling from StatusBanner component */
a,
button {
	transition: transform 0.2s ease;
}

a:hover,
button:hover {
	transform: scale(1.03);
}
</style>
