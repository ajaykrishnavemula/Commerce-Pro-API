# CommerceProAPI - Beginner's Learning Guide

## 📚 What is CommerceProAPI?

CommerceProAPI is a **complete e-commerce backend system**. Think of it as the engine behind online stores like Amazon or Shopify - it handles products, shopping carts, orders, payments, and everything needed to run an online store.

---

## 🎯 What Does This Project Do?

Imagine you're building an online store. You need:
- Products that customers can browse
- Shopping cart to collect items
- Checkout process to place orders
- User accounts to track purchases
- Reviews and ratings for products
- Admin dashboard to manage everything

**This project does ALL of that!**

---

## 🏗️ Project Structure (How Files Are Organized)

```
CommerceProAPI/
├── src/                          # All source code lives here
│   ├── controllers/              # Handle requests (the "brain")
│   │   ├── userController.ts    # User management (12 functions)
│   │   ├── productController.ts # Product management (11 functions)
│   │   ├── cartController.ts    # Shopping cart (8 functions)
│   │   ├── orderController.ts   # Order processing (6 functions)
│   │   ├── reviewController.ts  # Product reviews (8 functions)
│   │   ├── wishlistController.ts# Wishlists (9 functions)
│   │   ├── searchController.ts  # Search & recommendations (3 functions)
│   │   └── adminController.ts   # Admin analytics (4 functions)
│   ├── models/                   # Database structure
│   │   ├── User.ts              # Customer accounts
│   │   ├── Product.ts           # Product catalog
│   │   ├── Cart.ts              # Shopping carts
│   │   ├── Order.ts             # Customer orders
│   │   ├── Review.ts            # Product reviews
│   │   └── Wishlist.ts          # Saved items
│   ├── routes/                   # URL paths
│   │   ├── users.ts             # /api/v1/users/*
│   │   ├── products.ts          # /api/v1/products/*
│   │   ├── cart.ts              # /api/v1/cart/*
│   │   ├── orders.ts            # /api/v1/orders/*
│   │   ├── reviews.ts           # /api/v1/reviews/*
│   │   ├── wishlists.ts         # /api/v1/wishlists/*
│   │   ├── search.ts            # /api/v1/search/*
│   │   └── admin.ts             # /api/v1/admin/*
│   ├── middleware/               # Code that runs before controllers
│   │   ├── auth.ts              # Check if user is logged in
│   │   └── rate-limiter.ts      # Prevent spam
│   ├── services/                 # Business logic
│   ├── utils/                    # Helper functions
│   ├── config/                   # Settings
│   └── app.ts                    # Main application setup
├── .env                          # Secret keys
├── package.json                  # Project dependencies
└── tsconfig.json                 # TypeScript settings
```

---

## 🔑 Key E-Commerce Concepts Explained

### 1. **Product Catalog**

Products are items you sell:
```typescript
{
  name: "iPhone 15 Pro",
  description: "Latest Apple smartphone",
  price: 999.99,
  category: "Electronics",
  stock: 50,
  images: ["image1.jpg", "image2.jpg"],
  rating: 4.5,
  reviews: 120
}
```

### 2. **Shopping Cart**

Temporary storage before checkout:
```typescript
{
  userId: "user123",
  items: [
    {
      productId: "iphone15",
      quantity: 1,
      price: 999.99
    },
    {
      productId: "case",
      quantity: 2,
      price: 29.99
    }
  ],
  total: 1059.97
}
```

### 3. **Order Processing**

When customer completes purchase:
```typescript
{
  orderId: "ORD-2024-001",
  userId: "user123",
  items: [...],
  total: 1059.97,
  status: "pending",      // → processing → shipped → delivered
  shippingAddress: {...},
  paymentMethod: "card",
  createdAt: Date
}
```

### 4. **Inventory Management**

Track product availability:
```typescript
// When order is placed:
1. Check if product in stock
2. Reduce stock count
3. If stock = 0, mark as "out of stock"
4. Prevent overselling
```

### 5. **Reviews & Ratings**

Customer feedback:
```typescript
{
  productId: "iphone15",
  userId: "user123",
  rating: 5,              // 1-5 stars
  title: "Amazing phone!",
  comment: "Best purchase ever",
  helpful: 15,            // How many found it helpful
  verified: true          // Purchased the product
}
```

---

## 📖 How the Code Works (Step by Step)

### Example 1: Browsing Products

**What happens when someone visits the store?**

