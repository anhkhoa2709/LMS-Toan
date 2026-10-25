# Bàn giao: Tây Du Truyền Kỳ

Tài liệu cho người hoặc agent làm tiếp game này. Đọc hết file này trước khi sửa code.

## 1. Game là gì

Game nhập vai 2D góc nhìn từ trên xuống, chủ đề Tây Du Ký, chạy trên trình duyệt, ưu tiên màn hình dọc điện thoại.
Chủ dự án là người Việt, muốn làm game giống các MMORPG Trung Quốc đời cũ: thanh HUD trên cùng (máu, mana, VIP, nhiệm vụ, bản đồ nhỏ), nút "Tìm đường", khung chat nhiều kênh ở dưới.

- **Ngôn ngữ giao diện:** tiếng Việt. Trả lời chủ dự án bằng tiếng Việt.
- **Link chơi thử (Artifact claude.ai, riêng tư):** https://claude.ai/artifact/2D5P2Wi1i5b3UbUHBPu9eR
- **Repo:** `anhkhoa2709/LMS-Toan`, nhánh `master`, thư mục `tay-du-ky/`.

## 2. Cấu trúc file

```
tay-du-ky/
  index.html            toàn bộ game: CSS + HTML + JS, khoảng 950 dòng, không thư viện, không build
  assets/samurai/*.png  sprite nhân vật chính (xem mục 5)
  HANDOVER.md           file này
```

Chạy local: mở thẳng `index.html` trong trình duyệt. Không có server, không có npm.
Lưu ý: chạy bằng `file://` thì Google Fonts có thể lỗi, không sao.

`index.html` không có `<!doctype>/<html>/<head>/<body>`. Đây là chủ ý: khi publish lên Artifact, khung trang được bọc tự động. Mở bằng `file://` vẫn chạy bình thường.

## 3. Bản đồ code trong `index.html`

Các khối JS được đánh dấu bằng comment `/* ---------- tên ---------- */`. Dùng `grep -n "/\* ----------" index.html` để có số dòng hiện tại.

| Khối | Nội dung |
|---|---|
| tiện ích | `rng(seed)` ngẫu nhiên cố định, `pc()` vẽ hình tròn pixel, `R()` = fillRect |
| sprite tĩnh | cây, cây đào, đá, đảo mây vẽ bằng code, cache vào canvas `SPR` |
| nhân vật | `drawPerson`, `drawMonkey`, `drawPeachMob`, `drawEntity`, bảng `LOOK` (ngoại hình NPC và quái vẽ bằng code) |
| sprite nhân vật chính | `HERO`, `ANIM`, `heroFrame`, `drawHero`, `meH`: nạp PNG và chạy hoạt ảnh |
| bản đồ | `buildHQS()` = Hoa Quả Sơn 48×48 ô, `buildBDV()` = Bàn Đào Viên 40×44 ô; NPC, cổng dịch chuyển, vùng sinh quái `zones` |
| quái | `MOBDEF` (chỉ số theo cấp), `spawnMob`, `populate` |
| A* | `findPath` (8 hướng, không cắt góc) |
| nhiệm vụ | mảng `QUESTS` (talk/kill, có `repeat` cho nhiệm vụ hằng ngày) |
| trạng thái người chơi | object `P`, các hàm `maxHp/maxMp/atk/def/needExp` |
| lưu | `snapshot/save/load`, localStorage key `tdtk-save` |
| chat | log chat, kênh `khu/phai/thuong/sys`, bot chat giả `BOTS/LINES` |
| logic nhiệm vụ | `questReady`, `completeQuest`, `onKill` (trả về `{e,g}`, đánh dấu quái chết), `gainExp` |
| hội thoại | `openDialog/showDialog` |
| di chuyển | `goTo`, `changeMap`, `autoStep` (tự làm nhiệm vụ, kể cả qua cổng sang bản đồ khác) |
| chiến đấu theo lượt | biến toàn cục `B`, xem mục 4 |
| vòng lặp cập nhật | `update(dt)`, `tickFx`; khi `B` khác null thì chỉ chạy `stepBattle` |
| vẽ | `drawFx`, `drawFloats`, `renderBattle`, `render`, `drawMini` |
| HUD | `updateHud`, `toast`, `drawIcons`, `openMenu` (Bảng: chỉ số, tiệm thuốc, đổi tên, chơi lại) |
| input | `resize`, chạm/chuột, bàn phím |
| khởi động | `start(data)`, có hỗ trợ `window.claude.hot` của Artifact |

