Sioux
====================
.. image:: http://morgoth.cs.illinois.edu:8080/buildStatus/icon?job=python-utils
    :target: http://morgoth.cs.illinois.edu:8080/job/python-utils/
.. image:: https://semaphoreci.com/api/v1/projects/dc68ab4d-d1b7-4405-adca-b0c6af2e1aa0/1223617/badge.svg
    :target: https://semaphoreci.com/danyaljj/sioux-2
.. image:: https://img.shields.io/badge/%3E%3E%3E-Api%20Docs-brightgreen.svg
    :target: http://cogcomp.cs.illinois.edu/software/doc/sioux/

Run NLP tools such as Part-of-Speech tagging, Chunking, Named Entity Recognition, etc on your documents in Python with ease and breeze!

Installation
------------

1. Make sure `you have "pip" on your system <https://pip.pypa.io/en/stable/installing/>`_. 
2. Make sure you have installed Cython:

    pip install cython

3. Install:

    pip install sioux

4. Enjoy!

**Note:** The package should be compatible with Python 2.6+ and Python 3.3+

**Upgrading:** If you want to update your package: 

   pip install --upgrade sioux

If you want to upgrade upgrade it on a specific version replace :code:`pip` in the command above with :code:`pip2` or :code:`pip3`. 

Getting Started 
---------------
Here is a sample usage showing how yeezily you run Sioux: 

.. code-block:: python

   from sioux import remote_pipeline

   pipeline = remote_pipeline.RemotePipeline()
   doc = pipeline.doc("Hello, how are you. I am doing fine")
   print(doc.get_lemma) # will produce (hello Hello) (, ,) (how how) (be are) (you you) (. .) (i I) (be am) (do doing) (fine fine)
   print(doc.get_pos) # will produce (UH Hello) (, ,) (WRB how) (VBP are) (PRP you) (. .) (PRP I) (VBP am) (VBG doing) (JJ fine)

The default/easy usage has some restrictions as will deliniate in the next section. See the next section to 

**Api Docs:** Here is the `API docs <http://cogcomp.cs.illinois.edu/software/doc/sioux/pipeliner.m.html>`_ of our `Pipeliner` module.

Structure   
----------------------------- 
Sioux enables you accesss `CogComp pipeline <https://github.com/CogComp/cogcomp-nlp/tree/master/pipeline>`_ in different forms. The figure below summarizes these approaches: 

.. figure:: https://cloud.githubusercontent.com/assets/2441454/25446100/0e06d190-2a76-11e7-9438-f9a9bde717e0.png
   :scale: 50 %

The figure above gives a summary of possible usages, as well as their pros and cons. Next we will go through each item and elaborate: 

Remote Pipeline 
~~~~~~~~~~~~~~~~~~~~~~
In this setting, Sioux sends annotation requests to a remote machine. Hence there is not much memory burden on your local machine. Instead all the heavy-lifting is on the remote server. 

**Default remote server:**  This is the default setting in Sioux. The requests are sent to our remote server, hence requires a network connection. This option is here to demonstrate how things work, but it is not a viable solution for your big experiments since we limit the number of queries to our serve (current limit is *100 queries a day*). If you are a busy nlp user, you should use any of the other options. 

**Starting your own (remote) server:** If you have a big (remote) machine, this is probably a good option for you. 
You'll have to read the instructions on how to install the pipeline server in the `pipeline project documentation <https://github.com/CogComp/cogcomp-nlp/tree/master/pipeline#using-pipeline-webserver>`_. In summary: 

1. Clone our  `Cogcomp-NLP <https://github.com/CogComp/cogcomp-nlp/>`_ java project. 
2. Run :code:`pipeline/scripts/runWebserver.sh` to start the server. 
3. When you see :code:`Server:xxx - Started @xxxxxms`, the server is up and running: 
  
After making sure that the server is running, we can make python call to it: 

.. code-block:: python

   from sioux import remote_pipeline
   pipeline = remote_pipeline.RemotePipeline(server_api='http://www.fancyUrlName.com:8080') 
   # constructor declaration: RemotePipeline(server_api = None, file_name = None)
   # "server_api" is the address of the server as string. An example: http://www.fancyUrlName.com:8080
   # "file_name" is the config file used to set up pipeline (optional), please refer the latter section for more details

**Note:** This tool is based on CogComp's `pipeline project <https://github.com/CogComp/cogcomp-nlp/tree/master/pipeline>`_. Essentially annotator included in the pipeline should be accessible here. 
 
Local Pipeline 
~~~~~~~~~~~~~~~~~~~~~~
In this setting, Sioux will download the trained models and files required to run the pipeline locally. Since everything is run on your machine, it will probably require a lot of memory (the amount depends on which annotations you use). If you have a single big machine (i.e. memory > 15GB) for your expeirments, this is probably a good option for you. 

To download the models, run the following command:

  python -m sioux download

This will download model files into your home directly under :code:`~/.sioux/`. 

**Note:** Note that downloading the models require you to have Maven installed on your machine. If you don't, `here are some guidelines on how to install it <https://maven.apache.org/install.html>`_. 

**Note:** To use the pipelne locally (A) you have to make sure you have set :code:`JAVA_HOME` variable. In MacOS, you can verify it with :code:`echo "$JAVA_HOME"`. If it is not set, you can :code:`export JAVA_HOME=$(/usr/libexec/java_home)`. 

In the local pipeline the views are disabled by default. If you want to change specific behaviors, such as activating or deactivating specific components, you can specify the parameters while initializing local/remote pipeline module.

.. code-block:: python

   from sioux import local_pipeline
   pipeline = local_pipeline.LocalPipeline(enable_views=['POS','LEMMA']) 
   # constructor declaration: LocalPipeline(enable_views = None, disable_views = None, file_name = None)
   # "enable_views" will takes a list of the view names to be used as strings, each string is the name of the view. This parameter is important only if you're using the local pipeline (A). 
   # "file_name" is the config file used to set up pipeline (optional), please refer the latter section for more details
   
Setting from Configuration file 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
You can set settings on how to run Sioux via a local option too, rather than setting it programmatically. 
Here is how to: 

.. code-block:: python

   from sioux import local_pipeline
   pipeline = local_pipeline.LocalPipeline(file_name = 'path_to_custom_config_file')

   
The default keys and values (true/false) when models have been downloaded are specified below. If you want to use custom config file, please provide a file in similar format.


.. code-block:: bash

    [local_pipeline_setting]
    POS = false
    LEMMA = false
    NER_CONLL = false
    NER_ONTONOTES = false
    QUANTITIES = false
    SHALLOW_PARSE = false
    SRL_VERB = false
    DEPENDENCY_STANFORD = false
    DEPENDENCY = false
    PARSE_STANFORD = false
    SRL_PREP = false

    [remote_pipeline_setting]
    api = ADDRESS_OF_THE_SERVER # example: http://fancyUrlName.com:8080/
    

Questions/Suggestions/Comments 
------------------------------
Use comments or pull requests. 

About the name 
-------------- 
It is pronounced similar to "Sue". The *Sioux* are groups of Native American tribes and First Nations peoples in North America, mostly the tribal governments scattered across North Dakota, South Dakota, Nebraska, Minnesota, and Montana in the United States; and Manitoba and southern Saskatchewan in Canada. (`Read more <https://en.wikipedia.org/wiki/Sioux>`_)


