# 2022-spring-dov-level-ie
To run TANL:

  1. Install all files and folders from TANL scripts
  2. In base_dataset.py make any necessary changes to method generate_output_sentence (line 212) in class BaseDataset to get the model outputs (currently it is configured to output for MUC); the example variable is an InputExample object (reference input_example.py for more details) and should contain any information needed to be extracted
  3. Run a shell command of the form: !run.py dataset_name
  4. The dataset name will appear in the file config.ini (for example, muc_event corresponds to MUC)
  5. For more information, please reference the GitHub for TANL, found here: https://github.com/amazon-research/tanl

  
To run TANL on a new dataset:
  
  1. Identify the task you want to run with this new dataset
  2. In datasets.py create a new class for this dataset by copying the class of an exisiting dataset performing the same task and renaming
  3. Read through the methods used for inputting the dataset (usually called load_data_single_split and/or load_schema) to figure out the dictionary format for the input and format the new dataset accordingly
  4. Look for the method that does evaluation (usually called evaluate_dataset) and search for where the model directly outputs information (usually generate_output_sentences in base_dataset.py); write any code to write the model output to some text file
  5. Convert what's in the text file to the format accepted by the error analysis system; some examples here: https://github.com/IceJinx33/auto-err-template-fill/tree/main/model_outputs
  

Error Analysis script information here: https://github.com/IceJinx33/auto-err-template-fill
  
  
Random comments/thoughts:

  1. It's easy to get things running on Colab, but the memory constraints are problematic (usually can only input 512 tokens for training). If G2 works, train using that. The TANL GitHub mentioned earlier has information on how to change input token counts.
  2. There seems to be no good way to run SciREX on TANL. The ideal TANL task for SciREX is event extraction. However, it is difficult to do argument annotations on SciREX each entity in the 4-ary relations (denoting the templates) usually has many coreferring spans. Therefore, there's no good way of determining which span(s) correspond to an entity in any of the 4-ary relations. Note that the argument labeling method used by MUC (finding the first span in the text that corresponds to an argument span) does not work well here because SciREX is huge.
  3. I initially did the TANL NER task on SciREX. However, this doesn't correspond well with what the other models are doing (especially GTT). NER just extracts all spans that belong to one of several types we're interested in. The extracted spans provide no information on which spans are the most salient to the document (while event extraction aims at extracting as arguments only the spans that are most important). Furthermore, there will be too many spans for the error analysis system to run at all.
  4. If event extraction can't be done, a possible solution may be to generate templates heuristically by combining results from NER, coref resolution, and relation extraction tasks. The idea here is to use the number of coreffering spans and relations on a span or entity to assign a "saliency score" to each span. Then, use the highest scoring spans to fill out a template. This will require datasets that have annotations for all these tasks (some examples I looked at include DocRED and WLP datasets). When looking for possible datasets, also make sure that annotations are done on a document level.
  5. The current approach to trigger labeling for MUC is to find a set of most common words appearing in documents of each event type, then creating a list of possible triggers for each type. The list is used to label each document and might need additions depending on if each document can be labeled. When doing the labeling, make sure that each template has a trigger and that no single trigger (more precisely span) corresponds to multiple templates.
  6. Something I didn't get to try for MUC trigger labeling is to find triggers that are "in the middle of" all its corresponding arguments. This approach means that, given a set of spans corresponding to event arguments, we find a trigger that minimizes the average distance between it and all of the spans. The motivation here is that relevant tokens are closer together and the attention mechanism may perform better.
  7. I did not work on Dygie++. However, I think performance on it can be improved if we perform an event extraction task on it (instead of NER). It should be feasible now, at least with MUC, now that we have triggers labeled.


If you have any questions, feel free to email me: zc272@cornell.edu
