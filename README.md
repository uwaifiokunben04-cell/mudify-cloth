# MUDIFY — Global Clothing Marketplace

> Buy and sell clothing across borders. Fashion without limits.

![MUDIFY](https://img.shields.io/badge/MUDIFY-Live-C8956C?style=for-the-badge)
![Supabase](https://img.shields.io/badge/Supabase-Connected-3ECF8E?style=for-the-badge&logo=supabase)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Deployed-222222?style=for-the-badge&logo=github)

---

## What is MUDIFY?

MUDIFY is a global clothing marketplace where anyone can buy and sell fashion items across 180+ countries. Sellers list their items in minutes, buyers shop from around the world, and both sides communicate through a built-in real-time chat system.

---

## Features

- **Shop** — Browse clothing by category, search by name or brand, filter by style
- **Sell** — List items with photos, sizes, pricing, and worldwide shipping options
- **Real-time Chat** — Buyers and sellers communicate directly before and after purchase
- **Dashboard** — Sellers track orders, revenue, listings, and messages in one place
- **Cart & Checkout** — Add items, adjust quantities, and place orders securely
- **Authentication** — Email/password signup and Google OAuth login
- **Multi-currency** — Prices display in USD, EUR, GBP, NGN, GHS, KES, ZAR, JPY, CAD, AUD
- **Fully responsive** — Works on mobile, tablet, and desktop

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, Vanilla JavaScript |
| Database | Supabase (PostgreSQL) |
| Authentication | Supabase Auth |
| File Storage | Supabase Storage |
| Real-time Chat | Supabase Realtime |
| Hosting | GitHub Pages |

---

## Database Tables

The following tables are required in your Supabase project:

| Table | Purpose |
|---|---|
| `users` | Stores buyer and seller profiles |
| `products` | All clothing listings |
| `orders` | Purchase records between buyers and sellers |
| `messages` | Chat messages between users |
| `cart` | Items saved in a user's cart |
| `wishlist` | Items saved to a user's wishlist |
| `reviews` | Ratings and reviews after orders |

Full SQL to create all tables is in the `/sql` folder or see the **Database Setup** section below.

---

## Database Setup

Run the following SQL in your **Supabase SQL Editor** in this exact order:

### 1. Users
```sql
CREATE TABLE users (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  role TEXT CHECK (role IN ('buyer', 'seller', 'both')) DEFAULT 'buyer',
  country TEXT,
  avatar_url TEXT,
  bio TEXT,
  rating DECIMAL(2,1) DEFAULT 0.0,
  total_reviews INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 2. Products
```sql
CREATE TABLE products (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  seller_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  price DECIMAL(10,2) NOT NULL,
  currency TEXT DEFAULT 'USD',
  category TEXT CHECK (category IN ('Women','Men','Kids','Vintage','Shoes','Streetwear','Accessories')),
  condition TEXT CHECK (condition IN ('New with tags','Like new','Good','Fair')),
  brand TEXT,
  color TEXT,
  sizes TEXT[],
  images TEXT[],
  ships_from TEXT,
  shipping_fee DECIMAL(10,2) DEFAULT 0,
  ships_worldwide BOOLEAN DEFAULT TRUE,
  is_available BOOLEAN DEFAULT TRUE,
  views INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 3. Orders
```sql
CREATE TABLE orders (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  product_id UUID REFERENCES products(id),
  buyer_id UUID REFERENCES users(id),
  seller_id UUID REFERENCES users(id),
  quantity INT DEFAULT 1,
  amount DECIMAL(10,2) NOT NULL,
  currency TEXT DEFAULT 'USD',
  status TEXT CHECK (status IN ('pending','paid','shipped','delivered','cancelled','refunded')) DEFAULT 'pending',
  shipping_address JSONB,
  tracking_number TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4. Messages
```sql
CREATE TABLE messages (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  sender_id UUID REFERENCES users(id),
  receiver_id UUID REFERENCES users(id),
  product_id UUID REFERENCES products(id),
  content TEXT NOT NULL,
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 5. Cart
```sql
CREATE TABLE cart (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  product_id UUID REFERENCES products(id) ON DELETE CASCADE,
  quantity INT DEFAULT 1,
  added_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, product_id)
);
```

### 6. Wishlist
```sql
CREATE TABLE wishlist (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  product_id UUID REFERENCES products(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, product_id)
);
```

### 7. Reviews
```sql
CREATE TABLE reviews (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  order_id UUID REFERENCES orders(id),
  reviewer_id UUID REFERENCES users(id),
  seller_id UUID REFERENCES users(id),
  rating INT CHECK (rating BETWEEN 1 AND 5),
  comment TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 8. Row Level Security (run last)
```sql
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE wishlist ENABLE ROW LEVEL SECURITY;
ALTER TABLE cart ENABLE ROW LEVEL SECURITY;
ALTER TABLE reviews ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public profiles" ON users FOR SELECT USING (true);
CREATE POLICY "Edit own profile" ON users FOR UPDATE USING (auth.uid() = id);

CREATE POLICY "View all products" ON products FOR SELECT USING (true);
CREATE POLICY "Sellers can insert" ON products FOR INSERT WITH CHECK (auth.uid() = seller_id);
CREATE POLICY "Sellers can update" ON products FOR UPDATE USING (auth.uid() = seller_id);
CREATE POLICY "Sellers can delete" ON products FOR DELETE USING (auth.uid() = seller_id);

CREATE POLICY "Order access" ON orders FOR SELECT USING (auth.uid() = buyer_id OR auth.uid() = seller_id);
CREATE POLICY "Buyers can create orders" ON orders FOR INSERT WITH CHECK (auth.uid() = buyer_id);
CREATE POLICY "Sellers can update order status" ON orders FOR UPDATE USING (auth.uid() = seller_id);

CREATE POLICY "Message access" ON messages FOR SELECT USING (auth.uid() = sender_id OR auth.uid() = receiver_id);
CREATE POLICY "Send messages" ON messages FOR INSERT WITH CHECK (auth.uid() = sender_id);

CREATE POLICY "Own wishlist" ON wishlist USING (auth.uid() = user_id);
CREATE POLICY "Own cart" ON cart USING (auth.uid() = user_id);

CREATE POLICY "View reviews" ON reviews FOR SELECT USING (true);
CREATE POLICY "Write reviews" ON reviews FOR INSERT WITH CHECK (auth.uid() = reviewer_id);
```

---

## Supabase Storage Setup

1. Go to **Supabase → Storage**
2. Click **New bucket**
3. Name it exactly: `product-images`
4. Set it to **Public**
5. Click **Create bucket**

This bucket stores all product photos uploaded by sellers.

---

## Deployment (GitHub Pages)

1. Upload `index.html` to this repository
2. Go to **Settings → Pages**
3. Under **Source**, select **main** branch and **/ (root)** folder
4. Click **Save**
5. Your site will be live at:

```
https://yourusername.github.io/mudify
```

---

## Supabase URL Configuration

After deploying, add your live URL to Supabase so that authentication works correctly:

1. Go to **Supabase → Authentication → URL Configuration**
2. Set **Site URL** to:
```
https://yourusername.github.io
```
3. Add to **Redirect URLs**:
```
https://yourusername.github.io/mudify
```

---

## Environment Variables

The Supabase connection is configured directly in `index.html`:

```javascript
const SUPABASE_URL = 'https://xhfwslzubxfydfsxjihl.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key-here';
```

> The anon key is safe to use in frontend code. Never paste your `service_role` key in any public file.

---

## Project Structure

```
mudify/
├── index.html        # Complete single-file application
└── README.md         # This file
```

---

## Roadmap

- [ ] Payment integration (Stripe / Paystack / Flutterwave)
- [ ] Order tracking with map
- [ ] Seller verification badges
- [ ] Push notifications for new messages and orders
- [ ] Mobile app (React Native)
- [ ] AI-powered style recommendations
- [ ] Bulk listing tools for power sellers
- [ ] Discount codes and promotions

---

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

---

## License

MIT License — free to use, modify, and distribute.

---

## Contact

Built with ❤️ by the MUDIFY team.  
For support or business enquiries, open an issue on this repository.

---

*MUDIFY — Fashion without borders.*
