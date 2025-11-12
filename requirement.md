# Maze of Maxs — Tài liệu Yêu cầu (Requirement)

Version: 1.1
Date: 2025-11-12

## 1. Tổng quan

"Maze of Maxs" (hiển thị dưới tên Mê Cung Marx) là trò chơi mê cung chạy trong trình duyệt, vẽ bằng `<canvas>`. Người chơi điều khiển nhân vật (W/A/S/D hoặc phím mũi tên) để khám phá mê cung, giải các phòng (câu hỏi, rương, khiên) và tìm lối thoát. Trò chơi có cơ chế chọn hướng ban đầu (Đông/Tây/Bắc/Nam) ảnh hưởng tới gameplay, cơ chế "vượt tường" (bypass) được bật/tắt bằng phím Space, và mỗi ô sự kiện sẽ mở một "phòng" là một mê cung nhỏ (mini-maze) với lối ra ngẫu nhiên; phòng cuối (type boss) lớn hơn.

Mục tiêu tài liệu: mô tả rõ yêu cầu chức năng, phi chức năng, cơ chế hướng và phòng, cùng các tiêu chí chấp nhận/kiểm thử.

---

## 2. Phạm vi & Khái niệm chính

-   Mê cung chính: lưới N x N ô, mỗi ô là tường (1) hoặc đường (0). Một số ô chứa "sự kiện" (2..5).
-   Sự kiện: loại 2 = Câu hỏi, 3 = Rương/Thưởng, 4 = Khiên, 5 = Lối thoát / Boss.
-   Mini-room: khi bước vào ô sự kiện, người chơi sẽ vào 1 mê cung nhỏ (mini-maze) với ô xuất (exit) nằm ngẫu nhiên; tìm exit để hoàn thành phòng.
-   Chọn hướng ban đầu: trước khi bắt đầu, người chơi chọn 1 trong 4 hướng (east/west/north/south). Hướng này áp dụng suốt màn chơi và thay đổi trọng số, phần thưởng và/hoặc cơ chế.

---

## 3. Yêu cầu chức năng (Functional Requirements)

3.1 Khởi tạo & Giao diện

-   FR1: Hiển thị overlay khởi đầu với tiêu đề, hướng dẫn ngắn, nút "Bắt Đầu".
-   FR2: Trước khi bắt đầu, bắt buộc chọn 1 hướng (Đông/Tây/Bắc/Nam) trên sidebar; nếu chưa chọn thì không thể bắt đầu.
-   FR3: Khi bắt đầu, sinh mê cung chính ngẫu nhiên (N configurable) và hiển thị canvas + HUD.

    3.2 Di chuyển & điều khiển

-   FR4: Di chuyển bằng W/A/S/D hoặc phím mũi tên.
-   FR5: Space toggles bypassFlag (bật/tắt chế độ vượt tường). Khi bật, bước vào ô tường sẽ thực hiện bypass (vượt) nếu điều kiện cho phép.

    3.3 Sự kiện và phòng (mini-room)

-   FR6: Khi người chơi bước vào ô sự kiện (2..5), mở mini-room (một mê cung nhỏ với exit ngẫu nhiên). Người chơi điều khiển bên trong mini-room để tìm exit.
-   FR7: Sau khi tìm exit, áp dụng hiệu ứng tương ứng với loại sự kiện (câu hỏi, thưởng, khiên, thoát). Phòng boss (type 5) xử lý chiến thắng/mức thưởng đặc biệt.

    3.4 Cơ chế vượt tường (Wall Bypass)

-   FR8: Có `bypassFlag` (boolean). Khi người chơi bật (Space) và sau đó thực hiện bypass, flag sẽ tự tắt.
-   FR9: Nếu chọn hướng "south" (Nam), bypass khi bật là miễn phí (không mất điểm). Các hướng khác yêu cầu tiêu tốn `WALL_BYPASS_COST` điểm để bypass.

    3.5 Hiệu ứng theo hướng (direction perks)

-   FR10 (East/Đông): Câu hỏi thưởng nhiều điểm hơn (ví dụ +15 điểm cho mỗi câu đúng thay vì +5).
-   FR11 (West/Tây): Nếu trả lời sai, chỉ trừ điểm (ví dụ -5) thay vì mất mạng (nếu không có khiên).
-   FR12 (North/Bắc): Tăng xác suất rương và/hoặc phần thưởng lớn hơn (ví dụ rương có thể cho +50 điểm).
-   FR13 (South/Nam): Bypass miễn phí khi bật flag.

    3.6 Modal & callback

-   FR14: Modal/overlay hỗ trợ callback được truyền dưới dạng function hoặc tên hàm (string). Khi là string, gọi hàm tương ứng an toàn (ví dụ `window['funcName']()`), không dùng `eval`/`new Function`.

    3.7 Cấu hình

-   FR15: Các thông số dễ chỉnh: N (kích thước mê cung), CELL_SIZE, MAX_LIVES, WALL_BYPASS_COST, MAX_EVENTS, QUESTIONS_MIN/MAX.

