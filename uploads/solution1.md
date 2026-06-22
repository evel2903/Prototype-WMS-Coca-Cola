# Giải pháp Quản lý Kho & Phân phối cho Coca-Cola

## 01. Bối cảnh và mục tiêu

### 01.01. Hiện trạng vận hành

Mạng phân phối gồm ba luồng hàng:

| Mã | Luồng | Phương tiện | Đặc điểm |
|---|---|---|---|
| L1 | Nhà máy → Kho tập trung | Container | Hàng thành phẩm sau sản xuất, chuyển theo lô lớn. |
| L2 | Kho tập trung → Đại lý / Điểm bán | Xe tải (lớn/nhỏ tùy đơn) | Phân phối theo tuyến, nhiều điểm giao. |
| L3 | Khách tự tới Kho tập trung lấy hàng | Xe của khách | Tự lấy (self-pickup), khách trực tiếp đến kho. |

### 01.02. Ba tính chất bắt buộc của giải pháp

Mọi nghiệp vụ trong giải pháp phải đồng thời thỏa ba tính chất sau. Đây là kim chỉ nam thiết kế, không phải tùy chọn.

| Tính chất | Yêu cầu | Hệ quả thiết kế |
|---|---|---|
| **Đích danh** | Ngăn ngừa **trước** khi sự việc xảy ra (Prev Check). | Mọi đối tượng được định danh và đối chiếu với kế hoạch trước khi thao tác; sai thì **chặn ngay**, không sửa sau. |
| **Minh bạch** | Phải có dữ liệu trên hệ thống để kiểm tra, chống gian lận. | Mọi thay đổi tồn/trạng thái đều có bản ghi không sửa được, truy vết được ai/làm gì/khi nào/tại sao. |
| **Tức thời** | Thông tin cập nhật ngay lập tức để xử lý kịp thời. | Mọi thao tác phát sinh dữ liệu thời gian thực; điều phối/giám sát thấy trạng thái và cảnh báo ngay. |

### 01.03. Phạm vi

Trong phạm vi: quản lý tồn theo vị trí và lô, nhận hàng từ nhà máy, lưu kho, xuất cho đại lý/điểm bán, khách tự lấy, truy vết lô và thu hồi, vỏ/két trả về (tùy chọn), kiểm soát truy cập và đối soát.

Ngoài phạm vi: kế toán/công nợ đại lý (do hệ thống bán hàng/tài chính giữ; giải pháp chỉ nhận và áp cờ chặn nếu có), định tuyến vận tải chi tiết, thiết kế phần cứng/giao diện chi tiết.

## 02. Ba nguyên tắc thiết kế (chi tiết hóa ba tính chất)

### 02.01. Đích danh — Cổng kiểm soát ngăn ngừa (Prev Check)

**Định nghĩa**: Trước mỗi hành động làm đổi tồn hoặc bàn giao hàng, hệ thống đặt một **cổng kiểm soát**. Tại cổng, người thao tác phải **quét/định danh** đối tượng và hệ thống **đối chiếu với kế hoạch/chứng từ**. Nếu khớp → cho phép; nếu sai → **chặn cứng** (không cho hoàn tất). Mục tiêu là chặn lỗi/gian lận **trước khi xảy ra**, thay vì phát hiện sau khi hàng đã đi.

**Đối tượng cần định danh xuyên suốt**:

| Nhóm | Định danh | Đối chiếu với |
|---|---|---|
| Hàng | Mã SKU + **lô + ngày sản xuất (NSX) + hạn sử dụng (HSD)** | Master hàng, chứng từ nhập/xuất |
| Đơn vị lưu trữ | Mã pallet/kiện duy nhất (gọi là **mã LPN**, dán nhãn mã vạch) | Danh sách hàng trên chứng từ |
| Vị trí | Mã vị trí trong kho (zone/dãy/kệ/ô) | Quy tắc cất/lấy |
| Chuyến nhà máy (L1) | Số container + **số niêm phong (seal)** | Phiếu báo hàng đến (ASN) |
| Đơn hàng (L2/L3) | Số đơn + đại lý/điểm bán nhận (ship-to) + tuyến | Hệ thống bán hàng |
| Phương tiện | Biển số xe + tài xế | Lịch hẹn / đơn |
| Người nhận (L3) | Mã đơn/OTP + giấy ủy quyền + CMND/CCCD | Đơn được duyệt |

