<template>
	<Transition name="fade">
		<div
			v-if="modelValue"
			class="fixed inset-0 z-50 flex items-center justify-center overflow-hidden bg-black/40 backdrop-blur-sm">
			<!-- Modal -->
			<div
				:class="[
					'relative max-h-[95vh] w-[96vw] max-w-[86vw] overflow-hidden rounded-xl bg-white shadow-2xl transition-all duration-300',
					{
						'h-screen w-screen max-h-screen max-w-none rounded-none':
							isFullscreen,
					},
					{ 'flex flex-row h-[95vh]': isLandscape },
				]">
				<!-- Close Button (Floating) -->
				<button
					@click="closeItemModal"
					class="absolute right-3 top-3 z-30 flex h-8 w-8 sm:h-9 sm:w-9 items-center justify-center rounded-full bg-white/90 shadow-md transition-all hover:bg-gray-100 hover:scale-105">
					<i class="fas fa-xmark text-gray-700"></i>
				</button>

				<!-- Content Container -->
				<div :class="['flex h-full', isLandscape ? 'flex-row' : 'flex-col']">
					<!-- Image Section -->
					<div
						:class="[
							'relative overflow-hidden',
							isLandscape ? 'w-1/2 sm:w-6/12' : 'h-56 xs:h-64 sm:h-72',
						]">
						<div class="h-full w-full">
							<img
								:src="item.image_url || '/default-menu-image-placeholder.png'"
								:alt="item.name"
								@click="toggleFullscreen"
								class="h-full w-full cursor-pointer object-cover transition-transform duration-700 hover:scale-110" />
						</div>

						<!-- Image Overlay Gradient -->
						<div
							class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent"></div>

						<!-- Image Content Overlay -->
						<div class="absolute bottom-0 left-0 right-0 p-3 sm:p-4 md:p-5">
							<div class="mb-1 sm:mb-2 flex gap-2">
								<span
									v-if="item.special_label"
									class="rounded-lg bg-amber-500 px-2 py-0.5 sm:px-3 sm:py-1 text-xs sm:text-sm font-bold text-white shadow-sm">
									{{ item.special_label }}
								</span>
								<span
									v-if="item.preparation_time"
									class="flex items-center rounded-lg bg-gray-800/80 px-2 py-0.5 sm:px-3 sm:py-1 text-xs sm:text-sm font-bold text-white shadow-sm backdrop-blur-sm">
									<i class="fas fa-clock mr-1"></i>
									{{ item.preparation_time }} min
								</span>
							</div>

							<h2 class="text-xl sm:text-2xl font-bold text-white shadow-sm">
								{{ item.name }}
							</h2>
							<div class="text-lg sm:text-xl font-semibold text-white/90">
								€{{ formatPrice(item.price) }}
							</div>
						</div>
					</div>

					<!-- Content Section -->
					<div
						ref="modalContent"
						:class="[
							'flex-1 overflow-y-auto',
							isLandscape ? 'w-1/2 sm:w-6/12' : '',
						]">
						<!-- Content Wrapper -->
						<div class="p-4 sm:p-5 md:p-6">
							<!-- Quick Stats -->
							<div
								class="mb-4 sm:mb-5 md:mb-6 flex items-center justify-between">
								<div
									v-if="item.calories"
									class="flex items-center gap-1.5 rounded-full bg-orange-50 px-3 py-1 sm:px-4 sm:py-1.5 text-xs sm:text-sm font-medium text-orange-700">
									<i class="fas fa-fire text-orange-500"></i>
									<span>{{ item.calories }} kcal</span>
								</div>

								<div v-if="item.spiciness" class="flex gap-1 sm:gap-1.5">
									<div
										v-for="n in 5"
										:key="n"
										:class="[
											'h-2 w-2 sm:h-3 sm:w-3 rounded-full transition-all',
											n <= item.spiciness
												? `bg-red-${Math.min(n * 100 + 300, 600)}`
												: 'bg-gray-200',
										]"></div>
								</div>
							</div>

							<!-- Description Card -->
							<div
								class="mb-4 sm:mb-5 md:mb-6 rounded-xl bg-gray-50 p-3 sm:p-4">
								<p
									:class="[
										'text-sm sm:text-base text-gray-700 leading-relaxed',
										{ 'line-clamp-3': !showFullDescription },
									]">
									{{ item.description }}
								</p>
								<button
									v-if="item.description?.length > 100"
									@click="showFullDescription = !showFullDescription"
									class="mt-1.5 sm:mt-2 inline-flex items-center text-xs sm:text-sm font-medium text-blue-600 hover:text-blue-800">
									<span>{{
										showFullDescription ? "Show less" : "Show more"
									}}</span>
									<i
										:class="[
											'fas ml-1 transition-transform',
											showFullDescription ? 'fa-chevron-up' : 'fa-chevron-down',
										]"></i>
								</button>
							</div>

							<!-- Tags -->
							<div v-if="item.tags?.length" class="mb-4 sm:mb-5 md:mb-6">
								<h3
									class="mb-2 sm:mb-3 text-xs sm:text-sm font-medium uppercase tracking-wider text-gray-500">
									Tags e Allergeni
								</h3>
								<div class="flex flex-wrap gap-1.5 sm:gap-2">
									<div
										v-for="tag in item.tags"
										:key="tag.id"
										class="rounded-full bg-gray-100 px-3 py-1 sm:px-4 sm:py-1.5 text-xs sm:text-sm transition-colors hover:bg-gray-200">
										<i
											v-if="tag.icon"
											:class="[
												'fas',
												tag.icon,
												'mr-1.5 sm:mr-2 text-gray-500',
											]"></i>
										{{ tag.name }}
									</div>
								</div>
							</div>

							<!-- Ingredients -->
							<div v-if="item.ingredients?.length" class="mb-4 sm:mb-5 md:mb-6">
								<h3
									class="mb-2 sm:mb-3 text-xs sm:text-sm font-medium uppercase tracking-wider text-gray-500">
									Ingredients
								</h3>
								<div class="flex flex-wrap gap-1.5 sm:gap-2">
									<span
										v-for="ingredient in item.ingredients"
										:key="ingredient"
										class="rounded-lg bg-emerald-50 px-2 py-1 sm:px-3 sm:py-1.5 text-xs sm:text-sm font-medium text-emerald-700">
										{{ ingredient }}
									</span>
								</div>
							</div>

							<!-- Nutritional Info -->
							<div v-if="item.nutritional_info" class="mb-4">
								<h3
									class="mb-2 sm:mb-3 text-xs sm:text-sm font-medium uppercase tracking-wider text-gray-500">
									Nutritional Information
								</h3>
								<div class="grid grid-cols-2 gap-2 sm:gap-3 sm:grid-cols-3">
									<div
										v-for="(value, key) in item.nutritional_info"
										:key="key"
										class="group relative overflow-hidden rounded-xl border border-gray-100 bg-white p-2 sm:p-3 text-center shadow-sm transition-all hover:shadow-md hover:border-gray-200">
										<div
											class="absolute -right-12 -top-12 h-24 w-24 rounded-full bg-blue-50 opacity-0 transition-opacity group-hover:opacity-100"></div>
										<span
											class="relative block text-xs font-medium uppercase tracking-wider text-gray-500"
											>{{ key }}</span
										>
										<span
											class="relative block text-sm sm:text-lg font-semibold"
											>{{ value }}</span
										>
									</div>
								</div>
							</div>

							<!-- Action Buttons -->
							<div
								class="mt-4 sm:mt-6 md:mt-8 flex justify-end space-x-2 sm:space-x-4">
								<button
									@click="toggleFullscreen"
									class="flex items-center gap-1 sm:gap-2 rounded-lg border border-gray-200 px-2.5 py-1.5 sm:px-4 sm:py-2 text-xs sm:text-sm font-medium text-gray-700 transition-colors hover:bg-gray-50">
									<i
										:class="[
											'fas',
											isFullscreen ? 'fa-compress' : 'fa-expand',
										]"></i>

									<span class="xs:inline">{{
										isFullscreen ? "Exit fullscreen" : "Fullscreen"
									}}</span>
								</button>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</Transition>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const props = defineProps({
	modelValue: {
		type: Boolean,
		required: true,
	},
	item: {
		type: Object,
		required: true,
	},
});

