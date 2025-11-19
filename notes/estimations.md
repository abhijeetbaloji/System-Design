# System Design Capacity Estimation - Cheat Sheet

A comprehensive guide for back-of-the-envelope calculations in system design interviews.

---

## Table of Contents
1. [E-Commerce Platform](#1-e-commerce-platform-amazon-like)
2. [Food Delivery App](#2-food-delivery-app-uber-eatsdoordash)
3. [Ride-Hailing App](#3-ride-hailing-app-uberlyft)
4. [Video Streaming Platform](#4-video-streaming-platform-netflixyoutube)
5. [Social Media](#5-social-media-instagram-like)
6. [Messaging App](#6-messaging-app-whatsapptelegram)
7. [URL Shortener](#7-url-shortener-bitly)
8. [Key Formulas & Tips](#key-formulas--tips)

---

## 1. E-Commerce Platform (Amazon-like)

### Assumptions
- **500M** monthly active users (MAU)
- **40%** daily active users (DAU) = **200M**
- Each user views **20 products** per day
- **5%** of users make a purchase daily = **10M purchases**
- Average order has **3 items**
- **30%** of products have images (avg 5 images per product)
- Data stored for **7 years**

### Estimations

#### Read QPS (Product Views)
```
200M users × 20 views / 86,400 seconds ≈ 46,000 reads/sec
Peak (3x) ≈ 138,000 reads/sec
```

#### Write QPS (Orders)
```
10M orders / 86,400 seconds ≈ 115 orders/sec
Peak (5x during sales) ≈ 575 orders/sec
```

#### Storage Breakdown

| Component | Calculation | Size |
|-----------|-------------|------|
| **Product Catalog** | 100M products × 2.5KB | 250GB |
| **Images** | 100M × 30% × 5 images × 200KB | 30TB |
| **Order History (7 years)** | 10M orders/day × 600B × 365 × 7 | 15TB |
| **Total Storage** | Including indexes and replicas | ~45TB |

**Per Order Record:**
- 8B order_id + 8B user_id + 100B address + 500B items ≈ **600 bytes**

---

## 2. Food Delivery App (Uber Eats/DoorDash)

### Assumptions
- **100M** MAU
- **20%** daily active users = **20M DAU**
- Each user places **0.5 orders** per day = **10M orders/day**
- Average order has **4 items**
- **80%** of restaurants have menu photos
- Real-time location tracking every **10 seconds** during delivery
- Average delivery time: **30 minutes**
- Data stored for **3 years**

### Estimations

#### Read QPS (Menu Browsing)
```
20M users × 10 restaurant views / 86,400 ≈ 2,300 reads/sec
Peak (lunch/dinner 6x) ≈ 14,000 reads/sec
```

#### Write QPS (Orders)
```
10M orders / 86,400 ≈ 115 orders/sec
Peak ≈ 700 orders/sec
```

#### Location Updates QPS
```
Assume 20% of orders active at any time = 2M concurrent deliveries
2M / 10 seconds = 200,000 location updates/sec
```

#### Storage Breakdown

| Component | Calculation | Size |
|-----------|-------------|------|
| **Menu Data** | 2M restaurants × 50 items × 1KB | 100GB |
| **Images** | 2M × 80% × 10 images × 150KB | 2.4TB |
| **Order History (3 years)** | 10M × 1.5KB × 365 × 3 | 16TB |
| **Location Tracking (3 years)** | 200K updates/sec × 32B × 86,400 × 365 × 3 | 60TB |
| **Total Storage** | Including indexes | ~80TB |

**Per Location Update:**
- 8B delivery_id + 8B timestamp + 16B lat/long = **32 bytes**

---

## 3. Ride-Hailing App (Uber/Lyft)

### Assumptions
- **150M** MAU
- **15%** daily active riders = **22.5M**
- **5M** active drivers
- Average **1.5 rides** per day per active rider = **33M rides/day**
- Driver location updates every **5 seconds** when active
- **30%** of drivers online at peak
- Average trip duration: **20 minutes**
- Data stored for **5 years**

### Estimations

#### Read QPS (Finding Drivers)
```
22.5M riders × 3 searches / 86,400 ≈ 780 reads/sec
Peak ≈ 4,000 reads/sec
```

#### Write QPS (Ride Booking)
```
33M rides / 86,400 ≈ 380 rides/sec
Peak (3x) ≈ 1,200 rides/sec
```

#### Location Updates QPS
```
5M drivers × 30% active × 1 update/5 seconds = 300,000 updates/sec
```

#### Storage Breakdown

| Component | Calculation | Size |
|-----------|-------------|------|
| **User Profiles** | (150M riders + 5M drivers) × 2KB | 310GB |
| **Ride History (5 years)** | 33M × 630B × 365 × 5 | 38TB |
| **Location Tracking (5 years)** | 300K updates/sec × 32B × 86,400 × 365 × 5 | 150TB |
| **Map/Route Cache** | Popular routes and map tiles | 5TB |
| **Total Storage** | Including indexes | ~195TB |

**Per Ride Record:**
- 8B ride_id + 8B rider_id + 8B driver_id + 100B route + 500B metadata ≈ **630 bytes**

---

## 4. Video Streaming Platform (Netflix/YouTube)

### Assumptions
- **500M** MAU
- **50%** daily active users = **250M DAU**
- Average **2 hours** of video watched per user per day
- **10M videos** in catalog
- Average video: **10 minutes** duration
- Multiple quality levels:
  - 360p: 300MB
  - 720p: 1GB
  - 1080p: 2GB
  - 4K: 8GB
- Data stored **indefinitely**

### Estimations

#### Read QPS (Video Streaming)
```
250M users × 2 hours / 24 hours = 20.8M concurrent viewers
Each viewer = 1 video chunk request every 10 seconds
20.8M / 10 = 2M requests/sec
```

#### Write QPS (New Uploads)
```
Assume 10,000 new videos per day
10K / 86,400 ≈ 0.1 writes/sec (but bursty)
```

#### Storage Breakdown

| Component | Calculation | Size |
|-----------|-------------|------|
| **Video Storage** | 10M videos × 10 min × 4 qualities (avg 3GB) | 30PB |
| **Metadata** | 10M videos × 10KB | 100GB |
| **Thumbnails** | 10M videos × 5 thumbnails × 100KB | 5TB |
| **Watch History (5 years)** | 250M × 365 × 5 × 6 videos × 100B | 27TB |
| **Total Storage** | Including metadata | ~30PB |

#### Bandwidth Requirements
```
20.8M concurrent × average 5Mbps = 104 Petabits/sec = 13 PB/sec
```

---

## 5. Social Media (Instagram-like)

### Assumptions
- **2B** MAU
- **40%** daily active = **800M DAU**
- **3 posts** per user per day = **2.4B posts/day**
- **70%** posts contain photos, **20%** videos, **10%** text
- Average photo: **2MB**, video: **50MB**
- Users scroll **30 minutes/day** viewing **100 posts**
- Data stored for **10 years**

### Estimations

#### Write QPS (Posts)
```
2.4B posts / 86,400 ≈ 27,800 posts/sec
Peak (5x) ≈ 139,000 posts/sec
```

#### Read QPS (Feed Viewing)
```
800M users × 100 posts / 86,400 ≈ 925,000 reads/sec
Peak ≈ 2.8M reads/sec
```

#### Storage Breakdown

| Component | Calculation | Size |
|-----------|-------------|------|
| **Photos (10 years)** | 2.4B × 70% × 2MB × 365 × 10 | 12EB |
| **Videos (10 years)** | 2.4B × 20% × 50MB × 365 × 10 | 87EB |
| **Post Metadata (10 years)** | 2.4B × 2KB × 365 × 10 | 17.5PB |
| **Total Storage** | | ~100EB |

**Per Post Metadata:**
- Post_id, user_id, timestamp, caption, likes, comments ≈ **2KB**

---

## 6. Messaging App (WhatsApp/Telegram)

### Assumptions
- **3B** MAU
- **60%** daily active = **1.8B DAU**
- **50 messages** per user per day = **90B messages/day**
- **5%** messages contain media (avg 1MB)
- Data stored for **2 years**

### Estimations

#### Write QPS (Messages)
```
90B messages / 86,400 ≈ 1.04M messages/sec
Peak (2x) ≈ 2.1M messages/sec
```

#### Storage Breakdown

| Component | Calculation | Size |
|-----------|-------------|------|
| **Text Messages (2 years)** | 90B × 95% × 532B × 365 × 2 | 33PB |
| **Media Storage (2 years)** | 90B × 5% × 1MB × 365 × 2 | 329PB |
| **User Profiles** | 3B users × 5KB | 15TB |
| **Total Storage** | | ~362PB |

**Per Text Message:**
- 8B msg_id + 8B sender + 8B receiver + 8B timestamp + 500B content ≈ **532 bytes**

---

## 7. URL Shortener (bit.ly)

### Assumptions
- **100M** short URLs created per month
- **1B** redirects per day
- Store URLs for **10 years**
- **100:1** read-to-write ratio

### Estimations

#### Write QPS
```
100M / 30 / 86,400 ≈ 38 writes/sec
```

#### Read QPS (Redirects)
```
1B / 86,400 ≈ 11,500 reads/sec
Peak (3x) ≈ 35,000 reads/sec
```

#### Storage (10 years)
```
Per URL: 8B short_key + 2KB original_url + 500B metadata ≈ 2.5KB
100M/month × 12 × 10 × 2.5KB ≈ 30TB
```

---

## Key Formulas & Tips

### QPS Calculation
```
Daily requests / 86,400 seconds = Average QPS
Peak QPS = Average QPS × Peak factor (2-10x)
```

### Storage Calculation
```
Daily data × 365 days × Years retained = Total storage
Add 20-30% for indexes and replicas
```

### Bandwidth Calculation
```
Concurrent users × Average data rate = Bandwidth needed
```

### Common Size Estimates

| Data Type | Typical Size |
|-----------|--------------|
| User ID | 8 bytes (64-bit) |
| Timestamp | 8 bytes |
| UUID | 16 bytes |
| Short text | 100-500 bytes |
| Medium text | 1-2KB |
| Image (thumbnail) | 50-100KB |
| Image (standard) | 200KB - 2MB |
| Video (1 min SD) | 10-20MB |
| Video (1 min HD) | 30-50MB |
| Metadata per record | 1-2KB |

### Peak Traffic Multipliers

| System Type | Peak Factor | Peak Times |
|-------------|-------------|------------|
| E-commerce | 5-10x | Sales events, holidays |
| Food delivery | 6x | Meal times (12-2pm, 7-9pm) |
| Ride-hailing | 3-5x | Rush hours, weekends |
| Social media | 3-5x | Evenings, weekends |
| Streaming | 2-3x | Prime time (7-11pm) |
| Messaging | 2x | Daytime hours |

### Storage Units Conversion
```
1 KB = 1,000 bytes (10^3)
1 MB = 1,000 KB = 10^6 bytes
1 GB = 1,000 MB = 10^9 bytes
1 TB = 1,000 GB = 10^12 bytes
1 PB = 1,000 TB = 10^15 bytes
1 EB = 1,000 PB = 10^18 bytes
```

### Time Calculations
```
1 day = 86,400 seconds
1 month ≈ 2.6 million seconds (30 days)
1 year ≈ 31.5 million seconds
```

### Quick Mental Math Tricks

**For QPS from daily volume:**
```
Daily volume / 100,000 ≈ QPS
(More accurate: divide by 86,400, but 100K is easier for mental math)
```

**Example:**
```
10 million requests/day ÷ 100,000 = 100 QPS (actual: 115 QPS)
```

**For storage from daily volume:**
```
Daily storage × 365 × Years × 1.3 (for overhead)
```

### Common Percentages to Remember

- **20/80 Rule**: 20% of users generate 80% of traffic
- **Write:Read Ratios**:
  - Social media: 1:100
  - E-commerce: 1:10
  - URL shortener: 1:100
  - Analytics: 1:1

### Interview Tips

1. **Always state assumptions clearly** before calculations
2. **Round numbers** for easier mental math (use 100K instead of 86,400)
3. **Show your work** - interviewers want to see your thought process
4. **Verify scale** - does 1PB/day make sense for this system?
5. **Consider peak load** - average is not enough
6. **Don't forget overhead** - indexes, replicas, backups (add 20-30%)
7. **Think about growth** - systems typically plan for 2-3 years ahead

### Example Calculation Walkthrough

**Problem:** Estimate Instagram's photo storage needs

**Step 1: State assumptions**
```
- 2B users
- 40% daily active = 800M DAU
- 70% of 3 daily posts are photos = 1.68B photos/day
- Average photo size = 2MB
```

**Step 2: Calculate daily storage**
```
1.68B photos × 2MB = 3.36 PB/day
```

**Step 3: Calculate long-term storage**
```
3.36 PB/day × 365 days × 10 years = 12,264 PB ≈ 12 EB
```

**Step 4: Add overhead**
```
12 EB × 1.3 (for replicas, thumbnails) ≈ 15.6 EB
```

**Step 5: Sanity check**
```
Does 15+ Exabytes for 10 years of Instagram photos make sense? 
Yes - that's massive but reasonable for 2B users over 10 years.
```

---

## Additional Systems to Practice

### 8. Notification Service
- Push notifications to mobile devices
- Email notifications
- SMS notifications
- In-app notifications

### 9. Search Engine (Google-like)
- Web crawling
- Index storage
- Query processing
- Results ranking

### 10. Payment System
- Transaction processing
- Ledger storage
- Fraud detection
- Settlement processing

### 11. Music Streaming (Spotify)
- Audio file storage
- Playlist management
- Download vs streaming
- Recommendations

### 12. Cloud Storage (Dropbox/Google Drive)
- File uploads/downloads
- File synchronization
- Version history
- Sharing and permissions

### 13. Online Gaming Platform
- Real-time multiplayer state
- Match making
- Leaderboards
- Game saves and progress

### 14. Video Conferencing (Zoom)
- Real-time video/audio streams
- Screen sharing
- Recording storage
- Chat messages

### 15. News Feed (Facebook/Twitter)
- Post creation
- Feed generation
- Ranking algorithm
- Real-time updates

---

## Practice Problems

Try estimating these on your own:

1. **LinkedIn** - Professional network with 500M users
2. **TikTok** - Short video platform with 1B users
3. **Airbnb** - Vacation rental platform with 100M listings
4. **GitHub** - Code repository hosting with 100M repositories
5. **Stack Overflow** - Q&A platform with 20M questions

---

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Grokking the System Design Interview](https://www.educative.io/courses/grokking-the-system-design-interview)
- [ByteByteGo](https://bytebytego.com/)

---

**Last Updated:** November 2025

**Author:** System Design Study Notes

**License:** MIT - Feel free to use for interview preparation
This markdown file is well-structured with:

Clear table of contents
Consistent formatting
Tables for easy reading
Code blocks for calculations
Organized sections
Ready to push to Git
Easy to navigate and reference during interviews
You can save this as capacity-estimation-guide.md in your repository!