---

## 4. Yêu cầu phi chức năng (Non-Functional Requirements)

-   NFR1: Chạy trên trình duyệt hiện đại (Chrome, Edge, Firefox).
-   NFR2: Có thể chạy tĩnh (không cần backend). Khuyến nghị dùng static server để tránh hạn chế data URI.
-   NFR3: Tránh dynamic-eval; các callback chuỗi phải được gọi an toàn.
-   NFR4: Mã có chú thích, cấu trúc dễ đọc, tách logic (game) ra file riêng khi thuận tiện.
-   NFR5: Hiệu năng: render mượt với cấu hình mặc định (N=15, CELL_SIZE=40).

---

## 5. Thiết kế dữ liệu & cấu trúc chính

-   `maze`: mảng 2 chiều của mê cung chính.
-   `gameEvents`: danh sách các event { x, y, type, isCleared, ... }.
-   `player`: { x, y, lives, score, shield }.
-   `chosenDirection`: 'east'|'west'|'north'|'south' (do người chơi chọn trước khi bắt đầu).
-   `bypassFlag`: boolean bật/tắt bởi phím Space.
-   Mini-room: object { maze, exitX, exitY, type, eventRef } khi player đang ở trong phòng.

---

## 6. Luồng trò chơi (Game Flow)

1. Người chơi mở trang => hiển thị overlay khởi đầu và sidebar chọn hướng.
2. Người chơi chọn hướng (bắt buộc), đọc ghi chú (perks), nhấn Bắt Đầu.
3. Sinh mê cung chính và đặt `gameEvents`; phân bố loại sự kiện có thể bị ảnh hưởng bởi `chosenDirection` (ví dụ Bắc nhiều rương hơn, Đông nhiều câu hỏi hơn).
4. Người chơi di chuyển trong mê cung chính. Khi đi vào ô event:
    - Không xử lý trực tiếp; thay vào đó spawn mini-room (một mê cung nhỏ). Người chơi điều khiển bên trong để tìm ô exit.
    - Sau khi tìm exit, áp dụng hiệu ứng phòng (ví dụ mở modal câu hỏi, thưởng, tăng khiên, hoặc hoàn thành game nếu là exit/boss).
5. Vượt tường: bật `bypassFlag` bằng Space trước khi cố vượt tường. Khi dùng, flag tự tắt. Nếu `chosenDirection==='south'`, bypass miễn phí; khác thì trừ `WALL_BYPASS_COST`.

---

## 7. Hướng dẫn chạy (Run)

Khuyến nghị chạy bằng static server để tránh giới hạn Data URI/SVG:

```powershell
cd 'c:\Users\2132k\Desktop\code\workspace\JSproject'
python -m http.server 8000
# Mở http://localhost:8000
```

Hoặc mở trực tiếp `index.html` (có khả năng một số icon/data-uri không tải được trong một vài trình duyệt khi mở trực tiếp).

---

## 8. Kiểm thử & Tiêu chí chấp nhận (Acceptance)

TC1 — Start: Sau khi chọn hướng, nhấn Bắt Đầu => overlay ẩn, mê cung chính hiển thị, `chosenDirection` được lưu.

TC2 — Direction perks:

-   Đông: câu hỏi đúng cộng nhiều điểm (kiểm tra +15 thay vì +5).
-   Tây: trả lời sai trừ -5 điểm thay vì mất mạng.
-   Bắc: xuất nhiều rương hơn; rương có thể cho phần thưởng lớn (kiểm tra xuất +50).
-   Nam: bật Space rồi bước vào tường => bypass thành công mà không trừ điểm.

TC3 — Mini-room: bước vào ô event => hiện mini-room; di chuyển trong room; tìm ô đỏ (exit) để hoàn thành phòng.

TC4 — Bypass flag: nhấn Space bật/tắt; sử dụng 1 lần và auto tắt.

TC5 — Modal callbacks: mọi button action truyền tên hàm (string) hoặc function đều thực thi đúng (không dùng eval).

TC6 — Game Over / Win: Hết mạng => overlay Game Over; tìm exit boss => overlay Win.

---

## 9. Ghi chú triển khai / Next steps

-   Cải thiện UX: làm rõ thông báo modal (hiển thị điểm thực tế được cộng/trừ theo hướng).
-   Tách JS ra `game.js`, CSS ra file riêng để dễ bảo trì.
-   Thêm localStorage để lưu highscore và cài đặt người chơi.
-   Viết test unit cho hàm `generateMaze`, `placeEvents`, `enterRoom`, `tryWallBypass`.

---

## 10. Lịch sử thay đổi

-   1.0 — 2025-11-12: Phiên bản ban đầu.
-   1.1 — 2025-11-12: Bổ sung cơ chế hướng, bypass flag, mini-room/boss, cập nhật test cases và luồng game.

---

Nếu bạn muốn chỉnh thêm (ví dụ thay đổi các con số thưởng/phạt, hoặc chi tiết UX cho mini-room), cho tôi biết cụ thể mục nào để tôi chỉnh tiếp.