```typescript
// 1. User requests products
GET /api/v1/products?category=Electronics&sort=price&page=1

// 2. Code in productController.ts runs:
export const getAllProducts = async (req: Request, res: Response) => {
  // Get filters from URL
  const { category, minPrice, maxPrice, sort, page = 1 } = req.query;
  
  // Build database query
  const filters: any = {};
  if (category) filters.category = category;
  if (minPrice || maxPrice) {
    filters.price = {};
    if (minPrice) filters.price.$gte = Number(minPrice);
    if (maxPrice) filters.price.$lte = Number(maxPrice);
  }
  
  // Calculate pagination
  const limit = 20;
  const skip = (Number(page) - 1) * limit;
  
  // Get products from database
  const products = await Product.find(filters)
    .sort(sort === 'price' ? { price: 1 } : { createdAt: -1 })
    .skip(skip)
    .limit(limit);
  
  // Count total products
  const total = await Product.countDocuments(filters);
  
  // Return results
  res.json({
    success: true,
    products,
    pagination: {
      page: Number(page),
      limit,
      total,
      pages: Math.ceil(total / limit)
    }
  });
};
```

**Flow Diagram**:
```
User → Browse Products → Apply Filters → Sort Results → 
Paginate → Return Products
```

### Example 2: Adding to Cart

**What happens when someone adds item to cart?**

```typescript
// 1. User clicks "Add to Cart"
POST /api/v1/cart/items
{
  "productId": "iphone15",
  "quantity": 1
}

// 2. Code in cartController.ts runs:
export const addItemToCart = async (req: any, res: any) => {
  const { productId, quantity } = req.body;
  const userId = req.user.userId;
  
  // Get product details
  const product = await Product.findById(productId);
  if (!product) {
    throw new NotFoundError('Product not found');
  }
  
  // Check stock availability
  if (product.stock < quantity) {
    throw new BadRequestError('Insufficient stock');
  }
  
  // Find or create cart
  let cart = await Cart.findOne({ userId });
  if (!cart) {
    cart = await Cart.create({ userId, items: [] });
  }
  
  // Check if item already in cart
  const existingItem = cart.items.find(
    item => item.productId.toString() === productId
  );
  
  if (existingItem) {
    // Update quantity
    existingItem.quantity += quantity;
  } else {
    // Add new item
    cart.items.push({
      productId,
      quantity,
      price: product.price
    });
  }
  
  // Calculate total
  cart.total = cart.items.reduce(
    (sum, item) => sum + (item.price * item.quantity),
    0
  );
  
  await cart.save();
  
  res.json({
    success: true,
    cart
  });
};
```

**Flow Diagram**:
```
User → Add to Cart → Check Product → Check Stock → 
Find/Create Cart → Update Cart → Calculate Total → Save
```

### Example 3: Placing an Order

**What happens when someone checks out?**

```typescript
// 1. User clicks "Place Order"
POST /api/v1/orders
{
  "shippingAddress": {
    "street": "123 Main St",
    "city": "New York",
    "zipCode": "10001"
  },
  "paymentMethod": "card"
}

// 2. Code in orderController.ts runs:
export const createOrder = async (req: Request, res: Response) => {
  const userId = req.user.userId;
  const { shippingAddress, paymentMethod } = req.body;
  
  // Get user's cart
  const cart = await Cart.findOne({ userId }).populate('items.productId');
  if (!cart || cart.items.length === 0) {
    throw new BadRequestError('Cart is empty');
  }
  
  // Verify stock for all items
  for (const item of cart.items) {
    const product = await Product.findById(item.productId);
    if (product.stock < item.quantity) {
      throw new BadRequestError(`${product.name} is out of stock`);
    }
  }
  
  // Create order
  const order = await Order.create({
    userId,
    items: cart.items.map(item => ({
      productId: item.productId,
      quantity: item.quantity,
      price: item.price
    })),
    total: cart.total,
    shippingAddress,
    paymentMethod,
    status: 'pending'
  });
  
  // Update inventory
  for (const item of cart.items) {
    await Product.findByIdAndUpdate(item.productId, {
      $inc: { stock: -item.quantity }
    });
  }
  
  // Clear cart
  await Cart.findByIdAndUpdate(cart._id, {
    items: [],
    total: 0
  });
  
  // Send confirmation email (if implemented)
  // await sendOrderConfirmation(userId, order);
  
  res.status(201).json({
    success: true,
    order
  });
};
```

**Flow Diagram**:
```
User → Checkout → Get Cart → Verify Stock → Create Order → 
Update Inventory → Clear Cart → Send Confirmation
```

---