**Quy tắc cổng**: thao tác chỉ hợp lệ khi quét đủ và đúng; nhập tay (khi mã hỏng) là ngoại lệ cần lý do + người duyệt + ghi nhận.

### 02.02. Minh bạch — Dữ liệu kiểm tra được, chống gian lận

**Định nghĩa**: Không có thay đổi tồn nào xảy ra "ngầm". Mọi giao dịch để lại dấu vết đầy đủ và **không thể sửa/xóa lén**; chỉ đính chính bằng bản ghi mới có tham chiếu.

**Cơ chế**:

| Cơ chế | Mô tả |
|---|---|
| Giao dịch tồn (transaction) | Tồn chỉ đổi qua giao dịch có: người thao tác, thời điểm, từ/đến vị trí, lô, số lượng, chứng từ gốc. Không sửa số tồn trực tiếp. |
| Nhật ký không sửa được (audit) | Ghi cả giá trị trước/sau cho mọi thao tác quan trọng; chỉ thêm bản đính chính, không xóa. |
| Mã lý do chuẩn | Mọi ngoại lệ/can thiệp (điều chỉnh, đổi lô, hủy, override) phải gắn mã lý do + bằng chứng (ảnh) theo cấu hình. |
| Tách quyền (chống tự duyệt) | Người thực hiện ≠ người duyệt cho các hành động rủi ro (điều chỉnh tồn, thả hàng, hủy đơn). |
| Đối soát hai đầu | Hàng nhà máy gửi vs kho tập trung nhận phải khớp; lệch → tạo hồ sơ chênh lệch có người chịu trách nhiệm. |
| Truy vết lô đầu-cuối | Mỗi lô biết: đến từ chuyến/ASN nào; đã vào tồn nào; đã giao đại lý/điểm bán nào, qua đơn/xe nào. |

**Các kịch bản gian lận điển hình và cách chặn**:

| Rủi ro | Cách chặn |
|---|---|
| Lấy/giao nhiều hơn đơn (tuồn hàng) | Quét từng pallet/kiện lên xe đối chiếu đơn; thừa/thiếu → không cho đóng chuyến. |
| Giao sai đại lý / sai tuyến (bán phá tuyến) | Đơn gắn cứng đại lý + tuyến; xuất kho ghi rõ đại lý + lô; truy vết lô. |
| Đổi lô tốt lấy lô cận hạn | Hệ thống chỉ định đúng lô theo FEFO; quét sai lô → chặn; mọi thay lô có nhật ký. |
| Khai khống hao hụt/vỡ | Điều chỉnh tồn cần lý do + người duyệt + ảnh bằng chứng. |
| Tráo hàng trên đường (L1) | Đối chiếu số niêm phong (seal) khi container tới; lệch → chặn dỡ hàng. |
| Thất thoát vỏ/két (nếu áp dụng) | Đối soát số vỏ giao vs nhận theo từng đại lý. |

### 02.03. Tức thời — Thời gian thực

**Định nghĩa**: Dữ liệu cập nhật **ngay tại thời điểm thao tác**, không chờ nhập liệu cuối ca; điều phối và giám sát có thông tin + cảnh báo để can thiệp kịp.

**Cơ chế**:

| Cơ chế | Mô tả |
|---|---|
| Thiết bị cầm tay (RF/handheld/mobile) | Mỗi lần quét tạo bản ghi tức thì; xác nhận thao tác lên hệ thống ngay. |
| Theo dõi hàng trên đường (in-transit) | Container rời nhà máy → trạng thái "đang vận chuyển" hiển thị cho cả hai đầu. |
| Bảng điều khiển thời gian thực | Tồn, hàng chờ dock, đơn trễ, thiếu hàng, ngoại lệ hiển thị realtime. |
| Cảnh báo có chủ thể xử lý | Xe chờ quá lâu, đơn sắp trễ cut-off, thiếu hàng tại vị trí lấy → cảnh báo ngay tới đúng người. |
| Xác nhận giao hàng (POD) tức thời | Khi giao đại lý hoặc khách lấy xong → ký/ảnh xác nhận cập nhật ngay. |

