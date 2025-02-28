<template>
	<div>
		<button
			@click="showModal = true"
			class="group fixed top-0 left-0 right-0 z-50 flex items-center justify-between px-4 py-3 bg-white/80 backdrop-blur-md transition-all duration-300 hover:bg-white/90 border-b border-gray-100/50 shadow-sm">
			<!-- Left Side with Status -->
			<div class="flex items-center gap-2.5">
				<!-- Animated Status Indicator -->
				<div class="relative">
					<div class="relative flex items-center justify-center h-6 w-6">
						<span
							class="absolute inset-0 rounded-full animate-pulse"
							:class="isOpen ? 'bg-green-100' : 'bg-red-100'"></span>
						<div class="relative flex h-2.5 w-2.5">
							<span
								class="absolute inline-flex h-full w-full rounded-full opacity-75 animate-ping"
								:class="isOpen ? 'bg-green-400' : 'bg-red-400'"></span>
							<span
								class="relative inline-flex rounded-full h-2.5 w-2.5"
								:class="isOpen ? 'bg-green-500' : 'bg-red-500'"></span>
						</div>
					</div>
				</div>

				<!-- Status Badge -->
				<div
					class="flex items-center gap-1.5 px-2.5 py-1 rounded-full transition-colors duration-200"
					:class="isOpen ? 'bg-green-50' : 'bg-red-50'">
					<span
						class="text-xs font-medium"
						:class="isOpen ? 'text-green-700' : 'text-red-700'"
						v-html="isOpen ? 'Oggi <br /> Aperto' : 'Oggi <br /> Chiuso'">
					</span>
				</div>
			</div>

			<!-- Center Logo with subtle animation -->
			<div
				class="absolute left-1/2 -translate-x-1/2 transition-all duration-300 group-hover:scale-105">
				<img
					src="../assets/La-loggia-completo.svg"
					class="h-[75px] w-auto select-none pointer-events-none transition-opacity duration-300"
					alt="La Loggia Logo" />
			</div>

			<!-- Right Side Info Button -->
			<div class="flex items-center gap-2">
				<div
					class="flex items-center gap-1.5 px-2.5 py-1 rounded-full bg-gray-50 hover:bg-gray-100 transition-colors">
					<span class="text-xs font-medium text-gray-600">Info</span>
					<i
						class="fas fa-chevron-right text-[10px] text-gray-400 transition-transform duration-300 group-hover:translate-x-0.5"></i>
				</div>
			</div>

			<!-- Hover Effect Overlay -->
			<div
				class="absolute inset-0 opacity-0 group-hover:opacity-100 pointer-events-none transition-opacity duration-300 bg-gradient-to-r from-white/50 via-transparent to-white/50"></div>
		</button>

		<!-- Fullscreen Modal -->
		<Transition
			enter-active-class="transition-all duration-300 ease-out"
			enter-from-class="opacity-0"
			enter-to-class="opacity-100"
			leave-active-class="transition-all duration-200 ease-in"
			leave-from-class="opacity-100"
			leave-to-class="opacity-0">
			<div v-if="showModal" class="fixed inset-0 z-[60] flex flex-col bg-white">
				<!-- Modal Header -->
				<div
					class="flex-shrink-0 flex justify-between items-center p-4 sm:p-6 border-b border-gray-100 bg-white sticky top-0 z-10">
					<div>
						<img
							src="../assets/La-loggia-completo.svg"
							alt="La Loggia Logo"
							class="w-[150px]" />
						<p class="text-sm text-gray-500">Informazioni e contatti</p>
					</div>
					<button
						@click="showModal = false"
						class="p-2 text-gray-400 hover:text-gray-600 rounded-full hover:bg-gray-50/80 transition-colors">
						<i class="fas fa-times text-xl"></i>
					</button>
				</div>

				<!-- Scrollable Content -->
				<div class="flex-1 overflow-y-auto">
					<div class="max-w-2xl mx-auto px-4 py-6 space-y-6">
						<!-- Status Section -->
						<div
							class="p-4 rounded-xl border"
							:class="
								isOpen
									? 'bg-green-50 border-green-100'
									: 'bg-red-50 border-red-100'
							">
							<div class="flex items-center gap-2.5">
								<i
									:class="[
										'fas',
										isOpen ? 'fa-door-open' : 'fa-door-closed',
									]"></i>
								<div>
									<p
										class="font-medium"
										:class="isOpen ? 'text-green-800' : 'text-red-800'">
										{{ isOpen ? "Aperto" : "Chiuso" }}
									</p>
									<p class="text-sm text-gray-600">
										{{
											isOpen ? "Orario di chiusura: 22:30" : getClosedReason()
										}}
									</p>
								</div>
							</div>
						</div>

						<!-- Contact Info -->
						<div class="space-y-4">
							<a
								href="https://maps.google.com"
								target="_blank"
								class="block p-3.5 rounded-xl bg-gray-50 hover:bg-gray-100 transition-colors group">
								<div class="flex items-start gap-3">
									<i
										class="fas fa-location-dot mt-1 text-gray-400 group-hover:text-gray-600"></i>
									<div>
										<p class="font-medium text-gray-900">Indirizzo</p>
										<p class="text-sm text-gray-600">
											Via Example, 123<br />12345 City (PR)
										</p>
									</div>
									<i
										class="fas fa-arrow-up-right-from-square ml-auto mt-1 text-gray-400 opacity-0 group-hover:opacity-100 transition-opacity"></i>
								</div>
							</a>

							<a
								href="tel:+123456789"
								class="block p-3.5 rounded-xl bg-gray-50 hover:bg-gray-100 transition-colors group">
								<div class="flex items-center gap-3">
									<i
										class="fas fa-phone text-gray-400 group-hover:text-gray-600"></i>
									<div>
										<p class="font-medium text-gray-900">Telefono</p>
										<p class="text-sm text-gray-600">+39 123 456 789</p>
									</div>
									<i
										class="fas fa-arrow-up-right-from-square ml-auto text-gray-400 opacity-0 group-hover:opacity-100 transition-opacity"></i>
								</div>
							</a>

							<div class="p-3.5 rounded-xl bg-gray-50">
								<p class="font-medium text-gray-900 mb-3">Social</p>
								<div class="flex gap-3">
									<a
										href="#"
										target="_blank"
										class="p-2.5 rounded-lg bg-white shadow-sm hover:shadow transition-all text-gray-600 hover:text-gray-900">
										<i class="fab fa-facebook-f w-5 text-center"></i>
									</a>
									<a
										href="#"
										target="_blank"
										class="p-2.5 rounded-lg bg-white shadow-sm hover:shadow transition-all text-gray-600 hover:text-gray-900">
										<i class="fab fa-instagram w-5 text-center"></i>
									</a>
									<a
										href="#"
										target="_blank"
										class="p-2.5 rounded-lg bg-white shadow-sm hover:shadow transition-all text-gray-600 hover:text-gray-900">
										<i class="fab fa-tripadvisor w-5 text-center"></i>
									</a>
								</div>
							</div>
						</div>

						<!-- Opening Hours -->
						<div>
							<h4 class="text-lg font-medium text-gray-900 mb-4">
								Orari di apertura
							</h4>
							<div
								class="divide-y divide-gray-100 bg-gray-50 rounded-xl overflow-hidden">
								<div
									v-for="(hours, day) in openingHours"
									:key="day"
									class="flex items-center justify-between p-4"
									:class="{ 'bg-blue-50/50': isToday(day) }">
									<div class="flex items-center gap-2">
										<span
											class="text-sm font-medium"
											:class="isToday(day) ? 'text-blue-600' : 'text-gray-900'">
											{{ day }}
										</span>
										<span
											v-if="isToday(day)"
											class="px-1.5 py-0.5 text-[10px] font-medium rounded-full bg-blue-100 text-blue-600">
											oggi
										</span>
									</div>
									<span
										class="text-sm"
										:class="
											(isToday(day) && !isOpen) || hours === 'Chiuso'
												? 'font-medium text-red-500'
												: 'text-gray-600'
										">
										{{ isToday(day) ? (isOpen ? hours : "Chiuso") : hours }}
									</span>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</Transition>
	</div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const isTouchDevice = ref(false);
