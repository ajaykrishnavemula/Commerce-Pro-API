# CommerceProAPI

A robust e-commerce API built with Node.js, Express, TypeScript, and MongoDB. This project has been enhanced through three phases of development to include advanced search, customer features, admin dashboard, and analytics.

## Development Phases

This project was enhanced through three major development phases:

### Phase 1: Foundation Upgrade
- Migrated the codebase to TypeScript
- Implemented user authentication
- Enhanced product model
- Set up testing infrastructure

### Phase 2: Core Feature Enhancement
- Added shopping cart functionality
- Implemented order processing
- Added payment integration
- Enhanced product management

### Phase 3: Advanced Features
- Implemented advanced search with Elasticsearch
- Added customer features (wishlists, reviews)
- Created admin dashboard
- Implemented analytics

## Features

- **Product Management**: Create, read, update, and delete products with support for variants, inventory tracking, and SEO
- **User Authentication**: Register, login, and manage user profiles with JWT authentication
- **Shopping Cart**: Add, update, and remove items from cart with support for guest and user carts
- **Order Processing**: Create and manage orders with status tracking and inventory updates
- **Advanced Filtering**: Filter products by various criteria including name, price, rating, and more
- **Pagination & Sorting**: Paginate and sort results for better performance and user experience
- **Security**: Implement rate limiting, helmet security headers, and XSS protection
- **Error Handling**: Custom error handling with appropriate HTTP status codes
- **Logging**: Comprehensive logging for debugging and monitoring
- **Advanced Search**: Elasticsearch integration for powerful search capabilities
- **Customer Features**: Wishlists, product reviews, and ratings
- **Admin Dashboard**: Comprehensive admin interface for store management
- **Analytics**: Sales reports, customer insights, and inventory analytics

## Tech Stack

- **TypeScript**: For type safety and better developer experience
- **Node.js & Express**: For the server framework
- **MongoDB & Mongoose**: For database and ODM
- **JWT**: For authentication
- **Elasticsearch**: For advanced search capabilities
- **Stripe**: For payment processing
- **Helmet**: For security headers
- **XSS-Clean**: For sanitizing user input
- **Express-Rate-Limit**: For rate limiting
- **Winston**: For logging

## Project Structure

```
CommerceProAPI/
├── src/
│   ├── config/             # Configuration files
│   ├── controllers/        # Route controllers
│   ├── db/                 # Database connection
│   ├── errors/             # Custom error classes
│   ├── interfaces/         # TypeScript interfaces
│   ├── middleware/         # Express middleware
│   ├── models/             # Mongoose models
│   ├── routes/             # Express routes
│   ├── services/           # Business logic services
│   │   ├── elasticsearch/  # Elasticsearch service
│   │   ├── payment/        # Payment processing service
│   │   ├── analytics/      # Analytics service
│   │   └── email/          # Email service
│   ├── utils/              # Utility functions
│   └── app.ts              # Express app setup
├── public/                 # Static files
├── tsconfig.json           # TypeScript configuration
└── package.json            # Project dependencies
```

## API Endpoints

### Products

- `GET /api/v1/products` - Get all products with filtering, sorting, and pagination
- `GET /api/v1/products/:id` - Get a single product by ID
- `GET /api/v1/products/featured` - Get featured products
- `GET /api/v1/products/search` - Search products
- `GET /api/v1/products/categories` - Get product categories
- `POST /api/v1/products` - Create a new product (admin only)
- `PATCH /api/v1/products/:id` - Update a product (admin only)
- `DELETE /api/v1/products/:id` - Delete a product (admin only)
- `PATCH /api/v1/products/:id/inventory` - Update product inventory (admin only)

### Users

- `POST /api/v1/users/register` - Register a new user
- `POST /api/v1/users/login` - Login user
- `GET /api/v1/users/logout` - Logout user
- `GET /api/v1/users/me` - Get current user profile
- `PATCH /api/v1/users/me` - Update user profile
- `PATCH /api/v1/users/update-password` - Update user password
- `DELETE /api/v1/users/me` - Delete user account
- `GET /api/v1/users` - Get all users (admin only)
- `GET /api/v1/users/:id` - Get user by ID (admin only)
- `PATCH /api/v1/users/:id` - Update user (admin only)
- `DELETE /api/v1/users/:id` - Delete user (admin only)

### Orders

- `POST /api/v1/orders` - Create a new order
- `GET /api/v1/orders` - Get all orders for current user
- `GET /api/v1/orders/:id` - Get a single order by ID
- `PATCH /api/v1/orders/:id/status` - Update order status (admin only)
- `GET /api/v1/orders/admin/all` - Get all orders (admin only)
- `GET /api/v1/orders/admin/stats` - Get order statistics (admin only)

### Cart

- `GET /api/v1/cart` - Get or create cart
- `POST /api/v1/cart/items` - Add item to cart
- `PATCH /api/v1/cart/items/:itemId` - Update cart item quantity
- `DELETE /api/v1/cart/items/:itemId` - Remove item from cart
- `DELETE /api/v1/cart` - Clear cart
- `POST /api/v1/cart/discount` - Apply discount code to cart
- `POST /api/v1/cart/shipping` - Set shipping method for cart
- `POST /api/v1/cart/merge` - Merge guest cart with user cart on login

### Reviews