## 03. Đặc thù hàng hóa Coca-Cola và yêu cầu kéo theo

| Đặc thù | Mô tả | Yêu cầu giải pháp |
|---|---|---|
| Nước giải khát, bán nhanh | Sản lượng lớn, vòng quay nhanh | Bố trí hàng bán chạy gần khu xuất; thao tác chủ yếu theo pallet/thùng. |
| **Đơn vị bán chủ yếu: thùng + pallet** | Bán theo **thùng (case)** và **pallet** là chính; **lon lẻ và lốc rất hiếm hoặc không có** | Quy đổi **pallet ↔ thùng** là trục chính; lấy lẻ từng lon/lốc xem như **ngoại lệ có kiểm soát**, không thiết kế dây chuyền pick lẻ. |
| Lô + NSX + HSD | Mỗi sản phẩm có lô và hạn dùng in trên bao bì | Bắt buộc ghi nhận lô/NSX/HSD khi nhập; xuất theo **FEFO** (hết hạn trước, xuất trước). |
| Hạn còn tối thiểu khi giao | Đại lý/điểm bán yêu cầu HSD còn đủ dài | Cấu hình "số ngày hạn tối thiểu" theo đại lý; chặn cấp lô không đủ hạn. |
| An toàn thực phẩm / thu hồi | Phải thu hồi nhanh theo lô khi sự cố | Truy vết lô đầu-cuối; khóa thu hồi (recall hold) cho phạm vi lô. |
| Trọng lượng nặng (chất lỏng) | Pallet nặng, giới hạn tải kệ | Kiểm soát sức chứa theo **trọng lượng** và **số slot pallet**, không chỉ số lượng. |
| Lô lớn đồng nhất | Chủ yếu nhập/xuất theo nguyên pallet và thùng | Hai chế độ lấy hàng chính: **nguyên pallet** và **lấy theo thùng (case pick)**; không pick lẻ từng lon. |
| Vỏ chai thủy tinh + két nhựa (nếu có) | Bao bì quay vòng (returnable) | Quản lý vỏ/két như tài sản tuần hoàn; đối soát theo đại lý (mục 09). |
| Khuyến mãi/combo | Combo Tết, gift pack, bundle | Hỗ trợ **đóng combo (kitting)**; SKU khuyến mãi theo mùa. |
| Cao điểm Tết/hè | Sản lượng tăng đột biến | Bố trí lại vị trí hàng bán chạy; chuyển thẳng (cross-dock); cảnh báo quá tải. |
| Chống tuồn hàng/sai tuyến | Bảo vệ tuyến phân phối | Đơn gắn cứng đại lý/tuyến; truy vết lô; nhật ký đầy đủ. |

## 04. Kiến trúc giải pháp tổng thể

### 04.01. Các điểm trong mạng

| Điểm | Vai trò | Ghi chú |
|---|---|---|
| Nhà máy | Nguồn thành phẩm; đóng pallet, dán nhãn, niêm container | Là điểm xuất của L1. |
| Kho tập trung | Trung tâm phân phối: nhận – lưu – xuất | Trung tâm của cả ba luồng. |
| Đại lý / Điểm bán | Điểm nhận hàng | Nơi xác nhận giao (POD). |

### 04.02. Mô hình tồn kho (nền của minh bạch)

- Danh tính tồn tối thiểu: **Sản phẩm + Lô + Vị trí + Trạng thái + Số lượng**, gắn với **mã pallet (LPN)** khi quản lý theo pallet.
- Trạng thái tồn chính:

| Trạng thái | Ý nghĩa | Được phép xuất? |
|---|---|---|
| Khả dụng | Đã nhập, đạt kiểm tra, sẵn sàng bán | Có |
| Chờ kiểm tra | Vừa nhận, chờ lấy mẫu/kiểm | Không |
| Giữ / Cách ly | Nghi vấn, lỗi, hoặc thu hồi | Không |
| Đang vận chuyển (in-transit) | Đã rời nhà máy, chưa nhận tại kho tập trung | Không (chỉ hiển thị) |
| Hết hạn / cận hạn dưới ngưỡng | Không đủ điều kiện giao | Không |

