<properties
   pageTitle="開發 Apache Storm 的 Java 型拓撲 | Microsoft Azure"
   description="了解如何建立簡單字數拓撲，以使用 Java 建立 Storm 拓撲。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="jhubbard"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
   ms.service="hdinsight"
   ms.devlang="java"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="09/14/2016"
   ms.author="larryfr"/>

#使用 HDInsight 上的 Apache Storm 和 Maven 開發基本字數統計應用程式的 Java 型拓撲

了解如何使用 Maven 為 Apache Storm on HDInsight 建立 Java 型拓撲。您將逐步進行使用 Maven 和 Java 建立基本字數統計應用程式的程序，其中拓撲是以 Java 定義。然後，您將了解如何使用 Flux 架構定義拓撲。

> [AZURE.NOTE] Flux 架構可在 Storm 0.10.0 或更新版本中使用。Storm 0.10.0 則可在 HDInsight 3.3 及 3.4 中使用。

完成這份文件中的步驟之後，就會有可部署到 Apache Storm on HDInsight 的基本拓撲。

> [AZURE.NOTE] [https://github.com/Azure-Samples/hdinsight-java-storm-wordcount](https://github.com/Azure-Samples/hdinsight-java-storm-wordcount) 有提供本文件中建立之拓撲的完整版本。

##必要條件

* <a href="https://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html" target="_blank">Java Developer Kit (JDK) 第 7 版</a>

* <a href="https://maven.apache.org/download.cgi" target="_blank">Maven</a>：Maven 是 Java 專案的專案建置系統。

* 文字編輯器，例如記事本、<a href="http://www.gnu.org/software/emacs/" target="_blank">Emacs<a>、<a href="http://www.sublimetext.com/" target="_blank">Sublime Text</a>、<a href="https://atom.io/" target="_blank">Atom.io</a>、<a href="http://brackets.io/" target="_blank">Brackets.io</a>。或者您可以使用整合式開發環境 (IDE)，例如 <a href="https://eclipse.org/" target="_blank">Eclipse</a> (Luna 版本或更新版本)。

	> [AZURE.NOTE] 您的編輯器或 IDE 可能具有處理 Maven 的特定功能，但未記載在這份文件中。如需編輯環境功能的詳細資訊，請參閱所使用產品的文件。

##設定環境變數

當您安裝 Java 和 JDK 時可能會設定下列環境變數。不過，您應該檢查它們是否存在，以及它們是否包含您系統的正確值。

* **JAVA\_HOME** - 應該指向已安裝 Java 執行階段環境 (JRE) 的目錄。例如，在 Unix 或 Linux 散發套件上，它的值應該類似 `/usr/lib/jvm/java-7-oracle`在 Windows 中，它的值應該類似 `c:\Program Files (x86)\Java\jre1.7`

* **PATH** - 應該包含下列路徑：

	* **JAVA\_HOME** (或對等的路徑)

	* **JAVA\_HOME\\bin** (或對等的路徑)

	* 已安裝 Maven 的目錄

##建立新的 Maven 專案

從命令列中，使用下列命令建立名為 **WordCount** 的新 Maven 專案：

	mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.microsoft.example -DartifactId=WordCount -DinteractiveMode=false

這會在目前的位置中建立名為 **WordCount** 的新目錄，其內含基本 Maven 專案。

**WordCount** 目錄將會包含下列項目：

* **pom.xml**：內含 Maven 專案的設定。

* **src\\main\\java\\com\\microsoft\\example**：內含應用程式碼。

* **src\\test\\java\\com\\microsoft\\example**：內含您應用程式的測試。在此範例中，我們不會建立測試。

###移除範例程式碼

因為我們要建立應用程式，所以請刪除產生的測試和應用程式檔案：

*  **src\\test\\java\\com\\microsoft\\example\\AppTest.java**

*  **src\\main\\java\\com\\microsoft\\example\\App.java**

##加入屬性

Maven 可讓您定義稱為屬性的專案層級值。將以下內容加到 `<url>http://maven.apache.org</url>` 行之後：

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!--
        Storm 0.10.0 is for HDInsight 3.3 and 3.4.
        To find the version information for earlier HDInsight cluster
        versions, see https://azure.microsoft.com/documentation/articles/hdinsight-component-versioning/
        -->
        <storm.version>0.10.0</storm.version>
    </properties>

現在，我們可以在其他區段中使用這些值。例如，在指定 Storm 元件的版本時，我們可以使用 `${storm.version}` 而不是將值硬式編碼。

##新增相依性

因為這是 Storm 拓撲，所以您必須新增 Storm 元件的相依性。開啟 **pom.xml** 檔案並在 **&lt;dependencies>** 區段中加入下列程式碼：

	<dependency>
	  <groupId>org.apache.storm</groupId>
	  <artifactId>storm-core</artifactId>
      <version>${storm.version}</version>
	  <!-- keep storm out of the jar-with-dependencies -->
	  <scope>provided</scope>
	</dependency>

在編譯期間，Maven 會使用此資訊來查閱 Maven 儲存機制中的 **storm-core**。它會先查看本機電腦上的儲存機制。如果檔案不存在，則會從公用 Maven 儲存機制進行下載，並將它們儲存在本機儲存機制中。

> [AZURE.NOTE]  請注意在我們加入之區段中的 `<scope>provided</scope>` 行。此行要求 Maven 從我們建立的任何 JAR 檔案中排除 **storm-core**，因為其將由系統提供。這可讓您建立的封裝變得較小，並確保它們將使用 Storm on HDInsight 叢集上所含的 **storm-core** 位元。

##建置組態

Maven 外掛程式可讓您自訂專案的建置階段 (例如，如何編譯專案或如何將它封裝成 JAR 檔案)。開啟 **pom.xml** 檔案，並直接在 `</project>` 行上方加入下列程式碼。

	<build>
	  <plugins>
	  </plugins>
      <resources>
      </resources>
	</build>

此區段將用來新增外掛程式、資源，和其他組建組態選項。如需 __pom.xml__ 檔案的完整參考，請參閱 [http://maven.apache.org/pom.html](http://maven.apache.org/pom.html) (英文)。

###加入外掛程式

在 Storm 拓撲中，<a href="http://mojo.codehaus.org/exec-maven-plugin/" target="_blank">Exec Maven 外掛程式</a>十分有用，因為它可讓您輕鬆地在開發環境上以本機執行拓撲。將下列內容加入 **pom.xml** 檔案的 `<plugins>` 區段，以包括 Exec Maven 外掛程式：

	<plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
      <version>1.4.0</version>
      <executions>
        <execution>
        <goals>
          <goal>exec</goal>
        </goals>
        </execution>
      </executions>
      <configuration>
        <executable>java</executable>
        <includeProjectDependencies>true</includeProjectDependencies>
        <includePluginDependencies>false</includePluginDependencies>
        <classpathScope>compile</classpathScope>
        <mainClass>${storm.topology}</mainClass>
      </configuration>
    </plugin>

> [AZURE.NOTE] 請注意，`<mainClass>` 項目會使用 `${storm.topology}`。我們先前並未在 [屬性] 區段中定義此屬性 (但我們可以定義)。 相反地，我們會在稍後的步驟中，於開發環境執行拓撲時從命令列設定此值。

另一個有用的外掛程式是 <a href="http://maven.apache.org/plugins/maven-compiler-plugin/" target="_blank">Apache Maven 編譯器外掛程式</a>，其可用來變更編譯選項。我們需要此外掛程式的主要原因是要變更 Maven 用於您應用程式之來源和目標的 Java 版本。我們需要 1.7 版。

在 **pom.xml** 的 `<plugins>` 區段中加入下列內容，以包括 Apache Maven 編譯器外掛程式並將來源和目標版本設定為 1.7。

	<plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.3</version>
      <configuration>
        <source>1.7</source>
        <target>1.7</target>
      </configuration>
    </plugin>

###Configure resources

resources 區段可讓您包含非程式碼資源，例如拓撲中元件所需的組態檔。對於此範例，在 **pom.xml** 檔案的 `<resources>` 區段中新增下列內容。

    <resource>
        <directory>${basedir}/resources</directory>
        <filtering>false</filtering>
        <includes>
          <include>log4j2.xml</include>
        </includes>
    </resource>

這會將專案根目錄 (`${basedir}`) 中的 resources 目錄新增為包含資源的位置，並且包含稱為 __log4j2.xml__ 的檔案。這個檔案是用來設定拓撲要記錄哪些資訊。

##建立拓撲

Java 型 Storm 拓撲包含三個您必須編寫 (或參考) 為相依性的元件。

* **Spout**：讀取來自外部來源的資料，並將資料流發出到拓撲。

* **Bolt**：執行 Spout 或其他 Bolt 所發出資料流的處理，並發出一或多個資料流。

* **拓撲**：定義如何排列 Spout 和 Bolt，並提供拓撲的進入點。

###建立 Spout

若要減少設定外部資料來源的需求，下列 Spout 只會發出隨機的句子。它是隨附於 [Storm-Starter 範例](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter)的 Spout 修正版。

> [AZURE.NOTE] 如需從外部資料來源讀取之 Spout 的範例，請參閱下列其中一個範例：
>
> * [TwitterSampleSPout](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter/spout/TwitterSampleSpout.java)：從 Twitter 讀取的 Spout 範例
>
> * [Storm-Kafka](https://github.com/apache/storm/tree/0.10.x-branch/external/storm-kafka)：從 Kafka 讀取的 Spout

針對 Spout，在 **src\\main\\java\\com\\microsoft\\example** 目錄中建立名為 **RandomSentenceSpout.java** 的新檔案，並使用下列項目做為內容：

    /**
     * Licensed to the Apache Software Foundation (ASF) under one
     * or more contributor license agreements.  See the NOTICE file
     * distributed with this work for additional information
     * regarding copyright ownership.  The ASF licenses this file
     * to you under the Apache License, Version 2.0 (the
     * "License"); you may not use this file except in compliance
     * with the License.  You may obtain a copy of the License at
     *
     * http://www.apache.org/licenses/LICENSE-2.0
     *
     * Unless required by applicable law or agreed to in writing, software
     * distributed under the License is distributed on an "AS IS" BASIS,
     * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     * See the License for the specific language governing permissions and
     * limitations under the License.
     */

     /**
      * Original is available at https://github.com/apache/storm/blob/master/examples/storm-starter/src/jvm/storm/starter/spout/RandomSentenceSpout.java
      */

    package com.microsoft.example;

    import backtype.storm.spout.SpoutOutputCollector;
    import backtype.storm.task.TopologyContext;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseRichSpout;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Values;
    import backtype.storm.utils.Utils;

    import java.util.Map;
    import java.util.Random;

    //This spout randomly emits sentences
    public class RandomSentenceSpout extends BaseRichSpout {
      //Collector used to emit output
      SpoutOutputCollector _collector;
      //Used to generate a random number
      Random _rand;

      //Open is called when an instance of the class is created
      @Override
      public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
      //Set the instance collector to the one passed in
        _collector = collector;
        //For randomness
        _rand = new Random();
      }

      //Emit data to the stream
      @Override
      public void nextTuple() {
      //Sleep for a bit
        Utils.sleep(100);
        //The sentences that will be randomly emitted
        String[] sentences = new String[]{ "the cow jumped over the moon", "an apple a day keeps the doctor away",
            "four score and seven years ago", "snow white and the seven dwarfs", "i am at two with nature" };
        //Randomly pick a sentence
        String sentence = sentences[_rand.nextInt(sentences.length)];
        //Emit the sentence
        _collector.emit(new Values(sentence));
      }

      //Ack is not implemented since this is a basic example
      @Override
      public void ack(Object id) {
      }

      //Fail is not implemented since this is a basic example
      @Override
      public void fail(Object id) {
      }

      //Declare the output fields. In this case, an sentence
      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("sentence"));
      }
    }

