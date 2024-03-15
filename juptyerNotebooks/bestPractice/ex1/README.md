# About

I use Jupyter notebooks to process datasets. As such I am connecting the notebook to databases, external APIs, and other resources requiring connections I want to remain private and not be included in the notebook. 
In this example I will outline how I segment and separate the elements of the notebook so not everything is added directly to the notebook file.

## Title

Jupyter Notebook Best Practice - configuration

## Outline

For power users of Juptyer notebooks, there probably comes a time when your notebook file grows to become unweildy and difficult to scroll through because all of your markdown, code, and output. But how best to manage 
the notebook content? For me, the worker code is best left out of the notebook file, and especially any notebook configuration where you are dealing with sensitive database or API credentials which should not be saved
to a source-control repository. So what could this configuration look like? We will start with the configuration file.

Below are a list of files I use for my notebook configuration. All files are located within the same directory as the notebook file in this example.

### `_config.dev.json` (or maybe `_config.prod.json` or `_config.test.json`)

I store all of my notebook configuration within a JSON file. This allows for easy parsing of the object using Python since it is simple formatted text. So my database, API tokens, or other credentials, any paths or values specific to my environment would go here. This makes your notebook reusable by others or other environments (think development, test, and production environments for yourself) without needing to necessarily change your notebook code. 

```
{
    "aboutThisConfiguration":"This configuration file can contain anything you would normally place in a JSON file. The flexibility is the beauty.",
    "aboutThisProject":"The purpose of this project is to show how to configure a Jupyter notebook for more advanced operations and management.",
    "MYSQL_USERNAME": "userHvd8",
    "MYSQL_PASSWORD": "ByGcRz8uzZztz8bikD",
    "MYSQL_HOST": "host.docker.internal",
    "MYSQL_PORT": 3308,
    "MYSQL_DATABASE": "sampledb",
    "DOCKER_MOUNT_DIR": "/home/jovyan/DockerContainerMountDir/",
    "DOCKER_DATA_DIR": "data/",
    "blnUseLabels": "false",
    "surveyList": [
        {
            "frm24hCovidScreen": {
                "name": "24-hour COVID Screening",
                "id": "Rc8ffar7sqwx9zH"
            }
        }
    ]
}
```

### `_worker.py`

Next, we move on to the scripting file. Personally I use Python for most of my work, so this example will only cover Python within a Jupyter notebook. When I was first learning Jupyter I placed all of the Python code 
within the notebook. When I outgrew that I switched to this external Python file for storing my 'working' code. This is the code that performs operations for the notebook. Note this script contains the import statements needed for the notebook.

```
# @title Here we create a main working object so our configuration and database connections can more easily integrate. Also separating the code from the notebooks makes it easier to read and manage the notebook.
import pandas as pd
from pandas import DataFrame
import requests
import json
import time
import io
import os

# @title This is were we place the bulk of our code so functions can easily access the notebook configuration, database connections, etc.
class Worker:
    def __init__(self, strConfigFileName):
        f = open ("_config.json", "r") # read the notebook configuration settings
        self._config = json.loads(f.read()) # save the configuration to a `_config` variable within the Worker object
        f.close()

    # @title Return the notebook configuration
    def objConfig(self):
        return self._config

    # @title Simply retrieve a configuration value 
    def getAbout(self):
        print(self._config["aboutThisProject"])

    # @title Simply retrieve a configuration value 
    def runCalc(self):
        print(self._config["aboutThisProject"])
# --- end class Worker
```

### `_installer.py`

One final script that most probably do not need is 

```
# this script takes care of installing extra modules
import subprocess
import sys

def install(package):
    subprocess.check_call([sys.executable, "-m", "pip", "install", package])

install("mysql-connector-python")
install("pymysql")
```

### Notebook

One of the first Python blocks I create is an 'installer' block which reads our notebook configuration and installs any packages or modules the notebook requires. I like to begin and end my code blocks with simple print statements so I can easily see when the code begins and ends execution (without needing to rely on knowing where the 'working' icon is located within the Jupyter interactive environment such as JupyterLab or Google Colab).

```(Python code block)
print("Install things and initialize our Worker object")
import installer   # this is only if we need to install some additional packages we define within `_installer.py`.
%load_ext autoreload
%autoreload all
# we need the 'autoreload' above if we are actively making changes to the worker.py module and want to reload any changes to the module without restarting the notebook kernel

from worker import Worker
objWorker = Worker("_config.dev.json") # we should only need to call this once for the notebook session (to load our dev configuration)
print("- finished installing and importing modules")
```

Then additional Python code blocks would simply call a function within the Worker objects such as:

```(Python code block)
objWorker.getAbout() # which simply prints `The purpose of this project is to show how to configure a Jupyter notebook for more advanced operations and management.` from our configuration file
```
