# High level architecture AEM 6.5 + Adobe Commerce Integration with CIF

How to display **Configurable Product Data (dynamic pricing)** on the **Adobe Experience Manager (AEM) 6.5 Product Display Page (PDP)** using the **Commerce Integration Framework (CIF)** and **Adobe Commerce**.

---

## 📖 Overview

- **AEM 6.5** provides authoring and delivery of Product Display Pages (PDPs).
- **Adobe Commerce** supplies product catalog, dynamic pricing, promotions, and inventory.
- **CIF (Commerce Integration Framework)** enables seamless (headless) integration through GraphQL APIs.
- **Adobe I/O Runtime** is used as an integration layer to synchronize and normalize data.

This integration ensures that configurable product data (variants, dynamic prices, stock) is rendered dynamically in AEM while keeping commerce logic inside Adobe Commerce.

---

## 🏗 Architecture

The integration spans **four main domains**:  

1. **Authoring**  
   - AEM Author: Authors create PDP templates and link SKUs.  
   - AEM Publish: Delivers published content to customers.  

2. **Delivery**  
   - Dispatcher / CDN: Caches and delivers pages, assets, CSS/JS to the browser.  

3. **Commerce**  
   - Adobe Commerce manages catalog, prices, promotions, and inventory.  
   - Magento DB stores transactional data.  
   - Elasticsearch powers product indexing and search.  
   - GraphQL API exposes real-time product data.  

4. **Integrations**  
   - Adobe I/O Runtime hosts the CIF Integration Layer for synchronization and middleware logic.  

### Flowchart
![Architecture flow chart](https://i.imgur.com/ae2sgnV.jpeg)


## ⚡ Data Flow

1. **Authoring**
    - Authors create PDPs in AEM Author.
    - Products are linked via SKUs using CIF components.
    - Pages are published to AEM Publish.

2. **Delivery**
    - Customers request PDPs via Dispatcher/CDN.
    - Static content (HTML, JS, CSS, assets) is served from cache.
    - Dynamic product data is fetched from Adobe Commerce via GraphQL.

3. **Commerce**
   - GraphQL API retrieves configurable product data.
   - Magento DB + Elasticsearch provide catalog, pricing, and search.
   - Configurable options (size, color, etc.) and variant SKUs are resolved.

5. **Integrations**
    - CIF Integration Layer in Adobe I/O Runtime synchronizes catalog data.
    - Middleware enables custom mappings and third-party integrations.

### 🛠 Example GraphQL Query (Configurable Product)
```
query getConfigurableProduct($sku: String!) {
  products(filter: { sku: { eq: $sku } }) {
    items {
      __typename
      ... on ConfigurableProduct {
        sku
        name
        description { html }
        image { url }
        configurable_options {
          attribute_code
          label
          values { value_index label }
        }
        variants {
          product {
            sku
            name
            stock_status
            image { url }
            price_range {
              minimum_price {
                regular_price { value currency }
              }
            }
          }
          attributes { code value_index }
        }
      }
    }
  }
}

```