- `POST /api/v1/reviews` - Create a new product review
- `GET /api/v1/reviews/product/:productId` - Get all reviews for a product
- `GET /api/v1/reviews/user/:userId` - Get all reviews by a user
- `PATCH /api/v1/reviews/:id` - Update a review
- `DELETE /api/v1/reviews/:id` - Delete a review
- `PATCH /api/v1/reviews/:id/moderate` - Moderate a review (admin only)
- `POST /api/v1/reviews/:id/vote` - Vote on a review (helpful/not helpful)
- `GET /api/v1/reviews/pending` - Get pending reviews (admin only)

### Wishlists

- `POST /api/v1/wishlists` - Create a new wishlist
- `GET /api/v1/wishlists` - Get all wishlists for current user
- `GET /api/v1/wishlists/:id` - Get a single wishlist by ID
- `PATCH /api/v1/wishlists/:id` - Update a wishlist
- `DELETE /api/v1/wishlists/:id` - Delete a wishlist
- `POST /api/v1/wishlists/:id/items` - Add item to wishlist
- `DELETE /api/v1/wishlists/:id/items/:productId` - Remove item from wishlist
- `GET /api/v1/wishlists/public` - Get all public wishlists
- `GET /api/v1/wishlists/check/:productId` - Check if a product is in any of the user's wishlists

### Search

- `GET /api/v1/search/products` - Advanced product search with Elasticsearch
- `GET /api/v1/search/recommendations/:productId` - Get product recommendations

### Admin

- `GET /api/v1/admin/dashboard` - Get dashboard statistics
- `GET /api/v1/admin/analytics/sales` - Get sales analytics
- `GET /api/v1/admin/analytics/products` - Get product analytics
- `GET /api/v1/admin/analytics/users` - Get user analytics
- `POST /api/v1/search/reindex` - Reindex all products to Elasticsearch (admin only)

## Advanced Features

### Elasticsearch Integration

The CommerceProAPI integrates with Elasticsearch to provide powerful search capabilities:

- **Full-text search**: Search across product names, descriptions, and attributes
- **Fuzzy matching**: Find products even with typos or misspellings
- **Faceted search**: Filter products by categories, brands, price ranges, and more
- **Relevance ranking**: Sort results by relevance to search query
- **Product recommendations**: Get personalized product recommendations based on similar products
- **Search analytics**: Track popular searches and improve search results
- **Autocomplete suggestions**: Get search suggestions as you type

### Customer Features

Enhanced customer experience with personalized features:

#### Wishlists
- Create multiple wishlists
- Add products to wishlists
- Share wishlists with others
- Move items from wishlist to cart
- Get notifications for price drops on wishlist items

#### Reviews
- Write and read product reviews
- Rate products on a 5-star scale
- Upload images with reviews
- Vote on helpful reviews
- Verified purchase badges
- Review moderation system

### Admin Dashboard

Comprehensive admin interface for store management:

- **Dashboard overview**: Key metrics and performance indicators
- **Order management**: View, update, and process orders
- **Product management**: Add, edit, and delete products
- **User management**: Manage customer accounts and permissions
- **Review moderation**: Approve, reject, or edit customer reviews
- **Inventory management**: Track and update product inventory
- **Discount management**: Create and manage promotional codes

### Analytics

Detailed analytics for data-driven decision making:

- **Sales analytics**: Track sales performance over time
- **Product analytics**: Identify top-selling and underperforming products
- **Customer analytics**: Understand customer behavior and preferences
- **Inventory analytics**: Monitor stock levels and forecast inventory needs
- **Search analytics**: Analyze search queries and improve search results
- **Custom reports**: Generate custom reports for specific business needs

## Getting Started

### Prerequisites

- Node.js (v14 or higher)
- MongoDB
- Elasticsearch (optional, for advanced search features)

### Installation

1. Clone the repository
2. Install dependencies
   ```
   npm install
   ```
3. Create a `.env` file in the root directory with the following variables:
   ```
   PORT=5000
   MONGO_URI=your_mongodb_connection_string
   JWT_SECRET=your_jwt_secret
   JWT_LIFETIME=1d
   JWT_COOKIE_EXPIRE=1
   NODE_ENV=development
   ELASTICSEARCH_NODE=http://localhost:9200
   ELASTICSEARCH_USERNAME=elastic
   ELASTICSEARCH_PASSWORD=your_password
   STRIPE_SECRET_KEY=your_stripe_secret_key
   ```
4. Start Elasticsearch (optional, for advanced search features)
   ```
   docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.14.0
   ```
5. Build the TypeScript code
   ```
   npm run build
   ```
6. Start the server
   ```
   npm start
   ```
   
### Development

For development with hot reloading:
```
npm run dev
```

## Enhanced Features

### Product Management

- **Variants**: Support for product variants with different prices, SKUs, and inventory levels
- **Inventory Tracking**: Track inventory levels for products and variants
- **SEO**: Support for SEO metadata including title, description, and keywords
- **Rich Media**: Support for multiple product images and videos
- **Categories & Tags**: Organize products with categories and tags

### User Management

- **Role-Based Access Control**: Different access levels for customers and administrators
- **Profile Management**: Update user profile information and addresses
- **Password Reset**: Secure password reset functionality

### Shopping Cart

- **Guest Cart**: Support for shopping cart for non-logged in users
- **Cart Merging**: Merge guest cart with user cart on login
- **Discount Codes**: Apply discount codes to cart
- **Tax Calculation**: Calculate taxes based on configurable rates
- **Shipping Methods**: Support for multiple shipping methods

### Order Processing

- **Status Tracking**: Track order status with history
- **Inventory Updates**: Automatically update inventory levels on order creation and cancellation
- **Order Notes**: Add notes to orders for special instructions
- **Payment Processing**: Integration with Stripe for secure payment processing

## License

This project is licensed under the MIT License.