## 🎨 Features Implemented (What You Can Do)

### ✅ User Management (12 Functions)
1. **Register** - Create account
2. **Login** - Sign in
3. **Logout** - Sign out
4. **Get Profile** - View account
5. **Update Profile** - Change details
6. **Update Password** - Change password
7. **Delete Account** - Remove account
8. **Get All Users** - Admin only
9. **Get User by ID** - Admin only
10. **Update User** - Admin only
11. **Delete User** - Admin only
12. **JWT Authentication** - Secure access

### ✅ Product Management (11 Functions)
1. **Create Product** - Add new product
2. **Get All Products** - Browse catalog
3. **Get Product by ID** - View details
4. **Update Product** - Modify product
5. **Delete Product** - Remove product
6. **Get Featured Products** - Homepage items
7. **Search Products** - Find products
8. **Get Categories** - List categories
9. **Add Review** - Rate product
10. **Update Inventory** - Stock management
11. **Advanced Filtering** - Price, category, rating

### ✅ Shopping Cart (8 Functions)
1. **Get Cart** - View cart
2. **Add Item** - Add to cart
3. **Update Item** - Change quantity
4. **Remove Item** - Delete from cart
5. **Clear Cart** - Empty cart
6. **Apply Discount** - Use coupon code
7. **Set Shipping** - Choose shipping method
8. **Merge Cart** - Combine guest + user cart

### ✅ Order Management (6 Functions)
1. **Create Order** - Place order
2. **Get User Orders** - Order history
3. **Get Order by ID** - Order details
4. **Update Status** - Track order (admin)
5. **Get All Orders** - All orders (admin)
6. **Get Statistics** - Order analytics (admin)

### ✅ Review System (8 Functions)
1. **Create Review** - Write review
2. **Get Product Reviews** - Read reviews
3. **Get User Reviews** - My reviews
4. **Update Review** - Edit review
5. **Delete Review** - Remove review
6. **Moderate Review** - Admin approval
7. **Vote on Review** - Helpful/not helpful
8. **Get Pending Reviews** - Admin moderation

### ✅ Wishlist System (9 Functions)
1. **Create Wishlist** - New wishlist
2. **Get Wishlists** - My wishlists
3. **Get Wishlist** - View wishlist
4. **Update Wishlist** - Rename wishlist
5. **Delete Wishlist** - Remove wishlist
6. **Add Item** - Save product
7. **Remove Item** - Unsave product
8. **Get Public Wishlists** - Browse wishlists
9. **Check Product** - Is it saved?

### ✅ Search & Discovery (3 Functions)
1. **Search Products** - Full-text search
2. **Get Recommendations** - Suggested products
3. **Reindex Products** - Update search (admin)

### ✅ Admin Dashboard (4 Functions)
1. **Dashboard Stats** - Overview metrics
2. **Sales Analytics** - Revenue tracking
3. **Product Analytics** - Top products
4. **User Analytics** - User growth

---

## 📊 Database Models Explained

### Product Model
```typescript
{
  name: "iPhone 15 Pro",
  slug: "iphone-15-pro",
  description: "Latest Apple smartphone with A17 chip",
  price: 999.99,
  compareAtPrice: 1099.99,    // Original price (for discounts)
  category: "Electronics",
  subcategory: "Smartphones",
  brand: "Apple",
  sku: "IPH15PRO-256-BLK",    // Stock Keeping Unit
  stock: 50,
  lowStockThreshold: 10,
  images: [
    "https://cdn.example.com/iphone-1.jpg",
    "https://cdn.example.com/iphone-2.jpg"
  ],
  specifications: {
    "Screen Size": "6.1 inches",
    "Storage": "256GB",
    "Color": "Black"
  },
  tags: ["smartphone", "5g", "apple"],
  rating: 4.5,
  reviewCount: 120,
  isFeatured: true,
  isActive: true,
  createdAt: Date,
  updatedAt: Date
}
```

### Cart Model
```typescript
{
  userId: ObjectId,
  items: [
    {
      productId: ObjectId,
      quantity: 1,
      price: 999.99
    }
  ],
  subtotal: 999.99,
  discount: 50.00,              // From coupon code
  shippingCost: 10.00,
  tax: 89.99,
  total: 1049.98,
  discountCode: "SAVE50",
  shippingMethod: "express",
  expiresAt: Date               // Cart expires after 7 days
}
```