請用一些時間閱讀程式碼註解，以了解此 Spout 的運作方式。

> [AZURE.NOTE] 雖然此拓撲只使用一個 Spout，但是其他拓撲可能會有將資料從不同來源送入拓撲的數個 Spout。

###建立 Bolt

Bolt 會處理資料的處理。針對此拓撲，我們有兩個 Bolt：

* **SplitSentence**會將 **RandomSentenceSpout** 所發出的句子分割成個別單字。

* **WordCount**：計算每個單字的出現次數。

> [AZURE.NOTE] Bolt 幾乎可以包辦任何作業，例如計算、持續性或與外部元件交談。

在 **src\\main\\java\\com\\microsoft\\example** 目錄中，建立兩個新的檔案：**SplitSentence.java** 和 **WordCount.Java**。使用下列項目做為檔案的內容：

**SplitSentence**

    package com.microsoft.example;

    import java.text.BreakIterator;

    import backtype.storm.topology.BasicOutputCollector;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseBasicBolt;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Tuple;
    import backtype.storm.tuple.Values;

    //There are a variety of bolt types. In this case, we use BaseBasicBolt
    public class SplitSentence extends BaseBasicBolt {

      //Execute is called to process tuples
      @Override
      public void execute(Tuple tuple, BasicOutputCollector collector) {
        //Get the sentence content from the tuple
        String sentence = tuple.getString(0);
        //An iterator to get each word
        BreakIterator boundary=BreakIterator.getWordInstance();
        //Give the iterator the sentence
        boundary.setText(sentence);
        //Find the beginning first word
        int start=boundary.first();
        //Iterate over each word and emit it to the output stream
        for (int end=boundary.next(); end != BreakIterator.DONE; start=end, end=boundary.next()) {
          //get the word
          String word=sentence.substring(start,end);
          //If a word is whitespace characters, replace it with empty
          word=word.replaceAll("\\s+","");
          //if it's an actual word, emit it
          if (!word.equals("")) {
            collector.emit(new Values(word));
          }
        }
      }

      //Declare that emitted tuples will contain a word field
      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word"));
      }
    }

