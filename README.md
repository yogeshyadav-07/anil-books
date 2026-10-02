# 📚 KitabGhar — Online Book Store for an Independent Author

A simple, premium online book store built with **plain HTML, CSS and JavaScript** and **Supabase** as the backend. No frameworks, no build step. Open it in a browser and it already looks like a finished bookstore, with demo books, a hero slider, reviews and a footer.

---

## ✨ Features

**For readers**
- 6-image hero slider (arrows, dots, swipe, keyboard, auto-slide, pause on hover)
- Free and Paid books, Coming Soon section, instant search and category filters
- Book detail popup with about text, rating, price, release date and reviews
- Public reviews (name, 1–5 stars, comment)
- Cart saved in `localStorage` (add, increase, decrease, remove)
- Secure checkout with UPI payment link
- **My Orders** with Check Payment and secure download after payment
- English / हिंदी toggle and Light / Dark theme (both remembered)
- Contact form and a footer visitor counter (counted once per browser session)

**For the admin** (menu: Dashboard · Add Books · Coming Books · Payment · Contact · Settings · Logout)
- Add, edit and delete books with cover, full PDF and preview PDF upload
- Add, edit and delete coming-soon books
- Verify payments with **Mark Paid** or **Cancel**
- Read contact messages and mark them Unread / Read / Resolved
- Edit site name, author, UPI ID, hero text, 6 hero images and social links without touching code

---

## 🧱 Tech Stack

| Part | Technology |
|---|---|
| Frontend | HTML, CSS, Vanilla JavaScript |
| Backend | Supabase (Postgres, Auth, Storage, RPC, Edge Function) |
| Fonts | Playfair Display, Inter |

---

## 📁 Project Structure

```
kitabghar/
├── index.html                      # Page structure (HTML)
├── style.css                       # All styles + responsive rules
├── script.js                       # All JavaScript (Supabase keys are at the top)
├── supabase.sql                    # Tables, functions, RLS, storage, demo data
├── ADMIN_GUIDE.md                  # Step-by-step admin instructions
├── README.md
└── supabase/
    └── functions/
        └── get-download/
            └── index.ts            # Secure paid-book download (Edge Function)
```

**Optional split version:** you can move the `<style>` code into `style.css` and the `<script>` code into `script.js`, then link them:

```html
<link rel="stylesheet" href="style.css">
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.45.4/dist/umd/supabase.min.js"></script>
<script src="script.js"></script>
```
The Supabase CDN line must come **before** `script.js`.

---

## 🚀 Setup

1. **Create a Supabase project** at [supabase.com](https://supabase.com).
2. **Run the database setup:** open *SQL Editor*, paste all of `supabase.sql`, and click *Run*. It is safe to run again.
3. **Add your keys** in `index.html` (or `script.js`):
   ```js
   const SUPABASE_URL = "YOUR_SUPABASE_URL";
   const SUPABASE_ANON_KEY = "YOUR_SUPABASE_ANON_KEY";
   ```
   Find them in *Project Settings → API*. Use the **anon** key only.
4. **Create an admin user** in *Authentication → Users → Add user*, then run:
   ```sql
   update profiles set role = 'admin' where email = 'YOU@EMAIL.COM';
   ```
5. **Deploy the download function** (needed for paid downloads):
   ```bash
   supabase functions deploy get-download
   ```
6. **Open `index.html`.** Use VS Code Live Server or any static host (Netlify, Vercel, GitHub Pages).
7. **Log in as admin** with the faint **⚙ Admin** link in the footer, then upload PDFs in *Add Books*.

> If Supabase is not connected yet, the site shows built-in demo content so it never looks empty. Ordering, reviews, contact messages and admin need Supabase.

---

## 💳 Payment Flow

1. Customer places an order. The server calculates the price from the database through the `place_order` RPC and returns the order number, amount and a secret token.
2. The customer taps **Pay Now**, which opens a `upi://pay` link with the exact amount and order number.
3. The order stays **Pending**. Opening the UPI app or returning to the site never marks it as paid.
4. The admin checks the bank/UPI app, then clicks **Mark Paid** in *Admin → Payment*.
5. The customer opens **My Orders → Check Payment** and sees **Payment Successful → Download Book**.

Payment verification is separate from the rest of the site. To use Razorpay or Cashfree later, replace the UPI link in `showPay()` and let the gateway webhook update the order on the server. See the comment in the code.

---

## 🔐 Security

- **Row Level Security** is on for every table. Admin actions depend on `profiles.role = 'admin'` through `is_admin()`.
- Customers can never insert orders directly. They use `place_order`, and read status only through `order_status(order_id, token)`.
- Prices are never trusted from the browser.
- `paid-books` is a **private** bucket. The Edge Function verifies the order token, that the order is **Paid**, and that the book belongs to the order, then returns a signed URL valid for 60 seconds.
- The `service-role` key is used only inside the Edge Function and never in `index.html`.
- Users cannot promote themselves to admin.

---

## 🗄️ Storage Buckets

| Bucket | Access | Used for |
|---|---|---|
| `book-covers` | Public | Book cover images |
| `hero-images` | Public | Uploaded hero images |
| `free-books` | Public | Free book PDFs |
| `book-previews` | Public | 10-page previews of paid books |
| `paid-books` | **Private** | Full PDFs of paid books |

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---|---|
| Site shows demo data only | Check `SUPABASE_URL` and `SUPABASE_ANON_KEY`, and confirm `supabase.sql` ran (`select count(*) from books;`) |
| `invalid input syntax for type uuid: "d4"` | The cart holds old demo items. Run `localStorage.removeItem("kg_cart")` in the console and refresh |
| `gen_random_bytes does not exist` | Use the latest `supabase.sql` (token now uses `gen_random_uuid()`) |
| "You are not authorized" | Set `profiles.role = 'admin'` for your email |
| "File upload failed" | Run the full SQL first, and make sure you are logged in as admin |
| Paid download fails | The order must be **Paid** and `get-download` must be deployed |

---

## 🗺️ Roadmap

- Razorpay / Cashfree gateway with webhook
- Physical/printed book orders and shipping status
- Email notification when an order is verified

---

## 📄 License

Free to use for personal and learning projects. Add your own license before commercial use.
