# BU FS: CreditTech - Paylater

> **LLM INSTRUCTION:** Read this file completely — do not truncate or skip sections. Critical knowledge (corrections, gotchas, business rules) may appear in any section, especially `## Memory` at the end.

**Domain ID:** `7e23d41a-0baf-4b36-84c3-07fb850bdb6e`

## Description
- Data Domain: PayLater – Payment Transactions & Register
I. Transaction Data
Table: momovn-prod.BU_FI.PAYLATER_ALL_TRANS
Purpose: Stores all PayLater (VTS) transactions and repayments.
Note: Using result_code = 0 AND trans_type in ('pay_pl','pay_ins') to filter successful transactions when calculating MAU, GMV, or disbursement.

**1. Identity & Keys**
| Column | Type | Description |
|--------|------|-------------|
| id | INTEGER | Unique row identifier |
| core_id | INTEGER | MoMo core transaction ID. Dùng để COUNT số giao dịch & key để join với bảng khác |
| user_id | INTEGER | User identifier |
| paylater_loan_id | STRING | Loan ID (e.g. SL00000165607481) |
| parent_id | STRING | Parent transaction ID (nullable, for linked transactions) |

**2. Time**
| Column | Type | Description |
|--------|------|-------------|
| created_millis | INTEGER | Transaction timestamp in milliseconds |
| created_date | DATE | Transaction date (e.g. 2026-03-11) |

**3. Transaction Details**
| Column | Type | Description |
|--------|------|-------------|
| service_id | STRING | Service code (e.g. CGV01) |
| service_name | STRING | Service display name (e.g. CGV) |
| service_category | STRING | Service category (e.g. CINEMA) |
| newvertical | STRING | Vertical classification (e.g. CINEMA) |
| bu_group_code_l1 | STRING | Business unit group L1 (e.g. TOPBRAND ONLINE) |
| bill_id | STRING | Bill identifier (e.g. 93182950_202603) |
| bill_type | STRING | Bill type (e.g. ins_payment) |
| bill | STRING | Bill source (e.g. paylater) |
| amount | INTEGER | Transaction amount (VND) |
| trans_type | STRING | Transaction type: pay_ins (installment), pay_pl (paylater). Filter `IN ('pay_pl','pay_ins')` for payment transactions |
| account_type | INTEGER | Account type (e.g. 21) |
| typeid | STRING | Type IDs (array as string, e.g. []) |

**4. Result**
| Column | Type | Description |
|--------|------|-------------|
| result_code | INTEGER | Result code (0 = success). Filter `= 0` for successful transactions |
| error_desc | STRING | Error description in English (e.g. Success) |
| vi_error_desc | STRING | Error description in Vietnamese (e.g. Thành công) |

**5. User Profile**
| Column | Type | Description |
|--------|------|-------------|
| level_ | STRING | User level (e.g. BRONZE) |
| rank | STRING | User rank (e.g. BRONZE) |
| credit_limit | INTEGER | User's credit limit (VND) |
| transaction_age | INTEGER | Tuổi của user tại thời điểm giao dịch |

**6. Promotions**
| Column | Type | Description |
|--------|------|-------------|
| vc_amt | INTEGER | Voucher amount |
| cb_amt | INTEGER | Cashback amount |

**7. Partner**
| Column | Type | Description |
|--------|------|-------------|
| partner_agent | STRING | Thường là service_code ở bảng khác, dùng để phân biệt merchant/specialproject/usecase (e.g. billpaycgv) |
| partner_name | STRING | Partner name (e.g. TPB, MV...) |




II. PayLater MAU Data
Table: momovn-prod.BU_FI.PAYLATER_MAU_SEGMENT
Partitioned by month_trans
Purpose: This table provides a monthly user-level summary of PayLater (VTS) product, combining user segmentation (new/retain/reactive), first transaction details, spending behavior, promo usage, vertical breakdown, and demographics — designed to support MAU engagement and retention analysis.

## Tables

### momovn-prod.BU_FI.PAYLATER_MAU_SEGMENT
Partitioned by: month_trans. Monthly user-level summary of PayLater (VTS).

**1. Identity & Time**
| Column | Type | Description |
|--------|------|-------------|
| agent_id | STRING | Unique user identifier |
| month_trans | DATE | Transaction month (partition key) |
| date_trans | DATE | Date of user's first successful transaction in month_trans |

