
## 1. Data dictionary 

### fms_transaction: 

| Data field                | Value                                                                                                                  | Description                                |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| merc_id                   | 98 + merchant id                                                                                                       | 98: Key account + số merchant id           |
| merc_channel              | e.g, VPB, TCB, MICRO_MERCHANT, CHAINED_MERCHANT                                                                        | Source của merchant                        |
| merc_category             |                                                                                                                        | Business code được định nghĩa cho category |
| merc_name                 |                                                                                                                        | Tên merchant                               |
| merc_branch_id            |                                                                                                                        | Partner ID của merchant                    |
| merc_branch_name          |                                                                                                                        | Tên của partner                            |
| buyer_sf_acct_type        | APP: app ngân hàng<br>WALLET: ví điện tử <br>CARD: Thẻ <br>OTHERS: Nguồn tiền chưa được xác định(zalo pay, ....)       | Loại nguồn tiền giao dịch                  |
| buyer_sf_acct_subtype     | INTERNATIONAL_CARD<br><br>ATM_CARD<br><br>WALLET<br><br>BONUS<br><br>TOKEN<br><br>some other bank codes: SHB, VCCB,... | Chi tiết nguồn tiền giao dịch              |
| buyer_sf_acct_nbr         |                                                                                                                        | số thẻ/ số account / app id                |
| buyer_sf_acct_issuer      | CARD: VISA, MASTER, JCB, AMEX, NAPAS<br><br>WALLET: ZALOPAY,...<br><br>APP: VCB,BIDV,...                               | Nhà phát hành nguồn tiền                   |
| buyer_sf_issuer_country   | INSIDE_VN, OUTSIDE_VN                                                                                                  | CARD: phát hành trong hoặc ngoài Việt Nam  |
| buyer_sf_issuer_bin       |                                                                                                                        | CARD: 6 số đầu của card                    |
| oid_partner_oderno        |                                                                                                                        | số order no                                |
| og_created_date           |                                                                                                                        | ngày giao dịch được ghi nhận vào database  |
| og_status                 |                                                                                                                        | trạng thái giao dịch: 1 - thành công       |
| og_platform               | PAYMENT_GATEWAY<br>SMARTPOS<br>OTHERS                                                                                  | Platform thanh toán của merchant           |
| og_service_type           | POS<br><br>STATIC_QR<br><br>PAYMENT_GATEWAY<br><br>QR_CODE                                                             | Loại dịch vụ ứng với platform              |
| og_trans_type             | Purchase<br>Refund                                                                                                     |                                            |
| om_order_amount           |                                                                                                                        | Số tiền giao dịch gốc                      |
| om_sp_voucher_amount      |                                                                                                                        | Số tiền voucher được apply cho smartpay    |
| om_partner_voucher_amount |                                                                                                                        | Số tiền voucher được apply cho partner     |
| om_merc_voucher_amount    |                                                                                                                        | Số tiền voucher được apply cho merchant    |
| om_buyer_fee_amount       |                                                                                                                        | Phí buyer phải trả khi thực hiện giao dịch |
| om_buyer_cashback_amount  |                                                                                                                        | Số tiền được cashback cho buyer            |
| om_buyer_paid_amount      |                                                                                                                        | Số tiền thực thanh toán                    |
|                           |                                                                                                                        |                                            |

### fms_transaction_result:

| Data field           | Value                                               | Description       |
| -------------------- | --------------------------------------------------- | ----------------- |
| fee_type             | MDR/LEASING_FEE                                     |                   |
| fee_method           | auto: Tính tự động<br>force-update: Tính update lại |                   |
| pay_buyer_amount     |                                                     | Phí buyer trả     |
| pay_merchant_amount  |                                                     | Phí merchant trả  |
| pay_partner_amount   |                                                     | Phí partner trả   |
| pay_sp_amount        |                                                     | Phí Smartpay trả  |
| recv_buyer_amount    |                                                     | Phí buyer nhận    |
| recv_merchant_amount |                                                     | Phí merchant nhận |
| recv_partner_amount  |                                                     | Phí partner nhận  |
| recv_sp_amount       |                                                     | Phí Smartpay nhận |


## 2. Template config phí 

### 2.1. MDR Fee:
Mỗi biểu phí được config thành một file excel: 
ruleID-ruleName.xlsx
Example:  mm_id 00000525 

![[Pasted image 20240730151324.png]]

**Quy tắc đặt tên:** 