### Order Model
```typescript
{
  orderNumber: "ORD-2024-001",
  userId: ObjectId,
  items: [
    {
      productId: ObjectId,
      name: "iPhone 15 Pro",
      quantity: 1,
      price: 999.99,
      total: 999.99
    }
  ],
  subtotal: 999.99,
  discount: 50.00,
  shippingCost: 10.00,
  tax: 89.99,
  total: 1049.98,
  status: "pending",            // pending → processing → shipped → delivered
  paymentStatus: "paid",        // pending → paid → failed → refunded
  paymentMethod: "card",
  shippingAddress: {
    name: "John Doe",
    street: "123 Main St",
    city: "New York",
    state: "NY",
    zipCode: "10001",
    country: "USA",
    phone: "+1234567890"
  },
  trackingNumber: "TRACK123",
  notes: "Leave at door",
  createdAt: Date,
  updatedAt: Date
}
```

### Review Model
```typescript
{
  productId: ObjectId,
  userId: ObjectId,
  rating: 5,                    // 1-5 stars
  title: "Amazing product!",
  comment: "Best purchase I've made this year",
  images: ["review-photo.jpg"],
  verified: true,               // Purchased the product
  helpful: 15,                  // Helpful votes
  notHelpful: 2,                // Not helpful votes
  status: "approved",           // pending → approved → rejected
  moderatedBy: ObjectId,        // Admin who approved
  moderatedAt: Date,
  createdAt: Date,
  updatedAt: Date
}
```

---

## 🔐 Security Features

### 1. **Authentication**
```typescript
// JWT tokens for secure access
const token = jwt.sign(
  { userId: user._id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '7d' }
);
```

### 2. **Authorization**
```typescript
// Role-based access control
- Users: Can browse, buy, review
- Admins: Can manage products, orders, users
```

### 3. **Input Validation**
```typescript
// All inputs are validated
- Email format
- Price must be positive
- Quantity must be integer
- Stock cannot be negative
```

### 4. **Rate Limiting**
```typescript
// Prevent abuse
- Max 100 requests per 15 minutes
- Max 5 login attempts
- Max 10 cart updates per minute
```

---

## 🚀 How to Use This API

### Setup

```bash
# 1. Install dependencies
npm install

# 2. Create .env file
PORT=5000
MONGO_URI=mongodb://localhost:27017/commercepro
JWT_SECRET=your-secret-key
STRIPE_SECRET_KEY=your-stripe-key

# 3. Start server
npm run dev
```

### API Examples

#### Browse Products
```bash
curl http://localhost:5000/api/v1/products?category=Electronics&sort=price
```

#### Add to Cart
```bash
curl -X POST http://localhost:5000/api/v1/cart/items \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": "product123",
    "quantity": 1
  }'
```

#### Place Order
```bash
curl -X POST http://localhost:5000/api/v1/orders \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "shippingAddress": {
      "street": "123 Main St",
      "city": "New York",
      "zipCode": "10001"
    },
    "paymentMethod": "card"
  }'
```

---

## 🎓 Learning Path

### Beginner Level
1. ✅ Understand e-commerce basics
2. ✅ Learn about REST APIs
3. ✅ Understand databases
4. ✅ Learn about authentication

### Intermediate Level
1. ✅ Understand shopping cart logic
2. ✅ Learn order processing
3. ✅ Understand inventory management
4. ✅ Learn about payment gateways

### Advanced Level
1. ✅ Implement search algorithms
2. ✅ Build recommendation systems
3. ✅ Add analytics dashboards
4. ✅ Optimize for scale

---

## 🐛 Common Issues & Solutions

### Issue 1: "Product out of stock"
**Problem**: Trying to order unavailable product
**Solution**: Check stock before adding to cart

### Issue 2: "Cart is empty"
**Problem**: Trying to checkout with no items
**Solution**: Add items to cart first

### Issue 3: "Invalid discount code"
**Problem**: Coupon code doesn't exist or expired
**Solution**: Check code spelling and expiration

---

## 📚 Further Learning

### Concepts to Study
1. **E-commerce Fundamentals** - How online stores work
2. **Payment Gateways** - Stripe, PayPal integration
3. **Inventory Management** - Stock tracking
4. **Order Fulfillment** - Shipping and delivery
5. **Search Algorithms** - Product discovery
6. **Recommendation Systems** - Personalization

### Next Steps
1. Add payment integration (Stripe)
2. Implement email notifications
3. Add shipping rate calculation
4. Build admin dashboard UI
5. Add product variants (sizes, colors)

---

**Remember**: This is a production-ready e-commerce system. Study each feature, understand the flow, and experiment with the code!

**Happy Learning! 🛒**