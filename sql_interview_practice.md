# SQL Interview Practice (Advanced)

This sheet is focused on hard SQL interview questions commonly asked in live-coding rounds for Staff Data Analyst roles.
Each question includes a **problem statement**, **schema**, and a **reference solution**.

---

## 1) 7-day rolling retention by cohort
**Problem:** For each signup date, compute the percentage of users who returned within the next 7 days (inclusive).  
**Schema:**
```sql
users(user_id, signup_date)
user_events(user_id, event_date)
```
**Solution:**
```sql
WITH cohort AS (
  SELECT user_id, signup_date
  FROM users
), returns AS (
  SELECT
    c.signup_date,
    c.user_id,
    MAX(CASE
      WHEN e.event_date BETWEEN c.signup_date AND c.signup_date + INTERVAL '7 day'
      THEN 1 ELSE 0 END
    ) AS returned_7d
  FROM cohort c
  LEFT JOIN user_events e
    ON e.user_id = c.user_id
  GROUP BY c.signup_date, c.user_id
)
SELECT
  signup_date,
  ROUND(100.0 * AVG(returned_7d), 2) AS pct_returned_7d
FROM returns
GROUP BY signup_date
ORDER BY signup_date;
```

---

## 2) Median transaction amount per day (no median function)
**Problem:** Compute the median `amount` per day without using a built-in `median()`.
**Schema:**
```sql
transactions(txn_id, user_id, txn_date, amount)
```
**Solution:**
```sql
WITH ranked AS (
  SELECT
    txn_date,
    amount,
    ROW_NUMBER() OVER (PARTITION BY txn_date ORDER BY amount) AS rn,
    COUNT(*) OVER (PARTITION BY txn_date) AS cnt
  FROM transactions
)
SELECT
  txn_date,
  AVG(amount) AS median_amount
FROM ranked
WHERE rn IN ((cnt + 1) / 2, (cnt + 2) / 2)
GROUP BY txn_date
ORDER BY txn_date;
```

---

## 3) Consecutive active days (streaks)
**Problem:** Find users with an activity streak of **7+ consecutive days**.
**Schema:**
```sql
user_events(user_id, event_date)
```
**Solution:**
```sql
WITH daily AS (
  SELECT DISTINCT user_id, event_date
  FROM user_events
), groups AS (
  SELECT
    user_id,
    event_date,
    event_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_date)) * INTERVAL '1 day' AS grp
  FROM daily
), streaks AS (
  SELECT
    user_id,
    MIN(event_date) AS streak_start,
    MAX(event_date) AS streak_end,
    COUNT(*) AS streak_len
  FROM groups
  GROUP BY user_id, grp
)
SELECT *
FROM streaks
WHERE streak_len >= 7
ORDER BY streak_len DESC;
```

---

## 4) Top 2 products by revenue per category
**Problem:** Return the top 2 products by revenue within each category.
**Schema:**
```sql
order_items(order_id, product_id, quantity, unit_price)
products(product_id, category)
```
**Solution:**
```sql
WITH revenue AS (
  SELECT
    p.category,
    oi.product_id,
    SUM(oi.quantity * oi.unit_price) AS total_revenue
  FROM order_items oi
  JOIN products p ON p.product_id = oi.product_id
  GROUP BY p.category, oi.product_id
), ranked AS (
  SELECT
    *,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY total_revenue DESC) AS rnk
  FROM revenue
)
SELECT category, product_id, total_revenue
FROM ranked
WHERE rnk <= 2
ORDER BY category, total_revenue DESC;
```

---

## 5) Users whose first purchase was also their largest
**Problem:** Find users where their **first purchase amount** equals their **maximum purchase amount**.
**Schema:**
```sql
transactions(txn_id, user_id, txn_date, amount)
```
**Solution:**
```sql
WITH ordered AS (
  SELECT
    user_id,
    amount,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY txn_date, txn_id) AS rn,
    MAX(amount) OVER (PARTITION BY user_id) AS max_amt
  FROM transactions
)
SELECT user_id
FROM ordered
WHERE rn = 1 AND amount = max_amt;
```

