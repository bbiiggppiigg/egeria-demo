# Egeria Artifact 
Source code for the paper, Egeria: A Framework for Automatic Synthesis of HPC Advising Tools through Multi-Layered Natural Language Processing, submitted to SC'17.


The repo contains two parts: Egeria utils and the adiving tools generated by Egeria (cuda, opencl, and xeon). We first focus on the usage of demo systems and then explain how to construct a new advising tool for other HPC documents using Egeria utils. 

# Usage of Demo Systems
Three demo advising tools correspond to three folders: _cuda_, _opencl_, and _xeon_. The HTML documentations used are [Cuda Programming Guide](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html), [OpenCL Optimization Guide](http://developer.amd.com/amd-accelerated-parallel-processing-app-sdk/opencl-optimization-guide/), [Xeon Best Practice Guide](http://www.prace-ri.eu/best-practice-guide-intel-xeon-phi-html/) respectively. 

## Quick Start
To launch a demo system: 
* Install the following dependencies: Gunicorn, Flask, Textract, Gensim, NLTK, BeautifulSoup
* Open the project folder (e.g. cuda) 
* Setup the host IP address (host) and the port number (port) in Config.py and run.sh
* Run in the command line (linux only):
```./run.sh```

The demo will automatically create a website with a webpage presenting the advising sentences generated by the cuda advisor. The default number of worker are 4 and the port number is 5000. These can be changed in the Config.py file inside the folder. [Here](http://152.14.86.153:5000) is a running instance of cuda demo.


## Query Tips 
Since the techniques we used are the simple TF-IDF and VSM models. The adivising tools constructed by Egeria are quite sensitive to the the keywords used in the query. The documentations used in the demo usually contain only 100-200 advising sentences. The dictionaries built based on these advising sentences are very likely to miss lots of words. Here are the tips for using the advising tool:

* Various format of the same word will be regarded as one word because we stem each word in an input query before it is used to retrieve sentences.  For example, "warp" and "warps" are regarded as the same in the query. 
* NVVP reports should only be used in the cuda advisor even though advisors based on other documentations can also parse the reports. 
* If the query yields "No Relevant Sentences Found!", then try to change the keywords used in the query. It is very likely that the keywords do not exist in the advising sentences summary. 
* If too few sentences are selected, one can lower the similarity threshold to retrieve more sentences. 
* Either keywords or full sentences can be used as queries. VSM model works the same with a single word query or a query with many words. However, having the same word repeated in the query will increase the importance of that word when retrieving similar sentences. 
* As the dictionary is quite small, similar queries with different keywords might lead to very different responses. One can try to change some keywords in their query. 


# Usage of Egeria
Egeria is a framework for automatic synthesis of advising tools. To create a new advising tool, two folders are related: _advising_tool_template_ and _utils_. 
_advising_tool_template_ provides the web template for an advising tool and _utils_ provide the functions and classes to build an advising tool. 

To build advising tools for other documents, the process is as follows:
* Copy the _advising_tool_template_ folder and cd to the copied folder. 
* Setup the document name(guidename), host IP(host), host port(port) in Config.py.
* Implement the _parser_ function in the Parser.py to preprocess the document into a sequence of text blocks. 
	* input: filename (correspond to _guidename_ in Config.py)
	* output: a list of dictionary contains the following keys
		* "title": section title for the block,
		* "href": id for the title, 
		* "body": block content, 
		* "state": natural number used to describe the hierarchical level of the section.
* Enable CoreNLP according to the description on the [official website](https://stanfordnlp.github.io/CoreNLP/corenlp-server.html). 
* (Optional) Custimize the set of keywords and other configurations in Config.py. If it is a HTML-based documentation, then customizing _weblink_ could enable the hyperlink to the original documentation. 
* Invoke the advising tools through ```./run.sh```

The script starts a pipeline to build the advising tool:
1. preprocess the document using the _parser_ function defined in Parser.py
2. create _rawIndex.html_ using the entire sentences, recognize advising sentences and use the summary to create _index.html_. The two HTML files are saved in the _templates_folder. 
3. build dictionary and TF-IDF models and save them in the _model_ folder
4. start the flask app defined in _openk.py_ using gunicorn


## check-list ( meta information)
* Algorithm: Semantic Role Labeling (SRL), Dependency Parsing (DP), Word Tokenization, Stemming, Normalization, Term Frequency-Inverse Document Frequency (TF-IDF), Vector Space Model (VSM)
* Program: Python 2.7.10
* Output: suggestions on program optimiztion given a query

## Software depencencies
* Text-related: CoreNLP 3.7.0 (DP), Pycorenlp 0.3.0 (Python interface for CoreNLP), Pracnlptools 1.0 (SENNA), NLTK 3.2.1 (SnowballStemmer, WordNetLemmatizer, sent_tokenize, word_tokenize), Gensim (VSM, TF-IDF), Textract (NVVP report parser), 
* Web-related: Gunicorn (19.6.0), Flask 0.11.1, BeautifulSoup 4.4.1.
 
# Folders
* utils: contains the Egeria functions used to construct advising tool tools
* advising_tool_template: contains the templates related to web-based functions. 
* cuda/opencl/xeon: the advising tools
	* models: tfidf model and the dictionary
	* nvvp-reports(cuda only): sample NVVP reports, ground truth responses, and programs used in the evaluation.  
	* static: scripts used for the webpages
	* templates: htmls to be rendered
	* uploads: store the uploaded NVVP report
	* Config.py: configurations including keywords, weblink to the documentations, similairty score threshold, port and host, etc. 
	* openk.py: create a flask app
	* Parser.py: the parser for the original HTML documentation; specific to the documentation. The output of the parser should be a list of dict with the format:
		* "title": section title for the block,
		* "href": id for the title, 
		* "body": block content, 
		* "state": natural number used to describe the hierarchical level of the section.
	* Setup.py: build dictionary and tfidf model
	* run.sh: execute Python Setup.py and then run the advising tool

