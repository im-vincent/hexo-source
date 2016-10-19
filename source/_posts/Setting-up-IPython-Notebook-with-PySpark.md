title: Setting up IPython Notebook with PySpark
date: 2015-09-15 12:57:31
tags: spark
---
# Install Spark
Download latest spark 
- [spark-1.4.1-bin-hadoop2.6.tgz](http://www.apache.org/dyn/closer.cgi/spark/spark-1.4.1/spark-1.4.1-bin-hadoop2.6.tgz)
- tar xvfz spark-1.4.1-bin-hadoop2.6.tgz -C /usr/local/


Setup your environment variables for "SPARK_HOME" E.g. in Unix environments, add the following to ~/.bash_profile
- export SPARK_HOME=`<location of the install>`
- export PATH=$SPARK_HOME/bin:$PATH

# Install Anaconda
Download Anaconda which include python 2.7 and the main scientific
- [http://ipython.org/install.html](http://ipython.org/install.html)

# Run PySpark from IPython notebook
Ipython notebook is sort of similar to Mathematica. Its a web app that allows you to write descriptions, images, visualization and executing run code.
Download the following python setup script from Github to create a new pyspark profile for running Ipython notebook [https://github.com/felixcheung/vagrant-projects/blob/master/Spark-IPython-Zeppelin-Lightning/ipython-pyspark.py](https://github.com/felixcheung/vagrant-projects/blob/master/Spark-IPython-Zeppelin-Lightning/ipython-pyspark.py) Run
- python ipython-pyspark.py