- **Quy tắc lõi**: tồn chỉ thay đổi qua giao dịch hợp lệ; "Khả dụng" không bao gồm hàng đang giữ/cách ly/chờ kiểm/đang vận chuyển; chuyển kho không làm hàng "biến mất" (đi qua trạng thái đang vận chuyển).

### 04.03. Mô hình định danh (nền của đích danh)

Mỗi đối tượng có một mã định danh duy nhất, gắn mã vạch để quét:

- **SKU** (kèm cờ kiểm soát lô/HSD).
- **Pallet/kiện = LPN** (1 mã quét, biết chứa lô gì, số lượng bao nhiêu, đang ở đâu).
- **Container + seal** (chuyến nhà máy).
- **Đơn hàng + đại lý/ship-to + tuyến**.
- **Xe + tài xế**.
- **Người nhận** (L3).

## 05. Luồng L1 — Nhà máy → Kho tập trung (Container)

### 05.01. Các bước và cổng kiểm soát

| Bước | Nghiệp vụ | Cổng kiểm soát (Đích danh / Prev Check) |
|---|---|---|
| 1 | Lập phiếu điều hàng nhà máy → kho tập trung | Phiếu có đủ: hàng, lô dự kiến, số lượng, kho đi/đến. |
| 2 | Nhà máy đóng pallet, dán nhãn LPN, gắn lô/NSX/HSD | Mỗi pallet một mã LPN duy nhất. |
| 3 | Xếp container, niêm seal, phát phiếu báo hàng (ASN) | ASN ghi số container, **số seal**, danh sách LPN. |
| 4 | Xác nhận xuất khỏi nhà máy | Tồn nhà máy giảm → tồn "đang vận chuyển" tăng theo chuyến. |
| 5 | Container tới kho tập trung — **kiểm tại cổng** | Đối chiếu biển số, tài xế, **số seal = ASN**; sai → chặn dỡ hàng. |
| 6 | Dỡ và nhận theo từng LPN | Quét LPN; đối chiếu danh sách ASN; thiếu/thừa → ghi chênh lệch. |
| 7 | Kiểm tra chất lượng (lấy mẫu theo lô) | Ghi kết quả; lô chưa đạt → giữ/cách ly. |
| 8 | Đối soát hai đầu | Số xuất nhà máy = số đang vận chuyển = số nhận; lệch → hồ sơ chênh lệch. |
| 9 | Cất hàng (gợi ý vị trí theo FEFO/sức chứa/trọng lượng) | Quét vị trí đích; sai zone/quá tải → chặn. |
| 10 | Chuyển "Khả dụng" | Chỉ sau khi nhận + kiểm hợp lệ. |

### 05.02. Ba tính chất tại L1

- **Đích danh**: seal + container + LPN + lô đối chiếu ASN trước khi dỡ → chặn tráo hàng trên đường.
- **Minh bạch**: tồn "đang vận chuyển" thấy được hai đầu; đối soát chặn đóng chuyển nếu lệch; mọi LPN truy được về lô.
- **Tức thời**: trạng thái chuyến cập nhật từ lúc rời nhà máy → kho tập trung chủ động bố trí dock và nhân lực.

### 05.03. Cơ hội chuyển thẳng (cao điểm)

Nếu khi container về, kho đang có đơn đại lý chờ đúng sản phẩm đó, hàng có thể đẩy thẳng khu xuất thay vì cất kho (cross-dock) — vẫn phải qua kiểm chất lượng bắt buộc và giữ quét + nhật ký.

## 06. Luồng L2 — Kho tập trung → Đại lý/Điểm bán (Xe tải)

### 06.01. Các bước và cổng kiểm soát

