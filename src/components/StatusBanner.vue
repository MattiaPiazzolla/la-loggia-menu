<template>
	<div>
		<!-- Floating Status Button -->
		<button
			@click="showModal = true"
			:class="[
				'group fixed top-4 right-4 z-50 flex items-center gap-2 px-3 py-2 bg-white/90 backdrop-blur-lg rounded-full shadow-md transition-all duration-300 hover:shadow-lg hover:bg-white border border-gray-100/50',
				{ 'opacity-50': isScrollingDown },
			]">
			<span class="relative flex h-3 w-3">
				<span
					class="absolute inline-flex h-full w-full rounded-full opacity-50 animate-ping"
					:class="isOpen ? 'bg-green-400' : 'bg-red-400'"></span>
				<span
					class="relative inline-flex rounded-full h-3 w-3"
					:class="isOpen ? 'bg-green-500' : 'bg-red-500'"></span>
			</span>

			<span class="text-sm font-medium text-gray-800">
				{{ isOpen ? "Aperto" : "Chiuso" }}
			</span>

			<span
				class="ml-1 text-gray-400 transition-transform duration-300 group-hover:translate-x-0.5">
				<i class="fas fa-chevron-right text-[10px]"></i>
			</span>
		</button>

		<!-- Fullscreen Modal -->
		<Transition
			enter-active-class="transition-all duration-300 ease-out"
			enter-from-class="opacity-0 translate-y-4"
			enter-to-class="opacity-100 translate-y-0"
			leave-active-class="transition-all duration-200 ease-in"
			leave-from-class="opacity-100 translate-y-0"
			leave-to-class="opacity-0 translate-y-4">
			<div v-if="showModal" class="fixed inset-0 z-[60] flex flex-col bg-white">
				<!-- Modal Header -->
				<div
					class="flex justify-between items-center p-4 bg-white sticky top-0 z-10">
					<img
						src="../assets/La-loggia-completo.svg"
						alt="La Loggia"
						class="h-12 w-auto" />

					<button
						@click="showModal = false"
						class="p-2 rounded-full hover:bg-gray-100 transition-colors">
						<i class="fas fa-times"></i>
					</button>
				</div>

				<!-- Scrollable Content -->
				<div class="flex-1 overflow-y-auto">
					<div class="max-w-md mx-auto px-4 py-8 space-y-8">
						<!-- Status Circle -->
						<div class="flex justify-center">
							<div class="relative flex items-center justify-center h-32 w-32">
								<div class="text-center">
									<div
										class="text-3xl mb-1"
										:class="isOpen ? 'text-green-500' : 'text-red-500'">
										<i
											:class="['fas', isOpen ? 'fa-utensils' : 'fa-clock']"></i>
									</div>
									<p
										class="text-lg font-medium"
										:class="isOpen ? 'text-green-700' : 'text-red-700'">
										{{ isOpen ? "Aperto" : "Chiuso" }}
									</p>
									<p class="text-xs text-gray-500 mt-1">
										{{ isOpen ? getClosingInfo() : getNextOpeningInfo() }}
									</p>
								</div>
							</div>
						</div>

						<!-- Opening Hours -->
						<div class="bg-gray-50 rounded-2xl overflow-hidden">
							<div class="p-4 border-b border-gray-100">
								<h2 class="text-base font-medium text-gray-800">Orari</h2>
							</div>

							<div class="divide-y divide-gray-100">
								<div
									v-for="(hours, day) in openingHours"
									:key="day"
									class="p-4 flex items-center space-x-4"
									:class="{
										'bg-blue-50/30 border-l-4 border-blue-400': isToday(day),
									}">
									<!-- Day Circle -->
									<div
										class="h-10 w-10 rounded-full flex items-center justify-center text-sm font-medium"
										:class="
											isToday(day)
												? 'bg-blue-100 text-blue-800'
												: 'bg-gray-100 text-gray-800'
										">
										{{ getDayInitial(day) }}
									</div>

									<!-- Day Info -->
									<div class="flex-1">
										<div class="flex items-center">
											<p
												class="text-sm font-medium"
												:class="
													isToday(day) ? 'text-blue-800' : 'text-gray-800'
												">
												{{ day }}
											</p>
											<span
												v-if="isToday(day)"
												class="ml-2 px-1.5 py-0.5 text-[10px] font-medium rounded-full bg-blue-100 text-blue-800">
												oggi
											</span>
										</div>

										<!-- Hours -->
										<div v-if="hours === 'Chiuso'" class="mt-1">
											<span class="text-xs font-medium text-red-500"
												>Chiuso</span
											>
										</div>
										<div v-else class="mt-1 flex flex-wrap gap-2">
											<div
												v-if="hours.lunch"
												class="flex items-center text-xs text-gray-600">
												<i class="fas fa-sun text-amber-400 mr-1.5"></i>
												<span>{{ hours.lunch }}</span>
											</div>
											<div
												v-if="hours.dinner"
												class="flex items-center text-xs text-gray-600">
												<i class="fas fa-moon text-indigo-400 mr-1.5"></i>
												<span>{{ hours.dinner }}</span>
											</div>
										</div>
									</div>
								</div>
							</div>
						</div>

						<!-- Contact & Info Cards -->
						<div class="grid grid-cols-2 gap-3">
							<a
								href="https://maps.google.com"
								target="_blank"
								class="p-4 rounded-2xl bg-gray-50 hover:bg-gray-100 transition-colors flex flex-col items-center text-center group">
								<i
									class="fas fa-location-dot text-gray-400 group-hover:text-gray-600 text-xl mb-2"></i>
								<span class="text-xs text-gray-800">Via Example, 123</span>
							</a>

							<a
								href="tel:+123456789"
								class="p-4 rounded-2xl bg-gray-50 hover:bg-gray-100 transition-colors flex flex-col items-center text-center group">
								<i
									class="fas fa-phone text-gray-400 group-hover:text-gray-600 text-xl mb-2"></i>
								<span class="text-xs text-gray-800">+39 123 456 789</span>
							</a>

							<a
								href="#"
								target="_blank"
								class="p-4 rounded-2xl bg-gray-50 hover:bg-gray-100 transition-colors flex flex-col items-center text-center col-span-2 group">
								<div class="flex space-x-6 mb-2">
									<i
										class="fab fa-facebook-f text-gray-400 group-hover:text-blue-600"></i>
									<i
										class="fab fa-instagram text-gray-400 group-hover:text-pink-600"></i>
									<i
										class="fab fa-tripadvisor text-gray-400 group-hover:text-green-600"></i>
								</div>
								<span class="text-xs text-gray-800">Seguici sui social</span>
							</a>
						</div>

						<!-- Branding -->
						<div class="text-center pt-4">
							<img
								src="../assets/La-loggia-completo.svg"
								alt="La Loggia"
								class="h-12 w-auto mx-auto opacity-50" />
						</div>
					</div>
				</div>
			</div>
		</Transition>
	</div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import axios from "axios";