---

## 6) Sessionize events (30-minute inactivity)
**Problem:** Assign a session_id when the gap between consecutive events is > 30 minutes.
**Schema:**
```sql
events(user_id, event_time)
```
**Solution:**
```sql
WITH ordered AS (
  SELECT
    user_id,
    event_time,
    LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time) AS prev_time
  FROM events
), flags AS (
  SELECT
    *,
    CASE
      WHEN prev_time IS NULL OR event_time - prev_time > INTERVAL '30 minute' THEN 1
      ELSE 0
    END AS new_session
  FROM ordered
), sessions AS (
  SELECT
    user_id,
    event_time,
    SUM(new_session) OVER (PARTITION BY user_id ORDER BY event_time) AS session_id
  FROM flags
)
SELECT *
FROM sessions
ORDER BY user_id, event_time;
```

---

## 7) Find products never re-ordered
**Problem:** Return products that were ordered exactly once overall.
**Schema:**
```sql
order_items(order_id, product_id)
```
**Solution:**
```sql
SELECT product_id
FROM order_items
GROUP BY product_id
HAVING COUNT(DISTINCT order_id) = 1
ORDER BY product_id;
```

---

## 8) First and last touch attribution (same day)
**Problem:** For each user and day, return the first and last marketing channel touched.
**Schema:**
```sql
marketing_touches(user_id, touch_time, channel)
```
**Solution:**
```sql
WITH ranked AS (
  SELECT
    user_id,
    DATE(touch_time) AS touch_date,
    channel,
    ROW_NUMBER() OVER (PARTITION BY user_id, DATE(touch_time) ORDER BY touch_time) AS rn_first,
    ROW_NUMBER() OVER (PARTITION BY user_id, DATE(touch_time) ORDER BY touch_time DESC) AS rn_last
  FROM marketing_touches
)
SELECT
  user_id,
  touch_date,
  MAX(CASE WHEN rn_first = 1 THEN channel END) AS first_touch,
  MAX(CASE WHEN rn_last = 1 THEN channel END) AS last_touch
FROM ranked
GROUP BY user_id, touch_date
ORDER BY user_id, touch_date;
```

---

## 9) Detect overlapping bookings
**Problem:** Find overlapping bookings for the same room.
**Schema:**
```sql
bookings(booking_id, room_id, start_time, end_time)
```
**Solution:**
```sql
SELECT b1.booking_id AS booking_a, b2.booking_id AS booking_b, b1.room_id
FROM bookings b1
JOIN bookings b2
  ON b1.room_id = b2.room_id
 AND b1.booking_id < b2.booking_id
 AND b1.start_time < b2.end_time
 AND b2.start_time < b1.end_time
ORDER BY b1.room_id, b1.booking_id, b2.booking_id;
```

---

## 10) Pareto: users driving 80% of revenue
**Problem:** Return users whose cumulative revenue accounts for the top 80% of total revenue.
**Schema:**
```sql
transactions(txn_id, user_id, amount)
```
**Solution:**
```sql
WITH revenue AS (
  SELECT user_id, SUM(amount) AS total_revenue
  FROM transactions
  GROUP BY user_id
), ordered AS (
  SELECT
    user_id,
    total_revenue,
    SUM(total_revenue) OVER (ORDER BY total_revenue DESC) AS running_revenue,
    SUM(total_revenue) OVER () AS all_revenue
  FROM revenue
)
SELECT user_id, total_revenue
FROM ordered
WHERE running_revenue <= 0.8 * all_revenue
ORDER BY total_revenue DESC;
```

---

