
- **PublishYear**:: 2020 
- **Author**:: Martin Schultz, Marina Tropmann-Frick
- **Link**:: https://hdl.handle.net/10125/64408
- **Tags**:: #paper
- **Cite Key**:: [@schultzAutoencoderNeuralNetworks2020]

### Abstract
```
With the increasing complexity of business processes in today’s organizations and the ever-growing amount of structured accounting data, identifying erroneous or fraudulent business transactions and corresponding journal entries poses a major challenge for public accountants at annual audits. In current audit practice, mainly static rules are applied which check only a few attributes of a journal entry for suspicious values. Encouraged by numerous successful adoptions of deep learning in various domains we suggest an approach for applying autoencoder neural networks to detect unusual journal entries within individual financial accounts. The identified journal entries are compared to a list of entries that were manually tagged by two experienced auditors. The comparison shows high f-scores and high recall for all analyzed financial accounts. Additionally, the autoencoder identifies anomalous journal entries that have been overlooked by the auditors. The results underpin the applicability and usefulness of deep learning techniques in financial statement audits.
```

### Notes

**Journal Entries:** records of financial transactions in an accounting system.

### Problems:
- Sự phức tạp và phát triển của các business process
- Chỉ có thể check một vài record với một lượng rule nhất định để detect suspicious

### Proposed method: 
Apply Autoencoder Network để detect suspicious -> so sánh kết quả với kết quả từ 2 chuyên gia kiểm toán 

#### Anomaly detection with AutoEncoder
- 2 Components: Encoder and Decoder
- Encoder: Trích xuất các đặc trưng từ input đầu vào và biểu diễn thông tin được trích xuất bằng một vector $x -> e(x)$
- Decoder: từ vector thông tin, tái tạo lại output cho giống input nhất $d(e(x)) -> x$

### Analysis and Experiment

- Dataset: 72.917 journal entries
- Three suitable accounts are selected: 1) Revenue Domestic, 2) Revenue Foreign, 3) Expenses.
- Tất cả các attribute trong dataset là ở dạng Categorical ngoại trừ field amount là numeric 
- Three additional attributes are computed: **1) day of week** – derived from the creation date of the accounting document, **2) dates equal** - a Boolean attribute indicating whether all three date fields (posting date, document date, creation date) are equal or not, and **3) dates in accounting year** - a Boolean attribute representing whether the values of all three date fields lie within the date range of the fiscal year under review.
- 25 attributes in dataset: 
	- doc. number, 
	- doc. type,
	- posting key,
	- dates equal, 
	- dates in accounting year, 
	- day of week,
	- currency,
	- debit/ credit indicator,
	- tax code,
	- revenue indicator,
	- user id,
	- user group, 
	- debit account list,
	- credit account list,
	- doc. notes, 
	- doc. line item notes,
	- customer/ vendor number list, 
	- customer name list,
	- vendor name list,
	- customer/ vendor country list,
	- transaction code, 
	- recurring doc. number, \
	- reversal doc. number,
	- doc. origin 
	- amount (local currency).

#### Loss function
Mean Squared Error loss function (MSE): Công thức tính khoảng cách giữa 2 vector 
$$
	L(x,y) = argmin_{\theta}||x - y||_2
$$
#### Kiến trúc mô hình 
- Multiple fully-connected hidden layers
- Kiến trúc tối ưu: 9 lớp with \[128 (input layer), 64, 32, 16, 8, 4, 8, 16, 32, 64, 128 (output layer)] -> Số chiều trong mỗi layer

#### Data preprocessing: 
- Amount: StandardScaler - Scale data dựa trên giá trị mean và độ lệch chuẩn để đưa phân bố input đầu vào về phân bố chuẩn 
- Xác định outlier bằng các điểm nằm ở rìa phân bố (3*std)


---