**WordCount**

    package com.microsoft.example;

    import java.util.HashMap;
    import java.util.Map;
    import java.util.Iterator;

    import backtype.storm.Constants;
    import backtype.storm.topology.BasicOutputCollector;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseBasicBolt;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Tuple;
    import backtype.storm.tuple.Values;
    import backtype.storm.Config;

    // For logging
    import org.apache.logging.log4j.Logger;
    import org.apache.logging.log4j.LogManager;

    //There are a variety of bolt types. In this case, we use BaseBasicBolt
    public class WordCount extends BaseBasicBolt {
        //Create logger for this class
        private static final Logger logger = LogManager.getLogger(WordCount.class);
        //For holding words and counts
        Map<String, Integer> counts = new HashMap<String, Integer>();
        //How often we emit a count of words
        private Integer emitFrequency;

        // Default constructor
        public WordCount() {
            emitFrequency=5; // Default to 60 seconds
        }

        // Constructor that sets emit frequency
        public WordCount(Integer frequency) {
            emitFrequency=frequency;
        }

        //Configure frequency of tick tuples for this bolt
        //This delivers a 'tick' tuple on a specific interval,
        //which is used to trigger certain actions
        @Override
        public Map<String, Object> getComponentConfiguration() {
            Config conf = new Config();
            conf.put(Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS, emitFrequency);
            return conf;
        }

        //execute is called to process tuples
        @Override
        public void execute(Tuple tuple, BasicOutputCollector collector) {
            //If it's a tick tuple, emit all words and counts
            if(tuple.getSourceComponent().equals(Constants.SYSTEM_COMPONENT_ID)
                    && tuple.getSourceStreamId().equals(Constants.SYSTEM_TICK_STREAM_ID)) {
                for(String word : counts.keySet()) {
                    Integer count = counts.get(word);
                    collector.emit(new Values(word, count));
                    logger.info("Emitting a count of " + count + " for word " + word);
                }
            } else {
                //Get the word contents from the tuple
                String word = tuple.getString(0);
                //Have we counted any already?
                Integer count = counts.get(word);
                if (count == null)
                    count = 0;
                //Increment the count and store it
                count++;
                counts.put(word, count);
            }
        }

        //Declare that we will emit a tuple containing two fields; word and count
        @Override
        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            declarer.declare(new Fields("word", "count"));
        }
    }

