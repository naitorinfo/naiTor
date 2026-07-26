# naiTor

Stremio addon tìm torrent + phụ đề từ nhiều nguồn, hỗ trợ debrid để stream nhanh hơn và AI dịch phụ đề khi thiếu.

**Cài đặt:** https://naitor.onrender.com/configure

## Tính năng

- **Duyệt & tìm kiếm** phim/series (Discover + Search) qua Cinemeta
- **Nguồn torrent**: YTS, EZTV (gọi trực tiếp) · ThePirateBay/Apibay (qua FlareSolverr) · Nyaa (RSS, chuyên anime) · EXT.to (gọi trực tiếp)
- **Nguồn torrent đang tắt**: Torlock — cần FlareSolverr, tạm gác do giới hạn RAM free tier (code giữ nguyên, dễ bật lại)
- **Phụ đề**: SubDL, OpenSubtitles, Wyzie — gộp tất cả làm nhiều lựa chọn trong cùng addon
- **AI dịch phụ đề** (Gemini hoặc DeepL, user tự nhập key qua `/configure`): chỉ kích hoạt khi thiếu sub đúng ngôn ngữ đích, dịch từ bản tiếng Anh có sẵn, cache 7 ngày dùng chung cho mọi người
- **Debrid** (TorBox / Real-Debrid, user tự nhập key): resolve 3 kết quả seed cao nhất thành link HTTP trực tiếp, fallback về torrent thường nếu lỗi
- **Cache**: 2 tiếng cho stream/subtitle, 10 phút nếu rỗng (thử lại sớm), 7 ngày cho bản dịch AI

## Kiến trúc
Stremio ──▶ naiTor (Render) ──▶ Cinemeta (metadata)
│
├──▶ YTS / EZTV (trực tiếp)
├──▶ Apibay (qua FlareSolverr, hàng đợi tuần tự — vượt Cloudflare)
├──▶ Nyaa (RSS trực tiếp)
├──▶ EXT.to (trực tiếp)
├──▶ SubDL / OpenSubtitles / Wyzie (phụ đề)
├──▶ Gemini / DeepL (nếu user cấu hình — dịch AI)
└──▶ TorBox / Real-Debrid (nếu user cấu hình — resolve link nhanh)


## Công nghệ

- Node.js + `stremio-addon-sdk` (qua `getRouter`, không dùng `serveHTTP`)
- Express (routing tùy chỉnh, static landing page, rate limiting)
- `cheerio` (scrape HTML: EXT.to, Torlock dự phòng)
- `fast-xml-parser` (parse RSS Nyaa)
- `adm-zip` (giải nén phụ đề SubDL)
- FlareSolverr (bypass Cloudflare cho Apibay)
- Deploy: Render (free tier, cả `naitor` lẫn `flaresolverr`)

## Biến môi trường

| Biến | Bắt buộc | Mô tả |
|---|---|---|
| `FLARESOLVERR_URL` | Có | URL service FlareSolverr, kèm `/v1` |
| `SUBDL_API_KEY` | Không | Thiếu thì bỏ qua nguồn này |
| `OPENSUBTITLES_API_KEY` | Không | Thiếu thì bỏ qua nguồn này |
| `WYZIE_API_KEY` | Không | Thiếu thì bỏ qua nguồn này |
| `TMDB_API_KEY` | Không | Dự phòng cho Torlock (đang tắt) |
| `BASE_URL` | Có | URL public của chính naiTor, dùng cho logo/proxy link |
| `PORT` | Tự động | Render tự cấp |

*Gemini/DeepL API key do từng người dùng tự nhập qua `/configure`, không cấu hình ở server.*

## Giám sát

- UptimeRobot ping cả `naitor` và `flaresolverr` mỗi 5 phút — tránh cold-start, phát hiện sập sớm
- Health Check Path `/` đã bật trên Render cho `flaresolverr` — tự restart khi treo

## Lưu ý

Addon này tổng hợp link từ các nguồn công khai trên internet; không lưu trữ hay host nội dung. Người dùng tự chịu trách nhiệm về nội dung truy cập.