**2. MAU Segmentation**
| Column | Type | Description |
|--------|------|-------------|
| mau_type | STRING | 1.New (first-ever VTS payment), 2.Retain (active last month), 3.Reactive (inactive last month, returned) |
| is_retain_next_month | INTEGER | 1 if user active next month, else 0 |

**3. First Transaction in month_trans**
| Column | Type | Description |
|--------|------|-------------|
| first_trans_time | DATETIME | Timestamp of first successful transaction |
| first_trans_id | STRING | core_id of first successful transaction |
| first_trans_type | STRING | pay_pl (paylater) or pay_ins (installment) |
| first_usecase | STRING | Usecase of first transaction (e.g. SME OFFLINE, FNB, UTILITIES...) |
| first_trans_gift_type | STRING | 1.VTS gift / 2.BU gift / 3.Shop Xu / 4.organic |
| first_trans_gift_list | STRING (REPEATED) | Array of distinct gift IDs for first transaction |
| first_trans_ispromo | INTEGER | 1 if uses gift, 0 if not |

**4. Spending & Transactions**
| Column | Type | Description |
|--------|------|-------------|
| credit_limit | INTEGER | Credit limit in month_trans |
| spending_vts | FLOAT | Total spending amount |
| trans_vts | INTEGER | Count distinct transactions (core_id) |
| cnt_usecase | INTEGER | Count distinct usecases used |
| cb_amt | FLOAT | Total cashback amount |
| vc_amt | FLOAT | Total voucher amount |
| promo_amt | FLOAT | Total promo (cb_amt + vc_amt) |
| installment_trans | INTEGER | Count distinct installment transactions (pay_ins) |
| installment_amount | FLOAT | Total installment spending |
| is_using_promo | INTEGER | 1 if promo_amt > 0 |

**5. Top Use Case**
| Column | Type | Description |
|--------|------|-------------|
| most_spending_usecase | STRING | Usecase with highest spending |
| most_spending_amt | FLOAT | Spending of top usecase |
| most_usecase_ratio | FLOAT | most_spending_amt / spending_vts |
| most_freq_usecase | STRING | Usecase with most transactions |
| most_freq_trans | INTEGER | Transaction count of most frequent usecase |

**6. Per-Vertical Breakdown**
| Column | Type | Description |
|--------|------|-------------|
| spending_sps / sl_trans_sps | FLOAT / INT | SME OFFLINE |
| spending_fnb / sl_trans_fnb | FLOAT / INT | FNB |
| spending_retail / sl_trans_retail | FLOAT / INT | RETAIL |
| spending_utilities / sl_trans_utilities | FLOAT / INT | UTILITIES |
| spending_telco / sl_trans_telco | FLOAT / INT | AIRTIME + DATA |
| spending_marketplace / sl_trans_marketplace | FLOAT / INT | MARKETPLACE |
| spending_logistics / sl_trans_logistics | FLOAT / INT | LOGISTICS |
| spending_cinema / sl_trans_cinema | FLOAT / INT | CINEMA |
| spending_ota / sl_trans_ota | FLOAT / INT | OTA |
| spending_bu_online / sl_trans_bu_online | FLOAT / INT | ADS PAYMENT, APP STORE, DIGITAL CONTENT, GAME, LOGISTICS, MARKETPLACE, OTT |
| spending_bu_bill / sl_trans_bu_bill | FLOAT / INT | UTILITIES + PUBLIC SERVICE |
| spending_bu_eps / sl_trans_bu_eps | FLOAT / INT | FNB, MEDIUM SIZE POS MC, RETAIL |

**7. Demographics**
| Column | Type | Description |
|--------|------|-------------|
| age_group | STRING | [1] ≤22, [2] 23-26, [3] 27-30, [4] 31-35, [5] 36-40, [6] >40, [7] unknown |
| YOB | INTEGER | Year of birth |
| gender | STRING | Gender (defaults to 'unknown' if NULL) |
| region_name | STRING | Hồ Chí Minh, Hà Nội, KCN Miền Nam, KCN Miền Bắc, Miền Nam, Miền Bắc, Miền Trung, Other |
| most_city_a60 | STRING | Most frequent city from location data (last 60 days) |