| Bước | Nghiệp vụ | Cổng kiểm soát (Đích danh / Prev Check) |
|---|---|---|
| 1 | Nhận đơn từ hệ thống bán hàng | Đại lý hợp lệ, ship-to, tuyến, sản phẩm còn bán; áp **cờ chặn công nợ** nếu có. |
| 2 | Giữ hàng theo FEFO + hạn còn tối thiểu | Chỉ định đúng lô gần hết hạn nhất còn đủ điều kiện; chặn lô không đủ hạn của đại lý. |
| 3 | Chọn chế độ lấy: nguyên pallet hoặc theo thùng | Đơn chẵn pallet → nguyên pallet; còn lại → lấy theo thùng (case pick); pick lẻ lon/lốc chỉ khi ngoại lệ. |
| 4 | Gom đơn theo tuyến / cỡ xe / giờ cut-off | Tạo "đợt lấy hàng" (wave) theo chuyến và cỡ xe (lớn/nhỏ). |
| 5 | Lấy hàng (pick) | Quét vị trí + sản phẩm + lô + LPN; sai → chặn. |
| 6 | Kiểm trước khi xếp xe | Đối chiếu sản phẩm/số lượng/lô theo đơn. |
| 7 | Gán cửa xuất (dock) theo cỡ xe | Cửa phù hợp xe lớn/nhỏ; theo lịch. |
| 8 | Xếp lên đúng xe | Quét từng pallet/kiện vào chuyến = danh sách đơn; thiếu/thừa → không đóng chuyến. |
| 9 | Cho xe ra cổng | Đối chiếu xe/tài xế/đơn/chứng từ; chuyến phải ở trạng thái "đã xếp xong". |
| 10 | Ghi giảm tồn (xuất kho) tại cổng ra | Giảm đúng lô/số lượng; ghi rõ đại lý. |
| 11 | Xác nhận giao tại đại lý (POD) | Ký/ảnh xác nhận; cập nhật ngay. |

### 06.02. Xe lớn/nhỏ theo đơn

Loại xe là thuộc tính của chuyến: gom đơn theo tuyến rồi chọn cỡ xe; cửa xuất phải phù hợp loại xe. Đơn lớn/chẵn pallet → **nguyên pallet** để nhanh và ít chạm hàng; đơn nhỏ hơn → **lấy theo thùng (case)**. Lấy lẻ từng lon/lốc rất hiếm, nếu có thì xử lý như ngoại lệ có kiểm soát.

### 06.03. Thời điểm ghi giảm tồn — khuyến nghị "tại cổng ra"

Chọn ghi giảm tồn **khi xe thực sự ra cổng** (không phải lúc xếp xong): tồn chỉ giảm khi hàng đã rời kho với đúng xe/đúng hàng → tăng tính đích danh và minh bạch. Xác nhận giao tại đại lý (POD) là mốc riêng để xác nhận đã giao, tách khỏi mốc giảm tồn kho.

### 06.04. Ba tính chất tại L2

- **Đích danh**: hệ thống chỉ định đúng lô (FEFO) → người lấy quét đúng lô; xếp xe quét khớp danh sách → không giao thiếu/thừa/sai đại lý.
- **Minh bạch**: xuất kho ghi rõ lô + đại lý + xe → truy được "lô X đã giao đại lý Y" cho thu hồi và chống tuồn hàng.
- **Tức thời**: thiếu hàng tại vị trí lấy báo ngay để bổ sung; POD cập nhật trạng thái giao ngay khi tới đại lý.

## 07. Luồng L3 — Khách tự tới lấy hàng (Self-pickup)

Rủi ro lớn nhất là **mạo danh người nhận** và **lấy khống/sai** → trọng tâm là định danh người nhận khớp đơn đã duyệt.

### 07.01. Các bước và cổng kiểm soát

