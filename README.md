## Description
A **python3** caption eval lib, for MS COCO caption  and custom-task caption evaluation.

For coco caption eval: 

fork from [ruotianluo/coco-caption](https://github.com/ruotianluo/coco-caption)
 
- You will first need to download the Stanford CoreNLP 3.6.0 code and models for use by SPICE. To do this, run: bash get_stanford_models.sh
- Note: SPICE will try to create a cache of parsed sentences in ./pycocoevalcap/spice/cache/. This dramatically speeds up repeated evaluations. The cache directory can be moved by setting 'CACHE_DIR' in ./pycocoevalcap/spice. In the same file, caching can be turned off by removing the '-cache' argument to 'spice_cmd'.
- You will also need to download the Google News negative 300 word2vec model for use by WMD. To do this, run: bash get_google_word2vec_model.sh

For custom caption eval:

[kracwarlock/demo_cocoeval.py](https://gist.github.com/kracwarlock/c979b10433fe4ac9fb97)

## Custom caption eval
Script to evaluate Bleu, METEOR, CIDEr and ROUGE_L for any dataset using the coco evaluation api. Just requires the pycocoevalcap folder.
```python
    from custom_caption_eval.pyimport calculate_metrics
    rng = range(2)
    # label caption
    datasetGTS = {
        'annotations': [{u'image_id': 0, u'caption': u'the man is playing a guitar'},
                        {u'image_id': 0, u'caption': u'a man is playing a guitar'},
                        {u'image_id': 1, u'caption': u'a woman is slicing cucumbers'},
                        {u'image_id': 1, u'caption': u'the woman is slicing cucumbers'},
                        {u'image_id': 1, u'caption': u'a woman is cutting cucumbers'}]
        }
    # your generated caption
    datasetRES = {
        'annotations': [{u'image_id': 0, u'caption': u'man is playing guitar'},
                        {u'image_id': 1, u'caption': u'a woman is cutting vegetables'}]
        }
    # calculate score
    calculate_metrics(rng, datasetGTS, datasetRES)

```
and in `custom_caption_eval.py` select what are the criteria of your task.
```python
        # =================================================
        # Set up scorers
        # =================================================
        print('setting up scorers...')
        scorers = [
            (Bleu(4), ["Bleu_1", "Bleu_2", "Bleu_3", "Bleu_4"]),
            (Meteor(), "METEOR"),
            (Rouge(), "ROUGE_L"),
            (Cider(), "CIDEr"),
            # (Spice(), "SPICE"),
            # (WMD(),   "WMD"),
        ]
```

## Fixed bugs in  [ruotianluo/coco-caption](https://github.com/ruotianluo/coco-caption)
`pycocoevalcap/tokenizer/ptbtokenizer.py`:

replace the path of `java`, and add `shell=True` in subprocess.Popen()

replace
    
    ....
    
    cmd = ['java', '-cp', STANFORD_CORENLP_3_4_1_JAR, \
                     'edu.stanford.nlp.process.PTBTokenizer', \
                     '-preserveLines', '-lowerCase']
    ...
    cmd.append(os.path.basename(tmp_file.name))
    p_tokenizer = subprocess.Popen(cmd, cwd=path_to_jar_dirname, \
                    stdout=subprocess.PIPE)
    ....
    
with

    ....
    cmd = ['/usr/java/jdk1.8.0_191/bin/java -cp stanford-corenlp-3.4.1.jar edu.stanford.nlp.process.PTBTokenizer -preserveLines -lowerCase ']
    cmd[0] += os.path.join(path_to_jar_dirname, os.path.basename(tmp_file.name))
    # add shell=True
    p_tokenizer = subprocess.Popen(cmd, cwd=path_to_jar_dirname, \
    stdout=subprocess.PIPE, shell=True)
    ....


ERROR like:

    File "/..your project path.../pycocoevalcap/meteor/meteor.py", line 71, in __del__
        self.lock.acquire()
    AttributeError: 'Meteor' object has no attribute 'lock'


`pycocoevalcap/meteor/meteor.py`:
replace the path of `java`.
replace

       ....
       self.meteor_cmd = ['java', '-jar', '-Xmx2G', METEOR_JAR, \
                '-', '-', '-stdio', '-l', 'en', '-norm']
       ....
       
with

       ....
       self.meteor_cmd = ['/usr/java/jdk1.8.0_191/bin/java', '-jar', '-Xmx2G', METEOR_JAR, \
               '-', '-', '-stdio', '-l', 'en', '-norm']
       ....
