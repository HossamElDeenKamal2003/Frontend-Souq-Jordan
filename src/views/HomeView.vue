<template>
  <div class="parent">
    <CategoryFilterWithImages
      :categories="categories"
      @category-selected="handleCategorySelected"
    />

    <CarsComponent v-if="showCarsComponent" />
    <BuildingComponent v-if="showBuildingComponent" />
    <DevicesComponent v-if="showDevices" />
    <FurnitureComponent 
      v-if="showFurnitureComponent" 
      @furniture-selected="handleFurnitureSelected" 
    />
    <personalAccessoriesList v-if="showPersonalneeds" />
    <jobs-component v-if="jobsComponent" />
    <others-component v-if="othersComponent" />

    <div class="search-add-container">
      <input
        type="text"
        placeholder="Search products..."
        v-model="searchQuery"
        :class="['search-input', isDark ? 'dark-search' : '']"
      />
      <button class="add-offer-button">+Add Offer</button>
    </div>

    <div class="scrollable-container">
      <div class="products-container">
        <div
          v-for="(product, index) in filteredProducts"
          :key="product._id"
          :class="[
            'product-card',
            index % 2 === 0 ? 'white-card' : 'grey-card',
            isDark ? 'dark-card' : ''
          ]"
        >
          <img
            :src="product.images && product.images.length ? product.images[0] : require('../assets/jordan image.jpeg')"
            alt="Product Image"
            class="product-image"
          />
          <div :class="['product-info', isDark ? 'dark-text' : '']">
            <h3 :class="{ 'seen-title': product.isSeen }">{{ product.title }}</h3>
            <p>{{ product.description }}</p>
            <div class="indicators">
              <span v-if="product.isSeen" class="seen-indicator">👁️ Seen</span>
              <span v-if="product.isFavourite" class="favourite-indicator">❤️ Favourite</span>
            </div>
            <div v-if="product.price" class="price">
              {{ formatPrice(product.price) }}
            </div>
            <div class="time-location">
              <div class="time-difference">
                <i class="fas fa-clock"></i>
                {{ formatTimeDifference(product.createdAt) }}
              </div>
              <div v-if="product.location" class="location">
                <i class="fas fa-map-marker-alt"></i>
                {{ product.location }}
              </div>
            </div>
          </div>
        </div>
      </div>
      <div ref="loadMoreTrigger" class="loading-indicator">
        <p v-if="loading">Loading more products...</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ConstVariables } from "../../const.js";
import CategoryFilterWithImages from "../components/homeScreenFilter.vue";
import CarsComponent from "../components/cars/carsComponent.vue";
import BuildingComponent from "../components/cars/buildingComponent.vue";
import DevicesComponent from "../components/cars/devicesComponent.vue";
import FurnitureComponent from "../components/cars/furniturreComponent.vue"; // Fixed typo
import PersonalAccessoriesList from "../components/cars/personalneedsComponent.vue";
import JobsList from "../components/cars/jobsComponent.vue";
import OthersList from "../components/cars/otherComponent.vue";

