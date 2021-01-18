<template>
  <div>
    <div v-for="product in $page.Vendure.products.items" >
      <img :src="product.featuredAsset.preview" class="thumbnail" />
      <p :class="product.soldOut ? 'sold-out' : ''" class="product-title">{{ product.name }}</p>
      <p :class="product.soldOut ? 'sold-out' : ''">{{ product.variants[0].priceWithTax }}</p>
      <p v-if="product.soldOut">SOLD OUT</p>
    </div>
  </div>
</template>
<page-query>
query {
  Vendure {
    products {
      items {
        id
        name
        featuredAsset {
          preview
        }
        variants {
          priceWithTax
        }
      }
    }
  }
}
</page-query>
<script>
import {GraphQLClient} from 'graphql-request';

export default {
  async mounted() {
    const client = new GraphQLClient('https://yourwebshop-api.com/graphql');
    client.setHeader('vendure-token', 'demo')
    const {products: {items: products}} = await client.request(`{
                                                        products {
                                                            items {
                                                                id
                                                                variants {
                                                                    available
                                                                }
                                                            }
                                                        }
                                                      }`);
    this.$page.Vendure.products.items = this.$page.Vendure.products.items.map(product => {
      const clientSideProduct = products.find(p => p.id === product.id);
      if (clientSideProduct) {
        product.soldOut = !clientSideProduct.variants.find(variant => variant.available > 0);
      }
      return product;
    })

  }
}
</script>

<style>
.thumbnail {
  width: 20%;
}
.product-title {
  font-weight: bold;
}
.sold-out {
  text-decoration: line-through;
}
</style>
