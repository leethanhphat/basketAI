# NBA-AI: Xây dựng một Trung tâm Phân tích Bóng rổ Thông minh

> Phân tích chất lượng cấp chuyên gia với trải nghiệm người dùng đẳng cấp thế giới, miễn phí.

---

## Phần I: Nền tảng - Bản thiết kế Kiến trúc

### Kiến trúc Tổng quan

* **3 trụ cột kiến trúc**: Khả năng mở rộng, Phục hồi, Tốc độ phát triển
* **Ngôn ngữ chính**: Python
* **Backend**: FastAPI (async, WebSocket, Swagger UI)
* **CSDL**: PostgreSQL + TimescaleDB (dữ liệu chuỗi thời gian)
* **Frontend**: React + Recharts (hoặc Nivo cho biểu đồ nâng cao)

### Lớp Thu thập và Trừu tượng hóa Dữ liệu

#### Nguồn dữ liệu chính:

| Nguồn                | Truy cập     | Chi phí  | Giới hạn       | Độ sâu         | Ổn định          |
| -------------------- | ------------ | -------- | -------------- | -------------- | ---------------- |
| nba\_api             | Python lib   | Miễn phí | Không ổn định  | Rất sâu        | Không chính thức |
| balldontlie.io       | API REST     | Freemium | 5-600 req/phút | 1946-nay       | Rất tốt          |
| basketball-reference | Web Scraping | Miễn phí | \~20 req/phút  | Rất sâu        | Ổn định          |
| Sportradar           | API REST     | Trả phí  | Cao            | Thời gian thực | Rất cao          |

#### Data Abstraction Layer (DAL)

```python
# Pseudocode
async def get_player_career_stats(player_id):
    try:
        return await balldontlie.get_stats(player_id)
    except RateLimitError:
        return await nba_api.get_stats(player_id)
    except DataNotFound:
        return await scrape_bref(player_id)
```

### Backend: FastAPI

* Hiệu năng cao (Starlette + Pydantic)
* Hỗ trợ bất đồng bộ (`async/await`)
* WebSocket cho trận đấu trực tiếp
* Swagger UI + ReDoc hỗ trợ tài liệu

### CSDL: PostgreSQL + TimescaleDB

```sql
CREATE TABLE play_by_play (
  game_id BIGINT,
  event_timestamp TIMESTAMPTZ NOT NULL,
  event_type TEXT,
  player1_id INT,
  player2_id INT,
  score_margin INT,
  details JSONB
);
SELECT create_hypertable('play_by_play', 'event_timestamp');
```

### Phân tích ngay trong SQL

```sql
SELECT time_bucket('1 minute', event_timestamp) as minute,
       SUM(shot_made)::float / COUNT(shot_attempt) as fg_pct
FROM play_by_play
WHERE player_id = $1
GROUP BY minute;
```

### Frontend: React + Recharts

* Tái sử dụng component: `PlayerCard`, `TeamStats`, `ShotChart`
* Recharts: nhanh, dễ, khai báo
* Nivo (option nâng cao)

---

## Phần II: Các Tính năng Cốt lõi

### MVP: "Second Spectrum" cho mọi người

* **Trang Cầu thủ/Đội bóng Động**: biểu đồ trend, shot chart, percentile
* **Trung tâm Trận đấu Trực tiếp**:

  * WebSocket cập nhật play-by-play
  * Box Score + Mô hình xác suất thắng Live

### Cleaning the Glass - Triết lý Dữ liệu Sạch

* **Loại trừ Garbage Time** mặc định
* Dựa trên mô hình xác suất thắng / chênh lệch thời gian + điểm

### UX vượt trội hơn stats.nba.com

* Ít click, responsive, truy vấn nhanh

---

## Phần III: Động cơ AI

### Dự báo Trận đấu & Cầu thủ

* **Mô hình kết quả trận đấu**: logistic regression, XGBoost, LightGBM
* **Đặc trưng**: eFG%, TOV%, FT Rate, ORB%, Elo rating
* **Dự báo cầu thủ**: regression (Linear, RF, MLP)

### Fan Sentiment Index (FSI)

* NLP phân tích cảm xúc từ Twitter/X
* Chỉ số cảm xúc cầu thủ theo thời gian

### Giao diện Statmuse/NLP

* Giai đoạn 1: từ khóa đơn giản
* Giai đoạn 2: T5/BART fine-tune

### Tóm tắt trận đấu tự động (NLG)

```text
“Warriors đánh bại Lakers 114-108. Stephen Curry ghi 12 điểm liên tiếp ở cuối Q4...”
```

* Bước 1: Phát hiện sự kiện nổi bật (run, clutch, lead change)
* Bước 2: Tạo mẫu + NLG nâng cao (T5/BART sequence2sequence)

### Cá nhân hóa: Đề xuất theo vector hóa

* Vector người dùng + vector nội dung (Cosine Similarity)
* Xây dựng home feed cá nhân hóa

---

## Phần IV: Mã nguồn mở & Phát triển Bền vững

### Giai đoạn 1: Ra mắt Dự án

* `README.md` cực chi tiết, chuyên nghiệp
* License: **MIT** (open nhất, dễ áp dụng)

| License | Tự do thương mại | Copyleft | Bằng sáng chế |
| ------- | ---------------- | -------- | ------------- |
| MIT     | ✅                | ❌        | ❓             |
| Apache2 | ✅                | ❌        | ✅             |
| GPLv3   | ❌                | ✅        | ✅             |

### Giai đoạn 2: Xây dựng Cộng đồng

* `CONTRIBUTING.md`: hướng dẫn góp code, cài dev
* Issue gắn nhãn `good first issue`

### Giai đoạn 3: Bền vững & Monetize

* Open Core: bản miễn phí + tính năng trả phí
* SaaS: "NBA-AI Cloud" – không cần tự host
* Monetize API: như balldontlie.io

---

## 📌 Kết luận

**NBA-AI** không chỉ là một dự án  – mà còn là nền tảng open-source có thể phát triển thành một công cụ phân tích bóng rổ mạnh mẽ, lấy cảm hứng từ các công cụ chuyên nghiệp nhưng thân thiện với cộng đồng.

---

**Tác giả**: Phát Lê

**Giấy phép**: MIT

**Github**: *coming soon*
