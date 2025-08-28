# Technical Architecture Documentation

This document provides comprehensive technical architecture documentation for the RWD Hydrogen v1 e-commerce application.

## Table of Contents

1. [System Overview](#system-overview)
2. [Component Architecture](#component-architecture)
3. [Data Flow Architecture](#data-flow-architecture)
4. [Routing Architecture](#routing-architecture)
5. [Deployment Architecture](#deployment-architecture)
6. [API Integration Architecture](#api-integration-architecture)
7. [Technology Stack](#technology-stack)

## System Overview

The RWD Hydrogen v1 application is built on Shopify's Hydrogen framework, which provides a React-based headless commerce solution. The system follows a modern full-stack architecture with server-side rendering capabilities.

```mermaid
graph TB
    subgraph "Client Layer"
        Browser[Web Browser]
        Mobile[Mobile Browser]
    end
    
    subgraph "Application Layer"
        CDN[Shopify CDN]
        Oxygen[Shopify Oxygen Edge]
        App[Hydrogen App]
    end
    
    subgraph "Framework Layer"
        Remix[Remix Framework]
        React[React Components]
        Vite[Vite Build Tool]
    end
    
    subgraph "Data Layer"
        StorefrontAPI[Shopify Storefront API]
        CustomerAPI[Customer Account API]
        GraphQL[GraphQL Layer]
    end
    
    subgraph "Infrastructure Layer"
        Shopify[Shopify Platform]
        Analytics[Shopify Analytics]
    end
    
    Browser --> CDN
    Mobile --> CDN
    CDN --> Oxygen
    Oxygen --> App
    App --> Remix
    Remix --> React
    React --> Vite
    App --> GraphQL
    GraphQL --> StorefrontAPI
    GraphQL --> CustomerAPI
    StorefrontAPI --> Shopify
    CustomerAPI --> Shopify
    App --> Analytics
```

## Component Architecture

The application follows a modular component architecture with clear separation of concerns.

```mermaid
graph TD
    subgraph "Root Level"
        Root[root.jsx]
        Layout[Layout Component]
    end
    
    subgraph "Page Layout"
        PageLayout[PageLayout]
        Header[Header]
        Footer[Footer]
        Aside[Aside]
    end
    
    subgraph "Route Components"
        Index[_index.jsx]
        Products[products.$handle.jsx]
        Collections[collections.$handle.jsx]
        Cart[cart.jsx]
        Account[account.jsx]
        Search[search.jsx]
    end
    
    subgraph "UI Components"
        ProductItem[ProductItem]
        ProductImage[ProductImage]
        ProductPrice[ProductPrice]
        ProductForm[ProductForm]
        AddToCart[AddToCartButton]
        CartMain[CartMain]
        CartSummary[CartSummary]
        SearchForm[SearchForm]
        SearchResults[SearchResults]
    end
    
    subgraph "Utility Components"
        PaginatedResourceSection[PaginatedResourceSection]
        CartLineItem[CartLineItem]
        SearchFormPredictive[SearchFormPredictive]
        SearchResultsPredictive[SearchResultsPredictive]
    end
    
    Root --> Layout
    Layout --> PageLayout
    PageLayout --> Header
    PageLayout --> Footer
    PageLayout --> Aside
    
    Index --> ProductItem
    Products --> ProductImage
    Products --> ProductPrice
    Products --> ProductForm
    Collections --> ProductItem
    Cart --> CartMain
    Cart --> CartSummary
    Account --> SearchForm
    Search --> SearchResults
    
    ProductForm --> AddToCart
    CartMain --> CartLineItem
    SearchForm --> SearchFormPredictive
    SearchResults --> SearchResultsPredictive
    Collections --> PaginatedResourceSection
```

## Data Flow Architecture

The application implements a comprehensive data flow pattern using GraphQL for API communication and Remix loaders for data fetching.

```mermaid
sequenceDiagram
    participant Browser
    participant RemixRouter as Remix Router
    participant Loader as Route Loader
    participant Context as App Context
    participant GraphQL as GraphQL Layer
    participant StorefrontAPI as Storefront API
    participant CustomerAPI as Customer Account API
    participant Shopify as Shopify Backend
    
    Browser->>RemixRouter: Navigate to route
    RemixRouter->>Loader: Execute loader function
    Loader->>Context: Get context (storefront, cart, customer)
    
    par Critical Data Loading
        Loader->>GraphQL: Query header data
        GraphQL->>StorefrontAPI: HEADER_QUERY
        StorefrontAPI->>Shopify: Fetch menu data
        Shopify-->>StorefrontAPI: Return menu data
        StorefrontAPI-->>GraphQL: Return header data
        GraphQL-->>Loader: Return header data
    and Deferred Data Loading
        Loader->>GraphQL: Defer footer query
        GraphQL->>StorefrontAPI: FOOTER_QUERY
        StorefrontAPI->>Shopify: Fetch footer data
        Loader->>Context: Get cart data
        Loader->>CustomerAPI: Check login status
    end
    
    Loader-->>RemixRouter: Return loader data
    RemixRouter-->>Browser: Render page with data
    
    Note over Browser, Shopify: Deferred data loads after initial render
    GraphQL-->>Browser: Stream deferred data
```

## Routing Architecture

The application uses file-based routing provided by Remix, with a clear hierarchical structure.

```mermaid
graph TD
    subgraph "Root Routes"
        RootRoute[/ - _index.jsx]
        RobotsRoute[/robots.txt - robots.txt.jsx]
        SitemapRoute[/sitemap.xml - sitemap.xml.jsx]
        APIRoute[/api - api.$version.jsx]
    end
    
    subgraph "Product Routes"
        ProductsIndex[/products - products._index.jsx]
        ProductHandle[/products/:handle - products.$handle.jsx]
    end
    
    subgraph "Collection Routes"
        CollectionsIndex[/collections - collections._index.jsx]
        CollectionAll[/collections/all - collections.all.jsx]
        CollectionHandle[/collections/:handle - collections.$handle.jsx]
    end
    
    subgraph "Account Routes"
        AccountIndex[/account - account._index.jsx]
        AccountProfile[/account/profile - account.profile.jsx]
        AccountOrders[/account/orders - account.orders._index.jsx]
        AccountOrderDetail[/account/orders/:id - account.orders.$id.jsx]
        AccountAddresses[/account/addresses - account.addresses.jsx]
        AccountLogin[/account/login - account_.login.jsx]
        AccountLogout[/account/logout - account_.logout.jsx]
        AccountAuthorize[/account/authorize - account_.authorize.jsx]
    end
    
    subgraph "Commerce Routes"
        CartRoute[/cart - cart.jsx]
        CartLines[/cart/:lines - cart.$lines.jsx]
        SearchRoute[/search - search.jsx]
        DiscountRoute[/discount/:code - discount.$code.jsx]
    end
    
    subgraph "Content Routes"
        BlogsIndex[/blogs - blogs._index.jsx]
        BlogHandle[/blogs/:blogHandle - blogs.$blogHandle._index.jsx]
        ArticleHandle[/blogs/:blogHandle/:articleHandle - blogs.$blogHandle.$articleHandle.jsx]
        PagesHandle[/pages/:handle - pages.$handle.jsx]
        PoliciesIndex[/policies - policies._index.jsx]
        PolicyHandle[/policies/:handle - policies.$handle.jsx]
    end
    
    subgraph "Utility Routes"
        CatchAll[/* - $.jsx]
        SitemapDynamic[/sitemap/:type/:page.xml - sitemap.$type.$page.xml.jsx]
    end
    
    RootRoute --> ProductsIndex
    RootRoute --> CollectionsIndex
    RootRoute --> AccountIndex
    RootRoute --> CartRoute
    RootRoute --> SearchRoute
    RootRoute --> BlogsIndex
    RootRoute --> PagesHandle
    RootRoute --> PoliciesIndex
```

## Deployment Architecture

The application is designed to deploy on Shopify's Oxygen platform with edge computing capabilities.

```mermaid
graph TB
    subgraph "Development Environment"
        DevServer[Local Dev Server]
        HMR[Hot Module Reload]
        DevDB[Development Store]
    end
    
    subgraph "Build Process"
        ViteBuild[Vite Build]
        TypeGen[Type Generation]
        CodeGen[GraphQL Codegen]
        Bundle[Production Bundle]
    end
    
    subgraph "Oxygen Platform"
        OxygenEdge[Oxygen Edge Workers]
        OxygenRuntime[Oxygen Runtime]
        EdgeCache[Edge Caching]
    end
    
    subgraph "CDN Layer"
        ShopifyCDN[Shopify CDN]
        StaticAssets[Static Assets]
        ImageOptim[Image Optimization]
    end
    
    subgraph "Shopify Infrastructure"
        StorefrontAPI2[Storefront API]
        CustomerAPI2[Customer Account API]
        Analytics2[Shopify Analytics]
        Checkout[Shopify Checkout]
    end
    
    subgraph "Client Delivery"
        Browser2[Web Browsers]
        Mobile2[Mobile Devices]
        PWA[Progressive Web App]
    end
    
    DevServer --> ViteBuild
    HMR --> ViteBuild
    DevDB --> ViteBuild
    
    ViteBuild --> TypeGen
    ViteBuild --> CodeGen
    TypeGen --> Bundle
    CodeGen --> Bundle
    
    Bundle --> OxygenEdge
    OxygenEdge --> OxygenRuntime
    OxygenRuntime --> EdgeCache
    
    EdgeCache --> ShopifyCDN
    ShopifyCDN --> StaticAssets
    ShopifyCDN --> ImageOptim
    
    OxygenRuntime --> StorefrontAPI2
    OxygenRuntime --> CustomerAPI2
    OxygenRuntime --> Analytics2
    OxygenRuntime --> Checkout
    
    ShopifyCDN --> Browser2
    ShopifyCDN --> Mobile2
    ShopifyCDN --> PWA
```

## API Integration Architecture

The application integrates with multiple Shopify APIs through a GraphQL layer with proper caching and optimization.

```mermaid
graph LR
    subgraph "Client Layer"
        Components[React Components]
        Loaders[Remix Loaders]
        Actions[Remix Actions]
    end
    
    subgraph "GraphQL Layer"
        Fragments[GraphQL Fragments]
        Queries[GraphQL Queries]
        Mutations[GraphQL Mutations]
        CodeGen[Generated Types]
    end
    
    subgraph "Context Layer"
        HydrogenContext[Hydrogen Context]
        StorefrontClient[Storefront Client]
        CustomerClient[Customer Client]
        CartContext[Cart Context]
        SessionContext[Session Context]
    end
    
    subgraph "Caching Layer"
        CacheLong[Long Cache]
        CacheShort[Short Cache]
        EdgeCache2[Edge Cache]
        BrowserCache[Browser Cache]
    end
    
    subgraph "Shopify APIs"
        StorefrontAPI3[Storefront API 2024-10]
        CustomerAPI3[Customer Account API]
        AdminAPI[Admin API]
        WebhooksAPI[Webhooks API]
    end
    
    Components --> Loaders
    Components --> Actions
    Loaders --> Queries
    Actions --> Mutations
    
    Queries --> Fragments
    Mutations --> Fragments
    Fragments --> CodeGen
    
    Queries --> StorefrontClient
    Mutations --> StorefrontClient
    StorefrontClient --> HydrogenContext
    CustomerClient --> HydrogenContext
    CartContext --> HydrogenContext
    SessionContext --> HydrogenContext
    
    StorefrontClient --> CacheLong
    StorefrontClient --> CacheShort
    CacheLong --> EdgeCache2
    CacheShort --> EdgeCache2
    EdgeCache2 --> BrowserCache
    
    StorefrontClient --> StorefrontAPI3
    CustomerClient --> CustomerAPI3
    HydrogenContext --> AdminAPI
    HydrogenContext --> WebhooksAPI
```

## Technology Stack

### Frontend Technologies
- **React 18**: Component-based UI framework with concurrent features
- **Remix**: Full-stack web framework with nested routing and data loading
- **React Router 7**: Client-side routing with data loading capabilities
- **TypeScript**: Type-safe JavaScript development
- **Vite**: Fast build tool and dev server

### Backend Technologies
- **Shopify Hydrogen**: React-based framework for headless commerce
- **Shopify Oxygen**: Edge computing platform for deployment
- **GraphQL**: Query language for API communication
- **Node.js**: JavaScript runtime environment

### Development Tools
- **ESLint**: Code linting and quality enforcement
- **Prettier**: Code formatting
- **GraphQL Code Generator**: Automatic type generation from GraphQL schemas
- **Shopify CLI**: Development and deployment tooling

### Infrastructure
- **Shopify Platform**: E-commerce backend and infrastructure
- **Shopify CDN**: Content delivery network
- **Edge Workers**: Serverless compute at the edge
- **Progressive Web App**: Modern web app capabilities

### API Integration
- **Shopify Storefront API**: Product and store data access
- **Customer Account API**: Customer authentication and account management
- **Admin API**: Store administration capabilities
- **Webhooks**: Real-time event notifications

This architecture provides a scalable, performant, and maintainable foundation for modern e-commerce applications with excellent developer experience and optimal user performance.