| Bước | Nghiệp vụ | Cổng kiểm soát (Đích danh / Prev Check) |
|---|---|---|
| 1 | Đơn lấy hàng tạo trước trên hệ thống bán hàng | Đơn hợp lệ, đại lý, cờ công nợ. |
| 2 | Khách đến — **kiểm tại cổng** | Mã đơn/OTP + giấy ủy quyền + CMND/CCCD + biển số khớp đơn; sai → chặn. |
| 3 | Lấy hàng (hoặc đã lấy sẵn, để khu chờ) | FEFO + hạn còn tối thiểu. |
| 4 | Bàn giao | Quét từng pallet/kiện theo đơn; khách xác nhận. |
| 5 | Cho ra cổng | Đối chiếu đơn + người nhận + xe; chưa đủ → chặn. |
| 6 | Ghi giảm tồn + ký nhận tại chỗ | Giảm tồn; lưu chứng từ định danh người nhận. |

### 07.02. Khách đến không có đơn

→ **Chặn**, hoặc cho nhận tạm khi có **người duyệt** (điều phối) kèm lý do; không tự ý xuất khi chưa có đơn được duyệt.

### 07.03. Ba tính chất tại L3

- **Đích danh**: người nhận + đơn + xe phải khớp tại cổng trước khi giao (chống mạo danh).
- **Minh bạch**: giấy ủy quyền/OTP/CMND lưu kèm phiếu xuất; nhật ký người duyệt nhận tạm.
- **Tức thời**: xuất kho + ký nhận cập nhật ngay khi khách rời cổng.

## 08. An toàn thực phẩm: FEFO và Thu hồi (Recall)

### 08.01. FEFO và hạn còn tối thiểu

- Hệ thống luôn chỉ định **lô gần hết hạn nhất còn đủ điều kiện** khi giữ hàng cho đơn.
- Cấu hình **số ngày hạn tối thiểu** theo đại lý/sản phẩm; lô không đủ hạn bị **chặn** không cho cấp.
- Hàng hết hạn/cận hạn dưới ngưỡng tự động chuyển trạng thái không khả dụng, không lọt vào đơn giao.

### 08.02. Thu hồi theo lô

Khi có sự cố lô (đây là rủi ro cao của ngành nước giải khát):

- **Truy ngược**: lô đến từ chuyến/ASN/nhà máy nào.
- **Truy xuôi**: lô đã vào tồn nào còn trong kho; đã giao đại lý/điểm bán nào, qua đơn/xe nào.
- **Khóa thu hồi**: chặn giữ hàng/lấy/xuất cho phạm vi lô bị ảnh hưởng; đóng băng tồn còn trong kho.
- Lập nhanh danh sách đại lý đã nhận lô để phối hợp thu hồi.

→ Vì rủi ro pháp lý cao, năng lực truy vết lô + thu hồi cần được đưa vào **ngay từ giai đoạn đầu** (xem lộ trình mục 13).

## 09. Vỏ chai / két trả về (tùy chọn — nếu Coca dùng bao bì quay vòng)

| Nội dung | Thiết kế |
|---|---|
| Quản lý vỏ/két như tài sản tuần hoàn | Mã riêng cho vỏ/két; theo dõi "tồn vỏ tại kho" và "vỏ đang ở đại lý". |
| Nhận vỏ khi xe quay về | Quét đếm số vỏ/két thu hồi mỗi chuyến. |
| Đối soát theo đại lý | So vỏ đã giao vs vỏ đã nhận theo từng đại lý → minh bạch, chống thất thoát. |
| Xử lý vỏ hỏng | Loại bỏ vỏ hỏng có lý do + người duyệt. |

## 10. Đối tượng dữ liệu chính

| Đối tượng | Định danh | Thuộc tính then chốt |
|---|---|---|
| Sản phẩm (SKU) | Mã SKU | Cờ lô/HSD; đơn vị đo chính **thùng + pallet** (lon/lốc hiếm); quy đổi pallet↔thùng; trọng lượng; ngưỡng hạn tối thiểu |
| Pallet/kiện (LPN) | Mã LPN | Lô, NSX, HSD, số lượng, vị trí, trạng thái |
| Container (L1) | Số container + seal | ASN, danh sách LPN |
| Phiếu điều hàng (L1) | Số phiếu | Kho đi/đến, dòng hàng, số đang vận chuyển |
| Đơn hàng (L2/L3) | Số đơn | Đại lý/ship-to, tuyến, dòng hàng, cờ công nợ, cờ tự lấy |
| Đại lý/Điểm bán | Mã đại lý | Tuyến, ngưỡng hạn tối thiểu |
| Xe/Tài xế | Biển số/Mã tài xế | Cỡ xe, lịch hẹn |
| Vỏ/két (tùy chọn) | Mã tài sản | Tồn tại kho / tại đại lý |
| Giao dịch tồn | Mã giao dịch | Người, thời điểm, từ/đến, lô, chứng từ |