III. PayLater Register
### `momovn-prod.BU_FI.PAYLATER_REGISTER_USERS`
> Records of users who successfully registered for PayLater. Partitioned by date_range.

| Column | Type | Description |
|--------|------|-------------|
| `date_range` | DATE | Ngày user register thành công (được lender duyệt) |
| `agent_id` | STRING | User ID |
| `partner_loan_id` | STRING | Mã hợp đồng của user |
| `newtoMoMo` | STRING | User có phải là NewToMoMo không |
| `partner_name` | STRING | Lender name (TPB, MBV) |
| `CONTRACT_TYPE` | STRING | Loại contract (Card/Loan) |
| `latest_status` | STRING | Trạng thái ví gần nhất (ACTIVED, LOCKED, DEACTIVED) |
- question related CONTRACT_TYPE, filter: lastest_status IN ('ACTIVED', 'LOCKED') or lastest_status is null

IV. `project-5400504384186300846.REPORT.D_SERVICE_LIST_NEW`
Lookup table mapping service codes to merchant names, categories, and business group hierarchy. Join with PAYLATER_ALL_TRANS on partner_agent = SERVICE_CODE.

| Column | Type | Description |
|--------|------|-------------|
| `id` | INT64 | — |
| `service_code` | STRING | Service code (join key to partner_agent) |
| `service_description` | STRING | — |
| `bu_name` | STRING | — |
| `bu_group_code_l1` | STRING | Business group level 1 |
| `bu_group_code_l2` | STRING | Business group level 2 |
| `bu_group_code_l3` | STRING | Business group level 3 |
| `bu_group_code_l4` | STRING | Business group level 4 |
| `bu_group_code_l5` | STRING | Business group level 5 |
| `bu_group_code_l6` | STRING | Business group level 6 |
| `group_code_l1` | STRING | — |
| `merchant` | STRING | Merchant name |
| `key_merchant` | STRING | — |
| `key_merchant_2` | STRING | — |
| `key_merchant_3` | STRING | — |
| `newvertical` | STRING | New vertical classification |
| `newvertical_merchant` | STRING | — |
| `specialproject` | STRING | Special project tag |
| `valid_status` | STRING | — |
| `created_by` | STRING | — |
| `created_time` | TIMESTAMP | — |
| `updated_by` | STRING | — |
| `updated_time` | TIMESTAMP | — |
| `deleted_by` | STRING | — |
| `deleted_time` | TIMESTAMP | — |
| `end_time` | TIMESTAMP | — |


- Nếu user hỏi về dịch vụ COFFEE, CVS, SUPERMARKET thì map data để lấy theo bu_group_code_l4
SELECT partner_agent, bu_group_code_l4
 FROM `momovn-prod.BU_FI.PAYLATER_ALL_TRANS`  t1
  LEFT JOIN (
    SELECT *
    FROM `project-5400504384186300846.REPORT.D_SERVICE_LIST_NEW`
    GROUP BY ALL
    QUALIFY ROW_NUMBER() OVER(PARTITION BY SERVICE_CODE) = 1
  ) t2
  on t1.partner_agent = t2.SERVICE_CODE
WHERE created_date between ---- fitler date here 
AND result_code = 0
AND trans_type IN ('pay_ins', 'pay_pl')
group by all
V. momovn-prod.BU_FI.HTN_PAYLATER_PENETRATION_RAW
Daily aggregated PayLater penetration data by BU/specialproject
 **Partitioned by: date_trans.**

**Example row:**
month=2026-03-01, date_trans=2026-03-11, day=11, BU=1.ONLINE, specialproject=APPLICATION STORE, user=65122, VTS_TRANS=1, TRANS=1, GMV=274, VTS_GMV=274

| Column | Type | Description |
|--------|------|-------------|
| month | DATE | Tháng (first day of month) |
| date_trans | DATE | Ngày giao dịch |
| day | INTEGER | Ngày trong tháng |
| BU | STRING | Business unit group |
| specialproject | STRING | Usecase/vertical within BU |
| user | INTEGER | User ID |
| VTS_TRANS | INTEGER | Số giao dịch qua VTS (PayLater) |
| TRANS | INTEGER | Tổng số giao dịch (all payment methods) |
| GMV | FLOAT | Tổng GMV (all payment methods) |
| VTS_GMV | FLOAT | GMV qua VTS |

