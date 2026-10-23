# 📓 Style Guide — Video Animation 2D Dọc TikTok ("Sổ tay kinh tế")

> Phong cách + quy trình + checklist tái tạo được, ghi lại từ project thực tế
> "Tại sao GDP tăng mà dân vẫn thấy nghèo đi" (72s, 1080×1920, 14 scene, OpenMontage pipeline).
> Dùng lại cho mọi video explainer 2D dọc phong cách tương tự.

---

## 1. Nhận diện phong cách — "Sổ tay kinh tế"

Tính cách thị giác: **giấy kem kẻ dòng, mực dày bo tròn, vàng nghệ làm mồi nhấn, đỏ mộc con dấu** — ấm, thân quen, dí dỏm nhưng số liệu chính xác. Cảm giác là một cuốn sổ tay kinh tế học vẽ tay, không phải motion graphic corporate.

### Palette (bắt buộc theo vai trò)

| Vai trò | Hex | Quy tắc |
|---|---|---|
| Giấy kem (nền) | `#FFF7E8` | nền toàn video, kẻ dòng rất nhạt `#E8DBC3` |
| Mực (nét vẽ / chữ) | `#1D1D1B` | mọi stroke, text, viền dày |
| Đỏ (con dấu / rủi ro) | `#D62828` | **chỉ** con dấu + logic "bất lợi cho bạn" |
| Vàng (dòng tiền) | `#F4C430` | **chỉ** dòng vàng, hộp đếm, từ khóa phụ đề |
| Xanh (nước) | `#2A9D8F` | **chỉ** biển, khói, dải cuộn giấy |
| Gỗ | `#B9855C` | bảng giá chợ và đồ gỗ |

**Cấm:** gradient, neon, tím-AI, dashboard stock, 3D bóng bảy, slide chữ tĩnh, Ken Burns ảnh chụp.

### Font

**Be Vietnam Pro** (load qua `@remotion/google-fonts`, subset `vietnamese`+`latin`):
- weight **900** — tiêu đề / con số reveal
- weight **700** — nhãn, caption
- weight **500** — ghi chú nhỏ
- Tối đa 2 weight trong một khung.

### Signature device — con dấu đỏ

Mọi "khoảnh quyết định" bị đóng mộc: scale `2.6 → 1`, spring overshoot (damping 9–12, stiffness 130–170), giấy rung ~6px. **Chỉ dùng đúng 3 lần trong cả video** — dùng nhiều là mất sức nặng.

### Motion character

- Số reveal: **phải bùng spring**, không đổi chữ khô khan.
- Ẩn dụ nước: sóng sin 1.4–2.2Hz biên độ nhỏ; dòng tiền = `strokeDashoffset` chạy liên tục.
- Chữ typing 0.13–0.15s/ký tự.
- Nảy rồi yên — **không lò xo liên tục**. Tối đa 2 phần tử động cùng lúc; một ý / một khung.

---

## 2. Cấu trúc nội dung / nhịp kể chuyện

- Khung dọc **1080×1920, fps 30**, tổng ~72s (2160 frames).
- Nhịp mở từ video tham khảo: **hook → nói → 3 lý do → kick số → chốt** (chỉ mượn kết cấu, không mượn hình).
- 12–14 scene, mỗi scene một ẩn dụ thị giác riêng; **không hai scene liên tiếp cùng bố cục**.
- Ẩn dụ xuyên suốt (ví dụ: nền kinh tế = làng đóng tàu; GDP = sông; nhu cầu = 3 túi; lạm phát = bảng giá chợ).
- Phụ đề word-by-word: **pill kem viền mực, từ khóa tô vàng**, cố định đáy khung, bắt đầu từ scene 02 (scene mở không đè caption). Mỗi cluster ≤ 6 từ, ngắt khi gap > 0.55s.

---

## 3. Quy trình sản xuất (pipeline end-to-end)

| # | Stage | Việc chính | Tool |
|---|---|---|---|
| 1 | Research | lấy transcript video / nguồn số liệu | transcriber |
| 2 | Proposal | concept, playbook, art direction, lock `render_runtime=remotion`, `composition_mode=atelier` | proposal_packet |
| 3 | Script | 9 section, mỗi section có style note cho TTS | script.json |
| 4 | Scene plan | scene id / type / window + emphasis words | scene_plan.json |
| 5 | TTS | đọc thử 1 câu → render 9 file section | gemini TTS voice `algieba` (~$0.06/72s) |
| 6 | Word timings | transcribe 9 wav (faster-whisper small) → align vào script (SequenceMatcher, chống drift) | python |
| 7 | Props | `props.json`: 14 scene `{id, kind, audio, durationFrames, words, clusters, beats}` | python |
| 8 | Composition | viết **bespoke TSX** mỗi scene, không import stock registry | Remotion atelier |
| 9 | Audio slice | ffmpeg trim narration theo window từng scene | ffmpeg |
| 10 | Render | `npx remotion render` crf 18 | Remotion |
| 11 | QA | contact sheet + final_review + transcript-vs-script + frame spot-check | ffprobe/whisper |
| 12 | Publish | export bundle (metadata SEO, chapters, thumbnail concept) — không upload | export_bundle |