export default {
  name: "HomeView",
  components: {
    CategoryFilterWithImages,
    CarsComponent,
    BuildingComponent,
    DevicesComponent,
    FurnitureComponent,
    PersonalAccessoriesList,
    'jobs-component': JobsList, // Use kebab-case in template
    'others-component': OthersList // Use kebab-case in template
  },
  data() {
    return {
      products: [],
      page: 1,
      limit: 10,
      totalPages: 1,
      loading: false,
      observer: null,
      searchQuery: "",
      selectedCategory: null,
      selectedCarType: null,
      showCarsComponent: false,
      showBuildingComponent: false,
      showDevices: false,
      showPersonalneeds: false,
      showFurnitureComponent: false,
      jobsComponent: false,
      othersComponent: false,
      categories: []
    };
  },
  props: {
    isDark: Boolean,
  },
  computed: {
    filteredProducts() {
      return this.products.filter((product) => {
        return (
          (!this.selectedCategory || product.category === this.selectedCategory) &&
          (!this.searchQuery ||
            product.title.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
            product.description.toLowerCase().includes(this.searchQuery.toLowerCase()))
        );
      });
    },
  },
  created() {
    this.categories = this.getCategories();
  },
  mounted() {
    this.getTrips();
    this.$nextTick(this.setupInfiniteScroll);
  },
  beforeUnmount() {
    if (this.observer) this.observer.disconnect();
  },
  methods: {
    async getTrips() {
      if (this.loading || this.page > this.totalPages) return;

      try {
        this.loading = true;
        const userId = localStorage.getItem("userId") || "6762162e2be4997bad6b8237";
        const response = await axios.get(
          `https://backend.jordan-souq.com/product/posts/${userId}?page=${this.page}&limit=${this.limit}`
        );

        this.products.push(...response.data.products);
        this.totalPages = response.data.totalPages;
        this.page++;
      } catch (error) {
        console.error("Error fetching trips:", error.message);
      } finally {
        this.loading = false;
      }
    },
    setupInfiniteScroll() {
      this.observer = new IntersectionObserver(
        (entries) => {
          if (entries[0].isIntersecting) this.getTrips();
        },
        { threshold: 1 }
      );

      if (this.$refs.loadMoreTrigger) {
        this.observer.observe(this.$refs.loadMoreTrigger);
      }
    },
    handleCarTypeClick(carType) {
      console.log("Selected car type:", carType);
    },
    handleFurnitureSelected(furniture) {
      console.log("Selected furniture type:", furniture);
      this.selectedCategory = furniture;
    },
    handleCategorySelected(category) {
      // Reset all components to false first
      this.showCarsComponent = false;
      this.showBuildingComponent = false;
      this.showDevices = false;
      this.showFurnitureComponent = false;
      this.showPersonalneeds = false;
      this.jobsComponent = false;
      this.othersComponent = false;

      // Toggle the selected component
      if (category === "Cars") {
        this.showCarsComponent = !this.showCarsComponent;
        this.selectedCategory = this.showCarsComponent ? "Cars" : null;
        this.selectedCarType = this.showCarsComponent ? ConstVariables.carsTypeList : null;
      } else if (category === "Building") {
        this.showBuildingComponent = !this.showBuildingComponent;
        this.selectedCategory = this.showBuildingComponent ? "Building" : null;
      } else if (category === "Devices") {
        this.showDevices = !this.showDevices;
        this.selectedCategory = this.showDevices ? "Devices" : null;
      } else if (category === "Furniture") {
        this.showFurnitureComponent = !this.showFurnitureComponent;
        this.selectedCategory = this.showFurnitureComponent ? "Furniture" : null;
      } else if (category === "Personal Needs") {
        this.showPersonalneeds = !this.showPersonalneeds;
        this.selectedCategory = this.showPersonalneeds ? "Personal Needs" : null;
      } else if (category === "Jobs") {
        this.jobsComponent = !this.jobsComponent;
        this.selectedCategory = this.jobsComponent ? "Jobs" : null;
      } else if (category === "Others") {
        this.othersComponent = !this.othersComponent;
        this.selectedCategory = this.othersComponent ? "Others" : null;
      }
    },
    formatPrice(price) {
      return `$${price}`;
    },
    formatTimeDifference(createdAt) {
      const now = new Date();
      const createdDate = new Date(createdAt);
      const diffInSeconds = Math.floor((now - createdDate) / 1000);

      if (diffInSeconds < 60) return `${diffInSeconds} seconds ago`;
      if (diffInSeconds < 3600) return `${Math.floor(diffInSeconds / 60)} minutes ago`;
      if (diffInSeconds < 86400) return `${Math.floor(diffInSeconds / 3600)} hours ago`;
      return `${Math.floor(diffInSeconds / 86400)} days ago`;
    },
    getCategories() {
      return [
        { name: "All", image: "https://cdn-icons-png.flaticon.com/512/9696/9696976.png" },
        { name: "Cars", image: "https://ymimg1.b8cdn.com/uploads/car_model/10543/pictures/12904367/2023_Jetour_Dashing_Plus_Exterior_01.png", category: "سيارات", metaCategory: ConstVariables.carsTypeList },
        { name: "Building", image: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTn0dhO9J7R4JT424Qs-RcZqQRkw9tTCbJoxw&s", category: "building", metaCategory: ConstVariables.realStateSpacialTypes },
        { name: "Devices", image: "https://st.depositphotos.com/1682899/2107/i/450/depositphotos_21075061-stock-photo-electronic-devices.jpg", category: "devices", metaCategory: ConstVariables.devicesTypeList },
        { name: "Furniture", image: "https://m.media-amazon.com/images/I/71u3F2NZ9gL.jpg", category: "furniture", metaCategory: ConstVariables.furnitureList },
        { name: "Personal Needs", image: "https://t4.ftcdn.net/jpg/01/86/85/71/360_F_186857190_s4dfc0wfT6jcEcr7e3vzrFuUdysg6Gpp.jpg", category: "personal needs", metaCategory: ConstVariables.personalAccessoriesList },
        { name: "Jobs", image: "https://cdn-icons-png.flaticon.com/512/3850/3850285.png", category: "jobs", metaCategory: ConstVariables.jobsList },
        { name: "Others", image: "https://cdn-icons-png.flaticon.com/512/91/91356.png", category: "others", metaCategory: ConstVariables.othersTypeList },
      ];
    },
  },
};
</script>
<style scoped>
/* Same styles as before, no changes needed unless you want to adjust visuals */
.parent {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
  padding-top: 20px;
}

.car-types-container {
  display: flex;
  overflow-x: auto;
  gap: 10px;
  padding: 10px;
  margin-bottom: 20px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #ccc;
  width: 39%;
}

.car-type-item {
  padding: 10px 15px;
  background-color: white;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.car-type-item:hover {
  background-color: orange;
}

.search-add-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  max-width: 1000px;
  margin-bottom: 20px;
  padding: 0 10px;
  position: sticky;
  top: 0;
  z-index: 10;
}

.search-input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-right: 10px;
  font-size: 16px;
  background-color: white;
  color: #333;
}