請用一些時間閱讀程式碼註解，以了解每個 Bolt 的運作方式。

###定義拓撲

拓撲會將 Spout 和 Bolt 一起繫結至圖形，其中定義元件之間的資料流動方式。它也會提供 Storm 在叢集內建立元件的執行個體時所使用的平行處理原則提示。

以下是此拓撲之元件圖的基本圖。

![顯示 Spout 和 Bolt 排列的圖表](./media/hdinsight-storm-develop-java-topology/wordcount-topology.png)

若要實作拓撲，請在 **src\\main\\java\\com\\microsoft\\example** 目錄中建立名為 **WordCountTopology.java** 的新檔案。使用下列項目做為檔案的內容：

	package com.microsoft.example;

    import backtype.storm.Config;
    import backtype.storm.LocalCluster;
    import backtype.storm.StormSubmitter;
    import backtype.storm.topology.TopologyBuilder;
    import backtype.storm.tuple.Fields;

    import com.microsoft.example.RandomSentenceSpout;

    public class WordCountTopology {

      //Entry point for the topology
      public static void main(String[] args) throws Exception {
      //Used to build the topology
        TopologyBuilder builder = new TopologyBuilder();
        //Add the spout, with a name of 'spout'
        //and parallelism hint of 5 executors
        builder.setSpout("spout", new RandomSentenceSpout(), 5);
        //Add the SplitSentence bolt, with a name of 'split'
        //and parallelism hint of 8 executors
        //shufflegrouping subscribes to the spout, and equally distributes
        //tuples (sentences) across instances of the SplitSentence bolt
        builder.setBolt("split", new SplitSentence(), 8).shuffleGrouping("spout");
        //Add the counter, with a name of 'count'
        //and parallelism hint of 12 executors
        //fieldsgrouping subscribes to the split bolt, and
        //ensures that the same word is sent to the same instance (group by field 'word')
        builder.setBolt("count", new WordCount(), 12).fieldsGrouping("split", new Fields("word"));

        //new configuration
        Config conf = new Config();
        //Set to false to disable debug information
        // when running in production mode.
        conf.setDebug(false);

        //If there are arguments, we are running on a cluster
        if (args != null && args.length > 0) {
          //parallelism hint to set the number of workers
          conf.setNumWorkers(3);
          //submit the topology
          StormSubmitter.submitTopology(args[0], conf, builder.createTopology());
        }
        //Otherwise, we are running locally
        else {
          //Cap the maximum number of executors that can be spawned
          //for a component to 3
          conf.setMaxTaskParallelism(3);
          //LocalCluster is used to run locally
          LocalCluster cluster = new LocalCluster();
          //submit the topology
          cluster.submitTopology("word-count", conf, builder.createTopology());
          //sleep
          Thread.sleep(10000);
          //shut down the cluster
          cluster.shutdown();
        }
      }
    }

