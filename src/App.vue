<template>
	<div class="min-h-screen bg-gradient-to-b from-gray-100 to-gray-200">
		<StatusBanner />
		<!-- Header and search sections remain the same -->
		<div class="text-center text-black pt-20">
			<!-- <img
				src="./assets/La-loggia-completo.svg"
				class="mx-auto w-[200px] select-none pointer-events-none transition-opacity hover:opacity-90"
				alt="La Loggia Logo" /> -->
			<h3 class="text-lg font-medium uppercase">Menu</h3>
		</div>

		<div class="max-w-6xl mx-auto px-4 py-8">
			<!-- Search and Filter Bar - Hide on Mobile -->
			<div
				class="hidden md:flex flex-col mb-12 space-y-4"
				:class="{
					'translate-y-0 opacity-100': isVisible,
					'translate-y-8 opacity-0': !isVisible,
				}">
				<!-- Search input and categories remain the same -->
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
					<!-- Left Arrow -->
					<button
						@click="scrollCategories('left')"
						class="absolute left-0 top-1/3 -translate-y-1/2 z-10 w-8 h-8 flex items-center justify-center bg-white/90 rounded-full shadow-lg hover:shadow-xl transition-all transform hover:scale-110"
						:class="{ 'opacity-0': isScrolledLeft }">
						<i class="fas fa-chevron-left text-gray-600"></i>
					</button>

					<!-- Right Arrow -->
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
								@click="selectedCategory = 'all'"
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
								@click="selectedCategory = key"
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

					<!-- Reset Button -->
					<button
						v-if="hasActiveFilters"
						@click="resetFilters"
						class="mt-4 px-4 py-2 flex items-center gap-2 text-gray-600 hover:text-gray-900 transition-colors mx-auto">
						<i class="fas fa-undo-alt"></i>
						<span>Reset filtri</span>
					</button>
				</div>
			</div>

			<!-- Loading and Error states remain the same -->
			<div v-if="loading" class="flex items-center justify-center py-20">
				<i class="fas fa-spinner animate-spin text-3xl text-gray-600"></i>
			</div>

			<div v-else-if="error" class="text-center py-20">
				<i class="fas fa-circle-exclamation text-4xl text-rose-600 mb-4"></i>
				<p class="text-xl text-gray-800">Impossibile caricare il menu</p>
				<p class="text-gray-600 mt-2">Per favore, riprova più tardi</p>
			</div>

			<!-- Menu Content -->
			<div v-else class="space-y-16 mb-[100px]">
				<section
					v-for="(category, key, index) in groupedMenus"
					:key="key"
					class="space-y-8"
					:class="{
						'translate-y-0 opacity-100': isVisible,
						'translate-y-8 opacity-0': !isVisible,
					}"
					:style="{ transitionDelay: `${index * 100}ms` }">
					<h2
						class="text-3xl font-light tracking-wide text-gray-900 flex items-center gap-3">
						<i :class="category.icon" class="w-8 text-center text-gray-600"></i>
						{{ category.name }}
					</h2>

					<div
						class="grid gap-6 grid-cols-1 min-[450px]:grid-cols-2 min-[680px]:grid-cols-3 min-[1200px]:grid-cols-4 max-[450px]:grid-cols-1">
						<menu-item
							v-for="item in category.items"
							:key="item.id"
							:item="item"
							class="transform transition-all duration-500 hover:scale-[1.02] active:scale-95"
							@click="openItemModal(item)" />
					</div>
				</section>

				<!-- No Results State -->
				<div
					v-if="noResults"
					class="flex flex-col items-center justify-center py-20">
					<i class="fas fa-magnifying-glass text-4xl text-gray-400 mb-6"></i>
					<p class="text-xl text-gray-800">Nessun piatto trovato</p>
					<p class="text-gray-600 mt-2">Prova con termini di ricerca diversi</p>
				</div>
			</div>
		</div>

		<!-- Item Detail Modal -->
		<item-detail-modal
			v-if="selectedItem"
			:modelValue="!!selectedItem"
			:item="selectedItem"
			@update:modelValue="closeItemModal" />

		<!-- Back to Top Button -->
		<Transition
			enter-active-class="transition-all duration-300 ease-out"
			enter-from-class="opacity-0 translate-y-4"
			enter-to-class="opacity-100 translate-y-0"
			leave-active-class="transition-all duration-200 ease-in"
			leave-from-class="opacity-100 translate-y-0"
			leave-to-class="opacity-0 translate-y-4">
			<button
				v-show="showBackToTop"
				@click="scrollToTop"
				class="fixed bottom-8 right-8 z-40 p-3 w-12.5 h-12.5 rounded-full bg-black/80 backdrop-blur text-white shadow-xl hover:bg-black transition-all duration-300 hover:scale-110 transition-transform active:scale-95">
				<i class="fas fa-arrow-up"></i>
			</button>
		</Transition>

		<!-- Mobile Menu Button -->
		<Transition
			enter-active-class="transition-all duration-300 ease-out"
			enter-from-class="opacity-0 translate-y-4"
			enter-to-class="opacity-100 translate-y-0"
			leave-active-class="transition-all duration-200 ease-in"
			leave-from-class="opacity-100 translate-y-0"
			leave-to-class="opacity-0 translate-y-4">
			<button
				v-show="!isMobileMenuOpen"
				@click="toggleMobileMenu"
				class="md:hidden fixed bottom-8 left-8 z-40 p-3 w-12.5 h-12.5 rounded-full bg-black/80 backdrop-blur text-white shadow-xl hover:bg-black transition-all duration-300 hover:scale-110 transition-transform active:scale-95">
				<i class="fas fa-magnifying-glass"></i>
			</button>
		</Transition>

		<!-- Mobile Menu Overlay -->
		<Transition
			enter-active-class="transition-all duration-300 ease-out"
			enter-from-class="opacity-0 translate-x-full"
			enter-to-class="opacity-100 translate-x-0"
			leave-active-class="transition-all duration-200 ease-in"
			leave-from-class="opacity-100 translate-x-0"
			leave-to-class="opacity-0 translate-x-full">
			<div
				v-if="isMobileMenuOpen"
				class="md:hidden fixed inset-0 z-50 bg-white/95 backdrop-blur-sm overflow-y-auto">
				<!-- Mobile Menu Content -->
				<div class="h-full flex flex-col p-6">
					<!-- Close Button -->
					<button
						@click="toggleMobileMenu"
						class="self-end p-2 text-gray-600 hover:text-gray-900">
						<i class="fas fa-times text-2xl"></i>
					</button>

					<!-- Mobile Search Bar -->
					<div class="relative mt-4">
						<input
							type="text"
							v-model="searchQuery"
							placeholder="Cerca piatti o tag..."
							class="w-full px-6 py-3.5 bg-white/90 backdrop-blur border-0 rounded-full shadow-lg focus:ring-2 focus:ring-gray-300 focus:outline-none transition-all placeholder-gray-400/80"
							@input="() => (isMobileSearchActive = searchQuery.length > 0)"
							@keyup.enter="handleSearchSubmit" />
						<div class="absolute right-6 top-3.5 flex items-center gap-2">
							<i class="fas fa-search text-gray-400/60"></i>
						</div>
					</div>

					<!-- Search Results -->
					<div
						v-if="isMobileSearchActive && filteredMenus.length > 0"
						class="mt-6">
						<h3 class="text-lg font-medium text-gray-900 mb-4">
							Risultati ricerca
						</h3>
						<div class="space-y-3">
							<button
								v-for="item in filteredMenus.slice(0, 5)"
								:key="item.id"
								@click="selectItemAndClose(item)"
								class="w-full flex items-center gap-3 px-4 py-3 rounded-xl hover:bg-gray-100 transition-all text-left">
								<img
									v-if="item.image_url"
									:src="item.image_url"
									:alt="item.name"
									class="w-12 h-12 rounded-lg object-cover" />
								<div>
									<div class="font-medium">{{ item.name }}</div>
									<div class="text-sm text-gray-600">
										€{{ formatPrice(item.price) }}
									</div>
								</div>
							</button>
						</div>
					</div>

					<!-- Categories List -->
					<div
						class="flex flex-col gap-4 mt-8"
						:class="{ 'opacity-50': isMobileSearchActive }">
						<button
							@click="selectCategoryAndClose('all')"
							class="flex items-center gap-3 px-4 py-3 rounded-xl transition-all"
							:class="{
								'bg-black text-white': selectedCategory === 'all',
								'hover:bg-gray-100': selectedCategory !== 'all',
							}">
							<i class="fas fa-layer-group w-6"></i>
							<span>Tutto</span>
						</button>

						<button
							v-for="(cat, key) in categories"
							:key="key"
							@click="selectCategoryAndClose(key)"
							class="flex items-center gap-3 px-4 py-3 rounded-xl transition-all"
							:class="{
								'bg-black text-white': selectedCategory === key,
								'hover:bg-gray-100': selectedCategory !== key,
							}">
							<i :class="[cat.icon, 'w-6']"></i>
							<span>{{ cat.name }}</span>
						</button>
					</div>

					<!-- Mobile Reset Button -->
					<button
						v-if="hasActiveFilters"
						@click="resetFilters"
						class="mt-4 px-4 py-3 flex items-center justify-center gap-2 text-gray-600 hover:text-gray-900 transition-colors rounded-xl border border-gray-200">
						<i class="fas fa-undo-alt"></i>
						<span>Reset filtri</span>
					</button>
					<div class="py-3 text-white select-none">
						Created by Mattia Piazzolla
					</div>
				</div>
			</div>
		</Transition>
	</div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from "vue";