const isOpen = ref(false);
const showModal = ref(false);

const openingHours = {
	Lunedì: "19:00 - 22:30",
	Martedì: "Chiuso",
	Mercoledì: "19:00 - 22:30",
	Giovedì: "19:00 - 22:30",
	Venerdì: "19:00 - 22:30",
	Sabato: "19:00 - 23:00",
	Domenica: "19:00 - 22:30",
};

const dayMap = {
	0: "Domenica",
	1: "Lunedì",
	2: "Martedì",
	3: "Mercoledì",
	4: "Giovedì",
	5: "Venerdì",
	6: "Sabato",
};

const isToday = (day) => {
	const today = new Date().getDay();
	return day === dayMap[today];
};

const getClosedReason = () => {
	const today = new Date().getDay();
	if (today === 2) {
		return "Il ristorante è chiuso il martedì per riposo settimanale.";
	}
	return "Il ristorante oggi è chiuso.";
};

const checkRestaurantStatus = async () => {
	try {
		const response = await axios.get(
			"http://127.0.0.1:8001/api/restaurant-status"
		);
		const apiIsOpen = response.data.is_open === 1;

		if (!apiIsOpen) {
			isOpen.value = false;
			return;
		}

		const today = new Date().getDay();
		isOpen.value = today !== 2; // Closed on Tuesday (2)
	} catch (error) {
		console.error("Error fetching restaurant status:", error);
		isOpen.value = false;
	}
};

onMounted(() => {
	// Detect touch device
	isTouchDevice.value =
		"ontouchstart" in window ||
		navigator.maxTouchPoints > 0 ||
		navigator.msMaxTouchPoints > 0;

	// Add class to html element for touch detection in CSS
	if (isTouchDevice.value) {
		document.documentElement.classList.add("touch-device");
	}

	// Check status initially and every 5 minutes
	checkRestaurantStatus();
	setInterval(checkRestaurantStatus, 5 * 60 * 1000);
});
</script>

<style scoped>
/* Update hover effects */
@media (hover: hover) and (pointer: fine) {
	.group:hover {
	}
}

@media (hover: none) and (pointer: coarse) {
	.group:active {
	}
}

/* Add smooth scroll padding when header is fixed */
:root {
	scroll-padding-top: 76px;
}

/* Enhance hover effects */
.group:hover .backdrop-blur-md {
	--tw-backdrop-blur: blur(16px);
}

/* Add smooth shadow transition */
.group {
	--tw-shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1);
	transition: box-shadow 0.3s ease;
}

.group:hover {
	--tw-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
}
</style>