請用一些時間閱讀程式碼註解，以了解拓撲的定義方式，然後提交至叢集。

###設定記錄

Storm 使用 Apache Log4j 來記錄資訊。如果未設定記錄，拓撲就會發出許多難以讀取的診斷資訊。若要控制記錄哪些資訊，請在 __resources__ 目錄中建立名為 __log4j2.xml__ 的檔案。使用下列內容做為檔案的內容。

    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration>
    <Appenders>
        <Console name="STDOUT" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="com.microsoft.example" level="trace" additivity="false">
            <AppenderRef ref="STDOUT"/>
        </Logger>
        <Root level="error">
            <Appender-Ref ref="STDOUT"/>
        </Root>
    </Loggers>
    </Configuration>

這會設定 __com.microsoft.example__ 類別的新記錄器，而該類別中會包含此範例拓撲的元件。此記錄器的層級是設為 trace，這樣會擷取此拓撲中元件發出的任何記錄資訊。如果您回顧此專案中的程式碼，您會發現只有 WordCount.java 檔案實作記錄，它會記錄每個單字的字數。

`<Root level="error">` 區段會設定記錄的根層級 (不在 __com.microsoft.example__ 中的所有項目)，只記錄錯誤資訊。

> [AZURE.IMPORTANT] 雖然這樣可以大幅減少在開發環境中測試拓撲時所記錄的資訊，但不會移除所有在生產環境叢集執行時產生的偵錯資訊。若要減少這類資訊，您也必須在提交到叢集的組態中，將偵錯設為 false。如需範例，請參閱本文件中的 WordCountTopology.java 程式碼。

