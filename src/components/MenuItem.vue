<template>
	<div class="menu-item-wrapper">
		<div
			class="menu-item"
			:class="[cardColorClass, { unavailable: !item.is_available }]">
			<!-- Decorative Elements -->
			<div class="decorative-circle circle-1"></div>
			<div class="decorative-circle circle-2"></div>
			<div class="decorative-dots"></div>

			<!-- Unavailable Overlay -->
			<div v-if="!item.is_available" class="unavailable-overlay">
				<div class="unavailable-badge">
					<span class="unavailable-text">Non disponibile</span>
				</div>
			</div>

			<!-- Image Section with Creative Masking -->
			<div class="image-section">
				<div class="image-container">
					<img
						:src="item.image_url || '/default-menu-image-placeholder.png'"
						:alt="item.name"
						class="item-image" />
					<div class="image-overlay"></div>
				</div>

				<!-- Floating Price Badge -->
				<div class="price-badge">
					<div class="price-inner">
						<span class="price-currency">€</span>
						<span class="price-value">{{ formatPrice(item.price) }}</span>
					</div>
				</div>
			</div>

			<!-- Content Section with Creative Layout -->
			<div class="content-section">
				<div class="header-section">
					<h3 class="item-title">{{ item.name }}</h3>
					<div v-if="item.special_label" class="special-label">
						<span>{{ item.special_label }}</span>
					</div>
				</div>

				<!-- Description with stylized quotation marks -->
				<div class="description-container">
					<span class="quote-mark open-quote">"</span>
					<p class="item-description" :title="item.description">
						{{ truncateDescription(item.description) }}
					</p>
					<span class="quote-mark close-quote">"</span>
				</div>

				<!-- Beverage Category with Icon -->
				<div
					v-if="beverageTag"
					class="beverage-category"
					:class="beverageCategoryClass">
					<div class="beverage-icon-container">
						<i :class="['fas', getBeverageIcon(beverageTag.name)]"></i>
					</div>
					<span class="beverage-name">{{
						getBeverageDisplayName(beverageTag.name)
					}}</span>
				</div>

				<!-- Creative Tags Display -->
				<div class="tags-section">
					<div class="tags-container">
						<div
							v-for="(tag, index) in displayTags"
							:key="index"
							class="tag"
							:class="[
								`tag-${index}`,
								{ 'tag-hidden': isMobileWidth && index > 0 },
							]"
							:style="{ '--tag-index': index }">
							<i v-if="tag.icon" :class="['fas', tag.icon]"></i>
							<span>{{ tag.name }}</span>
						</div>

						<div v-if="shouldShowPlusSign" class="tag tag-more">
							<span>+{{ remainingTagsCount }}</span>
						</div>
					</div>
				</div>

				<!-- Metadata with Creative Visualization -->
				<div class="metadata-section">
					<div v-if="item.preparation_time" class="metadata-item time-item">
						<div class="metadata-icon-container">
							<i class="fas fa-clock"></i>
						</div>
						<div class="metadata-bar-container">
							<div
								class="metadata-bar"
								:style="{
									width: `${Math.min(item.preparation_time * 3, 100)}%`,
								}"></div>
						</div>
						<span class="metadata-value">{{ item.preparation_time }} min</span>
					</div>

					<div v-if="item.calories" class="metadata-item calorie-item">
						<div class="metadata-icon-container">
							<i class="fas fa-fire"></i>
						</div>
						<div class="metadata-bar-container">
							<div
								class="metadata-bar"
								:style="{
									width: `${Math.min(item.calories / 10, 100)}%`,
								}"></div>
						</div>
						<span class="metadata-value">{{ item.calories }} kcal</span>
					</div>

					<div v-if="item.spiciness" class="metadata-item spicy-item">
						<div class="spicy-meter">
							<div
								v-for="n in 5"
								:key="n"
								class="spicy-dot"
								:class="{ active: n <= item.spiciness }"></div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>
import { computed, ref, onMounted, onUnmounted } from "vue";

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

const getBeverageIcon = (tagName) => {
	const iconMap = {
		"bevande generiche": "fa-wine-bottle",
		"caffe e digestivi": "fa-mug-hot",
		birre: "fa-beer-mug-empty",
		vini: "fa-wine-glass",
	};
	return iconMap[tagName.toLowerCase()] || "fa-utensils";
};

const beverageTag = computed(() => {
	return props.item.tags.find((tag) => {
		const tagName = tag.name.toLowerCase();
		return ["bevande generiche", "caffe e digestivi", "birre", "vini"].includes(
			tagName
		);
	});
});