### Nhạc nền (bài học quan trọng)

API nhạc gen (Lyria) từng treo vô hạn (blocking call không trả). **Quy tắc:** gọi với timeout cứng ~6 phút; sau 2 lần thử thất bại → fallback **chạy không nhạc** và ghi vào decision log, đừng treo toàn pipeline. Lời bình tốt không cần nhạc vẫn hoàn chỉnh.

---

## 4. Yêu cầu kỹ thuật animation (Remotion atelier)

### Điều kiện bắt buộc

1. **Bespoke doctrine:** mọi scene là component vector path tự viết trong project; KHÔNG import từ stock creative registry (được tool check tự động — vi phạm là fail render).
2. File `art-direction.md` phải tồn tại và khai báo "động cơ khác biệt" trước khi viết code cảnh.
3. Entry render phải là **directory thật** bên trong `remotion-composer/projects/<slug>/` — nếu mirror từ đường dẫn khác, copy file thật (junction/symlink bị webpack resolve realpath → lỗi "Can't resolve '…'").
4. Tất cả JSON props/data phải có trong điều khiển `-` props input của composition (không hardcode số liệu trong tsx).

### Cấu trúc composition

```
Root.tsx          → <Composition id="..." durationInFrames={2160} fps={30} width={1080} height={1920}>
Composition.tsx   → <Audio src={staticFile("audio/sc01.wav")}/> cho từng scene + <Sequence> per scene
index.tsx         → registerRoot
ContactSheet.tsx  → QA grid (mock test frame=470 vẫn phải chạy pass trước render)
```

- Audio: mỗi scene 1 slice wav `sc01..sc14.wav`, start khớp `durationFrames` cộng dồn; gap ≈ 0.3–0.4s giữa scene.
- Props từ `props.json` passed qua `--props=...`, assets qua `--public-dir=...`.
- Caption cluster: `{t: [start, end], words: [...]}` — render pill đúng beat từ timing audio thật.

### Render command (chuỗi chuẩn)

```sh
npx remotion render <entry>/index.tsx <CompositionId> <out.mp4> \
  --props=<abs>/props.json --public-dir=<abs>/assets --crf=18 --concurrency=4
```

**Concurrency = số core CPU thực tế của máy** (đặt 6 trên máy 4 core → exit 1 ngay, validate trước khi render).
Output đích: `projects/<slug>/renders/final.mp4`; render dọc mất ~4–5 phút cho 72s.

### Spec video giao

- Container mp4, H.264 crf 18 + AAC 48kHz stereo
- 1080×1920 · 30fps · duration khớp narration thật (±0.5s), tail im lặng ngắn cho outro.

---

## 5. QA gates (tự động + thủ công)

1. **ContactSheet render pass** trước khi render thật (mock frame giữa video).
2. `final_review` của tool: technical_probe (ffprobe), visual_spotcheck, audio_spotcheck, promise_preservation (`render_runtime_used='remotion'`, cấm runtime swap), subtitle_check, atelier doctrine.
3. **Transcript-vs-script:** transcribe audio output → word-accuracy ≥ ~0.95, không từ rác ("dot", "comma").
4. Spot-check 5–7 frame thủ công (1.5/6/12/27/48/62/71s) xem art direction + dấu tiếng Việt.

### Lỗi encoding (Windows) — ghi nhớ kẻo hỏng tiếng Việt

- ❌ KHÔNG dùng `Get-Content | Set-Content` sửa .tsx/.json → round-trip cp1252 làm hỏng dấu tiếng Việt.
- ✅ Dùng tool `edit`/`write`, hoặc `[IO.File]::WriteAllText($path, $text, [Text.Encoding]::UTF8)`.
- Mọi script Python: `PYTHONIOENCODING=utf-8` + `[Console]::OutputEncoding=UTF8` (console cp1252 crash unicode).

---

## 6. Checklist tái tạo nhanh (một trang)

```text
□ Chốt ẩn dụ xuyên suốt + visual_variance≥5 (giấy kem ↔ nhựa mica ≠ các project cũ)
□ Lock palette 4 vai trò (kem/mực/đỏ/vàng) — mỗi màu đúng 1 vai trò
□ Script 9 section, 181 từ ≈ 67–72s giọng tự nhiên
□ TTS 1 câu thử → giọng đúng trước khi render loạt (gemini, voice algieba)
□ Whisper align → word timings → beats hand-coded mỗi scene
□ art-direction.md + palette + signature device (dấu đỏ ≤ 3 lần)
□ 14 scene bespoke TSX — không stock, không dashboard, không gradient
□ Mirror entry vào remotion-composer/projects/<slug>/ (directory thật, không symlink)
□ ContactSheet pass → render crf18 concurrency=#cores
□ QA: ffprobe + final_review pass + transcript ≥0.95 + 7 frame spot-check
□ Export bundle SEO (title/, description hashtag, chapter, thumbnail concept)
□ Ghi checkpoint mọi stage + decision log (nhạc fallback chẳng hạn)
```

---

*Ghi ngày 23-10-2026 · From: project `tai-sao-gdp-tang-van-ngheo` (OpenMontage, animated-explainer, budget $2 · chi thực ~$0.06).*