const isTouchDevice = ref(false);
const isOpen = ref(false);
const showModal = ref(false);
const openingHours = ref({});
const currentMealType = ref(null);
const globalStatus = ref(false);
const isScrollingDown = ref(false);
let lastScrollTop = 0;

const checkGlobalStatus = async () => {
	try {
		const statusResponse = await axios.get(
			"http://127.0.0.1:8001/api/restaurant-status"
		);
		globalStatus.value = statusResponse.data.is_open === 1;

		// If globally closed, ensure isOpen is false
		if (!globalStatus.value) {
			isOpen.value = false;
			currentMealType.value = null;
		} else {
			// If globally open, check schedule
			checkScheduleStatus();
		}
	} catch (error) {
		console.error("Error checking global status:", error);
		globalStatus.value = false;
		isOpen.value = false;
	}
};

const checkScheduleStatus = async () => {
	if (!globalStatus.value) return;

	try {
		const scheduleResponse = await axios.get(
			"http://localhost:8001/api/schedules"
		);
		const schedules = scheduleResponse.data.schedules;
		const today = new Date().getDay();
		const todaySchedule = schedules.find((s) => getDayNumber(s.day) === today);

		// Only check if the day is marked as open
		isOpen.value = todaySchedule?.is_open || false;

		// Set meal type based on schedule (prefer dinner if both are available)
		if (isOpen.value) {
			if (todaySchedule.dinner_opening) {
				currentMealType.value = "dinner";
			} else if (todaySchedule.lunch_opening) {
				currentMealType.value = "lunch";
			} else {
				currentMealType.value = null;
			}
		} else {
			currentMealType.value = null;
		}
	} catch (error) {
		console.error("Error checking schedule status:", error);
		isOpen.value = false;
		currentMealType.value = null;
	}
};

const getDayNumber = (day) => {
	const dayNumbers = {
		Sunday: 0,
		Monday: 1,
		Tuesday: 2,
		Wednesday: 3,
		Thursday: 4,
		Friday: 5,
		Saturday: 6,
	};
	return dayNumbers[day];
};

