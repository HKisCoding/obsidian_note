
- **PublishYear**:: 2020 
- **Author**:: Martin Schultz, Marina Tropmann-Frick
- **Link**:: 
- **Tags**:: #paper
- **Cite Key**:: [@schultzAutoencoderNeuralNetworks2020]

### Abstract
```
With the increasing complexity of business processes in today's organizations and the ever-growing amount of structured accounting data, identifying erroneous or fraudulent business transactions and corresponding journal entries poses a major challenge for public accountants at annual audits. In current audit practice, mainly static rules are applied which check only a few attributes of a journal entry for suspicious values. Encouraged by numerous successful adoptions of deep learning in various domains we suggest an approach for applying autoencoder neural networks to detect unusual journal entries within individual financial accounts. The identified journal entries are compared to a list of entries that were manually tagged by two experienced auditors. The comparison shows high f-scores and high recall for all analyzed financial accounts. Additionally, the autoencoder identifies anomalous journal entries that have been overlooked by the auditors. The results underpin the applicability and usefulness of deep learning techniques in financial statement audits.
```

### Notes

`Journal entry`: bản ghi chép các giao dịch tài chính trong sổ sách kế toán. Mỗi journal entry bao gồm các thông tin sau:

1. **Ngày giao dịch**: Ngày mà giao dịch tài chính xảy ra.
2. **Mô tả giao dịch**: Mô tả ngắn gọn về bản chất của giao dịch.
3. **Tài khoản nợ**: Tài khoản bị ảnh hưởng bởi giao dịch (số tiền ghi vào bên nợ).
4. **Tài khoản có**: Tài khoản bị ảnh hưởng bởi giao dịch (số tiền ghi vào bên có).
5. **Số tiền**: Số tiền của giao dịch.
6. **Số tham chiếu**: Một số hoặc mã duy nhất để nhận diện giao dịch (không bắt buộc nhưng hữu ích cho việc tra cứu sau này).

#### Problems:
`increasing complexity of business processes in today’s organizations and the ever-growing amount of structured accounting data` 

Theo phương pháp audit truyền thống, chỉ có thể apply một vài rule cố định trên một ít các thuộc tính của Journal Entry 

=> Ứng dụng Deep learning: AutoEncoder Neural Network -> detect journal entry bất thường

#### Paper Propose:



---

