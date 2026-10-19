---
marp: true
paginate: true
size: 16:9
theme: default
style: |
  section {
    font-family: "Noto Sans", "DejaVu Sans", "Segoe UI", Roboto, sans-serif;
    font-size: 21px; color: #0f172a; padding: 40px 46px 38px;
    display: flex !important; flex-direction: column !important;
    justify-content: flex-start !important; place-content: flex-start !important;
    letter-spacing: 0;
  }
  section :is(h1, marp-h1) { color: #1d4ed8 !important; font-size: 33px; margin: 0 0 12px; }
  section :is(h2, marp-h2) { color: #1d4ed8 !important; font-size: 28px; margin: 0 0 12px; }
  section :is(h3, marp-h3) { color: #334155 !important; font-size: 19px; margin: 8px 0 2px; }
  section :is(p, li) { line-height: 1.34; }
  section ul { margin: 4px 0 0; }
  section li { margin-bottom: 5px; }
  section table { font-size: 17px; border-collapse: collapse; width: 100%; }
  section :is(th, td) { border: 1px solid #cbd5e1; padding: 3px 7px; text-align: left; }
  section td:nth-child(2) { white-space: nowrap; }
  section th { background: #eff6ff; }
  section :is(code, tt) { font-size: 0.9em; }
  section strong { color: #b91c1c; }
  section.lead { justify-content: center !important; text-align: left; }
  section.lead :is(h1, marp-h1) { font-size: 44px; }
  .cols { display: grid; grid-template-columns: 1fr 1fr; gap: 24px; align-items: start; }
  .tag { display: inline-block; font-size: 15px; color: #475569; line-height: 1.3; }
  .refs { font-size: 15px; }
  .refs p { margin: 0 0 7px; line-height: 1.3; }
  .diagram { margin-top: 10px; }
---

<!--
S1 · 15 s

MỞ ĐẦU (nói khi còn ở slide này):
"Em chào thầy/cô. Nhóm 8 làm đồ án hướng nghiên cứu: tái hiện bài GTAE-IDS trên
IEEE TIFS 2025 — phát hiện xâm nhập chỉ bằng vài gói đầu của luồng — rồi đo lại số
liệu của bài và đề xuất một cải tiến."

KHÔNG đọc tên bài báo tiếng Anh trên slide (mất ~10 giây vô ích).

Ngân sách cả bài: S1 15 · S2 45 · S3 70 · S4 50 · S5 60 · S6 50 · S7 15.
Nguyên tắc: không đọc chữ trên slide; slide là gạch đầu dòng, miệng là nội dung.
-->

<!-- _class: lead -->

# GTAE-IDS: tái hiện &amp; cải tiến

## Phát hiện xâm nhập sớm bằng Graph Transformer Autoencoder

**Đồ án hướng nghiên cứu** · NT204.R11.ANTN — Hệ thống tìm kiếm, phát hiện và ngăn ngừa xâm nhập (IDPS)

Nhóm 8 · Báo cáo giữa kỳ · 19/10/2026

<span class="tag">Bài báo neo: J. Ghadermazi et al., “GTAE-IDS: Graph Transformer-Based Autoencoder Framework for Real-Time Network Intrusion Detection,” IEEE TIFS, vol. 20, pp. 4026–4041, 2025, doi: 10.1109/TIFS.2025.3557741</span>

---

# 1. Vấn đề của bài báo

**Ba hạn chế của NIDS dùng ML/DL hiện nay** — lý do bài báo đặt vấn đề

1. **Flow-based**: đặc trưng tổng hợp của cả luồng → quyết định chỉ ra sau khi luồng kết thúc/timeout ⇒ phù hợp phân tích hồi cứu, không phát hiện online
2. **Dữ liệu phẳng** (vector/lưới) → bỏ qua quan hệ cấu trúc giữa các thực thể mạng: PortScan = một nguồn → nhiều đích; DDoS = nhiều nguồn → một đích
3. **Phụ thuộc nhãn** (supervised/semi-supervised) → gán nhãn đồ thị đắt, và yếu trước tấn công có mẫu cấu trúc chưa thấy

**Khoảng trống bài báo tự xác lập** (khảo sát 15 công trình GNN-NIDS, Table I): chỉ **1/15** là unsupervised, và công trình đó vẫn **flow-based** ⇒ chưa có hệ **packet-based + unsupervised**.

**Câu hỏi của nhóm:** tái hiện được tới mức nào, và **cái giá sai (FPR)** của phương pháp là bao nhiêu?

<!--
S2 · 45 s

"Ba hạn chế của NIDS hiện nay. Một, học trên đặc trưng luồng nên chỉ ra quyết định sau
khi luồng kết thúc — tức là phân tích hồi cứu, không phải phát hiện online.
Hai, dữ liệu phẳng nên mất cấu trúc: PortScan là một nguồn quét nhiều đích, DDoS là
nhiều nguồn dồn một đích — nhìn từng luồng riêng lẻ thì không thấy.
Ba, phần lớn học có giám sát: cần nhãn tấn công và yếu với tấn công chưa thấy.

Bài báo khảo sát 15 công trình GNN cho IDS: chỉ một bài là unsupervised, và bài đó vẫn
flow-based. Vậy chưa có hệ vừa packet-based vừa unsupervised — GTAE-IDS lấp chỗ đó.
Câu hỏi của nhóm em là: tái hiện được tới đâu, và cái giá báo động giả là bao nhiêu."

THAO TÁC: giơ tay đếm 1–2–3 ở ba gạch đầu dòng; câu cuối hạ giọng, chậm lại.
TRÁNH: giải thích "GNN là gì" — để Q&A.
CHUYỂN: "Vậy bài báo làm gì để lấp khoảng trống đó? Ý tưởng gồm bốn bước."
-->

---

# 2. Ý tưởng &amp; phương pháp GTAE-IDS

<div class="cols">
<div>

**Biểu diễn: lưu lượng → đồ thị**

- Nút = `(IP, port)`, **không có đặc trưng**; cạnh = một luồng TCP
- Đặc trưng cạnh: **25 byte** header của **p gói đầu** (bỏ ETH, IP/TCP options, payload, IP/port/protocol), chuẩn hóa [0,1], mã hóa hướng bằng ô 50 chiều ⇒ **2·25·p** chiều
- p = 2…8 (bỏ p = 1 vì gói SYN vô thông tin); **7 mô hình riêng** cho 7 giá trị p

</div>
<div>

**Học &amp; phát hiện**

- **GTAE**: encoder graph transformer cập nhật **cạnh** (attention giữa các cạnh chung đầu mút ≈ line graph) + LapPE; decoder DNN; mất mát MSE
- **Huấn luyện chỉ trên benign** (one-class) — không dùng nhãn tấn công
- Sau train: **tách encoder** → embedding z → **4 detector** one-class → **bỏ phiếu đa số** → `0` / `−1`

</div>
</div>

<div class="diagram">
<svg viewBox="0 0 890 140" style="width:100%">
  <defs>
    <marker id="ar" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#475569"/>
    </marker>
  </defs>
  <g font-family="Noto Sans, DejaVu Sans, sans-serif" font-size="11" fill="#0f172a">
    <rect x="0" y="41" width="64" height="58" rx="8" fill="#f1f5f9" stroke="#94a3b8"/>
    <text x="32" y="66" text-anchor="middle">pcap</text>
    <text x="32" y="82" text-anchor="middle" fill="#475569">(TCP)</text>
    <rect x="78" y="20" width="244" height="100" rx="8" fill="#eff6ff" stroke="#1d4ed8"/>
    <text x="200" y="42" text-anchor="middle" font-weight="600">Đồ thị G(t,p)</text>
    <text x="200" y="61" text-anchor="middle">nút = (IP, port), không đặc trưng</text>
    <text x="200" y="78" text-anchor="middle">cạnh = luồng TCP</text>
    <text x="200" y="95" text-anchor="middle">x_e = 25 byte × p gói → 2·25·p</text>
    <text x="200" y="112" text-anchor="middle">mã hóa hướng bằng ô 50 chiều</text>
    <rect x="336" y="20" width="244" height="100" rx="8" fill="#eff6ff" stroke="#1d4ed8"/>
    <text x="458" y="42" text-anchor="middle" font-weight="600">Encoder graph transformer</text>
    <text x="458" y="61" text-anchor="middle">attention giữa các cạnh chung đầu mút</text>
    <text x="458" y="78" text-anchor="middle">LapPE + residual / LayerNorm / FFN</text>
    <text x="458" y="95" text-anchor="middle">bottleneck z: 8…64 chiều</text>
    <text x="458" y="112" text-anchor="middle" fill="#475569">7 mô hình cho p = 2…8</text>
    <rect x="594" y="8" width="296" height="60" rx="8" fill="#f8fafc" stroke="#94a3b8"/>
    <text x="742" y="32" text-anchor="middle">Decoder DNN — MSE tái tạo</text>
    <text x="742" y="50" text-anchor="middle" fill="#475569">huấn luyện: chỉ benign</text>
    <rect x="594" y="72" width="296" height="60" rx="8" fill="#fef2f2" stroke="#b91c1c"/>
    <text x="742" y="96" text-anchor="middle">4 detector one-class (OCSVM, IF, HBOS, INNE)</text>
    <text x="742" y="114" text-anchor="middle">+ bỏ phiếu đa số → 0 / −1</text>
    <line x1="64" y1="70" x2="76" y2="70" stroke="#475569" marker-end="url(#ar)"/>
    <line x1="322" y1="70" x2="334" y2="70" stroke="#475569" marker-end="url(#ar)"/>
    <line x1="580" y1="70" x2="592" y2="38" stroke="#475569" marker-end="url(#ar)"/>
    <line x1="580" y1="70" x2="592" y2="102" stroke="#475569" marker-end="url(#ar)"/>
  </g>
</svg>
</div>

<span class="tag">Kết quả công bố: TPR 99,04% / FPR 6,56% ở p = 8 (CIC-IDS2017); ≥ 98% ở p = 8 khi chuyển sang CIC-IDS2018 và ACI-IoT-2023; MDT 1,28 µs/gói.</span>

<!--
S3 · 70 s — slide trọng tâm

"Ý tưởng gồm bốn bước. Bước một — biến lưu lượng thành đồ thị: mỗi nút là một cặp IP và
port, nút không có đặc trưng; mỗi cạnh là một luồng TCP.

Bước hai — đặc trưng của cạnh: lấy p gói đầu, mỗi gói bỏ Ethernet, options, payload, IP
và port, giữ đúng 25 byte header, chia 255. Gói đi đặt vào 25 ô đầu, gói về đặt vào 25 ô
sau, nên p gói thành 2×25×p chiều.

Bước ba — một autoencoder kiểu graph transformer học tái tạo đặc trưng cạnh, chỉ trên lưu
lượng bình thường. Khác biệt là attention không chạy giữa các nút mà giữa các cạnh dùng
chung đầu mút — nên embedding của một luồng mang ngữ cảnh của các luồng xung quanh.

Bước bốn — vứt decoder, giữ encoder, đưa embedding cho bốn bộ dò bất thường: OCSVM,
Isolation Forest, HBOS, INNE — rồi bỏ phiếu đa số."

THAO TÁC: nói đến đâu chỉ tay đến khối đó trên sơ đồ. Nhấn "chỉ học trên benign".
TRÁNH: đọc dòng chú thích cuối slide (TPR 99,04% / FPR 6,56%) — nếu còn thời gian mới nói
"p chạy từ 2 đến 8, mỗi p một model riêng".
CHUYỂN: "Nhóm em không chỉ chạy lại, mà còn đọc lại số liệu — và tìm ra ba điểm."
-->

---

# 3. Ba điểm nhóm phát hiện khi đọc lại số liệu

<div class="cols">
<div>

| p | Ensemble TPR/FPR | OCSVM TPR/FPR |
|---|---|---|
| 2 | 84,47 / **8,94** | 81,56 / 3,40 |
| 4 | 94,72 / **6,60** | 94,39 / 3,98 |
| 8 | 99,04 / **6,56** | 98,46 / 4,03 |

<span class="tag">Table VI — ensemble tăng TPR nhưng FPR cao hơn **mọi** detector đơn ở cả 7 giá trị p (5,91–8,94% so với 3,40–5,36%)</span>

</div>
<div>

1. **Ensemble thực chất thiên về báo động.** FPR 6,56% ⇒ ở 10.000 luồng/giờ là ~656 cảnh báo giả/giờ. Suy luận từ số liệu: luật phải là “≥ 2/4 phiếu”, không phải “≥ 3/4”
2. **Chuyển miền chưa đo được vế sai.** Thí nghiệm trên CIC-IDS2018 và ACI-IoT-2023 **chỉ chứa lưu lượng độc hại** ⇒ không có FPR ở môi trường mới
3. **Bias chưa bị loại sạch.** 25 byte giữ TTL, IP ID, IP/TCP checksum, seq/ack: hai checksum là **hàm của IP/port đã bị xóa** (rò rỉ gián tiếp); TTL là “lối tắt” đã biết của dữ liệu lab

</div>
</div>

**⇒ Cơ sở cho hai việc nhóm làm thêm:** hiệu chỉnh FPR có bảo đảm + dừng sớm (P1) và kiểm tra shortcut header (P2).

<!--
S4 · 50 s — slide "chất" nhất

"Đây là ba điểm nhóm em phát hiện khi đọc lại số liệu của bài.

Thứ nhất, ensemble làm FPR tăng: ở cả bảy giá trị p, FPR của ensemble là 5,9 đến 8,9
phần trăm, trong khi mọi detector đơn chỉ 3,4 đến 5,4 phần trăm. Nghĩa là bỏ phiếu đa số
đang thiên về báo động — và suy ngược từ số liệu thì luật phải là 'hai trên bốn phiếu',
không phải ba trên bốn.

Thứ hai, thí nghiệm chuyển miền sang CIC-IDS2018 và ACI-IoT chỉ chạy trên lưu lượng độc
hại, nên không có FPR — vế báo động giả ở môi trường mới chưa hề được chứng minh.

Thứ ba, 25 byte giữ lại TTL, checksum và seq/ack; hai checksum được tính từ chính IP và
port đã bị xóa, nên thông tin rò rỉ gián tiếp."

THAO TÁC: giơ ba ngón tay, mỗi ngón một điểm. Nhấn con số "8,9% so với 5,4%".
TRÁNH: nói "bài báo sai" — nói "chưa được chứng minh", giọng bình luận, không đối đầu.
CHUYỂN: "Ba điểm đó dẫn trực tiếp đến phần thực nghiệm và đề xuất của nhóm em."
-->

---

# 4. Thực nghiệm dự kiến

<div class="cols">
<div>

**Dữ liệu &amp; phạm vi**

- **CIC-IDS2017** (pcap ~51 GB, 5 ngày), chỉ **TCP/IPv4**
- Nhãn gói = nhãn luồng: bảng lịch tấn công CIC (giờ + IP attacker/victim) **đối chiếu chéo** CSV flow; nhãn “UNCERTAIN” bị loại và đếm riêng
- **MUST**: p ∈ {2, 4, 8}; benign + **DoS Hulk** (volumetric) + **PortScan** (không volumetric)
- **SHOULD**: p = 2…8, đủ 11 lớp tấn công Table VII, 4 detector + bỏ phiếu

**Chống rò rỉ (leakage)**

- Chia theo **luồng**, không bao giờ chia gói cùng luồng vào 2 tập
- **Protocol T (chính)**: train = thứ Hai (benign), test = thứ Ba–Sáu → temporal; **Protocol R** (kiểu paper) báo kèm để so sánh
- 3 seed, báo **mean ± std**; unit test `strip_headers` theo đúng vector hex của Fig. 3

</div>
<div>

**Tiêu chí đánh giá**

- **TPR, FPR** (chỉ số chính, như paper) + precision, F1, **AUROC, PR-AUC**
- **TPR@FPR = 1%** — so sánh công bằng giữa các detector
- **Số gói trung bình đến khi ra quyết định** (med, p95 theo lớp)
- Latency: µs/gói (so 1,28 µs của paper) **và** ms/luồng end-to-end — kể cả thời gian chờ đủ p gói

**Đối chứng (baseline) &amp; ablation**

- **B0** đặc trưng thô không đồ thị (≈ Table X “không ngữ cảnh”), **B1** MLP-autoencoder, **B2** E-GraphSAGE-autoencoder
- A1–A3: bỏ LapPE / thay transformer bằng GNN / ngưỡng lỗi tái tạo thay 4 detector; **A5** header hygiene (bỏ TTL, checksum)
- Bảng “**paper vs ours**” kèm giải thích từng chênh lệch

</div>
</div>

<!--
S5 · 60 s

"Về thực nghiệm. Dữ liệu là CIC-IDS2017, khoảng 51 GB pcap, nhóm em chỉ dùng TCP. Nhãn
gói lấy từ nhãn luồng, theo bảng lịch tấn công của CIC đối chiếu với file CSV flow; luồng
nào không khớp chắc chắn thì đánh dấu UNCERTAIN và loại ra, có đếm tỉ lệ.

Nhóm làm p bằng 2, 4 và 8; benign cộng hai tấn công đại diện là DoS Hulk — volumetric —
và PortScan — không volumetric. Chia tập theo luồng và theo ngày, chạy ba seed.

Chỉ số chính là TPR và FPR; đo thêm AUROC, PR-AUC, TPR tại FPR bằng 1%, và số gói trung
bình đến khi ra quyết định. Đối chứng ba mức: B0 đặc trưng thô không có đồ thị — đúng cột
'không ngữ cảnh' trong bài; B1 là autoencoder MLP; B2 là E-GraphSAGE — để biết đóng góp
thật đến từ đồ thị hay từ transformer."

THAO TÁC: nói "theo ngày" chậm và rõ — đây là điểm giảng viên hay gật đầu.
TRÁNH: sa vào chi tiết gán nhãn, một câu là đủ.
CẮT NẾU TRỄ: bỏ hẳn phần B0/B1/B2, chỉ nói "có ba mức đối chứng".
CHUYỂN: "Từ đó nhóm em đề xuất cải tiến P1."
-->

---

# 5. Cải tiến đề xuất P1 &amp; lộ trình

<div class="cols">
<div>

**P1 — Calibrated Early-Exit GTAE** *(cải tiến chính)*

- Thay 4 nhãn cứng + bỏ phiếu bằng **điểm liên tục** → **conformal p-value** tính trên tập calibration benign (không thuộc tập train)
- Gộp 4 p-value (Fisher / min–Bonferroni) → báo động khi p-value &lt; α ⇒ **FPR được chặn trên bởi α** (lý thuyết conformal), thay vì 6–9% không kiểm soát
- **Dừng sớm**: xét lần lượt p = 2 → 8 với ngân sách α chia theo bước → mỗi luồng có `p_decision` (đúng tinh thần “early detection” mà paper chỉ đề xuất, không đánh giá)
- **Tiêu chí thắng**: FPR thực đo ≤ α + 1% (α ∈ {1%, 5%}); TPR@FPR = 1% cao hơn bỏ phiếu đa số; số gói đến quyết định &lt; 8 mà TPR ≥ TPR(p = 8) − 1 điểm

<span class="tag">Không huấn luyện lại encoder ⇒ chi phí GPU ~0, đo được bằng số. P2 (header hygiene) chạy như một ablation.</span>

</div>
<div>

**Lộ trình 12 tuần (M = milestone)**

| | Tuần | Nội dung |
|---|---|---|
| M1 | 1–2 | Đọc paper, tài liệu hóa, chốt scope với GVHD (đã xong) |
| M2 | 3–4 | Parse pcap + gán nhãn + QA; encoder byte + dựng đồ thị; B0 |
| M3 | 5–6 | Cài GTAE, train p = 2, 4, 8 |
| M4 | 7–8 | 4 detector + bỏ phiếu; bảng Table VI/VII của nhóm; B1, B2 |
| M5 | 9–10 | Ablation + **P1** (3 seed); bảng so sánh với paper |
| M6 | 11–12 | Báo cáo, slide, demo, tổng duyệt |

**Kết quả dự kiến:** pipeline chạy end-to-end 1 lệnh train + 1 lệnh eval; đường cong TPR/FPR theo p; bảng “paper vs ours” có giải thích; P1 đo bằng TPR@FPR cố định + số gói đến quyết định. Mục tiêu là **tái hiện đúng xu hướng**, không đặt chỉ tiêu “đạt 99%”.

</div>
</div>

<!--
S6 · 50 s

"Từ đó nhóm em đề xuất P1. Thay vì bốn nhãn cứng rồi bỏ phiếu — vốn không có gì bảo đảm
về FPR — nhóm dùng conformal p-value: dành riêng một tập benign để hiệu chỉnh, mỗi
detector cho một điểm, quy thành p-value, báo động khi p-value nhỏ hơn alpha. Nếu tập
hiệu chỉnh và tập test hoán đổi được cho nhau, FPR bị chặn trên bởi alpha — có bảo đảm
về lý thuyết, không phải chọn tay.

Kèm theo là dừng sớm: xét lần lượt p từ 2 đến 8, chia ngân sách alpha theo từng bước;
luồng nào đủ bất thường thì báo ngay, nên mỗi luồng có một p_decision thay vì luôn chờ
đủ 8 gói.

Nhóm định trước tiêu chí thắng: FPR thực đo không vượt alpha cộng 1 điểm phần trăm; TPR
tại FPR một phần trăm cao hơn bỏ phiếu đa số; và số gói đến quyết định nhỏ hơn 8.
Lộ trình 12 tuần: M1 đã xong, M2 dữ liệu và đồ thị tuần 4, M3 train GTAE tuần 6, M4
detector tuần 8, M5 ablation và P1 tuần 10, M6 báo cáo và demo tuần 12."

THAO TÁC: phần lộ trình lướt tay ngang dòng chữ, đọc nhanh — đừng đọc từng chữ.
TRÁNH: giải thích công thức conformal; nếu bị hỏi sâu, trả lời "đếm tỉ lệ mẫu benign có
điểm bất thường cao hơn mẫu đang xét".
CẮT NẾU TRỄ: bỏ phần lộ trình, chỉ nói "12 tuần, hiện ở milestone 2".
-->

---

# 6. Kết luận &amp; tài liệu tham khảo

<div class="cols">
<div>

**Kết luận**

1. Tái hiện **có kiểm soát** GTAE-IDS trên CIC-IDS2017 với split theo thời gian, đánh giá FPR nghiêm ngặt hơn paper
2. Mọi siêu tham số paper **không công bố** (chiều LapPE k, chiều ẩn d, sampler subgraph, luật hòa phiếu) đều được ghi thành giả định + ablation
3. Cải tiến **P1** nhắm đúng điểm yếu đo được: FPR không kiểm soát của ensemble và cơ chế dừng sớm chưa được đánh giá

**Xin ý kiến GVHD:** chốt scope (bỏ ACI-IoT-2023), dùng protocol T làm kết quả chính, và có nên dùng nhãn đã sửa của Engelen et al. để đo nhiễu nhãn?

</div>
<div>

**Tài liệu tham khảo chính**

<div class="refs">

[1] J. Ghadermazi, S. Hore, A. Shah, N. D. Bastian, “GTAE-IDS: Graph Transformer-Based Autoencoder Framework for Real-Time Network Intrusion Detection,” *IEEE Trans. Inf. Forensics Security*, vol. 20, pp. 4026–4041, 2025. doi: 10.1109/TIFS.2025.3557741

[2] J. Ghadermazi, A. Shah, N. D. Bastian, “Towards Real-Time Network Intrusion Detection with Image-Based Sequential Packets Representation,” *IEEE Trans. Big Data*, vol. 11, no. 1, pp. 157–173, 2025. doi: 10.1109/TBDATA.2024.3403394

[3] E. Caville et al., “Anomal-E: A self-supervised network intrusion detection system based on graph neural networks,” *Knowledge-Based Systems*, vol. 258, Art. 110030, 2022. doi: 10.1016/j.knosys.2022.110030

[4] I. Sharafaldin, A. H. Lashkari, A. A. Ghorbani, “Toward Generating a New Intrusion Detection Dataset and Intrusion Traffic Characterization,” *ICISSP*, 2018, pp. 108–116. doi: 10.5220/0006639801080116

[5] Y. Mirsky et al., “Kitsune: An Ensemble of Autoencoders for Online Network Intrusion Detection,” *NDSS*, 2018. doi: 10.14722/ndss.2018.23204

[6] A. N. Angelopoulos, S. Bates, “A Gentle Introduction to Conformal Prediction and Distribution-Free Uncertainty Quantification,” arXiv:2107.07511, 2021

</div>

<span class="tag">Danh mục đầy đủ 24 tài liệu: `docs/GTAE-IDS_Research_docs.md`, mục Bibliography.</span>

</div>
</div>

<!--
S7 · 15 s

"Tóm lại: nhóm em tái hiện có kiểm soát, ghi rõ mọi giả định vì bài báo không công bố hết
siêu tham số, và P1 nhắm đúng điểm yếu đo được là FPR. Danh mục đầy đủ 24 tài liệu nằm
trong tài liệu thiết kế của nhóm. Em xin hết phần trình bày."

THAO TÁC: đứng thẳng, nhìn thầy/cô ở câu cuối, không nhìn slide.
TRÁNH: "cảm ơn mọi người đã lắng nghe" — sáo rỗng, mất 5 giây.

=== Q&A — trả lời 1–2 câu, đừng diễn giải dài ===

Hỏi: Sao gọi unsupervised khi vẫn cần benign?
 Đáp: Đúng hơn là one-class — train không dùng nhãn tấn công, nhưng vẫn cần tập benign sạch.

Hỏi: Bỏ IP/port mà sao vẫn dựng đồ thị bằng IP/port?
 Đáp: IP/port để xác định ai nói với ai, không đi vào đặc trưng; nhưng cấu trúc vẫn mang
 thông tin môi trường — đó là hạn chế nhóm em đã ghi.

Hỏi: Sao không dùng p = 1?
 Đáp: Gói đầu là SYN, benign và tấn công giống nhau nên vô thông tin.

Hỏi: P1 khác gì bài gốc?
 Đáp: Không đổi encoder; chỉ thay tầng quyết định — bỏ phiếu cứng thành p-value có hiệu
 chỉnh, kèm dừng sớm.

Hỏi: Kết quả nhóm có bằng paper không?
 Đáp: Không kỳ vọng bằng — khác split, khác nhãn, và vài siêu tham số bài không công bố.
 Mục tiêu là tái hiện xu hướng, có bảng giải thích chênh lệch.
-->