const fetchSchedules = async () => {
	try {
		const response = await axios.get("http://localhost:8001/api/schedules");
		const schedules = response.data.schedules;
		const status = response.data.status;

		const formattedHours = schedules.reduce((acc, schedule) => {
			// Convert English day names to Italian
			const dayName = translateDayToItalian(schedule.day);

			// Check if this is today's schedule
			const isToday = getDayNumber(schedule.day) === new Date().getDay();

			// If it's today and global status is closed, mark as closed regardless of schedule
			if (isToday && status.is_open === 0) {
				acc[dayName] = "Chiuso";
				return acc;
			}

			// Normal schedule processing
			if (!schedule.is_open) {
				acc[dayName] = "Chiuso";
				return acc;
			}

			acc[dayName] = {};

			if (schedule.lunch_opening && schedule.lunch_closing) {
				acc[
					dayName
				].lunch = `${schedule.lunch_opening} - ${schedule.lunch_closing}`;
			}

			if (schedule.dinner_opening && schedule.dinner_closing) {
				acc[
					dayName
				].dinner = `${schedule.dinner_opening} - ${schedule.dinner_closing}`;
			}

			if (!acc[dayName].lunch && !acc[dayName].dinner) {
				acc[dayName] = "Chiuso";
			}

			return acc;
		}, {});

		openingHours.value = formattedHours;
	} catch (error) {
		console.error("Error fetching schedules:", error);
	}
};

// English to Italian day translation
const translateDayToItalian = (englishDay) => {
	const translations = {
		Sunday: "Domenica",
		Monday: "Lunedì",
		Tuesday: "Martedì",
		Wednesday: "Mercoledì",
		Thursday: "Giovedì",
		Friday: "Venerdì",
		Saturday: "Sabato",
	};

	return translations[englishDay] || englishDay;
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

// Get first letter of day for the circle display
const getDayInitial = (day) => {
	return day.charAt(0);
};

const getClosingInfo = () => {
	if (!currentMealType.value) return "Oggi aperto";

	const today = new Date().getDay();
	const currentDay = dayMap[today];
	const hours = openingHours.value[currentDay];

	if (typeof hours === "object") {
		let timeRange;
		if (currentMealType.value === "lunch" && hours.lunch) {
			timeRange = hours.lunch;
		} else if (currentMealType.value === "dinner" && hours.dinner) {
			timeRange = hours.dinner;
		}

		if (timeRange) {
			const [opening, closing] = timeRange.split(" - ");
			return `${opening} - ${closing}`;
		}
	}

	return "Oggi aperto";
};

const getNextOpeningInfo = () => {
	const today = new Date().getDay();
	let nextOpenDay = today;
	let daysChecked = 0;

	// Find the next open day
	while (daysChecked < 7) {
		nextOpenDay = (nextOpenDay + 1) % 7;
		const nextDay = dayMap[nextOpenDay];

		if (
			openingHours.value[nextDay] &&
			openingHours.value[nextDay] !== "Chiuso"
		) {
			const isItTomorrow = nextOpenDay === (today + 1) % 7;

			if (isItTomorrow) {
				return "Siamo aperti domani";
			} else {
				return `Apriamo ${nextDay}`;
			}
		}

		daysChecked++;
	}

	return "Controlla gli orari";
};

const handleScroll = () => {
	const st = window.pageYOffset || document.documentElement.scrollTop;
	isScrollingDown.value = st > lastScrollTop;
	lastScrollTop = st <= 0 ? 0 : st;
};

onMounted(() => {
	isTouchDevice.value =
		"ontouchstart" in window ||
		navigator.maxTouchPoints > 0 ||
		navigator.msMaxTouchPoints > 0;

	if (isTouchDevice.value) {
		document.documentElement.classList.add("touch-device");
	}

	// Initial checks
	checkGlobalStatus();
	fetchSchedules();

	// Check global status more frequently (every 15 seconds)
	setInterval(checkGlobalStatus, 15 * 1000);

	// Update schedules display every hour
	setInterval(fetchSchedules, 60 * 60 * 1000);

	window.addEventListener("scroll", handleScroll);
});

onUnmounted(() => {
	window.removeEventListener("scroll", handleScroll);
});
</script>

<style scoped>
/* Floating status button shadow */
.fixed {
	filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.05));
}

/* Smooth transitions */
* {
	transition-property: color, background-color, border-color,
		text-decoration-color, fill, stroke, opacity, box-shadow, transform, filter,
		backdrop-filter;
	transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
	transition-duration: 150ms;
}

/* Add subtle scale on hover for all interactive elements */
a,
button {
	transition: transform 0.2s ease;
}

a:hover,
button:hover {
	transform: scale(1.03);
}
</style>
