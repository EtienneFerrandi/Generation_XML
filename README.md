# Generation_XML

We plan to automatically generate XML/EAD encoding of documentary units (<c> inventory holdings description tags <dsc>). We have tried to proceed in two different ways.

## Fine-tuning: review and results

First, we attempted to fine-tune an LLM on our data. This was a question/answer dataframe with the text of the documentary unit in the first column and its XML/EAD encoding in the second column. There could be one to two documentary units (two <c> tags and their content) per question. If possible, the maximum number of tokens per question and per answer should not exceed 512.

We tried fine-tuning the open-source LLMs 7B, Llama 2 from Meta and Mistral-Hermès from Mistral AI, which we loaded into a quantized version using an A100 GPU rented from Google Colab ([cf. Open-source LLMs fine-tuned](https://github.com/EtienneFerrandi/Generation_XML/tree/d73287cc57ad9758d046722072fda00eda2c77fe/Open-source%20LLMs%20fine-tuned))