Format: MID_{merc_id}\_{service_code}-{ruleName}
service_code:
- 00001: SMARTPOS CARD Transaction - Fee_cardpayment_smartpos.xlsx

- 00002: GATEWAY_transaction - BBDS_Monthly_Temp_Editting_data.xlsm

- 00003: SMARTPOS_QR Transaction - BBDS_Monthly_Temp_Editting_data.xlsm

- 00004: Leasing Fee - Leasing fee_05.2024.xlsm

- 00005: Manual Fee config from contract

ruleID: MID_{mm_id}\_00005
ruleName: Tên của loại phí áp dụng ( hoặc mô tả về merchant và dịch vụ biểu phí)

=> Tên: ` MID_00000525_00005-SMARTPOS_FEE.xlsx`

#### Cấu trúc config: 
Ví dụ: MID_0000084_00001-BệnhViệnNhiĐồng1\_Gateway.xlsx

![[Pasted image 20240730152615.png]]

**Config:**
![[Pasted image 20240730155540.png]]
![[Pasted image 20240730163440.png]]

> [!warning] **Lưu ý:** 
> - `schemeCode` bắt buộc phải là unique cho mỗi scheme và nên đặt tên để dễ quản lý và update
> - Bắt buộc phải có `additional_condition` với attribute: function. Đối với scheme tính accumulate thì cần có thêm 2 trường của layer `additional_condition`: acc_scope, acc_field và layer `acc_condition`
> - Chỉ được update, thêm, xoá ở những layer selector: ['condition_trans_selector', 'condition_entity_selector', 'preEval_trans_selector', 'preEval_entity_selector']. Những layer khác bắt buộc ko xoá
> - layer `effective_date` bắt buộc value phải ở dạng 'yyyy/dd/mm' và có dấu ' ở đầu để excel convert về dạng text (không format cell)
#### Bảng mô tả config

| Field      | Mô tả                         | Gía trị                                                    | Trường dữ liệu             | Detail                                                                                                                  |
| ---------- | ----------------------------- | ---------------------------------------------------------- | -------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| schemeCode | code unique cho từng scheme   | FOP:{ruleID}\_{stt scheme}\_{stt sub scheme}               |                            | FOP:MID_00000084_00001_01_01<br>FOP:MID_00000084_00001_03_01<br>FOP:MID_00000084_00001_03_02                            |
| schemeName | Tên mô tả cho scheme          |                                                            |                            | Thanh toán thẻ Quốc tế<br>Ma QR: So luong giao dich hang thang < 10000<br>Ma QR: So luong giao dich hang thang >= 10000 |
| layer      |                               |                                                            | condition_trans_selector   | Điều kiện thuộc tính giao dịch để tính phí                                                                              |
|            |                               |                                                            | condition_entity_selector  | Điều kiện thuộc tính của merchant/buyer/partner để tính phí                                                             |
|            |                               |                                                            | fee_setup                  | Các thành phần của phí                                                                                                  |
|            |                               |                                                            | effective_date             | Khoảng thời gian effective của biểu phí                                                                                 |
|            |                               |                                                            | additional_condition       | Các điều kiện bổ sung                                                                                                   |
|            |                               |                                                            | pay_fee_ratio              | Tỉ lệ trả phí                                                                                                           |
|            |                               |                                                            | recv_fee_ratio             | Tỉ lệ nhận phí                                                                                                          |
|            |                               |                                                            | preEval_trans_selector     | Điều kiện thuộc tính giao dịch để tính các giá trị GMV/ Num giao dịch                                                   |
|            |                               |                                                            | preEval_entity_selector    | Điều kiện thuộc tính merchant / buyer / partner để tính các giá trị GMV/ Num giao dịch                                  |
|            |                               |                                                            | preEval_timestamp_selector | Điều kiện để xét khoảng thời gian tính GMV                                                                              |
| attribute  | Các trường thuộc tính của phí |                                                            |                            |                                                                                                                         |
| operator   | toán tử so sánh               | **=,<,>,<=,>=**<br><br>**in**: Khi phải chọn nhiều giá trị |                            |                                                                                                                         |
| value      | Giá trị thuộc tính của phí    |                                                            |                            |                                                                                                                         |
| runType    | Loại biểu phí                 | rule-mdr<br>rule-acc<br>rule-leasing                       |                            |                                                                                                                         |
| frequency  | thời điểm tính phí            | immediately<br>EOM                                         |                            |                                                                                                                         |
| status     |                               | active <br>disable                                         |                            |                                                                                                                         |