### Hai canvas
- `#world`: canvas pixel-art, kích thước `VW*S × VH*S`, transform `setTransform(S,0,0,S,...)`. Toạ độ vẽ là **đơn vị thế giới** (1 ô = 16 đơn vị). `S` = 2 hoặc 3 (nút zoom).
- `#labels`: canvas độ phân giải cao (nhân DPR) cho chữ: tên, số sát thương, thanh máu trong trận. Đổi toạ độ bằng `(x-camX)*S`.

Trong trận, `camX=camY=0` và mọi vị trí tính theo khung nhìn: `fpos(i)` là vị trí quái, `mpos()` là vị trí người chơi.

## 4. Chiến đấu theo lượt

- **Vào trận:** chạm quái trên bản đồ (hoặc "Tìm đường" tự chạm), hoặc quái chủ động (`aggro`) chạm vào người.
  - `startBattle(mob)` gom quái cùng loại ở gần.
  - Số quái mỗi trận: dưới cấp 3 là 1 con, cấp 3–5 là 2 con, từ cấp 6 là 3 con.
- **Máy trạng thái `B.phase`:** `intro → input → anim → (foesTurn) → input … → end`.
- **Hoạt cảnh:** `play(steps, done)` chạy lần lượt các bước `{d, start, tick(k), end}`. `dash(actor, from, to, hit, opts)` tạo chuỗi lao tới, đánh, lùi về. Các tuỳ chọn của `opts`:

  | Tuỳ chọn | Ý nghĩa |
  |---|---|
  | `run` | hoạt ảnh lúc lao tới |
  | `anim` | hoạt ảnh lúc đánh |
  | `d` | thời lượng bước đánh |
  | `at` | thời điểm gây sát thương, 0..1 |
  | `gap` | khoảng cách đứng trước mục tiêu |
  | `dashD` | thời gian lao tới |

- **Hành động** (`doAction`):

  | Hành động | Chi tiết |
  |---|---|
  | `atk` | Công |
  | `spell` | `SPELLS`: Phá Thiên Trảm 12 MP đánh một mục tiêu; Vạn Kiếm Quy Tông 24 MP đánh tất cả |
  | `heal` | `HEALS`: Hồi Xuân Thuật 16 MP, bình máu, bình mana |
  | `skip` | phòng thủ: giảm 50% sát thương lượt đó, hồi 8% MP |

- **Tự đánh:** ô `#bAuto`, logic trong `autoPick()`. Tự bật khi đang dùng "Tìm đường".
- **Kết thúc:**
  - Thắng: `winBattle`, gọi `onKill` cho từng quái.
  - Thua: `loseBattle` chạy hoạt ảnh ngã rồi hồi sinh.
  - Cả hai đều kết thúc bằng `endBattle`, sau đó người chơi có `P.safe=1.5` giây không bị quái gây hấn.
- **Hiệu ứng** (mảng `fx`, thời lượng trong `FXLIFE`, vẽ trong `drawFx`):
  `slash, crescent, bigslash, shock, aura, qi, xslash, ring, cast, boom, heal, shield, lvup`.
  Rung màn hình: `B.shake`.
- **Phím tắt:** 1 Công, 2 Phép, 3 Hồi phục, 4 Bỏ lượt, Enter để đóng bảng kết quả.

## 5. Sprite nhân vật chính (võ sĩ áo đen)

Mỗi file PNG là một dải ngang, mỗi khung **128×128**, nền trong suốt, nhân vật **quay mặt sang phải**, chân chạm đáy khung (y≈127).
`ax` là toạ độ x của thân trong khung, dùng làm điểm neo.

| File | Khung | ax | Dùng cho (trong `ANIM`) |
|---|---|---|---|
| idle.png | 3 | 72 | `idle`: thủ thế trong trận; ảnh đại diện HUD cắt từ đây |
| stand.png | 6 | 33 | `stand`: đứng yên trên bản đồ |
| walk.png | 9 | 59 | `walk`: đi bộ trên bản đồ (điều khiển tay) |
| run.png | 8 | 50 | `run`: chạy khi tự tìm đường, lao tới khi Công |
| jump.png | 9 | 66 | `jump`: bật nhảy khi dùng Phá Thiên Trảm |
| combo.png | 4 | 53 | `combo`: Công |
| heavy.png | 5 | 58 | `heavy`: Phá Thiên Trảm |
| thrust.png | 4 | 53 | `thrust`: Vạn Kiếm Quy Tông; `ready` dùng khung 0–1 |
| guard.png | 2 | 68 | `guard`: Bỏ lượt / phòng thủ |
| death.png | 6 | 60 | `death`: thua; `hurt` = khung 2 |