**Penetration:** VTS_TRANS / TRANS hoặc VTS_GMV / GMV

**BU → specialproject mapping:**
| BU | specialproject |
|----|---------------|
| 1.ONLINE | ADS PAYMENT, APPLICATION STORE, DIGITAL CONTENT, GAME, LOGISTICS, MARKETPLACE, OTT |
| 2.TELCO | AIRTIME, DATA |
| 3.BILLPAY | PUBLIC SERVICE, UTILITIES |
| 4.EPS | FNB, RETAIL, MEDIUM SIZE POS MC |
| 5.SPS | SME OFFLINE |
| 6.MDS | CINEMA, OTA |
| 7.OTHER | INSURANCE |




VI. Note from DA
- VTS = ví trả sau = paylater
- MAU = monthly active user, là user có hành vi thanh toán với Paylater trong tháng, tính tới ngày t-1 so với ngày lấy data
- MAU_SEGMENT = '1.New' là new MAU (first time active PayLater), không phải New register
- User thanh toán VTS là user có transaction_type = pay_pl, pay_ins, send_pl
- GMV = SUM(amount) của các trans_type = pay_ins, pay_pl, send_pl
- Luôn lấy data đến ngày hôm qua, không lấy data hôm nay vì không đủ
- contract_type: Card = hợp đồng Card, Loan = hợp đồng Loan, null = không có thông tin
- Filter thêm partner_name để chi tiết theo partner
- Service categories: Online (Ads Payment, Application Store, Digital Content, Game, Logistics, Marketplace, OTT), Telco (Airtime, Data), BillPay (Public Service, Utilities), EPS (F&B, Retail, Medium Size POS MC), SPS (SME Offline), MDS (Cinema, OTA)
- Khi được hỏi tới tên merchant, dùng giá trị cột service_name trong PAYLATER_ALL_TRANS. Top service_name: VIETTEL, TIKTOK, MOBIFONE, APPLE, GRAB-ENDUSER, VINAPHONE, TOPUP VIETTEL, GOOGLE, MWG - BACH HOA XANH, BE GROUP, GSM, CIRCLE K, EVN HO CHI MINH, ADSL FPT, CGV, P2P, GS25, EVN HA NOI, MINISTOP, HIGHLANDS COFFEE, LAZADA, FAMILYMART, NETFLIX, VIETLOTT, SMS FUNTEK, SPOTIFY, VIETNAMOBILE, INTERNET TRA SAU VIETTEL, PHARMACITY, GALAXY CINEMA, PETROLIMEX, JOLLIBEE, LOTTE CINEMA, BETA CINEMA, MBB 247, VNPT TOAN QUOC, CO.OPMART, BIG C - GO - TOPS MARKET, NHA THUOC LONG CHAU, 7-ELEVEN, VÉ XE RẺ, PHƯƠNG TRANG, EVN BEN TRE, EVN DONG NAI, DI DONG VIETTEL TRA SAU, SIEU THI AEON, CINESTAR, VINAPHONE - TRA SAU, TIKTOK LIVE, PHUC LONG, KATINAT, MOBIFONE - TRA SAU, COOP FOOD, BHD STAR, METRO HCM, VIETJET AIR, HỒNG TRÀ NGÔ GIA, EVN CA MAU, KINGFOOD, TIKI, TTT+, BIDV 247, EVN LONG AN, LOTTE MART, EVN HUNG YEN, COMBO DATA VIETTEL, LOTTERIA, VNPT HCM, FACEBOOK, NUOC CHO LON, EVN BINH DUONG, BUS OTA (VXR, FUTA), CON CUNG, PHE LA, EVN BAC GIANG, THE COFFEE HOUSE, NUOC SACH HA NOI, EVN HAI DUONG, EVN QUANG NAM, VE TAU - DUONG SAT VIET NAM (THU HO), GUARDIAN, MM MEGA MARKET, COMECO, EVN DAK LAK, EVN DONG THAP, COOP SMILE, NUOC TRUNG AN, EVN DA NANG, ADSL FPT TRA TRUOC, VETC, CANVA PTY LTD, NUOC THU DUC, EVN QUANG BINH, GOLDENGATE, EVN TIEN GIANG, EVN BINH DINH, VDTC VIETINBANK, EVN CAN THO, EVN BAC NINH