## 11. Ma trận điểm kiểm soát × ba tính chất

Bảng trung tâm: mỗi cổng đảm bảo cả ba tính chất ra sao.

| Cổng kiểm soát | Đích danh (ngăn ngừa) | Minh bạch | Tức thời |
|---|---|---|---|
| Cổng container (L1) | Seal + biển số + tài xế khớp ASN → chặn nếu sai | Bản ghi cổng + ảnh seal | Trạng thái cập nhật ngay |
| Nhận theo LPN (L1) | Quét LPN khớp ASN; kiểm lô | Đối soát hai đầu; truy vết lô | Sự kiện nhận realtime |
| Cất hàng (L1) | Quét vị trí đúng zone/sức chứa/trọng lượng | Giao dịch tồn | Cập nhật vị trí ngay |
| Giữ hàng cho đơn (L2/L3) | FEFO khóa đúng lô + hạn tối thiểu | Nhật ký lô được giữ; lý do nếu đổi | Báo thiếu hàng ngay |
| Lấy hàng (L2/L3) | Quét vị trí/sản phẩm/lô/LPN → chặn sai | Bản ghi lấy có người + lô | Sự kiện lấy realtime |
| Xếp xe (L2/L3) | Quét từng kiện vào đúng xe/đơn | Thiếu/thừa → không đóng chuyến | Cảnh báo ngay |
| Cổng người nhận (L3) | OTP/ủy quyền/CMND + biển số khớp đơn | Chứng từ định danh lưu kèm | Bản ghi cổng realtime |
| Cổng ra (L2/L3) | Xe/tài xế/đơn/chứng từ khớp; chuyến đã xếp xong | Bản ghi cổng + chứng từ | Sự kiện ra cổng ngay |
| Ghi giảm tồn | Trừ đúng lô/đại lý/số lượng tại cổng ra | Giao dịch xuất kho | Cập nhật tồn ngay |
| Xác nhận giao (POD) | Đúng người/đại lý nhận | Chữ ký/ảnh lưu | Cập nhật giao ngay |
| Thu hồi | Khóa thu hồi theo phạm vi lô | Truy vết xuôi/ngược đầy đủ | Khóa và cảnh báo ngay |

## 12. Phân quyền, ngoại lệ và xử lý

### 12.01. Phân quyền và tách quyền (chống gian lận)

- Tách quyền **lấy/xếp hàng** khỏi quyền **duyệt điều chỉnh/ghi đè**.
- Tách quyền **kiểm chất lượng** khỏi quyền **thả hàng ra bán**.
- Bảo vệ/cổng có quyền kiểm xe nhưng **không** thay nghiệp vụ nhận/xuất kho.
- Người dùng chỉ thấy/thao tác trong phạm vi kho và đại lý được phân quyền.

### 12.02. Ngoại lệ trọng yếu và hành vi mặc định

| Tình huống | Hành vi mặc định |
|---|---|
| Sai số niêm phong (seal) container | Chặn dỡ hàng; điều tra; lưu bằng chứng. |
| Lấy/giao sai lô (không theo FEFO) | Chặn xác nhận; yêu cầu lấy đúng lô. |
| Xếp xe thiếu/thừa/sai đơn | Cảnh báo, không đóng chuyến; người giám sát xử lý. |
| Khách tự lấy không đúng người/không đơn | Chặn ra cổng; điều phối xử lý. |
| Lô cận/hết hạn lọt vào đơn | Chặn; chuyển giữ hàng; xử lý hủy/đổi. |
| Lệch vỏ/két với đại lý | Tạo hồ sơ chênh lệch + lý do; đối soát. |
| Đồng bộ với hệ thống bán hàng/ERP lỗi sau khi xuất | Giữ giao dịch đã xác nhận + thử lại; không để mất dữ liệu. |