- **Tỉ lệ vẽ:** trên bản đồ `drawHero(..., sc=.5)`, khi `S=2` thì mỗi pixel sprite đúng bằng 1 pixel màn hình. Trong trận `sc=1`.
- **Hướng:** `face=-1` để lật sang trái. Trong trận người chơi luôn quay trái.
- **Dự phòng:** nếu PNG chưa tải xong (`HERO.ready=false`), game dùng nhân vật vẽ bằng code `LOOK.player`.
- **Đo neo cho sprite mới:** dùng Pillow, lấy trung điểm các pixel không trong suốt của vùng đầu (10 hàng đầu của bbox) hoặc của chân (y 112–127). Ví dụ:
  ```bash
  pip install Pillow
  python3 -c "from PIL import Image; im=Image.open('x.png').convert('RGBA'); print([im.crop((i*128,0,i*128+128,128)).getbbox() for i in range(im.width//128)])"
  ```
- **Thêm hoạt ảnh:** thêm vào `HERO.sheets` (tên file + `ax`), thêm mục vào `ANIM` (`s` = sheet, `f` = danh sách khung, `fps`, `loop`). Đổi hoạt ảnh trong trận bằng `setMe(tên)`.

## 6. Việc đang dở, ưu tiên theo thứ tự

1. **Bộ sprite cung thủ** (áo cam đỏ, đeo bao tên): mũi tên, bắn cung, chém kiếm, ngã, đứng giương cung (6 tấm).
   - Chủ dự án đã gửi nhưng **chưa nhận được file** (xem mục 7).
   - Đề xuất: làm **nhân vật thứ hai người chơi chọn được** lúc vào game, đánh xa bằng mũi tên bay (sprite mũi tên riêng), chiêu bắn tất cả "Liên Châu Tiễn".
   - Cần thêm: màn chọn nhân vật, `P.cls`, bộ `ANIM`/`HERO` riêng theo class, `dash` thay bằng bắn tại chỗ.
2. **Bộ sprite võ sĩ áo đỏ cầm cờ** (đứng 2, ngã 6, chém liên hoàn 4, chém bổ 5, đâm 4).
   - Cũng **chưa nhận được file**.
   - Đề xuất: boss "Xích Giáp Tướng Quân". Cần hệ sprite cho quái: hiện `drawEntity` chỉ vẽ quái bằng code, phải thêm nhánh dùng ảnh giống `drawHero`, và cho `foesTurn` dùng `anim` của quái.
   - Chưa hỏi lại chủ dự án là chọn boss hay nhân vật chơi được. **Hỏi trước khi làm.**
3. Hai bộ sprite trên đều làm theo cùng định dạng 128×128 như bộ võ sĩ đen, nên có thể tái dùng cách neo ở mục 5.

Ý tưởng chủ dự án từng được gợi ý nhưng chưa chọn: túi đồ và trang bị, thêm bản đồ và nhiệm vụ, máy chủ để nhiều người chơi cùng (chat thật, tài khoản).

## 6b. Folder `background/` mới tải lên (chưa dùng)

Chủ dự án đã tải được cả folder qua GitHub (commit "Add files via upload" trên `master`), nằm ở **`/background/` gốc repo**, ngoài thư mục `tay-du-ky/`.

- **Nội dung:** 50 file `1.png`…`50.png`, mỗi file **512×512, tổng khoảng 18 MB**.
- **Thực tế đây là chân dung nhân vật**, không phải ảnh nền: tranh vẽ tay phong cách fantasy châu Âu (dân làng, hiệp sĩ, vua, hoàng hậu, phù thủy, cung thủ trùm đầu…), nền tối.
- **Chưa hỏi chủ dự án định dùng làm gì.** Đề xuất:
  - Ảnh chân dung NPC trong hộp hội thoại `#dialog`, thay cho canvas `#dPortrait`.
  - Ảnh đại diện để người chơi chọn.
- **Trước khi đưa vào game:**
  - Thu nhỏ còn khoảng 128×128 (ImageMagick `convert x.png -resize 128x128 out.png`), đặt vào `tay-du-ky/assets/portraits/` để nhẹ. Ảnh vẽ tay mịn nên **không** dùng `image-rendering: pixelated` cho ảnh này.
  - Phong cách châu Âu lệch với chủ đề Tây Du, nên **hỏi chủ dự án trước** khi gán cho Tôn Ngộ Không hay Thiên Vương.