如需針對 Log4j 設定記錄的詳細資訊，請參閱 [http://logging.apache.org/log4j/2.x/manual/configuration.html](http://logging.apache.org/log4j/2.x/manual/configuration.html) (英文)。

> [AZURE.NOTE] Storm 0.10.0 版使用 Log4j 2.x。舊版的 Storm 使用 Log4j 1.x，它們使用不同格式的記錄設定。如需舊版組態的詳細資訊，請參閱 [http://wiki.apache.org/logging-log4j/Log4jXmlFormat](http://wiki.apache.org/logging-log4j/Log4jXmlFormat) (英文)。

##在本機測試拓撲

儲存檔案之後，請使用下列命令在本機測試拓撲。

	mvn compile exec:java -Dstorm.topology=com.microsoft.example.WordCountTopology

它執行時，拓撲將顯示啟動資訊。然後在從 Spout 發出句子並由 Bolt 處理時，開始顯示類似如下的內容。

    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
    17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word snow

查看 WordCount Bolt 所發出的記錄，我們可以得知 'and' 已發出 113 次。只要拓撲還在執行，次數就會繼續增加，因為 Spout 會持續發出相同的句子。

另外，在發出單字和計算次數之間有 5 秒的間隔。這是因為 __WordCount__ 元件設定為只在計時 Tuple 抵達時發出資訊，而且它預設會要求這類 Tuple 每隔 5 秒才傳送。

## 將拓撲轉換為 Flux

Flux 是可在 Storm 0.10.0 中使用的新架構，可讓您區隔組態與實作。您的元件 (Bolt 和 Spout) 仍會以 Java 定義，但拓撲則會使用 YAML 檔案來定義。

YAML 檔案會定義要用於拓撲的元件、兩者間的資料流動方式，以及在初始化元件時要使用的值。您可以在部署 YAML 檔案時將它包含在含有專案的 jar 檔案中成為其一部分，或是在啟動拓撲時使用外部 YAML 檔案。

1. 將 __WordCountTopology.java__ 檔案移出專案。先前是以此檔案定義拓撲，但在 Flux 中我們不使用它。

2. 在 __resources__ 目錄中，建立名為 __topology.yaml__ 的新檔案。使用下列程式碼做為此檔案的內容。

        # topology definition

        # name to be used when submitting. This is what shows up...
        # in the Storm UI/storm command-line tool as the topology name
        # when submitted to Storm
        name: "wordcount"

        # Topology configuration
        config:
        # Hint for the number of workers to create
        topology.workers: 1

        # Spout definitions
        spouts:
        - id: "sentence-spout"
            className: "com.microsoft.example.RandomSentenceSpout"
            # parallelism hint
            parallelism: 1

        # Bolt definitions
        bolts:
        - id: "splitter-bolt"
            className: "com.microsoft.example.SplitSentence"
            parallelism: 1

        - id: "counter-bolt"
            className: "com.microsoft.example.WordCount"
            constructorArgs:
            - 10
            parallelism: 1

        # Stream definitions
        streams:
        - name: "Spout --> Splitter" # name isn't used (placeholder for logging, UI, etc.)
            # The stream emitter
            from: "sentence-spout"
            # The stream consumer
            to: "splitter-bolt"
            # Grouping type
            grouping:
            type: SHUFFLE

        - name: "Splitter -> Counter"
            from: "splitter-bolt"
            to: "counter-bolt"
            grouping:
            type: FIELDS
            # field(s) to group on
            args: ["word"]

    請花一點時間看過並了解每個區段的用途，以及它與 __WordCountTopology.java__ 檔案中的 Java 型定義的關係。

3. 對 __pom.xml__ 檔案進行下列變更。

    * 在 `<dependencies>` 區段新增下列新的相依性︰

            <!-- Add a dependency on the Flux framework -->
            <dependency>
                <groupId>org.apache.storm</groupId>
                <artifactId>flux-core</artifactId>
                <version>${storm.version}</version>
            </dependency>

    * 對 `<plugins>` 區段新增下列外掛程式。此外掛程式會負責建立專案的封裝 (jar 檔案)，並在建立封裝時套用一些 Flux 特定的轉換。

            <!-- build an uber jar -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <transformers>
                        <!-- Keep us from getting a "can't overwrite file error" -->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer" />
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                        <!-- We're using Flux, so refer to it as main -->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            <mainClass>org.apache.storm.flux.Flux</mainClass>
                        </transformer>
                    </transformers>
                    <!-- Keep us from getting a bad signature error -->
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

    * 在 __exec-maven-plugin__ `<configuration>` 區段中，將 `<mainClass>` 的值變更為 `org.apache.storm.flux.Flux`。這可讓 Flux 負責執行我們在開發時於本機執行的拓撲。

    * 在 `<resources>` 區段中，對 `<includes>` 新增下列內容。這包括會將拓撲定義為專案一部分的 YAML 檔案。
    
            <include>topology.yaml</include>

## 在本機測試 Flux 拓撲

1. 請使用下列命令，以 Maven 編譯和執行 Flux 拓撲。

        mvn compile exec:java -Dexec.args="--local -R /topology.yaml"
    
    如果您要使用 PowerShell，請使用下列命令︰
    
        mvn compile exec:java "-Dexec.args=--local -R /topology.yaml"

    如果您是在 Linux/Unix/OS X 系統上，而且[已在開發環境中安裝 Storm](http://storm.apache.org/releases/0.10.0/Setting-up-development-environment.html)，您可以改用下列命令︰

        mvn compile package
        storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local -R /topology.yaml

    `--local` 參數會在開發環境上以本機模式執行拓撲。`-R /topology.yaml` 參數會使用 jar 檔案中的 `topology.yaml` 檔案資源來定義拓撲。

    它執行時，拓撲將顯示啟動資訊。然後在從 Spout 發出句子並由 Bolt 處理時，開始顯示類似如下的內容。

        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
        17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
        17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
        17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
    
    每一批記錄資訊之間會有 10 秒的延遲，因為 `topology.yaml` 檔案會在 WordCount 元件建立時傳遞 `10` 的值。這會將計時 Tuple 的延遲間隔設定為 10 秒。

2.  從專案複製 `topology.yaml` 檔案。將它命名為 `newtopology.yaml` 之類的名稱。在檔案中找到下列區段，然後將 `10` 的值變更為 `5`。這會將發出單字計算的批次之間的間隔從 10 秒變更為 5 秒。

          - id: "counter-bolt"
            className: "com.microsoft.example.WordCount"
            constructorArgs:
            - 5
            parallelism: 1

3. 若要執行拓撲，請使用下列命令：

        mvn exec:java -Dexec.args="--local /path/to/newtopology.yaml"

    或者，如果您有位於 Linux/Unix/OS X 開發環境的 Storm︰

        storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local /path/to/newtopology.yaml

    將 `/path/to/newtopology.yaml` 變更為您在上一個步驟中建立的 newtopology.yaml 檔案的路徑。此命令會使用 newtopology.yaml 做為拓撲定義。因為我們並未包含 `compile` 參數，Maven 會重複使用先前步驟中建置的專案版本。

    拓撲啟動後，您應該會注意到批次發出間隔時間已變更，反映了 newtopology.yaml 中的值。因此您可以看到，您可以透過 YAML 檔案變更組態，而不需要重新編譯拓撲。

Flux 還提供其他許多未在本文中討論到的功能，例如根據執行階段所傳遞的參數替代 YAML 檔案中的變數，或從環境變數來替代。如需Flux 架構的這些功能和其他功能的詳細資訊，請參閱 [Flux (https://storm.apache.org/releases/0.10.0/flux.html)](https://storm.apache.org/releases/0.10.0/flux.html)。

##Trident

Trident 是 Storm 提供的高層級抽象。它支援具狀態的處理。Trident 的主要優點是它可以保證進入拓撲的每則訊息都只處理一次。這在原始 Java 拓撲中很難達到，而原始 Java 拓撲保證訊息將至少處理一次。還有其他差異，例如可供使用的內建元件，而不是建立 Bolt。事實上，較不一般的元件 (例如篩選、投影和函數) 會完全取代 Bolt。

可以使用 Maven 專案來建立 Trident 應用程式。您使用與本文稍早所呈現的相同基本步驟—只有程式碼不同。Trident (目前) 也無法與 Flux 架構搭配使用。

如需 Trident 的詳細資訊，請參閱 <a href="http://storm.apache.org/documentation/Trident-API-Overview.html" target="_blank">Trident API 概觀</a>。

如需 Trident 應用程式的範例，請參閱 [Twitter 的趨勢主題與 Apache Storm on HDInsight](hdinsight-storm-twitter-trending.md)。

##後續步驟

您已經了解如何使用 Java 建立 Storm 拓撲。現在要了解如何：

* [部署和管理 HDInsight 上的 Apache Storm 拓撲](hdinsight-storm-deploy-monitor-topology.md)

* [使用 Visual Studio 開發 Apache Storm on HDInsight 的 C# 拓撲](hdinsight-storm-develop-csharp-visual-studio-topology.md)

您可透過瀏覽 [Storm on HDInsight 的範例拓撲](hdinsight-storm-example-topology.md)找到更多範例 Storm 拓撲。

<!---HONumber=AcomDC_0921_2016-->