% Jython - Improving Java Integration
% Jim Baker, Rackspace


Problem
=======

* Support large-scale distributed computation systems like Hadoop, Storm, GraphLab
* With language of choice
* Example: Jython has seen some success on Hadoop with Pig for defining UDFs


Storm
=====

* "Real-time" complex event processing system
* Runs topology of storms, bolts to process events ("tuples")
* Can support at-least-once, exactly-once semantics


In Jython
=========

Pieces of imports:

````python
from backtype.storm import Config, Constants
from backtype.storm.topology import TopologyBuilder
from backtype.storm.topology.base import BaseRichBolt, BaseRichSpout
from backtype.storm.tuple import Fields, Values

from clamp import PackageProxy
````

`PolicyBolt`nitoringSpout`
=================

````python
class PolicyBolt(BaseRichBolt):

    __proxymaker__ = PackageProxy("otter")

  # noarg constructor; depends on Storm initializing
  # through prepare
  def prepare(self, conf, context, collector):
    self._collector = collector
    self.policies = \
      defaultdict(partial(Policy, age=15.))

  def declareOutputFields(self, declarer):
    declarer.declare(Fields(["asg", "decision"]))

  def getComponentConfiguration(self):
    # Every 5 seconds, make a decision -
    # speed up because we're impatient humans
    return { Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS: 5 }
````


Supporting code
===============

````python
from org.python.compiler import CustomMaker

class SerializableProxies(CustomMaker):
  # NOTE: SerializableProxies is itself a java proxy,
  # but it's not a custom one!

  def doConstants(self):
    self.classfile.addField("serialVersionUID",
      CodegenUtils.ci(java.lang.Long.TYPE),
      Modifier.PUBLIC | Modifier.STATIC | Modifier.FINAL)
    code = self.classfile.addMethod("<clinit>", 
      ProxyCodeHelpers.makeSig("V"), Modifier.STATIC)
    code.visitLdcInsn(java.lang.Long(1))
    code.putstatic(self.classfile.name, 
      "serialVersionUID",
      CodegenUtils.ci(java.lang.Long.TYPE))
    code.return_()

    #...
````


Clamp
=====

* Standard issue - our Jython code needs to work with your system on the JVM
* Java is the standard level of interop
* Java, from a classfile perspective
* For Storm: serializablity (mostly don't care about Kryo optimization - just moving code here), resolution on the `CLASSPATH`
* In general: support a specific output shape
* => clamp project to bind again `__proxymaker__` protocol


Exposing
========

* Core type of Jython runtime is `PyObject`
* due to lack of interface injection

Pipeline: presentations as code
===============================

Uses a nice pipeline of

* github
* markdown
* pandoc
* beamer
* \LaTeX, but only for isolated bits
* Maybe a good template for your own presentations, feel free to use!


https://github.com/jimbaker/jvm-language-summit

