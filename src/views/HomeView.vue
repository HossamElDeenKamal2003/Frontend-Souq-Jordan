<template>
  <div class="parent">
    <!-- Search and Add Offer Section -->
    <div class="search-add-container">
      <input
        type="text"
        placeholder="Search products..."
        v-model="searchQuery"
        :class="['search-input', isDark ? 'dark-search' : '']"
      />
      <button class="add-offer-button">+Add Offer</button>
    </div>

    <!-- Scrollable Products Container -->
    <div class="scrollable-container">
      <!-- Products Container -->
      <div class="products-container">
        <div
          v-for="(product, index) in filteredProducts"
          :key="product._id"
          :class="[
            'product-card',
            index % 2 === 0 ? 'white-card' : 'grey-card',
            isDark ? 'dark-card' : ''  // Apply dark theme
          ]"
        >
          <!-- Image with fallback -->
          <img
            :src="product.images && product.images.length ? product.images[0] : require('../assets/jordan image.jpeg')"
            alt="Product Image"
            class="product-image"
          />

          <!-- Product Info -->
          <div :class="['product-info', isDark ? 'dark-text' : '']">
            <h3>{{ product.title}}</h3>
            <p>{{ product.description }}</p>
          </div>
        </div>
      </div>

      <!-- Load More Indicator -->
      <div ref="loadMoreTrigger" class="loading-indicator">
        <p v-if="loading">Loading more products...</p>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";

export default {
  name: "HomeView",
  data() {
    return {
      products: [], 
      page: 1, 
      limit: 10, 
      totalPages: 1, 
      loading: false,
      observer: null, 
      searchQuery: "", // Search query data property
    };
  },
  props: {
    isDark: Boolean, // Dark theme prop
  },
  computed: {
    // Filter products based on search query
    filteredProducts() {
      if (!this.searchQuery) {
        return this.products; // Return all products if search query is empty
      }
      const query = this.searchQuery.toLowerCase();
      return this.products.filter((product) => {
        return (
          product.title.toLowerCase().includes(query) || // Search by name
          product.description.toLowerCase().includes(query) // Search by description
        );
      });
    },
  },
  methods: {
    async getTrips() {
      if (this.loading || this.page > this.totalPages) return;

      try {
        this.loading = true;
        const userId = localStorage.getItem("userId") || "6762162e2be4997bad6b8237";
        const response = await axios.get(`https://backend.jordan-souq.com/product/posts/${userId}?page=${this.page}&limit=${this.limit})`);
        
        this.products.push(...response.data.products);
        this.totalPages = response.data.totalPages;
        this.page++; // Increment page for next request
      } catch (error) {
        console.error("Error fetching trips:", error.message);
      } finally {
        this.loading = false;
      }
    },
    setupInfiniteScroll() {
      this.observer = new IntersectionObserver((entries) => {
        if (entries[0].isIntersecting) {
          this.getTrips(); // Load more products when user reaches bottom
        }
      }, { threshold: 1 });

      if (this.$refs.loadMoreTrigger) {
        this.observer.observe(this.$refs.loadMoreTrigger);
      }
    }
  },
  mounted() {
    this.getTrips(); // Initial load
    this.$nextTick(() => this.setupInfiniteScroll()); 
  },
  beforeUnmount() {
    if (this.observer) {
      this.observer.disconnect(); 
    }
  }
};
</script>
<style scoped>
.parent {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
  padding-top: 20px; /* Add padding to ensure search bar doesn't overlap */
}

.search-add-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  max-width: 1000px; /* Match the width of product cards */
  margin-bottom: 20px; /* Space between search bar and products */
  padding: 0 10px; /* Add padding for better alignment */
  position: sticky; /* Make search bar sticky */
  top: 0; /* Stick to the top */
  z-index: 10; /* Ensure search bar stays above other content */
}

.search-input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-right: 10px; /* Space between input and button */
  font-size: 16px;
  background-color: white; /* Default light theme */
  color: #333; /* Default light theme */
}

.dark-search {
  background-color: #333; /* Dark theme background */
  color: white; /* Dark theme text */
  border-color: #555; /* Dark theme border */
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
  background-color: darkorange; /* Darker shade on hover */
}

.scrollable-container {
  width: 100%;
  max-width: 1000px; /* Match the width of product cards */
  overflow-y: auto; /* Make the container scrollable */
  max-height: calc(100vh - 120px); /* Adjust height to fit remaining space */
  padding: 0 10px; /* Add padding for better alignment */
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
  max-width: 1000px; /* Adjust based on preference */
  padding: 10px;
  height: 150px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
  background-color: white; /* Or grey if needed */
}

.product-card:hover {
  transform: scale(1.02);
}

.product-image {
  width: 150px;
  height: 150px;
  object-fit: cover;
  border-radius: 5px;
  margin-right: 10px; /* Space between image and text */
}

.product-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.product-info h3 {
  margin: 0;
  font-size: 16px;
  color: #009688; /* Match text color */
}

.product-info p {
  margin: 5px 0 0;
  font-size: 14px;
  color: #666;
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

/* Dark Theme */
.dark-card {
  background-color: #222 !important;
  color: #fff !important;
  box-shadow: 0 4px 6px rgba(255, 255, 255, 0.1);
}

/* Dark Theme Text */
.dark-text h3,
.dark-text p {
  color: white !important; /* Ensure text is white in dark mode */
}
</style>