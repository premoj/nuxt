<template>
  <div id="main" class="flex justify-center">
    <div id="loading" v-if="$apollo.queries.categories.loading">
      <div class="spinner-square"></div>
    </div>

    <div v-else id="layout" class="container mx-2 my-grid my-10">
      <div id="side-bar" class="flex-col px-6 mb-4 md:mb-0">
        <p class="title text-gray-500 pt-4 text-2xl">Categories</p>

        <ul class="py-2 flex flex-row md:flex-col flex-wrap">
          <li
            v-for="item in categories.items[0].children"
            :key="item.uid"
            @click="getCategoryId(item.uid)"
            class="category-list-item p-2"
          >
            {{ item.name }}
          </li>
        </ul>
      </div>

      <div
        id="view"
        class="p-4 flex flex-col justify-between"
        :class="[
          categoryUid ? null : 'items-center',
          $apollo.queries.products.loading
            ? 'items-center justify-center'
            : null,
        ]"
      >
        <p class="text-2xl" v-if="!categoryUid">
          Hello, please choose a category.
        </p>
        <div
          id="loading"
          class="flex justify-center items-center"
          v-if="$apollo.queries.products.loading"
        >
          <div class="spinner-square"></div>
        </div>
        <div
          v-else-if="categoryUid && !$apollo.queries.products.loading"
          id="products"
          class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-5"
        >
          <div
            v-for="item in products.items"
            :key="item.sku"
            class="item flex-col justify-between my-2"
          >
            <p class="p-4 font-bold">{{ item.name }}</p>
            <p class="price">
              {{ item.price_range.minimum_price.regular_price.value }} USD
            </p>
          </div>
        </div>
        <div
          v-if="products.total_count"
          id="pagination"
          class="mt-4 flex justify-center"
        >
          <span
            v-for="page in pagesCount"
            @click="setCurrentPage(page)"
            :key="page"
            class="mx-1 pagination-item"
          >
            {{ page }}</span
          >
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import gql from "graphql-tag";
export default {
  data() {
    return {
      categories: [],
      products: [],
      categoryUid: null,
      currentPage: 1,
      pageSize: 10,
    };
  },
  name: "App",
  components: {},
  computed: {
    pagesCount: function () {
      return Math.ceil(this.products.total_count / this.pageSize);
    },
  },
  methods: {
    getCategoryId(id) {
      console.log(id);
      this.categoryUid = id;
      this.currentPage = 1;
    },
    setCurrentPage(page) {
      console.log(page);
      this.currentPage = page;
    },
  },
  apollo: {
    categories: {
      query: gql`
        query {
          categories {
            total_count
            items {
              uid
              level
              name
              path
              children_count
              children {
                uid
                level
                name
                path
                children_count
                children {
                  uid
                  level
                  name
                  path
                }
              }
            }
            page_info {
              current_page
              page_size
              total_pages
            }
          }
        }
      `,
    },
    products: {
      query: gql`
        query ($categoryUid: String, $currentPage: Int, $pageSize: Int) {
          products(
            filter: { category_uid: { eq: $categoryUid } }
            pageSize: $pageSize
            currentPage: $currentPage
          ) {
            total_count
            items {
              name
              sku
              price_range {
                minimum_price {
                  regular_price {
                    value
                    currency
                  }
                }
              }
            }
            page_info {
              page_size
              current_page
            }
          }
        }
      `,
      variables() {
        return {
          categoryUid: this.categoryUid,
          currentPage: this.currentPage,
          pageSize: this.pageSize,
        };
      },
      skip() {
        return !this.categoryUid;
      },
    },
  },
};
</script>

<style>
body {
  margin: 0;
}

#main {
  font-family: "Avenir", Helvetica, sans-serif;
  min-height: 100vh;
  height: 100%;
  background: rgb(2, 0, 36);
  background: linear-gradient(
    90deg,
    rgba(2, 0, 36, 1) 0%,
    rgb(89, 89, 238) 0%,
    rgba(0, 212, 255, 1) 100%
  );
}

#side-bar,
#view {
  height: auto;
  background-color: white;
}

#loading {
  display: flex;
  justify-content: center;
  align-items: center;
}

@media screen and (min-width: 768px) {
  .my-grid {
    display: grid;
    grid-template-columns: 200px 1fr;
    grid-column-gap: 10px;
    justify-items: stretch;
    align-items: stretch;
  }

  #side-bar,
  #view {
    border-radius: 10px;
  }
}

.item {
  cursor: pointer;
  position: relative;
  border-radius: 10px;
  height: 150px;
  width: 90%;
  margin-left: auto;
  margin-right: auto;
}

.item:nth-child(even) {
  background-color: antiquewhite;
}

.item:nth-child(odd) {
  background-color: rgb(214, 214, 214);
}

.item:hover {
  background-color: rgb(198, 237, 240);
  transition: 0.5s;
}

.category-list-item {
  cursor: pointer;
  border-radius: 4px;
  background-color: transparent;

  height: fit-content;
}

.category-list-item:hover {
  background-color: rgb(198, 237, 240);
  transition: 0.5s;
}

.products {
  margin: auto;
}

.price {
  position: absolute;
  right: 1rem;
  bottom: 1rem;
}

.spinner-square {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  height: 32px;
  width: 32px;
  background: black;
  animation: flipSquare 1s linear infinite;
}

@keyframes flipSquare {
  50% {
    transform: rotateX(180deg);
  }
  100% {
    transform: rotateX(180deg) rotateY(180deg);
  }
}

.pagination-item {
  cursor: pointer;
}
</style>
