# Roadmap for automatic XML/EAD encoding of archival inventories

# Annotation

The aim of this section is to organize the archival description of each archival inventory. It involves thinking about the organization of the documentary units: their nesting and their properties. To achieve this, we recommend the following procedure: 

Start by copying and pasting the archival inventory's description (.docx or .pdf format) into a CSV. 

Make sure that each documentary unit fits into a single cell. Clean up the data if necessary (delete empty or excess columns). If you need to add encompassing document units, you can do so by adding rows.

Two options are available. Either add a second column to the CSV file, and write the number of its title level opposite each cell containing a documentary unit. Or you can use an automatic annotation tool that uses deep learning to recognize named entities to label the correct title level for each documentary unit.

If the archival description is small (less than 100 documentary units, for example), automatic annotation may be a waste of time. In the case of manual annotation, after adding "text" and "label" column headers, simply import our two-column CSV into the drive and proceed directly to II. 

In the case of automatic annotation, here's what happens next: 

1. Copy and paste the column into a txt file.
If the archival description is in the form of a table, copy and paste into a CSV and : 
ensure that the document is encoded in utf-8
merge all columns and their contents
make sure there is no isolated data in an additional column
copy and paste the single column into a txt file, ensuring that one document unit corresponds to one line in the text file.

2. open label studio label-studio start

3. create a new project and give it a name

4. go to Labelling Setup / Natural Language Processing / Named Entity Recognition / Code and copy and paste the following template: 
<View>
  <Labels name="label" toName="text">
	<Label value="Title 1" background="#D4380D"/>
	<Label value="Title 2" background="#FFA39E"/>
	<Label value="Title 3" background="#D4380D"/>
	<Label value="Title 4" background="#FFC069"/>
  </Labels>
  <Text name="text" value="$text" granularity="word"/>
</View>

If there are more than four levels of titles, add rows to the Labels list by adding +1 after "Title". 

5. Go to Data Import and load the archival inventory's description file obtained following step 1. Select the "List of tasks" button. Press "Save".

6. An interface appears, with the text divided into a list of tasks.

You can click on a task and then annotate the text: title 1 for a documentary unit that is the parent of other children's documentary units, title 2 for a level below, and so on. It is not necessary to annotate the entire text for each task. You can annotate the part of each text most relevant to its title level, e.g. for "I - this", "II- that", "Title 1" only for "I- " and "II- " and not "this" and "that".

It's also a question of annotating the most representative tasks, to enable the named entity recognition model to automatically identify the title levels of other tasks.

7. Go to Settings/Machine Learning and add a model if you haven't already done so. 
Start the machine learning backend server with label-studio-ml start my-ml-backend. Once the machine learning backend server has been activated, you can start training ("Start training" button).

8. Once training is complete, you return to the annotation interface.

Check all the tasks and, from the "N tasks" drop-down menu, click on "Retrieve predictions". The model predicts the title level for each task. 
Select all tasks and click on "Delete annotations". All annotations previously created manually are deleted.
You can then click on "Create annotations from predictions" to save the predictions as annotations.
Correct any annotations from predictions that turn out to be incorrect, making sure to click on the "Update" button for each one. If you have several annotations for a task, you need only keep one - the correct one - regardless of its position in the text.

9. Finally, export the annotated archival inventory by clicking on the "Export" button in the task display interface. Be sure to check "CSV". The file is downloaded; you can rename it, for example, with the archival description number, followed by "_labels".

## XML/EAD encoding generation

1. Launch the notebook. Launch the various installations and imports.
2. Load schematics. 
It's a good idea to place examples in the "c" schema from the archival inventory you're about to process. For example, you can place an example for each title level. If the documentary units are quite varied and complex, you can multiply the examples, so that the LLM can recognize the different cases that present themselves at the time of the inference (generation) task. 
Be sure to add a prefix "[Title n]" before the extract from the archival description, and to place a number after "c", the n of "[Title n]". Similarly, if you add an example with "scopecontent" or "bioghist", you must number the keys "p" if there are several, for example: 
```
      "- Tous correspondants – classé par correspondants"
              "- Sans répertoire"
              """,
              """
              "p1" : "- Tous correspondants – classé par correspondants"
              "p2" : "- Sans répertoire"
```

There is a limit to the size of the prompt with gpt 3.5, which is 4096 tokens. This is why you can't multiply the examples ad infinitum. On the other hand, if you have long documentary units, you'll need to increase the max_tokens parameter in ChatOpenAI, since this number is taken into account when calculating the number of prompt tokens, which is limited, as we said, to 4096. 
```
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",
    max_tokens=1024,
)
``` 
3. Load the llm and the data extraction chain.
4. Import the archival description of the archival inventory, whether annotated manually or automatically, into the drive attached to colab and transform each documentary unit into a list item.
5. The XML/EAD encoding of each list item is then automatically generated.  Depending on the length of the archival inventory, the inference time may vary.
6. Check that all the elements in the list have been processed by the LLM. If this is not the case, we isolate these elements and retry a generation. If this second attempt is unsuccessful, we list the elements that could not be generated and their index, and, knowing their position, we can manually add their XML/EAD encoding (in JSON format) to the JSON file resulting from the list we have formed.
7. We measure the cosine similarity between only the end-to-end values of each element in the generation list and the elements in the annotation list (minus the elements that could not be generated). Elements in each list with a low cosine similarity (e.g. less than 0.6) are displayed.
8. The list of generations is saved in a JSON file. This file is opened.
9. The next step is to check the accuracy of the encoding in the JSON file. The following corrections can be targeted:
-check that there are no "Titles" in the file, and that the correct numbers have been added after the "c" keys
-identify in the JSON the position of list items that have not been generated, and insert them 
-we correct in the JSON the elements of the two lists that have been identified as having a low cosine similarity
-finally, we check the accuracy of the remaining elements in the JSON file and make any necessary corrections.
10. Once the corrections have been made, we load the corrected JSON file into our drive and transform each JSON element into an XML <c> tag.
11. Load an xslt stylesheet, which converts c1 tags into c2 parent tags, and so on. Templates can be added to the xslt style sheet if any are missing.
12. We then convert the initial XML into an XML with <c> tags nesting.

