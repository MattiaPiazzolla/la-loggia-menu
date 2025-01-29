<template>
	<div class="group relative h-full">
		<!-- Container with Apple-style frosted glass effect -->
		<div
			class="relative overflow-hidden rounded-3xl backdrop-blur-xl transition-all duration-500 ease-out hover:scale-[1.02] hover:shadow-2xl h-full flex flex-col"
			:class="[cardColorClass, { 'opacity-60': !item.is_available }]">
			<!-- Unavailable State Overlay -->
			<div
				v-if="!item.is_available"
				class="absolute inset-0 z-20 flex items-center justify-center backdrop-blur-sm">
				<div
					class="rounded-full bg-black/80 px-6 py-2.5 text-sm font-medium text-white backdrop-blur-sm">
					Non disponibile
				</div>
			</div>

			<!-- Image Section -->
			<div class="relative aspect-[4/3] overflow-hidden flex-shrink-0">
				<img
					:src="
						item.image_url
							? item.image_url
							: '../../public/default-menu-image-placeholder.png'
					"
					:alt="item.name"
					class="h-full w-full object-cover transition-transform duration-700 group-hover:scale-105" />

				<!-- Elegant Price Badge -->
				<div class="absolute right-6 top-6">
					<div
						class="rounded-2xl bg-white/90 px-5 py-2.5 backdrop-blur-md transition-all duration-500 group-hover:translate-y-1">
						<span class="text-lg font-medium text-gray-900">
							€{{ formatPrice(item.price) }}
						</span>
					</div>
				</div>

				<!-- Modern Tags Display -->
				<div class="absolute bottom-6 left-6 flex flex-wrap gap-2">
					<span
						v-for="tag in displayTags"
						:key="tag.id"
						class="rounded-full bg-white/80 px-4 py-2 text-sm font-medium text-gray-800 backdrop-blur-md transition-all duration-300 hover:bg-white">
						<i v-if="tag.icon" :class="['fas', tag.icon, 'mr-2']"></i>
						#{{ tag.name }}
					</span>
					<span
						v-if="hasMoreTags"
						class="rounded-full bg-white/80 px-4 py-2 text-sm font-medium text-gray-800 backdrop-blur-md">
						+{{ item.tags.length - maxDisplayTags }}
					</span>
				</div>
			</div>

			<!-- Content Section with refined spacing -->
			<div class="space-y-4 p-8 flex-grow flex flex-col justify-between">
				<div class="space-y-4">
					<div class="flex items-start justify-between">
						<h3 class="text-2xl font-semibold text-gray-900">
							{{ item.name }}
						</h3>
						<span
							v-if="item.special_label"
							class="rounded-full bg-red-50 px-4 py-1.5 text-sm font-medium text-red-600">
							{{ item.special_label }}
						</span>
					</div>

					<p
						class="line-clamp-3 text-base leading-relaxed text-gray-600"
						:title="item.description">
						{{ truncateDescription(item.description) }}
					</p>

					<!-- New Beverage Label -->
					<div
						v-if="beverageTag"
						class="text-lg font-bold text-gray-600 uppercase">
						<i
							v-if="beverageTag.icon"
							:class="['fas', beverageTag.icon, 'mr-2']"></i>
						{{ getBeverageDisplayName(beverageTag.name) }}
					</div>
				</div>

				<!-- Refined metadata display -->
				<div class="flex items-center gap-6 text-sm text-gray-500 mt-auto pt-4">
					<span v-if="item.preparation_time" class="flex items-center">
						<i class="fas fa-clock mr-2"></i>
						{{ item.preparation_time }} min
					</span>
					<span v-if="item.calories" class="flex items-center">
						<i class="fas fa-fire mr-2"></i>
						{{ item.calories }} kcal
					</span>
					<span v-if="item.spiciness" class="flex items-center gap-0.5">
						<i
							v-for="n in item.spiciness"
							:key="n"
							class="fas fa-pepper-hot text-red-500"></i>
					</span>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
	item: {
		type: Object,
		required: true,
		default: () => ({
			name: "",
			description: "",
			price: 0,
			tags: [],
			is_available: true,
			image_url: "",
			preparation_time: null,
			calories: null,
			spiciness: 0,
			special_label: "",
		}),
	},
	maxDisplayTags: {
		type: Number,
		default: 3,
	},
});

const beverageDisplayNames = {
	"bevande generiche": "Bibite",
	"caffe e digestivi": "Caffè e Digestivi",
	birre: "Birre",
	vini: "Vini",
};

const getBeverageDisplayName = (tagName) => {
	return beverageDisplayNames[tagName.toLowerCase()] || tagName;
};

const beverageTag = computed(() => {
	return props.item.tags.find((tag) => {
		const tagName = tag.name.toLowerCase();
		return ["bevande generiche", "caffe e digestivi", "birre", "vini"].includes(
			tagName
		);
	});
});

const displayTags = computed(() => {
	const filteredTags = props.item.tags.filter((tag) => {
		const tagName = tag.name.toLowerCase();
		return ![
			"bevande generiche",
			"caffe e digestivi",
			"birre",
			"vini",
		].includes(tagName);
	});
	return filteredTags.slice(0, props.maxDisplayTags);
});

const hasMoreTags = computed(
	() => props.item.tags.length > props.maxDisplayTags
);

const formatPrice = (price) => Number(price).toFixed(2);

const truncateDescription = (text) => {
	if (!text) return "";
	return text.length > 150 ? text.substring(0, 150) + "..." : text;
};

const cardColorClass = computed(() => {
	const beverageTag = props.item.tags.find((tag) => {
		const tagName = tag.name.toLowerCase();
		return ["bevande generiche", "caffe e digestivi", "birre", "vini"].includes(
			tagName
		);
	});

	if (!beverageTag) return "bg-white/80";

	const colorMap = {
		"bevande generiche": "bg-blue-100/80",
		"caffe e digestivi": "bg-orange-100/80",
		birre: "bg-yellow-500/80",
		vini: "bg-rose-50/80",
	};

	return colorMap[beverageTag.name.toLowerCase()] || "bg-white/80";
});
</script>

<style scoped>
.line-clamp-3 {
	display: -webkit-box;
	-webkit-line-clamp: 2;
	-webkit-box-orient: vertical;
	overflow: hidden;
}
</style>