const emit = defineEmits(["update:modelValue"]);
const showFullDescription = ref(false);
const isFullscreen = ref(false);
const modalContent = ref(null);
const isLandscape = ref(false);

// Custom breakpoint for extra small screens
const xs = ref(false);
const checkXsBreakpoint = () => {
	xs.value = window.innerWidth >= 400;
};

const formatPrice = (price) => Number(price).toFixed(2);

const closeItemModal = () => {
	emit("update:modelValue", false);
};

const toggleFullscreen = () => {
	isFullscreen.value = !isFullscreen.value;
};

const checkOrientation = () => {
	isLandscape.value = window.innerWidth > window.innerHeight;
};

// Lifecycle hooks
onMounted(() => {
	document.body.style.overflow = "hidden";
	checkOrientation();
	checkXsBreakpoint();
	window.addEventListener("resize", checkOrientation);
	window.addEventListener("resize", checkXsBreakpoint);
	window.addEventListener("orientationchange", checkOrientation);
	window.addEventListener("orientationchange", checkXsBreakpoint);
});

onUnmounted(() => {
	document.body.style.overflow = "";
	window.removeEventListener("resize", checkOrientation);
	window.removeEventListener("resize", checkXsBreakpoint);
	window.removeEventListener("orientationchange", checkOrientation);
	window.removeEventListener("orientationchange", checkXsBreakpoint);
});
</script>