const beverageCategoryClass = computed(() => {
	if (!beverageTag.value) return "";

	const classMap = {
		"bevande generiche": "beverage-soft",
		"caffe e digestivi": "beverage-coffee",
		birre: "beverage-beer",
		vini: "beverage-wine",
	};

	return classMap[beverageTag.value.name.toLowerCase()] || "";
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

const formatPrice = (price) => {
	const [whole, decimal] = Number(price).toFixed(2).split(".");
	return `${whole}.${decimal}`;
};

const truncateDescription = (text) => {
	if (!text) return "";
	return text.length > 120 ? text.substring(0, 120) + "..." : text;
};

const cardColorClass = computed(() => {
	if (!beverageTag.value) return "card-default";

	const colorMap = {
		"bevande generiche": "card-soft-drink",
		"caffe e digestivi": "card-coffee",
		birre: "card-beer",
		vini: "card-wine",
	};

	return colorMap[beverageTag.value.name.toLowerCase()] || "card-default";
});

const isMobileWidth = ref(false);
const checkMobileWidth = () => {
	isMobileWidth.value = window.innerWidth < 640;
};

onMounted(() => {
	checkMobileWidth();
	window.addEventListener("resize", checkMobileWidth);
});

onUnmounted(() => {
	window.removeEventListener("resize", checkMobileWidth);
});

const remainingTagsCount = computed(() => {
	if (isMobileWidth.value) {
		return props.item.tags.length - 1;
	}
	return props.item.tags.length - props.maxDisplayTags;
});

const shouldShowPlusSign = computed(() => {
	if (isMobileWidth.value) {
		return props.item.tags.length > 1;
	}
	return props.item.tags.length > props.maxDisplayTags;
});
</script>

<style scoped>
/* Base Styles with Creative Approach */
.menu-item-wrapper {
	position: relative;
	height: 100%;
	perspective: 1000px;
	z-index: 1;
}

.menu-item {
	position: relative;
	height: 100%;
	display: flex;
	flex-direction: column;
	border-radius: 24px;
	overflow: hidden;
	background-color: #ffffff;
	box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
	transition: all 0.5s cubic-bezier(0.25, 1, 0.5, 1);
	transform-style: preserve-3d;
	z-index: 1;
}

.menu-item:hover {
	transform: translateY(-8px) rotateX(2deg);
	box-shadow: 0 20px 40px rgba(0, 0, 0, 0.12);
}

/* Decorative Elements */
.decorative-circle {
	position: absolute;
	border-radius: 50%;
	opacity: 0.4;
	z-index: -1;
	transition: all 0.6s ease;
}

.circle-1 {
	width: 120px;
	height: 120px;
	top: -30px;
	right: -30px;
	background: radial-gradient(
		circle,
		rgba(255, 255, 255, 0.8) 0%,
		rgba(255, 255, 255, 0) 70%
	);
}

.circle-2 {
	width: 80px;
	height: 80px;
	bottom: 20px;
	left: -20px;
	background: radial-gradient(
		circle,
		rgba(255, 255, 255, 0.6) 0%,
		rgba(255, 255, 255, 0) 70%
	);
}

.menu-item:hover .circle-1 {
	transform: scale(1.2) translateX(10px);
}

.menu-item:hover .circle-2 {
	transform: scale(1.3) translateY(-10px);
}

.decorative-dots {
	position: absolute;
	top: 50%;
	right: 20px;
	width: 40px;
	height: 100px;
	background-image: radial-gradient(
		circle,
		rgba(0, 0, 0, 0.1) 1px,
		transparent 1px
	);
	background-size: 8px 8px;
	z-index: -1;
	opacity: 0.5;
}

/* Unavailable State */
.unavailable {
	opacity: 0.7;
	filter: grayscale(0.5);
}

.unavailable-overlay {
	position: absolute;
	inset: 0;
	z-index: 20;
	display: flex;
	align-items: center;
	justify-content: center;
	backdrop-filter: blur(4px);
	background-color: rgba(255, 255, 255, 0.2);
}

.unavailable-badge {
	background: rgba(0, 0, 0, 0.8);
	color: white;
	padding: 10px 24px;
	border-radius: 30px;
	transform: rotate(-5deg);
	box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
	animation: pulse 2s infinite;
}

.unavailable-text {
	font-weight: 600;
	letter-spacing: 0.5px;
	font-size: 16px;
}

@keyframes pulse {
	0% {
		transform: rotate(-5deg) scale(1);
	}
	50% {
		transform: rotate(-5deg) scale(1.05);
	}
	100% {
		transform: rotate(-5deg) scale(1);
	}
}

/* Creative Image Section */
.image-section {
	position: relative;
	height: 200px;
	overflow: hidden;
}

.image-container {
	position: relative;
	height: 100%;
	overflow: hidden;
	clip-path: polygon(0 0, 100% 0, 100% 85%, 0 100%);
}

.item-image {
	width: 100%;
	height: 100%;
	object-fit: cover;
	transition: transform 0.7s cubic-bezier(0.25, 1, 0.5, 1);
}

.menu-item:hover .item-image {
	transform: scale(1.08) translateY(-5px);
}

.image-overlay {
	position: absolute;
	inset: 0;
	background: linear-gradient(
		to bottom,
		rgba(0, 0, 0, 0) 50%,
		rgba(0, 0, 0, 0.4) 100%
	);
	z-index: 1;
}

/* Creative Price Badge */
.price-badge {
	position: absolute;
	top: 20px;
	right: 20px;
	z-index: 10;
}

.price-inner {
	display: flex;
	align-items: baseline;
	background: rgba(255, 255, 255, 0.95);
	padding: 8px 16px;
	border-radius: 20px;
	box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
	transform: rotate(3deg);
	transition: all 0.3s ease;
}

.menu-item:hover .price-inner {
	transform: rotate(0deg) scale(1.05);
}

.price-currency {
	font-size: 16px;
	font-weight: 600;
	color: #333;
	margin-right: 2px;
}

.price-value {
	font-size: 22px;
	font-weight: 700;
	color: #222;
}

.price-decimal {
	font-size: 14px;
	opacity: 0.7;
}

/* Content Section */
.content-section {
	flex-grow: 1;
	display: flex;
	flex-direction: column;
	padding: 24px;
	position: relative;
	z-index: 2;
}

/* Header Section */
.header-section {
	display: flex;
	justify-content: space-between;
	align-items: flex-start;
	margin-bottom: 16px;
}

.item-title {
	font-size: 22px;
	font-weight: 700;
	color: #222;
	margin: 0;
	line-height: 1.3;
	position: relative;
	display: inline-block;
}

.item-title::after {
	content: "";
	position: absolute;
	bottom: -6px;
	left: 0;
	width: 40px;
	height: 3px;
	background-color: currentColor;
	opacity: 0.3;
	transition: width 0.3s ease;
}

.menu-item:hover .item-title::after {
	width: 100%;
}

.special-label {
	background: linear-gradient(135deg, #ff6b6b 0%, #ff8e8e 100%);
	color: white;
	padding: 6px 14px;
	border-radius: 20px;
	font-size: 13px;
	font-weight: 600;
	transform: rotate(3deg);
	box-shadow: 0 3px 10px rgba(255, 107, 107, 0.3);
	transition: all 0.3s ease;
}

.menu-item:hover .special-label {
	transform: rotate(0deg) translateY(-2px);
}

/* Creative Description */
.description-container {
	position: relative;
	margin-bottom: 20px;
	padding: 0 20px;
}

.item-description {
	font-size: 15px;
	line-height: 1.6;
	color: #555;
	font-style: italic;
	display: -webkit-box;
	-webkit-line-clamp: 2;
	-webkit-box-orient: vertical;
	overflow: hidden;
	margin: 0;
}

.quote-mark {
	position: absolute;
	font-size: 40px;
	line-height: 1;
	font-family: Georgia, serif;
	opacity: 0.15;
	color: #000;
}

.open-quote {
	top: -10px;
	left: 0;
}

.close-quote {
	bottom: -20px;
	right: 0;
}

/* Beverage Category */
.beverage-category {
	display: flex;
	align-items: center;
	margin-bottom: 20px;
	padding: 10px 16px;
	border-radius: 12px;
	background-color: rgba(0, 0, 0, 0.03);
	transition: all 0.3s ease;
}

.menu-item:hover .beverage-category {
	transform: translateX(5px);
}

.beverage-icon-container {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 36px;
	height: 36px;
	border-radius: 50%;
	margin-right: 12px;
	background-color: rgba(0, 0, 0, 0.1);
	color: #333;
	font-size: 16px;
}

.beverage-name {
	font-size: 16px;
	font-weight: 600;
	letter-spacing: 0.5px;
	text-transform: uppercase;
}

/* Beverage Category Variations */
.beverage-soft .beverage-icon-container {
	background-color: rgba(59, 130, 246, 0.1);
	color: #3b82f6;
}

.beverage-coffee .beverage-icon-container {
	background-color: rgba(217, 119, 6, 0.1);
	color: #d97706;
}

.beverage-beer .beverage-icon-container {
	background-color: rgba(245, 158, 11, 0.1);
	color: #f59e0b;
}

.beverage-wine .beverage-icon-container {
	background-color: rgba(220, 38, 38, 0.1);
	color: #dc2626;
}

/* Creative Tags Display */
.tags-section {
	margin-bottom: 20px;
}

.tags-container {
	display: flex;
	flex-wrap: wrap;
	gap: 10px;
}

.tag {
	display: inline-flex;
	align-items: center;
	padding: 6px 14px;
	border-radius: 30px;
	font-size: 13px;
	font-weight: 500;
	background-color: rgba(0, 0, 0, 0.05);
	color: #555;
	transition: all 0.3s cubic-bezier(0.25, 1, 0.5, 1);
	transform: translateY(calc(var(--tag-index) * 2px));
	animation: tagAppear 0.5s calc(var(--tag-index) * 0.1s) both;
}

@keyframes tagAppear {
	from {
		opacity: 0;
		transform: translateY(10px);
	}
	to {
		opacity: 1;
		transform: translateY(calc(var(--tag-index) * 2px));
	}
}

.tag i {
	margin-right: 6px;
	font-size: 12px;
}

.tag:hover {
	background-color: rgba(0, 0, 0, 0.1);
	transform: translateY(calc(var(--tag-index) * 2px - 2px));
}

.tag-hidden {
	display: none;
}

.tag-more {
	background-color: rgba(0, 0, 0, 0.03);
	border: 1px dashed rgba(0, 0, 0, 0.2);
}

/* Creative Metadata Visualization */
.metadata-section {
	margin-top: auto;
	padding-top: 20px;
	border-top: 1px dashed rgba(0, 0, 0, 0.1);
}

.metadata-item {
	display: flex;
	align-items: center;
	margin-bottom: 12px;
}

.metadata-icon-container {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 30px;
	height: 30px;
	border-radius: 50%;
	background-color: rgba(0, 0, 0, 0.05);
	color: #666;
	margin-right: 12px;
}

.metadata-bar-container {
	flex-grow: 1;
	height: 6px;
	background-color: rgba(0, 0, 0, 0.05);
	border-radius: 3px;
	overflow: hidden;
	margin-right: 12px;
}

.metadata-bar {
	height: 100%;
	border-radius: 3px;
	transition: width 1s cubic-bezier(0.25, 1, 0.5, 1);
}

.time-item .metadata-bar {
	background-color: #3b82f6;
}

.calorie-item .metadata-bar {
	background-color: #f59e0b;
}

.metadata-value {
	font-size: 14px;
	font-weight: 500;
	color: #666;
	min-width: 60px;
	text-align: right;
}

/* Creative Spiciness Meter */
.spicy-item {
	justify-content: flex-start;
}

.spicy-meter {
	display: flex;
	align-items: center;
	gap: 6px;
}

.spicy-dot {
	width: 12px;
	height: 12px;
	border-radius: 50%;
	background-color: rgba(0, 0, 0, 0.1);
	transition: all 0.3s ease;
}

.spicy-dot.active {
	background-color: #ef4444;
	box-shadow: 0 0 8px rgba(239, 68, 68, 0.5);
}

/* Card Color Variations */
.card-default {
	background-color: #ffffff;
}

.card-soft-drink {
	background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%);
}

.card-soft-drink .item-title::after {
	background-color: #3b82f6;
}

.card-coffee {
	background: linear-gradient(135deg, #fff7ed 0%, #ffedd5 100%);
}

.card-coffee .item-title::after {
	background-color: #d97706;
}

.card-beer {
	background: linear-gradient(135deg, #fffbeb 0%, #fef3c7 100%);
}

.card-beer .item-title::after {
	background-color: #f59e0b;
}

.card-wine {
	background: linear-gradient(135deg, #fff1f2 0%, #ffe4e6 100%);
}

.card-wine .item-title::after {
	background-color: #dc2626;
}

/* Responsive Adjustments */
@media (max-width: 640px) {
	.image-section {
		height: 180px;
	}

	.price-inner {
		padding: 6px 12px;
	}

	.price-currency {
		font-size: 14px;
	}

	.price-value {
		font-size: 18px;
	}

	.content-section {
		padding: 20px;
	}

	.item-title {
		font-size: 18px;
	}

	.special-label {
		padding: 4px 10px;
		font-size: 12px;
	}

	.item-description {
		font-size: 14px;
	}

	.beverage-name {
		font-size: 14px;
	}
}
</style>
