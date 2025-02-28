<template>
  <div class="category-filter">
    <div
      v-for="category in categories"
      :key="category.name"
      class="category-item"
      @click="selectCategory(category.name)"
      :class="{ active: selectedCategory === category.name }"
    >
      <div class="icon-container">
        <img :src="category.image" :alt="category.name" class="category-icon" />
      </div>
      <span class="category-name">{{ category.name }}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "CategoryFilterWithImages",
  props: {
    categories: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      selectedCategory: null,
    };
  },
  methods: {
    selectCategory(category) {
      this.selectedCategory = category;
      this.$emit("filter", category); // Emit the selected category for filtering
      this.$emit("category-selected", category); // Emit a custom event for category selection
    },
  },
};
</script>


<style scoped>
.category-filter {
  display: flex;
  gap: 15px;
  padding: 10px;
  overflow-x: auto;
  background-color: #f5f5f5;
  border-bottom: 1px solid #ccc;
}

.category-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  padding: 10px;
  border-radius: 8px;
  background-color: white;
  transition: transform 0.2s;
  width: 150px; /* Match the width of product cards */
  text-align: center;
}

.category-item:hover {
  transform: scale(1.05);
}

.category-item.active {
  background-color: orange;
}

.icon-container {
  width: 100px; /* Large icon size */
  height: 100px; /* Large icon size */
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 10px;
}

.category-icon {
  width: 100%; /* Ensure the icon fills the container */
  height: 100%; /* Ensure the icon fills the container */
  object-fit: cover;
  border-radius: 8px;
}

.category-name {
  font-size: 14px;
  text-align: center;
}
.category-filter{
	width: 39%;
}
@media (max-width: 2000px) {
	.category-filter{
		width: 95%;
	}
}
</style>