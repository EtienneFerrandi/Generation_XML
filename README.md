# Generation_XML

We plan to automatically generate XML/EAD encoding of documentary units (<c> inventory holdings description tags <dsc>). We have tried to proceed in two different ways.

## Fine-tuning: review and results

First, we attempted to fine-tune an LLM on our data. This was a question/answer dataframe with the text of the documentary unit in the first column and its XML/EAD encoding in the second column. There could be one to two documentary units (two <c> tags and their content) per question. If possible, the maximum number of tokens per question and per answer should not exceed 512.

We tried fine-tuning the open-source LLMs 7B, Llama 2 from Meta and Mistral-Hermès from Mistral AI, which we loaded into a quantized version using an A100 GPU rented from Google Colab ([cf. Open-source LLMs fine-tuned](https://github.com/EtienneFerrandi/Generation_XML/tree/d73287cc57ad9758d046722072fda00eda2c77fe/Open-source%20LLMs%20fine-tuned)).

We also tried fine-tuning gpt-3.5, owned by OpenAI, using the OpenAI API (cf. [GPT 3.5 fine-tuned](https://github.com/EtienneFerrandi/Generation_XML/tree/a468fa948ef148b33fddeabe54adae30bd9c9d90/GPT%203.5%20fine-tuned))

Overall, inference, i.e. text generation from the various fine-tuned LLMs, was disappointing. This can be explained by the fact that the XML/EAD format we wish to generate is highly structured.

## Schematic prompt: balance sheet and results

We adopted the function calling approach. This is a method of indicating the expected response pattern during inference. In this way, the expected XML/EAD format is precisely defined in an upstream JSON version. The output is an XML/EAD generation in the previously defined format.

Function calling is possible with Mistral-Hermès. We used a fine-tuned LLM on our data, which we called "neural-hermes-7b-ead". Secondly, we also used function calling with the OpenAI API (cf. [Function-calling](https://github.com/EtienneFerrandi/Generation_XML/tree/56ef44ba80e5c53861be37d51288ed0c3316cab5/Function-calling)).

In both cases, the results are encouraging, but to reach a higher level of execution, it was necessary to use a more systematic and accomplished tool, Kor (cf.[Kor_XML](https://github.com/EtienneFerrandi/Generation_XML/tree/e6b3877d9e80102eea264cb95eaad802bc24626a/Kor_XML)).

This method involves preparing annotated archival descriptions in advance, either automatically or manually. Each documentary unit is given its own tag level. A higher <c> tag has a level n+1 of a lower <c> tag. To automate such annotation, we used LabelStudio.

Once the annotation has been made, you are invited to use the Kor_EAD notebook according to the corresponding ["Roadmap"](https://github.com/EtienneFerrandi/Generation_XML/blob/e6b3877d9e80102eea264cb95eaad802bc24626a/Kor_XML/ReadME.md).

The results are very good. However, there is still room for improvement. It is possible to add numerous examples in the "c" schema, i.e. document units whose XML/EAD encoding is specified in JSON format. This is provided that the limit of 4096 tokens for the prompt is not exceeded when using gpt 3.5. When using gpt 4, or even gpt 4 32k, this limit is increased to 8192, or even 32,768 tokens.

## Conclusion

Fine-tuning LLMs on our data, i.e. document units as questions and their XML/EAD encoding as answers, didn't produce the results we'd hoped for, probably because an LLM is designed to generate text, not structure. 
The solution lies in calling up a schematic answer in the prompt. The schema is precisely the XML/EAD structure in JSON format. The generation process therefore consists in filling in the key values of the JSON object that is the document unit. In this case, the prompt is very large. Care must be taken to respect the limit of tokens authorized by the LLM you are using. But you can exceed this limit by using gpt 4 32k. It is possible to multiply the number of examples in the "c" schema, so that the model will be able to generate a wide variety of documentary units. This method acts as a fine-tuned model on question/answer data.
 
We therefore invite the user to continue testing the Kor_EAD notebook with the recommendations given.
