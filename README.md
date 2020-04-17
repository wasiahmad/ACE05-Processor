# ACE05-Processor

The [ACE 2005](https://catalog.ldc.upenn.edu/LDC2006T06) dataset contains entity, relation, and event annotations for an assortment of newswire and online text. To the best of my knowledge, there are no publicly available implementation to preprocess the ACE05 dataset that supports all the 3 languages, Arabic, Chinese, and English. Hence, I decided to build a pipeline that can process the raw files from ACE05 and dump the processed data into nice JSON format.

**[Note]** We use UDPipe v2.5 models to perform preprocessing on all the languages. 

## ACE 2005

I assume the ACE 2005 is downloaded and kept in a directory `ace_2005` at the root directory of this repository.
The tree structure of the directory hierarchy (upto level 3) is shown below.

```
ace_2005
├── data
│   ├── Arabic
│   │   ├── bn
│   │   ├── nw
│   │   └── wl
│   ├── Chinese
│   │   ├── bn
│   │   ├── nw
│   │   └── wl
│   └── English
│       ├── bc
│       ├── bn
│       ├── cts
│       ├── nw
│       ├── un
│       └── wl
├── docs
│   ├── README
│   └── file.tbl
├── dtd
│   ├── ace_source_sgml.v1.0.2.dtd
│   ├── ag-1.1.dtd
│   └── apf.v5.1.1.dtd
└── index.html
```

## Preprocessing in Action

The entire preprocessing can be executed by running the [setup.sh](https://github.com/wasiahmad/ACE05-Processor/blob/master/setup.sh) script. Please make sure the required packages are installed as listed in [requirements.txt](https://github.com/wasiahmad/ACE05-Processor/blob/master/requirements.txt) before starting the preprocessing.

Once the preprocessing is done, we will get the data under the `processed-data` directory. The structure of the directory (upto level 2) would be as follows.

```
processed-data
├── Arabic
│   ├── train
│   ├── dev
│   └── test
├── Chinese
│   ├── train
│   ├── dev
│   └── test
└── English
    ├── train
    ├── dev
    └── test
```

During preprocessing, we split the ACE05 data into train/dev/test splits. We use the dataset split provided by the first author of this [work](https://www.aclweb.org/anthology/D19-1038.pdf). However, we can use any dataset split. To use your own split, just put the file lists under the `filelist` directory located at the root directory of this repo. The structure of the `filelist` directory should be as follows. 

```
filelist/
├── ace.ar.dev.txt
├── ace.ar.test.txt
├── ace.ar.train.txt
├── ace.en.dev.txt
├── ace.en.test.txt
├── ace.en.train.txt
├── ace.zh.dev.txt
├── ace.zh.test.txt
└── ace.zh.train.txt
```

Once the preprocessing is done, inside the `processed-data`, say for example, inside `processed-data/Arabic/train`, we will see a list of files associated with each document from ACE05 dataset. There will be three files per document. For example,

```
APW_ENG_20030406.0191.conllu
APW_ENG_20030406.0191.v1.json
APW_ENG_20030406.0191.v2.json 
```

Where, `APW_ENG_20030406.0191` is a document filename. To learn about the `conllu`, `v1.json`, and `v2.json` files, keep reading the details.


## Details of Preprocessing

The following is the description of how I perform the preprocessing. There are two steps in the dataset preprocessing.

- **[Step1]** Extract the entities, events, and relations from the raw ACE05 dataset.
- **[Step2]** Extract the word indices for the entities, events, and relations. Due to the complexity of matching character indices to the word indices, after preprocessing the ACE05 dataset, we lose a few entity/event/relation information.


### Step1

In step1, we segment the raw document files (with `.sgm` extension, e.g., `bn/adj/NTV20001002.1530.0917.sgm`) into sentences and extract the entity, relations and event information from the raw files (with `.apf.xml` extension, e.g., `bn/adj/NTV20001002.1530.0917.apf.xml`). The outputs will be stored in `.conllu` and `v1.json` files.

Once the step1 is finished, we will see a summary of the extracted entities, relations and events as follows.

```
******************** English ********************

+-------------------+--------+-------+-------+--------+
| Attribute         |  Train |   Dev |  Test |  Total |
+-------------------+--------+-------+-------+--------+
| #Documents        |    479 |    60 |    60 |    599 |
| #Sentences        |  16105 |  2015 |  1598 |  19718 |
| #Words            | 260490 | 35392 | 28274 | 324156 |
| Entity Mentions   |  49148 |  6697 |  5476 |  61321 |
| Relation Mentions |   6940 |   878 |   920 |   8738 |
| Event Mentions    |   4165 |   619 |   565 |   5349 |
| Event Arguments   |   7615 |  1114 |  1064 |   9793 |
+-------------------+--------+-------+-------+--------+

******************** Chinese ********************

+-------------------+--------+-------+-------+--------+
| Attribute         |  Train |   Dev |  Test |  Total |
+-------------------+--------+-------+-------+--------+
| #Documents        |    507 |    63 |    63 |    633 |
| #Sentences        |   6450 |   755 |   814 |   8019 |
| #Words            | 205010 | 24493 | 26077 | 255580 |
| Entity Mentions   |  32312 |  3958 |  3954 |  40224 |
| Relation Mentions |   7525 |   962 |   830 |   9317 |
| Event Mentions    |   2664 |   293 |   376 |   3333 |
| Event Arguments   |   6521 |   667 |   844 |   8032 |
+-------------------+--------+-------+-------+--------+

******************** Arabic ********************

+-------------------+--------+-------+-------+--------+
| Attribute         |  Train |   Dev |  Test |  Total |
+-------------------+--------+-------+-------+--------+
| #Documents        |    323 |    40 |    40 |    403 |
| #Sentences        |   2555 |   301 |   262 |   3118 |
| #Words            | 114117 | 12560 | 10857 | 137534 |
| Entity Mentions   |  29232 |  3148 |  2898 |  35278 |
| Relation Mentions |   3848 |   434 |   449 |   4731 |
| Event Mentions    |   1793 |   230 |   247 |   2270 |
| Event Arguments   |   3882 |   516 |   577 |   4975 |
+-------------------+--------+-------+-------+--------+
```

### Step2

In step2, we resolve he character-level indices to word-level indices for the spans associated with entities, relations, and events. To do that, we use the character offsets provided by UDPipe and match them with the character-offsets provided in the ACE05 dataset. The outputs will be stored in `v2.json` files.

Once the step2 is finished, we will see a summary as follows.


```
******************** English ********************

-------------------- Train --------------------
[Entities] Total 49148, Skipped 4430, Dropped 721, Wrong-Head  0.
[Events] Total 4165, Dropped 406, Wrong-Trigger 0.
[Relations] Total 6940, Dropped 286.
-------------------- Dev --------------------
[Entities] Total 6697, Skipped 583, Dropped 126, Wrong-Head 0.
[Events] Total 619, Dropped 67, Wrong-Trigger  0.
[Relations] Total 878, Dropped 55.
-------------------- Test --------------------
[Entities] Total 5476, Skipped 456, Dropped 128, Wrong-Head 0.
[Events] Total 565, Dropped 74, Wrong-Trigger 0.
[Relations] Total 920, Dropped 71.

******************** Arabic ********************

-------------------- Train --------------------
[Entities] Total 29232, Skipped 1898, Dropped 3086, Wrong-Head 18.
[Events] Total 1793, Dropped 1719, Wrong-Trigger 0.
[Relations] Total 3848, Dropped 1240.
-------------------- Dev --------------------
[Entities] Total 3148, Skipped 208, Dropped 312, Wrong-Head 0.
[Events] Total 230, Dropped 222, Wrong-Trigger 0.
[Relations] Total 434, Dropped 129.
-------------------- Test --------------------
[Entities] Total 2898, Skipped 196, Dropped 391, Wrong-Head 0.
[Events] Total 247, Dropped  235, Wrong-Trigger 0.
[Relations] Total 449, Dropped 144.

******************** Chinese ********************

-------------------- Train --------------------
[Entities] Total 32312, Skipped 4013, Dropped 599, Wrong-Head  6.
[Events] Total 2664, Dropped  140, Wrong-Trigger 19.
[Relations] Total 7525, Dropped 237.
-------------------- Dev --------------------
[Entities] Total 3958, Skipped 453, Dropped 58, Wrong-Head 1.
[Events] Total 293, Dropped 18, Wrong-Trigger 3.
[Relations] Total 962, Dropped 27.
-------------------- Test --------------------
[Entities] Total 3954, Skipped 520, Dropped 79, Wrong-Head 0.
[Events] Total 376, Dropped 12, Wrong-Trigger 2.
[Relations] Total 830, Dropped 32.
```

#### Transformations from Step1 to Step2

Here, we present one example of entities, relations, and events that we extract in **step1** and then modify in **step2**.

##### Example of an Entity


```
---------------After Step1---------------

"entity-id": "AFP_ENG_20030304.0250-E1-3", 
"entity-type": "ORG:Medical-Science", 
"text": "The Davao Medical Center", 
"position": [493, 516], 
"head": {
	"text": "Davao Medical Center", 
	"position": [497, 516]
}

---------------After Step2---------------

"entity-id": "AFP_ENG_20030304.0250-E1-3", 
"entity-type": "ORG:Medical-Science", 
"text": "The Davao Medical Center", 
"sent_id": "6", 
"position": [0, 3], 
"head": {
	"text": "Davao Medical Center", 
	"position": [1, 3]
}
```

##### Example of a Relation

```
---------------After Step1---------------

"relation-id": "AFP_ENG_20030304.0250-R1-1", 
"relation-type": "PART-WHOLE:Geographical", 
"text": "southern Philippines airport", 
"position": [253, 280], 
"arguments": [
	{
		"text": "southern Philippines airport", 
		"position": [253, 280], 
		"role": "Arg-1", 
		"entity-id": "AFP_ENG_20030304.0250-E26-32"
	}, 
	{
		"text": "southern Philippines", 
		"position": [253, 272], 
		"role": "Arg-2", 
		"entity-id": "AFP_ENG_20030304.0250-E28-33"
	}
]

---------------After Step2---------------

"relation-id": "AFP_ENG_20030304.0250-R1-1", 
"relation-type": "PART-WHOLE:Geographical", 
"text": "southern Philippines airport", 
"sent_id": "4", 
"position": [14, 16], 
"arguments": [
	{
		"text": "southern Philippines airport", 
		"sent_id": "4", 
		"position": [14, 16], 
		"role": "Arg-1", 
		"entity-id": "AFP_ENG_20030304.0250-E26-32"
	}, 
	{
		"text": "southern Philippines", 
		"sent_id": "4", 
		"position": [14, 15], 
		"role": "Arg-2", 
		"entity-id": "AFP_ENG_20030304.0250-E28-33"
	}
]
```



##### Example of an Event


```
---------------After Step1---------------

"event-id": "AFP_ENG_20030304.0250-EV1-1", 
"event_type": "Life:Die", 
"arguments": [
	{
		"text": "At least 19 people", 
		"position": [181, 198], 
		"role": "Victim", 
		"entity-id": "AFP_ENG_20030304.0250-E24-29"
	},
	{
		"text": "southern Philippines airport", 
		"position": [253, 280], 
		"role": "Place", 
		"entity-id": "AFP_ENG_20030304.0250-E26-32"
	}, 
	{
		"text": "Tuesday", 
		"position": [243, 249], 
		"role": "Time-Within", 
		"entity-id": "AFP_ENG_20030304.0250-T2-1"
	}
], 
"text": "At least 19 people were killed and 114 people were wounded in\nTuesday's southern Philippines airport blast, officials said, but\nreports said the death toll could climb to 30",
"position": [181, 353], 
"trigger": {
	"text": "killed", 
	"position": [205, 210]
}

---------------After Step2---------------

"event-id": "AFP_ENG_20030304.0250-EV1-1", 
"event_type": "Life:Die", 
"arguments": [
	{
		"text": "At least 19 people", 
		"sent_id": "4", 
		"position": [0, 3], 
		"role": "Victim", 
		"entity-id": "AFP_ENG_20030304.0250-E24-29"
	}, 
	{
		"text": "southern Philippines airport", 
		"sent_id": "4", 
		"position": [14, 16], 
		"role": "Place", 
		"entity-id": "AFP_ENG_20030304.0250-E26-32"
	}
], 
"text": "At least 19 people were killed and 114 people were wounded in\nTuesday's southern Philippines airport blast, officials said, but\nreports said the death toll could climb to 30", 
"sent_id": "4", 
"position": [0, 31], 
"trigger": {
	"text": "killed", 
	"position": [5, 5]
}
```


### Note

- Following previous work, timex2 and value mentions are not treated as entity mentions and are skipped.
- We drop the entities/relations/events if we are unable to resolve their (or their head's/trigger's) word indices.
- We assume entity spans do not cross sentence boundaries. (Otherwise they are ignored)
- According to the conllu format, we assume `sent_id` 1 refers to the first sentence in a document.

### FAQ

- How do we map character-level offsets to word-level offsets for the entities, heads and triggers?

We use the `Tokenrange` information provided by UDPipe and match them with the character-level offsets provided in the ACE05 dataset. According to the API documentation of UDPipe (see [more](https://ufal.mff.cuni.cz/udpipe/api-reference)):

> The format of the feature (inspired by Python) is TokenRange=start:end, where start is zero-based document-level index of the start of the token (counted in Unicode characters) and end is zero-based document-level index of the first character following the token (i.e., the length of the token is end-start).


- What does dropped entities/relations/events mean?

We drop (do not include in the processed output) the entities/relations/events if we are unable to resolve their character-level offsets to word-level offsets.

- What does skipped entities mean?

Following previous work, timex2 and value mentions are not treated as entity mentions and are skipped.

- What does wrong head or wrong trigger mean?

If we are unable to resolve the character-level offset to a word-level offset for entity-heads or event-triggers, we label them as wrong heads or triggers. We ignore such entities and events.


## Contact

For any question/comment/concern/feedback/suggestion, please open an issue. I may not be able to respond personally.