Mọi ngoại lệ có vòng đời: phát hiện → ghi nhận (lý do + bằng chứng) → giao người xử lý → duyệt nếu cần → xử lý → đóng. Không xóa hồ sơ ngoại lệ để "làm sạch".

## 13. Chỉ số đo lường (KPI)

| Nhóm | Chỉ số |
|---|---|
| Chất lượng tồn | Độ chính xác tồn; tuân thủ FEFO; tỷ lệ hàng cận/hết hạn. |
| Nhập (L1) | Thời gian quay vòng container; tỷ lệ chênh lệch nhận; thời gian "đang vận chuyển". |
| Xuất (L2/L3) | Độ chính xác đơn; giao đúng giờ; tỷ lệ thiếu hàng khi lấy. |
| An toàn thực phẩm | **Thời gian dựng được danh sách đại lý đã nhận theo lô** (sẵn sàng thu hồi). |
| Vỏ/két (nếu có) | Tỷ lệ đối soát vỏ/két theo đại lý. |
| Chống gian lận | Số ngoại lệ tuồn hàng/sai tuyến phát hiện & chặn tại cổng. |

## 14. Lộ trình triển khai

| Giai đoạn | Nội dung | Mục tiêu |
|---|---|---|
| GĐ 0 — Nền tảng | Dữ liệu sản phẩm (kèm lô/HSD), đại lý/tuyến, kho/vị trí, mã pallet (LPN), phân quyền. | Chuẩn dữ liệu và định danh trước khi vận hành. |
| GĐ 1 — Luồng lõi | Khép kín 3 luồng tối thiểu: nhận container (L1), xuất xe tải (L2), khách tự lấy (L3); thiết bị quét; ghi giảm tồn; **bắt buộc ghi lô/NSX/HSD + FEFO + truy vết xuôi**. | Phiên bản dùng được đầu tiên, đã có đích danh/minh bạch/tức thời ở mức cốt lõi. |
| GĐ 2 — Hoàn thiện | "Đang vận chuyển" + đối soát hai đầu chặt; **thu hồi theo lô đầy đủ**; vỏ/két trả về; báo cáo & KPI. | An toàn thực phẩm + chống thất thoát. |
| GĐ 3 — Điều phối | Lịch hẹn cổng/dock cho container & xe tải; điều phối nhân lực cao điểm. | Giảm ùn tắc, tối ưu ca. |
| GĐ 4 — Tối ưu | Bố trí lại vị trí hàng bán chạy theo mùa (Tết/hè); chuyển thẳng (cross-dock) mạnh; tự động hóa nếu có. | Tăng năng suất cao điểm. |

**Lưu ý quan trọng**: do an toàn thực phẩm, **truy vết lô + thu hồi** nên được đưa vào ngay từ GĐ 1 ở mức tối thiểu (ghi lô/NSX/HSD + truy xuôi theo phiếu xuất), không để đến giai đoạn cuối.

## 15. Kết luận

Giải pháp giải quyết trọn ba luồng phân phối của Coca-Cola và đặt **ba tính chất bắt buộc** làm xương sống:

- **Đích danh** = mỗi thao tác đi qua một **cổng kiểm soát ngăn ngừa**: quét, đối chiếu kế hoạch, chặn cứng nếu sai — ngăn lỗi/gian lận **trước khi xảy ra**.
- **Minh bạch** = mọi thay đổi tồn là giao dịch có nhật ký không sửa được, đối soát hai đầu và truy vết lô đầu-cuối — kiểm tra được, chống gian lận.
- **Tức thời** = thiết bị quét + theo dõi hàng trên đường + bảng điều khiển + cảnh báo realtime — xử lý kịp thời.

Đặc thù Coca-Cola (FEFO, lô/HSD, thu hồi an toàn thực phẩm, trọng lượng nặng, nguyên pallet, vỏ/két, cao điểm mùa vụ) được phản ánh trực tiếp trong quy tắc giữ hàng, kiểm chất lượng, sức chứa và ưu tiên triển khai truy vết lô từ sớm.
