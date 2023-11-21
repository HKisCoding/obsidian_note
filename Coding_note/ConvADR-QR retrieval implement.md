## Referrence Paper: [link](obsidian://open?vault=Obsidian%20Vault&file=3.Papers%2FOpen-Domain%20Conversational%20Question%20Answering%20with%20Historical%20Answers)

### Preproduce step: 
1. From dialogue -> split question-answer pair
2. Looping through each pair to concat the embedding
3. Embedding the conversation for each question 
4. Calculate distance to get the user intention(Smartpay product)

### Source Code implementation:
```python

# Padding embedding to specific length (Bert)
def pad_input_ids_with_mask(input_ids,
                            max_length,
                            pad_on_left=False,
                            pad_token=0):
    padding_length = max_length - len(input_ids)
    padding_id = [pad_token] * padding_length

    attention_mask = []

    if padding_length <= 0:
        input_ids = input_ids[:max_length]
        attention_mask = [1] * max_length
    else:
        if pad_on_left:
            input_ids = padding_id + input_ids
        else:
            attention_mask = [1] * len(input_ids) + [0] * padding_length
            input_ids = input_ids + padding_id

    assert len(input_ids) == max_length
    assert len(attention_mask) == max_length

    return input_ids, attention_mask

def evaluate(retriever_model, retriever_tokenizer, conversation):
	inp, answer = spliting_conversations(conversation)
	concat_ids = []
	concat_id_mask = []
	concat_ids.append(retriever_tokenizer.cls_token_id)
	# Looping through each q-a pair and concat 
    for q, a in zip(inp, answers):
	    concat_ids.extend(retriever_tokenizer.convert_tokens_to_ids(
		    retriever_tokenizer.tokenize(q)
	    ))
	    concat_ids.append(retriever_tokenizer.sep_token_id)
	    if a not in ["CANNOTANSWER", "NOTRECOVERED", "empty"]:
                        concat_ids.extend(
                            retriever_tokenizer.convert_tokens_to_ids(
                                retriever_tokenizer.tokenize(a)
                        ))
	    concat_ids.append(retriever_tokenizer.sep_token_id)

		# Padding input ids to get desire length
		concat_ids, concat_id_mask = pad_input_ids_with_mask(
            concat_ids, args.max_concat_length)
        ids, id_mask = (for ele in [torch.to_tensor([concat_ids]), torch.tensor([concat_id_mask])])
```