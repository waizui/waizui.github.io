<head>
  <script>
    window.MathJax = {
      tex: {
        inlineMath: [['$', '$'], ['\\(', '\\)']],
        displayMath: [['$$', '$$'], ['\\[', '\\]']]
      },
      options: {
        skipHtmlTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
      }
    };
  </script>

  <script
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js"
    async>
  </script>
</head>

# Let's build a GPT-2 (IV)

In this part, I will introduce how to fine-tune a GPT2 model with instructions, which can make the model follow instructions.

## Why Instruction Fine Tuning

A pre-trained model can complete a sentence, but it cannot reliably follow human instructions. For example, I use this as the input of a 355M GPT2 model:

```
What is the capital of China?
```
and the model generated:

```
What is the capital of China? For this question is a bit ridiculous. You can find more information about the capital of China on the Wikipedia page here but I would be a bit more skeptical. So I have to add capital to it. So what? It does not help to answer ...
```

The generated text does not answer the question. The expected answer is something like:
```
The capital of China is Beijing.
```

In order to make the model follow instructions, we need to use **instruction fine-tuning**.

## Prepare Data

We use **Alpaca** prompt style for training, given an instruction-answer pair:
```
{
    "instruction": "Edit the following sentence for grammar.",
    "input": "He go to the park every day.",
    "output": "He goes to the park every day."
},
```
Alpaca converts it to:

```
Below is an instruction that describes a task. Write a response that appropriately completes the request.

### Instruction:
Edit the following sentence for grammar.

### Input:
He go to the park every day.

### Response:
He goes to the park every day.
```

We use the same training function just like part III. The difference is that we use instruction sets instead of cutting out chunks from long text.

Instructions vary in length. We need to pad them to make them a batch. 

For example: If the instruction batch is: 

```
[
    [a, b, c],
    [d, e],
]
```
after padding it becomes this:

```
inputs:
[
    [a, b,     c],
    [d, e, 50256],
]

targets:
[
    [b,     c, 50256],
    [e, 50256,  -100],
]
```
where inputs and targets are used to calculate the cross-entropy loss. 
For instructions that are shorter than maximum instruction length, we pad with 50256 to create inputs.

Targets are created by shifting instructions to the left by one token, then pad with 50256 and fill the tokens after the first 50256 with -100.
Tokens with id equal to -100 will be ignored in the calculation of cross-entropy loss.

The following code shows how this process is done.

```python
def custom_collate_fn(
    batch,  # [batch_size]
    pad_token_id=50256,  # end of sequence id
    ignore_index=-100, # token with this id will be ignored by cross-entropy loss function
    allowed_max_len=None,
    device="cuda",
):
    batch_max_len = max(len(item) + 1 for item in batch)
    input_lst, target_lst = [], []

    for item in batch:
        new_item = item.copy()
        new_item.append(pad_token_id)

        padded = new_item + [pad_token_id] * (batch_max_len - len(new_item))

        # same as part II, we want to minimize the cross-entropy loss of model predictions and targets
        inputs = torch.tensor(padded[:-1])
        targets = torch.tensor(padded[1:])

        mask = targets == pad_token_id
        indices = torch.nonzero(mask).squeeze()
        if indices.numel() > 1:
            # fill all padding tokens with ignore_index except the first one
            targets[indices[1:]] = ignore_index

        if allowed_max_len is not None:
            inputs = inputs[:allowed_max_len]
            targets = targets[:allowed_max_len]

        input_lst.append(inputs)
        target_lst.append(targets)

    inputs_tensor = torch.stack(input_lst).to(device)  # [batch_size,batch_max_len-1]
    targets_tensor = torch.stack(target_lst).to(device) 

    return inputs_tensor, targets_tensor
```

## Training
The training is based on a pre-trained 355M-parameter GPT2 model, apart from data preparation, everything is same as in part II.

Here are some fine-tuned model generated results:

```
### Instruction:                                                                                                                                                                                            
Classify the following statement into one of these labels: [positive, negative, neutral]                                                                                                                    
                                                                                                                                                                                                            
### Input:                                                                                                                                                                                                  
My computer crashed.                                                                                                                                                                                        
                                                                                                                                                                                                            
### Response:                                                                                                                                                                                               
Negative                                                                                                                                                                                                    
Negative
```

```
### Instruction:
Provide a synonym for 'clever.'

### Response:
A synonym for 'clever' is 'start.' 

```

```
### Instruction:                                                                                                                                                                                            
What is the capital of India?                                                                                                                                                                               
                                                                                                                                                                                                            
### Response:                                                                                                                                                                                               
The capital of India is Tokyo.
```

Considering the dataset I used and relatively small number of model parameters, these results are good enough to illustrate how to make models follow instructions.
Maybe the synonym of clever is start from some philosophical perspectives.


## References
- [Alpaca](https://github.com/tatsu-lab/stanford_alpaca)
- [Build a Large Language Model](https://github.com/rasbt/LLMs-from-scratch)