- **Cách tải folder đã chạy được:** kéo folder vào trang upload GitHub. Lần sau chủ dự án gửi sprite (cung thủ, võ sĩ áo đỏ) thì nên đi đường này, rồi agent `git pull origin master`.

## 7. Lưu ý về file chủ dự án gửi (quan trọng)

- **Ảnh dán vào chat chỉ thành file trên máy khi gửi ở đầu lượt**, lúc agent đã trả lời xong.
  - Ảnh gửi **giữa lúc agent đang làm** chỉ hiện để nhìn, không có file, không cắt khung được. Đã xảy ra 2 lần.
  - Ảnh nhận được nằm ở thư mục `images/` của phiên làm việc. Đường dẫn ghi trong tin nhắn dạng `[Image: source: ...]`.
- Chủ dự án muốn tải **cả folder**:
  - Máy chạy agent **chỉ ra được GitHub**. Google Drive, Dropbox, Mediafire, transfer.sh đều bị proxy chặn.
  - Cách đã hướng dẫn: kéo folder vào https://github.com/anhkhoa2709/LMS-Toan/upload/master trên máy tính, rồi Commit. Cách này **đã thành công** với folder `background/` (xem mục 6b).
  - Agent tiếp theo nên `git fetch origin` để xem có file mới chưa.
- Sprite có thể lấy từ game khác. Đã nhắc chủ dự án: chỉ dùng riêng thì được, phát hành công khai cần sprite có giấy phép.

## 8. Cách kiểm thử

Đã dùng Playwright có sẵn trên máy (`$(npm root -g)/playwright`, Chromium cài sẵn). Mẫu script:

```js
const {chromium}=require(process.env.PW);
(async()=>{const b=await chromium.launch();const p=await b.newPage({viewport:{width:400,height:820},deviceScaleFactor:2});
const errs=[];p.on('pageerror',e=>errs.push(e.message));
await p.goto('file:///home/user/LMS-Toan/tay-du-ky/index.html');await p.waitForTimeout(1000);
// biến toàn cục truy cập được qua evaluate: P, B, curMap, MAPS, QI, QP, startBattle, changeMap, tc...
await p.evaluate(()=>{P.lv=6;changeMap('hqs',tc(12),tc(38));const m=curMap.mobs.find(m=>!m.dead);P.target={mob:m}});
await p.waitForTimeout(3000);await p.click('.act[data-act=atk]');await p.waitForTimeout(600);
await p.screenshot({path:'shot.png'});console.log(errs.join('\n')||'no errors');await b.close()})();
```
Chạy: `PW=$(npm root -g)/playwright node test.js`. Lỗi `ERR_CERT_AUTHORITY_INVALID` khi tải Google Fonts là bình thường trong sandbox.

## 9. Publish và commit

- **Cập nhật link chơi thử:** gọi tool Artifact publish với `file_path=tay-du-ky/index.html`, `url=https://claude.ai/artifact/2D5P2Wi1i5b3UbUHBPu9eR`.
  - Ở phiên mới, phải `action: "read"` URL đó trước.
  - Truyền thêm `files` cho mọi PNG mới, ví dụ `{"assets/samurai/run.png":"tay-du-ky/assets/samurai/run.png"}`. File đã publish trước đó vẫn được giữ.
- **Commit:** lên `master` bằng message tiếng Anh ngắn, rồi `git push -u origin master`. Mỗi lần xong một tính năng thì commit và push, vì container tạm có thể bị xoá.
- Repo có nhánh `claude/lms-lop-toan-handover-8wv8wn` của một phiên khác (thư mục `phu-hieu/`, làm phù hiệu). **Không liên quan đến game, đừng đụng vào.**

## 10. Quy ước code

- Giữ một file `index.html`, JS thuần, không thư viện.
- Code gọn, dày, giống phần đang có. Comment ngắn bằng tiếng Việt.
- Mọi màu và kích thước HUD nằm trong `:root` của CSS (`--ink, --slate, --jade, --gold, --cinnabar, --azure`…). Giao diện chỉ có một theme tối.
- Đặt `[hidden]{display:none!important}` để dùng `el.hidden`.
- Bọc mọi chỗ dùng localStorage trong try/catch.
- Sau mỗi thay đổi: chạy kiểm thử Playwright, xem ảnh chụp, rồi publish, commit, push.