#### Additional_condition - function:
Các template được định nghĩa sẳn để tạo config phí từ file excels:
- `MDR-FeePerTrans`: Áp dụng cho phí MDR ko tính Accumulate
- `Acc-FeePerTrans`: Áp dụng cho phí MDR có điều kiện Accumulate

#### Effective_date: (Giao dịch thực hiện trong khoảng này sẽ được áp dụng phí)
- `start_date`: Ngày bắt đầu
- `end_date`: Ngày hết hạn 
**lưu ý:** 
- Định dạng 'yyyy/mm/dd' ở dạng text (thêm dấu ' ở đầu trong excel)
- Khi tính backdate: ops cung cấp ngày tính backdate, thông báo cho team data trigger lại transaction history

#### Fee setup
Phí sẽ được tính theo công thức: 
$$fee = \begin{cases} 
x1 * pct\_fee + fix\_fee &\text{if } min\_fee \lt  fee \lt max\_fee \\
min\_fee &\text{if } min\_fee \geq fee \geq  0\\
max\_fee &\text{if }  0 \leq max\_fee \leq fee 
\end{cases}$$

**Attributes:**
- x1: field giao dịch lựa chọn trong **fms_transaction** để tính phí (`om_order_amount`)
- pct_fee: phần trăm phí áp dụng
- fix_fee: phí cứng áp dụng
- min_fee: số tiền phí thu tối thiểu 
- max_fee: số tiền phí thu tối đa

#### Phí tính theo điều kiện GMV: (accumulate transaction)

> [!note] Tương tự như một câu query tính GMV: 
> ``` SQL
select sum({acc_field}) as AccAmount, count(trans_id) as NumTrans
from fms_transaction 
where {preEval_trans_selector} and {preEval_entity_selector}
group by {acc_scope}```

Bổ sung thêm layers để làm điều kiện lựa chọn giao dịch tính accumulate
- `preEval_trans_selector`
- `preEval_entity_selector`
- `preEval_timestamp_selector`: ví dụ: 
	- time\_unit = 1_month -> Tính gmv trong 1 tháng 
	- time\_unit = 2_month -> Tính gmv trong 2 tháng 

**Additional_conditions:**
- acc_scope: Field dữ liệu trong `fms_transaction` được sử dụng để group. Ví dụ: merc_id
- acc_field: Field dữ liêụ trong `fms_transaction` dùng để tính GMV

**Điều kiện accumulate:**
- Tính theo GMV: `acc_condition`: {attributes: 'AccAmount'}
- Tính theo số lượng giao dịch: `acc_condition`: {attributes: 'NumTrans'}

### 2.2. LeasingFee:

Từ file tính phí Leasing hàng tháng và update của fullfillment: 
Ví dụ: 
sheet **Scheme**:
![[Pasted image 20240808143859.png]]
=> Tạo các template phí cho từng Scheme (1)
sheet **PARTNER**:
![[Pasted image 20240808144025.png]]

sheet **Monthly data**:
![[Pasted image 20240808144345.png]]
Bổ sung cột **Order ID** tương ứng với ID của mỗi máy ứng với từng merchant ID
=> Tạo các Scheme tính phí theo từng merchant tương ứng với Scheme code: 
-  Tìm valid device trong Monthly data: 
	- Merchant ID ko null
	- Scheme != {`FREE4EVR`, `NOFEE`}
	- Recall Date >= 01/ previousMonth/currentYear
- Từ data device phải chịu phí => Lấy danh sách merchant cần tạo scheme phí (2)
-  Từ danh sách (1) và (2) => Tạo các scheme phí cho từng merchant => Lưu vào 1 file **SmartPosLeasingFee_RuleTrigger.xlsx**

#### Phương thức trigger giao dịch tính phí:
Dựa vào sheet `Monthly data` => Tạo giao dịch ảo **POSTING-LEASING-FEE**:
> [!note] POSTING-LEASING-FEE 
>- `om_order_amount`: SmartPOS Fee
>- `oid_merc_transid`: \[Handover Date - Recall Date]
>- `buyer_id`: Order ID
>- `buyer_sf_acct_type`: Type of SmartPOS
>- `merc_id`: Merchant ID
>- `key`: Device Serial
>- `og_trans_type`: Scheme

#### Template phí cho leasing fee:
Quy tắc đặt code cho scheme: FOP:MID_{merc_id}\_00004\_01\_01
Tương tự như MDR template ngoại trừ:

| layer                    | attribute      | value               | runType      |
| ------------------------ | -------------- | ------------------- | ------------ |
| condition_trans_selector | fms_trans_type | POSTING_LEASING_FEE | rule-leasing |
- Đối với scheme không có điều kiện accumulate: `frequency: active`
- Đối với scheme có điều kiện accumulate: `frequency: EOM`

#### Template phí có điều kiện miễn phí theo khoảng thời gian:
Ví dụ: Scheme `FREE3MON`: Miễn phí 3 tháng đầu

| layer                | attribute           | operator | value               | runType     |
| -------------------- | ------------------- | -------- | ------------------- | ----------- |
| fee_setup            | free_time_duration  | =        | 3                   | immediately |
| fee_setup            | free_time_unit      | =        | month               |             |
| additional_condition | function            | =        | Duration-FeeLeasing |             |
| pay_fee_ratio        | pay_merchant_amount | =        | 1                   |             |
| recv_fee_ration      | recv_sp_amount      | =        | 1                   |             |
![[Pasted image 20240808150809.png]]

Công thức tính phí:
$$\begin{cases}
\text{currentDate} - \text{cancelDate} < 1 \text{month}, \text{ nếu có cancelDate} \\
\text{currentDate} - \text{HandoverDate} > \text{free\_time\_duration(1) free\_time\_unit (month)}, \text{ nếu không có cancelDate}
\end{cases}
$$
#### Template phí có điều kiện miễn phí máy mỗi GMV:
Ví dụ: Scheme `FREEGMV500M`:Miễn phí nếu đạt doanh số 500tr, mỗi 500tr free 1 máy

| layer                      | attribute           | operator | value           | runType |
| -------------------------- | ------------------- | -------- | --------------- | ------- |
| fee_setup                  | free_time_duration  | =        | 1               | EOM     |
| fee_setup                  | free_time_unit      | =        | month           |         |
| fee_setup                  | endow_amount        | =        | 500000000       |         |
| pay_fee_ratio              | pay_merchant_amount | =        | 1               |         |
| recv_fee_ration            | recv_sp_amount      | =        | 1               |         |
| preEval_entity_selector    | merc_id             | =        |                 |         |
| preEval_trans_selector     | og_trans_type       | =        | Purchase        |         |
| preEval_trans_selector     | og_platform         | =        | SMARTPOS        |         |
| preEval_timestamp_selector | time_unit           | =        | 1_month         |         |
| additional_condition       | function            | =        | Acc-FeeLeasing  |         |
| additional_condition       | acc_scope           | =        | merc_id         |         |
| additional_condition       | acc_field           | =        | om_order_amount |         |
**Công thức tính phí:**
Tính phí cho những máy có:
$$
\text{device Id} > \frac{\text{GMV}}{\text{endow\_amount}}
$$
#### Đề xuất các scheme mới:

**Example1:** 
![[Pasted image 20240808152533.png]]
Tạo scheme phí với effective_date: end_date là 1 năm sau ngày lắp đặt máy đầu tiên
Tính phí tương tự như template của `NOSCH` scheme tính phí bình thường
=> Merchant sẽ có 2 scheme 1 scheme `NOSCH`và 1 scheme:

| layer           | attribute            | operator | value                           | runType |
| --------------- | -------------------- | -------- | ------------------------------- | ------- |
| pay_fee_ratio   | recv_merchant_amount | =        | 1                               |         |
| recv_fee_ration | pay_sp_amount        | =        | 1                               |         |
| effective_date  | end_date             | =        | 1 năm sau ngày lắp máy đầu tiên |         |
=> Sau khi tính tổng lại thì phí merchant = 0

**Example2:**
Merchant có 900 máy được free toàn bộ nếu trong tháng GMV >= 150 tỉ. 30 máy được lắp đặt sau đó được free nếu:
- 900 máy ban đầu được free (GMV > 150 tỉ)
- Với mỗi 200 tr dư ra  free 1 máy trong 30 máy
=> 2 Scheme Tính phí:
**Scheme1:**
$$
	\text{AccAmount(GMV)} < 150.000.000.000
$$
**Scheme2:**
$$
	condition = \begin{cases}
	\text{deviceID} > \frac{AccAmount - 150000000000}{200000000} + 900 \\
	\text{AccAmount(GMV)} > 150.000.000.000
	\end{cases}
$$
