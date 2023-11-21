# Task: 
- **Benchmark Hyperparameters:**
	- Finetune 
	- Inference
- **Training strategy**
- **Post_processing:**
	- End conversation
	- String generation respond
# Prompt design
- Context: "Ngữ cảnh", "Chủ đề", "Chat"
- Specific prompt and completion
- Add Vietnamese customer culture
- Add keywords
- Variety of keyword
- Break down conversation into smaller concrete info 
# Issue:
- Wrong/Misunderstanding intent -> Need utilize common KB
- Leak information -> Due to long/short memory 
- Wrong fact related with number 
# :luc_arrow_big_right: Solution 
- Config Temp: 0.1 - 0.3
- Presence: 0.5 - 1
- Analyze Finetuning result (train/val)
- Augment Trainset to 200 samples
- Redesign Prompting: Include more general KB

# Experiment
1. Davinci 22/06/2023
- Epoch: 6
- BS: 4 
- Prompt design: Simple, concrete question - answer + additional context for conversation 
- Train - val result 
![[Pasted image 20230626002130.png]]

```
             step  elapsed_tokens  elapsed_examples  training_loss  \
count  139.000000      139.000000         139.00000     139.000000   
mean    70.000000   171385.611511         280.00000       0.129591   
std     40.269923   100640.865962         161.07969       0.128390   
min      1.000000     4548.000000           4.00000       0.011796   
25%     35.500000    85502.000000         142.00000       0.040637   
50%     70.000000   169368.000000         280.00000       0.082334   
75%    104.500000   256978.000000         418.00000       0.183154   
max    139.000000   344140.000000         556.00000       0.668766   

       training_sequence_accuracy  training_token_accuracy  validation_loss  \
count                   139.00000               139.000000        18.000000   
mean                      0.06295                 0.912804         0.143848   
std                       0.13491                 0.068337         0.111461   
min                       0.00000                 0.723044         0.032935   
25%                       0.00000                 0.860241         0.080710   
50%                       0.00000                 0.933333         0.099053   
75%                       0.00000                 0.973266         0.160374   
max                       0.75000                 0.994269         0.478401   

       validation_sequence_accuracy  validation_token_accuracy  
count                          18.0                  18.000000  
mean                            0.0                   0.916240  
std                             0.0                   0.058769  
min                             0.0                   0.798387  
25%                             0.0                   0.897064  
50%                             0.0                   0.935965  
75%                             0.0                   0.957501  
max                             0.0                   0.986198  
```

=> Loss seem to convergence although decrease trend
=> Need more data and epoch for better analyse.

2. DaVinci 21/06/2023
- Epoch: 6
- BS: 1 
- Dataset: 40 samples (raw)
![[Pasted image 20230626003515.png]]

```
             step  elapsed_tokens  elapsed_examples  training_loss  \
count  115.000000      115.000000        115.000000     115.000000   
mean    58.000000    34599.008696         58.000000       0.223775   
std     33.341666    19747.828737         33.341666       0.257420   
min      1.000000      425.000000          1.000000       0.003323   
25%     29.500000    18841.500000         29.500000       0.027703   
50%     58.000000    34146.000000         58.000000       0.114424   
75%     86.500000    50302.500000         86.500000       0.344131   
max    115.000000    68171.000000        115.000000       1.099441   

       training_sequence_accuracy  training_token_accuracy  validation_loss  \
count                  115.000000               115.000000        15.000000   
mean                     0.130435                 0.920078         0.452870   
std                      0.338255                 0.077425         0.705567   
min                      0.000000                 0.730909         0.046083   
25%                      0.000000                 0.866135         0.152839   
50%                      0.000000                 0.946667         0.236078   
75%                      0.000000                 0.987684         0.380859   
max                      1.000000                 1.000000         2.898939   

       validation_sequence_accuracy  validation_token_accuracy  
count                          15.0                  15.000000  
mean                            0.0                   0.821958  
std                             0.0                   0.238219  
min                             0.0                   0.000000  
25%                             0.0                   0.822012  
50%                             0.0                   0.875256  
75%                             0.0                   0.943441  
max                             0.0                   0.969231  
```

3. DaVinci 27/06/2023 
- Epoch: 10
- Batch size: 8
- Dataset: 200 training images.
![[Pasted image 20230628005136.png]]
```
             step  elapsed_tokens  elapsed_examples  training_loss  \
count  251.000000    2.510000e+02        251.000000     251.000000   
mean   126.000000    7.198175e+05       1008.000000       0.052754   
std     72.601653    4.141579e+05        580.813223       0.074234   
min      1.000000    8.456000e+03          8.000000       0.001576   
25%     63.500000    3.623000e+05        508.000000       0.005358   
50%    126.000000    7.227360e+05       1008.000000       0.018045   
75%    188.500000    1.076900e+06       1508.000000       0.073345   
max    251.000000    1.431384e+06       2008.000000       0.422803   

       training_sequence_accuracy  training_token_accuracy  
count                  251.000000               251.000000  
mean                     0.347610                 0.953963  
std                      0.378066                 0.060210  
min                      0.000000                 0.758019  
25%                      0.000000                 0.923682  
50%                      0.125000                 0.985771  
75%                      0.750000                 0.998368  
max                      1.000000                 1.000000  
```
=> Model seems to be converged at 170 iterations - 6.8 epoch.
=> Need more data or change batch size to 4. 

- Sequence accuracy improve.