<template>
	<div
		class="hidden md:flex flex-col mb-12 space-y-4"
		:class="{
			'translate-y-0 opacity-100': isVisible,
			'translate-y-8 opacity-0': !isVisible,
		}">
		<!-- Search input -->
		<div class="relative flex-grow w-full group">
			<input
				type="text"
				v-model="searchQuery"
				placeholder="Cerca piatti o tag..."
				class="w-full px-6 py-3.5 bg-white/90 backdrop-blur border-0 rounded-full shadow-lg focus:ring-2 focus:ring-gray-300 focus:outline-none transition-all group-hover:shadow-xl placeholder-gray-400/80" />
			<div class="absolute right-6 top-2/6 flex items-center gap-2">
				<i class="fas fa-search text-gray-400/60"></i>
			</div>
		</div>

		<!-- Categories Scroll Container -->
		<div class="relative group">
			<button
				@click="scrollCategories('left')"
				class="absolute left-0 top-1/3 -translate-y-1/2 z-10 w-8 h-8 flex items-center justify-center bg-white/90 rounded-full shadow-lg hover:shadow-xl transition-all transform hover:scale-110"
				:class="{ 'opacity-0': isScrolledLeft }">
				<i class="fas fa-chevron-left text-gray-600"></i>
			</button>

			<button
				@click="scrollCategories('right')"
				class="absolute right-0 top-1/3 -translate-y-1/2 z-10 w-8 h-8 flex items-center justify-center bg-white/90 rounded-full shadow-lg hover:shadow-xl transition-all transform hover:scale-110"
				:class="{ 'opacity-0': isScrolledRight }">
				<i class="fas fa-chevron-right text-gray-600"></i>
			</button>

			<div
				ref="categoriesContainer"
				@scroll="checkScroll"
				class="flex items-center overflow-x-auto scrollbar-hide px-2 pb-8 pt-1 scroll-smooth">
				<div class="flex gap-3 flex-nowrap">
					<button
						@click="selectCategory('all')"
						class="flex-shrink-0 px-5 py-2.5 rounded-full transition-all duration-200 hover:shadow-xl cursor-pointer flex items-center gap-2"
						:class="{
							'bg-black text-white shadow-md': selectedCategory === 'all',
							'bg-white/90 shadow-sm hover:bg-gray-50':
								selectedCategory !== 'all',
						}">
						<i class="fas fa-layer-group text-sm"></i>
						<span class="whitespace-nowrap">Tutto</span>
					</button>

					<button
						v-for="(cat, key) in categories"
						@click="selectCategory(key)"
						:key="key"
						class="flex-shrink-0 px-4 py-2.5 rounded-full transition-all duration-200 flex items-center gap-2 hover:shadow-xl cursor-pointer"
						:class="{
							'bg-black text-white shadow-md': selectedCategory === key,
							'bg-white/90 shadow-sm hover:bg-gray-50':
								selectedCategory !== key,
						}">
						<i :class="cat.icon" class="text-sm w-5 text-center"></i>
						<span class="whitespace-nowrap">{{ cat.name }}</span>
					</button>
				</div>
			</div>

			<button
				v-if="hasActiveFilters"
				@click="resetFilters"
				class="mt-4 px-4 py-2 flex items-center gap-2 text-gray-600 hover:text-gray-900 transition-colors mx-auto">
				<i class="fas fa-undo-alt"></i>
				<span>Reset filtri</span>
			</button>
		</div>
	</div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";

const props = defineProps({
	categories: {
		type: Object,
		required: true,
	},
	modelValue: String,
	selectedCategory: String,
	isVisible: Boolean,
});

const emit = defineEmits(["update:modelValue", "update:selectedCategory"]);

const categoriesContainer = ref(null);
const isScrolledLeft = ref(true);
const isScrolledRight = ref(false);

const searchQuery = computed({
	get: () => props.modelValue,
	set: (value) => emit("update:modelValue", value),
});

const hasActiveFilters = computed(() => {
	return searchQuery.value !== "" || props.selectedCategory !== "all";
});

const selectCategory = (category) => {
	emit("update:selectedCategory", category);
};

const resetFilters = () => {
	emit("update:modelValue", "");
	emit("update:selectedCategory", "all");
};

const scrollCategories = (direction) => {
	if (!categoriesContainer.value) return;
	const container = categoriesContainer.value;
	const scrollAmount = container.clientWidth * 0.75;
	container.scrollBy({
		left: direction === "left" ? -scrollAmount : scrollAmount,
		behavior: "smooth",
	});
};

const checkScroll = () => {
	if (!categoriesContainer.value) return;
	const container = categoriesContainer.value;
	isScrolledLeft.value = container.scrollLeft <= 0;
	isScrolledRight.value =
		container.scrollLeft + container.clientWidth >= container.scrollWidth;
};

onMounted(() => {
	checkScroll();
});
</script>

<style scoped>
.scrollbar-hide::-webkit-scrollbar {
	display: none;
}

.scrollbar-hide {
	-ms-overflow-style: none;
	scrollbar-width: none;
}
</style>
