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
Ngân sách thời gian (tổng ~300 s):
S1 15s · S2 45s · S3 70s · S4 50s · S5 60s · S6 50s · S7 15s
Nguyên tắc: mỗi slide 1 thông điệp; số liệu đọc từ slide, không đọc lại chữ.
-->

<!-- _class: lead -->

# GTAE-IDS: tái hiện &amp; cải tiến

## Phát hiện xâm nhập sớm bằng Graph Transformer Autoencoder

**Đồ án hướng nghiên cứu** · NT204.R11.ANTN — Hệ thống tìm kiếm, phát hiện và ngăn ngừa xâm nhập (IDPS)

Nhóm 8 · Báo cáo giữa kỳ · 19/10/2026

<span class="tag">Bài báo neo: J. Ghadermazi et al., “GTAE-IDS: Graph Transformer-Based Autoencoder Framework for Real-Time Network Intrusion Detection,” IEEE TIFS, vol. 20, pp. 4026–4041, 2025, doi: 10.1109/TIFS.2025.3557741</span>

<!--
Mở đầu 15s: "Nhóm em chọn hướng nghiên cứu: tái hiện có kiểm soát một bài báo TIFS 2025,
đo lại các con số của bài, và đề xuất một cải tiến dựa trên điểm yếu đo được."
KHÔNG đọc tên bài báo dài — chỉ nói "bài TIFS 2025 về GTAE-IDS".
-->

---

# 1. Vấn đề của bài báo

**Ba hạn chế của NIDS dùng ML/DL hiện nay** — lý do bài báo đặt vấn đề

1. **Flow-based**: đặc trưng tổng hợp của cả luồng → quyết định chỉ ra sau khi luồng kết thúc/timeout ⇒ phù hợp phân tích hồi cứu, không phát hiện online
2. **Dữ liệu phẳng** (vector/lưới) → bỏ qua quan hệ cấu trúc giữa các thực thể mạng: PortScan = một nguồn → nhiều đích; DDoS = nhiều nguồn → một đích
3. **Phụ thuộc nhãn** (supervised/semi-supervised) → gán nhãn đồ thị đắt, và yếu trước tấn công có mẫu cấu trúc chưa thấy

**Khoảng trống bài báo tự xác lập** (khảo sát 15 công trình GNN-NIDS, Table I): chỉ **1/15** là unsupervised, và công trình đó vẫn **flow-based** ⇒ chưa có hệ **packet-based + unsupervised**.

**Câu hỏi của nhóm:** tái hiện được tới mức nào, và **cái giá sai (FPR)** của phương pháp là bao nhiêu?

<!--
45s. Điểm nhấn: gap là "packet-based + unsupervised" (đúng theo bảng khảo sát của chính tác giả).
Câu chốt cuối slide là cầu nối sang slide 4 (đóng góp của nhóm = đo lại FPR).
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
70s — slide trọng tâm. Nói theo mạch: "luồng thành cạnh, gói thành đặc trưng cạnh 25 byte,
học chỉ trên benign, rồi tách encoder cho 4 detector bỏ phiếu."
Nhấn: nút KHÔNG có đặc trưng (chỉ cấu trúc); attention là giữa CÁC CẠNH (khác transformer chuẩn trên nút).
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
50s. Đây là phần "chất" nhất: không chỉ tái hiện mà đọc lại số liệu để tìm điểm yếu.
Câu chốt: FPR của ensemble là chi phí vận hành thật, và paper không kiểm soát nó.
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
60s. Nói nhanh, đây là slide "cam kết phương pháp".
Nhấn 2 điểm GVHD hay hỏi: (i) split theo thời gian chứ không random; (ii) đo FPR chứ không chỉ TPR.
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
50s. Mạch nói: "đề xuất không nằm ở kiến trúc mà ở tầng quyết định: hiệu chỉnh FPR và dừng sớm."
Nếu bị hỏi "sao không đổi kiến trúc?": chi phí thấp, đo được, và nhắm đúng điểm yếu đã chứng minh từ Table VI,
còn thay kiến trúc (P3/P4/P5) thì khó quy kết nguyên nhân và vượt thời gian.
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
15s. Chốt: "Xin thầy/cô cho ý kiến về 3 câu hỏi ở cột trái và về P1 có đủ tính 'cải tiến phương pháp' không."
-->
