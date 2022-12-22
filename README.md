# Cordiance Project

In this project, we aim to map categories between 2 files by designing an efficient text-matching algorithm. The first file, Avalara_goods_and_services.xlsx, is a well-structured file of goods and services with somewhat uniform fields; whereas the second file, UNSPSC_English.csv is hierarchical, but the fields are messy and non-uniform.

Our task is to map categories from the reference file: Avalara_goods_and_services.xlsx to the file UNSPSC_English.csv, clean and process data, design matching algorithms and report the number of valid matches and corresponding matching percentages.

In our “Improved Brute Force” iteration, our matching percentage being roughly 2691 matches/2000 rows(test data)* 8 columns * 8 words(an approximate average number of words in each field), which is about 2%). In our trie algorithm, we matched around 70% of the items with 30% matching accuracy.

The objective of the project is to design and implement an efficient algorithm that can accurately match two categories using as little readily available libraries or trained data as possible. 

# Text pre-processing

We wrote a text-preprocessing function using NLP libraries and regular expressions to filter out unwanted punctuation, stopwords, singularize plural words, and stem the words to keep prefixes of words. For example, the word “electronically” should be matched with “electronic” or “electrical”, we stemmed the words using Porter stemming from NLTK, which keeps the main prefixes of words to help us find more matches in our data. We kept "-" and "/" as a string separator instead of filtering them out because the words before and after those punctuations are helpful to the matching.

<img width="900" alt="Screen Shot 2022-12-22 at 5 10 05 PM" src="https://user-images.githubusercontent.com/101501539/209099248-ab98a79b-83e5-44f1-93a6-20e132478a82.png">

# Why Levenshtein Distance would be less accurate

Initially we tried to match strings by calculating the Levenshtein distance between words, which calculates the ratio of match between two words based on how many edits are needed to change one string to another. However, this not only proved inefficient and less accurate, since the Levenshtein ratio only takes in account on number of edits are needed, but not the nature nor meaning of the word. For example, the words "mice" and "rice" have a very high Levenshtein ratio, yet they are very different words. 

# Trie Data Structure 

To speed up the matching process, we tried turning the UNSPSC data into  a Trie data structure. Each “Trie Node” will consist of the word list of the UNSPSC data according to its levels (Segment, Family, Class, Commodity), in that order. However, after many trials we found that the Family level was too broad and not very helpful, so we only worked with Segment, Class and Commodity level. The first level of the Trie is the Segment level. To increase matches we merged Segment Titles with Segment Definitions, Class Titles with Commodity Titles for the second level, and Commodity Titles for the third level of matching. Finally at the end of the Trie we have the corresponding UNSPSC Commodity code.

<img width="487" alt="Screen Shot 2022-12-22 at 5 19 53 PM" src="https://user-images.githubusercontent.com/101501539/209102532-9a1510e4-02c7-405a-a3cf-a9f234924568.png">
We implemented the Trie data structure using nested dictionaries. 
<br>
Essentially, our structure would look like the following:
<br>
{Segment:{Commodity+Class:{Commodity Title:{Commodity Code:{12345 }}}}}

An example of the trie would look like the following:
<br>
<img width="900" alt="Screen Shot 2022-12-22 at 5 20 09 PM" src="https://user-images.githubusercontent.com/101501539/209102653-ed15bbe0-23b9-494b-93e8-d9dc33ada628.png">

## Matching algorithm

```python

import copy
from prompt_toolkit.shortcuts.progress_bar.formatters import D
from tqdm import tqdm

def compareWordList(newTree,data_list):
  
   def get_max_score(tree,data_list,score):
       matches = {k:score for k in tree.keys()}
       # matches = defaultdict(int)
       curr_tree = tree
       for word in data_list:
           for key,val in curr_tree.items():
            
               if word in set(key):
                   matches[key]+= 1


       # print("matches",matches)      
       max_score = max(matches.values())
       #when there are ties in number of matches, put all candidates in a list
       get_max_match = [(k,v,curr_tree[k]) for k,v in matches.items() if v == max_score]
       # print("get_max_match",get_max_match)      

       return get_max_match

   #search trie with highest match score
   max_match = get_max_score(newTree,data_list,0)

   #level 2 search: commodity and class
   for key,score,level in max_match:
       new_max_match = get_max_score(level,data_list,score)
       # print("new_max_match",new_max_match)
  
   #level 3 search: commodity level matching
   curr_res = []
   for key,score,level in new_max_match:
       comm_max_match = get_max_score(level,data_list,score)
       curr_res.extend(copy.deepcopy(comm_max_match))
  
   # print(curr_res)
   max_value = -1
   max_level = None
   for key,score,level in curr_res:
       if score > max_value:
           max_value = score
           max_level = level
   return max_level["comm_code"]  

```

# Instruction to run 

1. Unzip "files" 
2. Import unzipped files (Avalara_goods_and_services.xlsx, UNSPSC_English.xls) to google drive, in the path of "/content/drive/MyDrive/Colab_Notebooks/files".
3. In RunTime tab, "Run All"

# Project 
Debra's Copy of Cordiance Project: Improved Brute Force: Tree, non-stop and stop
<br>
CordianceProject2.ipynb: final project result 

# Final report
Full PDF report in "FinalProjectReport.pdf"

# Result

We picked out 15 sample results for demonstration in "results.csv" as the last column (corresponding code column) in the Avalara file.
<br>
About 30% of the matches appeared to be accurate matches, while some matches are completely off. Some matches are inaccurate but related in meaning or category, as shown below:

<img width="1443" alt="Screen Shot 2022-12-22 at 5 30 33 PM" src="https://user-images.githubusercontent.com/101501539/209103413-2f216108-1889-4e23-a2a6-efa6fe685ed0.png">

# Conclusion and Improvements

We used the Trie data structure with Segment level being the first level of filter. Even though we combined the segment title with definitions, we realized for some results, the first level is still too broad. This would result in less accurate matches in the first level for some data. 
<br>
Matching Algorithm could improve accuracy if we alter the Trie data structure to include more information on the first level, however the runtime would be longer because there are now more keys to match on the first level.

# Authors
Amanda Au-Yeung, Wenqiao Xu, Yian Chen