.dark-search {
  background-color: #333;
  color: white;
  border-color: #555;
}

.add-offer-button {
  padding: 10px 20px;
  background-color: orange;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.add-offer-button:hover {
  background-color: darkorange;
}

.scrollable-container {
  width: 100%;
  max-width: 1000px;
  overflow-y: auto;
  max-height: calc(100vh - 120px);
  padding: 0 10px;
}

.products-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.product-card {
  display: flex;
  align-items: center;
  width: 100%;
  max-width: 1000px;
  padding: 10px;
  height: 150px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
  background-color: white;
  overflow: hidden;
  position: relative;
}

.product-card:hover {
  transform: scale(1.02);
}

.product-image {
  width: 150px;
  height: 150px;
  object-fit: cover;
  border-radius: 5px;
  margin-right: 10px;
}

.product-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  overflow: hidden;
}

.product-info h3 {
  margin: 0;
  font-size: 16px;
  color: #009688;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.product-info h3.seen-title {
  color: red;
}

.product-info p {
  margin: 5px 0 0;
  font-size: 14px;
  color: #666;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}

.indicators {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.seen-indicator,
.favourite-indicator {
  font-size: 14px;
  color: #666;
}

.dark-text .seen-indicator,
.dark-text .favourite-indicator {
  color: white;
}

.price {
  position: absolute;
  bottom: 10px;
  left: 10px;
  font-size: 14px;
  font-weight: bold;
  color: #009688;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 2px 5px;
  border-radius: 3px;
}

.time-location {
  position: absolute;
  bottom: 10px;
  right: 10px;
  display: flex;
  gap: 15px;
  align-items: center;
}

.time-difference {
  font-size: 14px;
  color: #666;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 2px 5px;
  border-radius: 3px;
  display: flex;
  align-items: center;
  gap: 5px;
}

.location {
  font-size: 14px;
  color: #666;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 2px 5px;
  border-radius: 3px;
  display: flex;
  align-items: center;
  gap: 5px;
}

.white-card {
  background-color: white;
}

.grey-card {
  background-color: #f5f5f5;
}

.loading-indicator {
  text-align: center;
  margin: 20px 0;
  font-weight: bold;
  color: #666;
}

.dark-card {
  background-color: #222 !important;
  color: #fff !important;
  box-shadow: 0 4px 6px rgba(255, 255, 255, 0.1);
}

.dark-text p {
  color: white !important;
}

@media (max-width: 2000px) {
  .category-filter {
    width: 95%;
  }
}
</style>