# FTEC5660 Homework 1: Receipt Chain

Build a LangChain pipeline that reads every supermarket receipt in a folder
with the vision-capable DeepSeek Flash model and answers these two questions:

1. How much money did I spend in total for these bills?
2. How much would I have had to pay without the discount?

For this homework, **amount spent** means the final payment after the receipt's
rounding line. **Without the discount** means the sum of the original positive
item prices: add back every promotion, coupon, member, app, packaging-damage,
and percentage discount, but do not add back rounding.

## Student task

Only edit the two functions in `hw1.py` that contain `### YOUR CODE HERE`:

- `build_chain()` creates your LangChain chain.
- `answer_queries()` runs the chain on the receipt images and returns one final
  response for each question.

You may use prompt chaining, routing, parallel calls, reflection, or a
combination. Your final responses should each contain one HKD amount. Do not
hard-code filenames or public answers; grading uses unseen receipt folders.

## Setup and public test

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Put your DeepSeek key after `DEEPSEEK_API_KEY=` in `.env`, then run:

```bash
python3 hw1.py --image-folder public_test
```

The program creates `results.csv` in the current directory. Its columns are
`query`, `model_response`, and `correctness`. The public answers are in
`public_test/ground_truth.json`. The starter intentionally returns the dummy
response `please design your chain to answer these two queries.` so it runs
before you add any API code.

The required model is `deepseek-v4-flash-vision-exp`, the vision-capable
DeepSeek Flash model. JPEG, PNG, GIF, and WebP inputs are accepted by the
homework runner.


## Homework 1 solution: 
Mostly AI generated, with my adjustment in *\text*.

My solution implements an end-to-end multimodal LangChain chain that directly extracts structured JSON data from supermarket receipt images and performs precise financial aggregation in Python. I used `ChatPromptTemplate.from_messages` to build a multimodal prompt containing both textual instructions and an `image_url` content block, which is then passed to the vision-capable `deepseek-v4-flash-vision-exp` model. To mitigate visual hallucinations and blurry text common in long receipts, I added strict anti-hallucination rules in the prompt, such as "ONLY extract text that is actually visible", "DO NOT invent or guess", and specific guidance to "read only the rightmost column for prices" and to "ignore all lines after ROUNDING" *(This is because I notice that the model will recognize repeated data, which always appear after ROUNDING. It might lead to overfitting but I have no simple alternatives)*. The model outputs structured data through `JsonOutputParser`, yielding a list of items and `total_payment` per receipt. In the `answer_queries` function, I use Python's `Decimal` type for financial-grade precision: I iterate over all receipt dictionaries, extract `total_payment`, and sum all negative item amounts to compute total discounts. QUERY_1 (total spent) is the sum of all `total_payment` values. QUERY_2 (total without discount) is computed as `sum(total_payment - discount)` where `discount` is the sum of negative amounts per receipt. Finally, I format both results as strings in the form `"HK$xx.xx"`. The chain processes receipts sequentially in a loop to ensure stable parsing across images. This design ensures accurate extraction and aggregation despite noisy visual input.
*I used basic Python print to help me debug since I am not familiar with powershell debugging tools. The trace code is commented in my homework code.*