import axios from "axios";
import MenuItem from "./components/MenuItem.vue";
import StatusBanner from "./components/StatusBanner.vue";
import ItemDetailModal from "./components/ItemDetailModal.vue";

export default {
	components: {
		MenuItem,
		StatusBanner,
		ItemDetailModal,
	},

	setup() {
		const loading = ref(true);
		const error = ref(false);
		const searchQuery = ref("");
		const selectedCategory = ref("all");
		const menus = ref([]);
		const isVisible = ref(false);
		const selectedItem = ref(null);
		const categoriesContainer = ref(null);
		const isScrolledLeft = ref(true);
		const isScrolledRight = ref(false);
		const showBackToTop = ref(false);
		const isMobileMenuOpen = ref(false);
		const isMobileSearchActive = ref(false);
		const showFullDescription = ref(false);

		const categories = {
			antipasto: { icon: "fas fa-leaf", name: "Antipasti" },
			pizza: { icon: "fas fa-pizza-slice", name: "Pizze" },
			primo: { icon: "fas fa-utensils", name: "Primi Piatti" },
			secondo: { icon: "fas fa-drumstick-bite", name: "Secondi Piatti" },
			contorno: { icon: "fas fa-carrot", name: "Contorni" },
			dolce: { icon: "fas fa-cake-candles", name: "Dolci" },
			bevande: { icon: "fas fa-wine-bottle", name: "Bevande" },
		};

		const filteredMenus = computed(() => {
			let filtered = menus.value;

			if (selectedCategory.value !== "all") {
				filtered = filtered.filter(
					(item) => item.category === selectedCategory.value
				);
			}

			if (searchQuery.value) {
				const query = searchQuery.value.toLowerCase();
				filtered = filtered.filter((item) => {
					const nameMatch = item.name.toLowerCase().includes(query);
					const tagMatch = item.tags.some((tag) =>
						tag.name.toLowerCase().includes(query)
					);
					return nameMatch || tagMatch;
				});
			}

			return filtered;
		});

		const beverageOrder = {
			"bevande generiche": 1,
			"caffe e digestivi": 2,
			birre: 3,
			vini: 4,
		};

		const sortBeverages = (items) => {
			return items.sort((a, b) => {
				const tagA = a.tags.find((t) => beverageOrder[t.name.toLowerCase()]);
				const tagB = b.tags.find((t) => beverageOrder[t.name.toLowerCase()]);

				const orderA = tagA ? beverageOrder[tagA.name.toLowerCase()] : 999;
				const orderB = tagB ? beverageOrder[tagB.name.toLowerCase()] : 999;

				return orderA - orderB;
			});
		};

		const groupedMenus = computed(() => {
			const grouped = {};

			Object.keys(categories).forEach((categoryKey) => {
				const items = filteredMenus.value.filter(
					(item) => item.category === categoryKey
				);
				if (items.length > 0) {
					grouped[categoryKey] = {
						...categories[categoryKey],
						items: categoryKey === "bevande" ? sortBeverages(items) : items,
					};
				}
			});

			return grouped;
		});

		const noResults = computed(
			() => Object.keys(groupedMenus.value).length === 0 && !loading.value
		);

		const fetchMenu = async () => {
			try {
				const response = await axios.get("http://127.0.0.1:8001/api/menus");
				if (response.data.success) {
					menus.value = response.data.data;
				}
				loading.value = false;
			} catch (err) {
				console.error("Error fetching menu:", err);
				error.value = true;
				loading.value = false;
			}
		};

		const formatPrice = (price) => Number(price).toFixed(2);

		const openItemModal = (item) => {
			selectedItem.value = item;
			document.body.style.overflow = "hidden";
		};

		const closeItemModal = () => {
			selectedItem.value = null;
			document.body.style.overflow = "";
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

		const scrollToTop = () => {
			window.scrollTo({ top: 0, behavior: "smooth" });
		};

		const handleScroll = () => {
			showBackToTop.value = window.scrollY > 400;
		};

		const toggleMobileMenu = () => {
			isMobileMenuOpen.value = !isMobileMenuOpen.value;
			document.body.style.overflow = isMobileMenuOpen.value ? "hidden" : "";
		};

		const selectCategoryAndClose = (category) => {
			selectedCategory.value = category;
			toggleMobileMenu();
		};

		const selectItemAndClose = (item) => {
			openItemModal(item);
			toggleMobileMenu();
			isMobileSearchActive.value = false;
			searchQuery.value = "";
		};

		const handleSearchSubmit = () => {
			if (searchQuery.value.trim()) {
				toggleMobileMenu();
				isMobileSearchActive.value = false;
			}
		};

		const hasActiveFilters = computed(() => {
			return searchQuery.value !== "" || selectedCategory.value !== "all";
		});

		const resetFilters = () => {
			searchQuery.value = "";
			selectedCategory.value = "all";
			isMobileSearchActive.value = false;
		};

		const truncateModalDescription = (text) => {
			if (!text || showFullDescription.value) return text;
			return text.length > 300 ? text.substring(0, 300) + "..." : text;
		};

		const toggleFullDescription = () => {
			showFullDescription.value = !showFullDescription.value;
		};

		const beverageDisplayNames = {
			"bevande generiche": "Bibite",
			"caffe e digestivi": "Caffè e Digestivi",
			birre: "Birre",
			vini: "Vini",
		};

		const getBeverageDisplayName = (tagName) => {
			return beverageDisplayNames[tagName.toLowerCase()] || tagName;
		};

		const getBeverageTag = (item) => {
			if (!item?.tags) return null;
			return item.tags.find((tag) => {
				const tagName = tag.name.toLowerCase();
				return [
					"bevande generiche",
					"caffe e digestivi",
					"birre",
					"vini",
				].includes(tagName);
			});
		};

		const getNonBeverageTags = (item) => {
			if (!item?.tags) return [];
			return item.tags.filter((tag) => {
				const tagName = tag.name.toLowerCase();
				return ![
					"bevande generiche",
					"caffe e digestivi",
					"birre",
					"vini",
				].includes(tagName);
			});
		};

		onMounted(() => {
			fetchMenu();
			setTimeout(() => {
				isVisible.value = true;
			}, 100);
			checkScroll();
			window.addEventListener("scroll", handleScroll);
		});

		onUnmounted(() => {
			window.removeEventListener("scroll", handleScroll);
			document.body.style.overflow = "";
		});

		return {
			loading,
			error,
			searchQuery,
			selectedCategory,
			categories,
			groupedMenus,
			noResults,
			isVisible,
			selectedItem,
			openItemModal,
			closeItemModal,
			formatPrice,
			categoriesContainer,
			isScrolledLeft,
			isScrolledRight,
			scrollCategories,
			checkScroll,
			showBackToTop,
			scrollToTop,
			isMobileMenuOpen,
			toggleMobileMenu,
			selectCategoryAndClose,
			isMobileSearchActive,
			selectItemAndClose,
			handleSearchSubmit,
			hasActiveFilters,
			resetFilters,
			filteredMenus, // Add this line
			truncateModalDescription,
			toggleFullDescription,
			showFullDescription,
			getBeverageDisplayName,
			getBeverageTag,
			getNonBeverageTags,
		};
	},
};
</script>

<style lang="scss" scoped>
.scrollbar-hide::-webkit-scrollbar {
	display: none;
}

.scrollbar-hide {
	-ms-overflow-style: none;
	scrollbar-width: none;
}

.transition-backdrop {
	transition-property: backdrop-filter, background-color;
	transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
	transition-duration: 300ms;
}

.smooth-entrance {
	transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.scrollbar-hide::-webkit-scrollbar {
	display: none;
}

.mobile-menu-enter-active,
.mobile-menu-leave-active {
	transition: transform 0.3s ease-in-out;
}

.mobile-menu-enter-from,
.mobile-menu-leave-to {
	transform: translateX(100%);
}

/* Add these styles for better mobile menu scrolling */
.mobile-menu-content {
	max-height: 100vh;
	overflow-y: auto;
	-webkit-overflow-scrolling: touch;
}
</style>