## 11) Daily active users and week-over-week change
**Problem:** Compute DAU and WoW % change.
**Schema:**
```sql
user_events(user_id, event_date)
```
**Solution:**
```sql
WITH dau AS (
  SELECT event_date, COUNT(DISTINCT user_id) AS dau
  FROM user_events
  GROUP BY event_date
), lagged AS (
  SELECT
    event_date,
    dau,
    LAG(dau, 7) OVER (ORDER BY event_date) AS dau_7d_ago
  FROM dau
)
SELECT
  event_date,
  dau,
  ROUND(100.0 * (dau - dau_7d_ago) / NULLIF(dau_7d_ago, 0), 2) AS wow_pct_change
FROM lagged
ORDER BY event_date;
```

---

## 12) Identify first-time repeat purchasers (second purchase date)
**Problem:** Return each user and their **second purchase date** (if any).
**Schema:**
```sql
transactions(txn_id, user_id, txn_date)
```
**Solution:**
```sql
WITH ranked AS (
  SELECT
    user_id,
    txn_date,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY txn_date, txn_id) AS rn
  FROM transactions
)
SELECT user_id, txn_date AS second_purchase_date
FROM ranked
WHERE rn = 2
ORDER BY user_id;
```

---

## 13) Fill missing dates (calendar table)
**Problem:** Create a daily spine and left join metrics to fill missing dates.
**Schema:**
```sql
metrics(metric_date, value)
```
**Solution (Postgres):**
```sql
WITH calendar AS (
  SELECT DATE '2024-01-01' + (n * INTERVAL '1 day') AS dt
  FROM generate_series(0, 31) AS t(n)
)
SELECT
  c.dt,
  COALESCE(m.value, 0) AS value
FROM calendar c
LEFT JOIN metrics m
  ON m.metric_date = c.dt
ORDER BY c.dt;
```

---

## 14) Detect churned users (no events in last 30 days)
**Problem:** Find users who have not been active in the last 30 days relative to a reference date.
**Schema:**
```sql
user_events(user_id, event_date)
```
**Solution:**
```sql
WITH last_seen AS (
  SELECT user_id, MAX(event_date) AS last_event
  FROM user_events
  GROUP BY user_id
)
SELECT user_id
FROM last_seen
WHERE last_event < DATE '2024-06-01' - INTERVAL '30 day'
ORDER BY user_id;
```

---

## 15) Funnel conversion with drop-off
**Problem:** For a three-step funnel, compute users at each step and drop-off rates.
**Schema:**
```sql
events(user_id, event_time, event_type)
-- event_type in ('landing', 'signup', 'purchase')
```
**Solution:**
```sql
WITH step_users AS (
  SELECT
    user_id,
    MAX(CASE WHEN event_type = 'landing' THEN 1 ELSE 0 END) AS did_landing,
    MAX(CASE WHEN event_type = 'signup' THEN 1 ELSE 0 END) AS did_signup,
    MAX(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) AS did_purchase
  FROM events
  GROUP BY user_id
)
SELECT
  SUM(did_landing) AS landing_users,
  SUM(CASE WHEN did_landing = 1 AND did_signup = 1 THEN 1 ELSE 0 END) AS signup_users,
  SUM(CASE WHEN did_signup = 1 AND did_purchase = 1 THEN 1 ELSE 0 END) AS purchase_users,
  ROUND(100.0 * (SUM(did_landing) - SUM(CASE WHEN did_landing = 1 AND did_signup = 1 THEN 1 ELSE 0 END)) / NULLIF(SUM(did_landing), 0), 2) AS landing_to_signup_dropoff_pct,
  ROUND(100.0 * (SUM(CASE WHEN did_landing = 1 AND did_signup = 1 THEN 1 ELSE 0 END) - SUM(CASE WHEN did_signup = 1 AND did_purchase = 1 THEN 1 ELSE 0 END)) / NULLIF(SUM(CASE WHEN did_landing = 1 AND did_signup = 1 THEN 1 ELSE 0 END), 0), 2) AS signup_to_purchase_dropoff_pct
FROM step_users;
```
