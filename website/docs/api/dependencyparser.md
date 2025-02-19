---
title: DependencyParser
tag: class
source: spacy/pipeline/dep_parser.pyx
teaser: 'Pipeline component for syntactic dependency parsing'
api_base_class: /api/pipe
api_string_name: parser
api_trainable: true
---

A transition-based dependency parser component. The dependency parser jointly
learns sentence segmentation and labelled dependency parsing, and can optionally
learn to merge tokens that had been over-segmented by the tokenizer. The parser
uses a variant of the **non-monotonic arc-eager transition-system** described by
[Honnibal and Johnson (2014)](https://www.aclweb.org/anthology/D15-1162/), with
the addition of a "break" transition to perform the sentence segmentation.
[Nivre (2005)](https://www.aclweb.org/anthology/P05-1013/)'s **pseudo-projective
dependency transformation** is used to allow the parser to predict
non-projective parses.

The parser is trained using an **imitation learning objective**. It follows the
actions predicted by the current weights, and at each state, determines which
actions are compatible with the optimal parse that could be reached from the
current state. The weights are updated such that the scores assigned to the set
of optimal actions is increased, while scores assigned to other actions are
decreased. Note that more than one action may be optimal for a given state.

## Config and implementation {#config}

The default config is defined by the pipeline component factory and describes
how the component should be configured. You can override its settings via the
`config` argument on [`nlp.add_pipe`](/api/language#add_pipe) or in your
[`config.cfg` for training](/usage/training#config). See the
[model architectures](/api/architectures) documentation for details on the
architectures and their arguments and hyperparameters.

> #### Example
>
> ```python
> from spacy.pipeline.dep_parser import DEFAULT_PARSER_MODEL
> config = {
>    "moves": None,
>    "update_with_oracle_cut_size": 100,
>    "learn_tokens": False,
>    "min_action_freq": 30,
>    "model": DEFAULT_PARSER_MODEL,
> }
> nlp.add_pipe("parser", config=config)
> ```

| Setting                       | Description                                                                                                                                                                                                                                                                                                           |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `moves`                       | A list of transition names. Inferred from the data if not provided. Defaults to `None`. ~~Optional[TransitionSystem]~~                                                                                                                                                                                                |
| `update_with_oracle_cut_size` | During training, cut long sequences into shorter segments by creating intermediate states based on the gold-standard history. The model is not very sensitive to this parameter, so you usually won't need to change it. Defaults to `100`. ~~int~~                                                                   |
| `learn_tokens`                | Whether to learn to merge subtokens that are split relative to the gold standard. Experimental. Defaults to `False`. ~~bool~~                                                                                                                                                                                         |
| `min_action_freq`             | The minimum frequency of labelled actions to retain. Rarer labelled actions have their label backed-off to "dep". While this primarily affects the label accuracy, it can also affect the attachment structure, as the labels are used to represent the pseudo-projectivity transformation. Defaults to `30`. ~~int~~ |
| `model`                       | The [`Model`](https://thinc.ai/docs/api-model) powering the pipeline component. Defaults to [TransitionBasedParser](/api/architectures#TransitionBasedParser). ~~Model[List[Doc], List[Floats2d]]~~                                                                                                                   |

```python
%%GITHUB_SPACY/spacy/pipeline/dep_parser.pyx
```

## DependencyParser.\_\_init\_\_ {#init tag="method"}

> #### Example
>
> ```python
> # Construction via add_pipe with default model
> parser = nlp.add_pipe("parser")
>
> # Construction via add_pipe with custom model
> config = {"model": {"@architectures": "my_parser"}}
> parser = nlp.add_pipe("parser", config=config)
>
> # Construction from class
> from spacy.pipeline import DependencyParser
> parser = DependencyParser(nlp.vocab, model)
> ```

Create a new pipeline instance. In your application, you would normally use a
shortcut for this and instantiate the component using its string name and
[`nlp.add_pipe`](/api/language#add_pipe).

| Name                          | Description                                                                                                                                                                                                                                                                                         |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `vocab`                       | The shared vocabulary. ~~Vocab~~                                                                                                                                                                                                                                                                    |
| `model`                       | The [`Model`](https://thinc.ai/docs/api-model) powering the pipeline component. ~~Model[List[Doc], List[Floats2d]]~~                                                                                                                                                                                |
| `name`                        | String name of the component instance. Used to add entries to the `losses` during training. ~~str~~                                                                                                                                                                                                 |
| `moves`                       | A list of transition names. Inferred from the data if not provided. ~~Optional[List[str]]~~                                                                                                                                                                                                         |
| _keyword-only_                |                                                                                                                                                                                                                                                                                                     |
| `update_with_oracle_cut_size` | During training, cut long sequences into shorter segments by creating intermediate states based on the gold-standard history. The model is not very sensitive to this parameter, so you usually won't need to change it. Defaults to `100`. ~~int~~                                                 |
| `learn_tokens`                | Whether to learn to merge subtokens that are split relative to the gold standard. Experimental. Defaults to `False`. ~~bool~~                                                                                                                                                                       |
| `min_action_freq`             | The minimum frequency of labelled actions to retain. Rarer labelled actions have their label backed-off to "dep". While this primarily affects the label accuracy, it can also affect the attachment structure, as the labels are used to represent the pseudo-projectivity transformation. ~~int~~ |

## DependencyParser.\_\_call\_\_ {#call tag="method"}

Apply the pipe to one document. The document is modified in place, and returned.
This usually happens under the hood when the `nlp` object is called on a text
and all pipeline components are applied to the `Doc` in order. Both
[`__call__`](/api/dependencyparser#call) and
[`pipe`](/api/dependencyparser#pipe) delegate to the
[`predict`](/api/dependencyparser#predict) and
[`set_annotations`](/api/dependencyparser#set_annotations) methods.

> #### Example
>
> ```python
> doc = nlp("This is a sentence.")
> parser = nlp.add_pipe("parser")
> # This usually happens under the hood
> processed = parser(doc)
> ```

| Name        | Description                      |
| ----------- | -------------------------------- |
| `doc`       | The document to process. ~~Doc~~ |
| **RETURNS** | The processed document. ~~Doc~~  |

## DependencyParser.pipe {#pipe tag="method"}

Apply the pipe to a stream of documents. This usually happens under the hood
when the `nlp` object is called on a text and all pipeline components are
applied to the `Doc` in order. Both [`__call__`](/api/dependencyparser#call) and
[`pipe`](/api/dependencyparser#pipe) delegate to the
[`predict`](/api/dependencyparser#predict) and
[`set_annotations`](/api/dependencyparser#set_annotations) methods.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> for doc in parser.pipe(docs, batch_size=50):
>     pass
> ```

| Name           | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| `docs`         | A stream of documents. ~~Iterable[Doc]~~                      |
| _keyword-only_ |                                                               |
| `batch_size`   | The number of documents to buffer. Defaults to `128`. ~~int~~ |
| **YIELDS**     | The processed documents in order. ~~Doc~~                     |

## DependencyParser.initialize {#initialize tag="method" new="3"}

Initialize the component for training. `get_examples` should be a function that
returns an iterable of [`Example`](/api/example) objects. The data examples are
used to **initialize the model** of the component and can either be the full
training data or a representative sample. Initialization includes validating the
network,
[inferring missing shapes](https://thinc.ai/docs/usage-models#validation) and
setting up the label scheme based on the data. This method is typically called
by [`Language.initialize`](/api/language#initialize) and lets you customize
arguments it receives via the
[`[initialize.components]`](/api/data-formats#config-initialize) block in the
config.

<Infobox variant="warning" title="Changed in v3.0" id="begin_training">

This method was previously called `begin_training`.

</Infobox>

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> parser.initialize(lambda: [], nlp=nlp)
> ```
>
> ```ini
> ### config.cfg
> [initialize.components.parser]
>
> [initialize.components.parser.labels]
> @readers = "spacy.read_labels.v1"
> path = "corpus/labels/parser.json
> ```

| Name           | Description                                                                                                                                                                                                                                                                                                                                                                                                            |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `get_examples` | Function that returns gold-standard annotations in the form of [`Example`](/api/example) objects. ~~Callable[[], Iterable[Example]]~~                                                                                                                                                                                                                                                                                  |
| _keyword-only_ |                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `nlp`          | The current `nlp` object. Defaults to `None`. ~~Optional[Language]~~                                                                                                                                                                                                                                                                                                                                                   |
| `labels`       | The label information to add to the component, as provided by the [`label_data`](#label_data) property after initialization. To generate a reusable JSON file from your data, you should run the [`init labels`](/api/cli#init-labels) command. If no labels are provided, the `get_examples` callback is used to extract the labels from the data, which may be a lot slower. ~~Optional[Dict[str, Dict[str, int]]]~~ |

## DependencyParser.predict {#predict tag="method"}

Apply the component's model to a batch of [`Doc`](/api/doc) objects, without
modifying them.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> scores = parser.predict([doc1, doc2])
> ```

| Name        | Description                                                   |
| ----------- | ------------------------------------------------------------- |
| `docs`      | The documents to predict. ~~Iterable[Doc]~~                   |
| **RETURNS** | A helper class for the parse state (internal). ~~StateClass~~ |

## DependencyParser.set_annotations {#set_annotations tag="method"}

Modify a batch of [`Doc`](/api/doc) objects, using pre-computed scores.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> scores = parser.predict([doc1, doc2])
> parser.set_annotations([doc1, doc2], scores)
> ```

| Name     | Description                                                                                                                           |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `docs`   | The documents to modify. ~~Iterable[Doc]~~                                                                                            |
| `scores` | The scores to set, produced by `DependencyParser.predict`. Returns an internal helper class for the parse state. ~~List[StateClass]~~ |

## DependencyParser.update {#update tag="method"}

Learn from a batch of [`Example`](/api/example) objects, updating the pipe's
model. Delegates to [`predict`](/api/dependencyparser#predict) and
[`get_loss`](/api/dependencyparser#get_loss).

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> optimizer = nlp.initialize()
> losses = parser.update(examples, sgd=optimizer)
> ```

| Name              | Description                                                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `examples`        | A batch of [`Example`](/api/example) objects to learn from. ~~Iterable[Example]~~                                                  |
| _keyword-only_    |                                                                                                                                    |
| `drop`            | The dropout rate. ~~float~~                                                                                                        |
| `sgd`             | An optimizer. Will be created via [`create_optimizer`](#create_optimizer) if not set. ~~Optional[Optimizer]~~                      |
| `losses`          | Optional record of the loss during training. Updated using the component name as the key. ~~Optional[Dict[str, float]]~~           |
| **RETURNS**       | The updated `losses` dictionary. ~~Dict[str, float]~~                                                                              |

## DependencyParser.get_loss {#get_loss tag="method"}

Find the loss and gradient of loss for the batch of documents and their
predicted scores.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> scores = parser.predict([eg.predicted for eg in examples])
> loss, d_loss = parser.get_loss(examples, scores)
> ```

| Name        | Description                                                                 |
| ----------- | --------------------------------------------------------------------------- |
| `examples`  | The batch of examples. ~~Iterable[Example]~~                                |
| `scores`    | Scores representing the model's predictions. ~~StateClass~~                 |
| **RETURNS** | The loss and the gradient, i.e. `(loss, gradient)`. ~~Tuple[float, float]~~ |

## DependencyParser.score {#score tag="method" new="3"}

Score a batch of examples.

> #### Example
>
> ```python
> scores = parser.score(examples)
> ```

| Name        | Description                                                                                                                                                              |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `examples`  | The examples to score. ~~Iterable[Example]~~                                                                                                                             |
| **RETURNS** | The scores, produced by [`Scorer.score_spans`](/api/scorer#score_spans) and [`Scorer.score_deps`](/api/scorer#score_deps). ~~Dict[str, Union[float, Dict[str, float]]]~~ |

## DependencyParser.create_optimizer {#create_optimizer tag="method"}

Create an [`Optimizer`](https://thinc.ai/docs/api-optimizers) for the pipeline
component.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> optimizer = parser.create_optimizer()
> ```

| Name        | Description                  |
| ----------- | ---------------------------- |
| **RETURNS** | The optimizer. ~~Optimizer~~ |

## DependencyParser.use_params {#use_params tag="method, contextmanager"}

Modify the pipe's model, to use the given parameter values. At the end of the
context, the original parameters are restored.

> #### Example
>
> ```python
> parser = DependencyParser(nlp.vocab)
> with parser.use_params(optimizer.averages):
>     parser.to_disk("/best_model")
> ```

| Name     | Description                                        |
| -------- | -------------------------------------------------- |
| `params` | The parameter values to use in the model. ~~dict~~ |

## DependencyParser.add_label {#add_label tag="method"}

Add a new label to the pipe. Note that you don't have to call this method if you
provide a **representative data sample** to the [`initialize`](#initialize)
method. In this case, all labels found in the sample will be automatically added
to the model, and the output dimension will be
[inferred](/usage/layers-architectures#thinc-shape-inference) automatically.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> parser.add_label("MY_LABEL")
> ```

| Name        | Description                                                 |
| ----------- | ----------------------------------------------------------- |
| `label`     | The label to add. ~~str~~                                   |
| **RETURNS** | `0` if the label is already present, otherwise `1`. ~~int~~ |

## DependencyParser.set_output {#set_output tag="method"}

Change the output dimension of the component's model by calling the model's
attribute `resize_output`. This is a function that takes the original model and
the new output dimension `nO`, and changes the model in place. When resizing an
already trained model, care should be taken to avoid the "catastrophic
forgetting" problem.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> parser.set_output(512)
> ```

| Name | Description                       |
| ---- | --------------------------------- |
| `nO` | The new output dimension. ~~int~~ |

## DependencyParser.to_disk {#to_disk tag="method"}

Serialize the pipe to disk.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> parser.to_disk("/path/to/parser")
> ```

| Name           | Description                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `path`         | A path to a directory, which will be created if it doesn't exist. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| _keyword-only_ |                                                                                                                                            |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~                                                |

## DependencyParser.from_disk {#from_disk tag="method"}

Load the pipe from disk. Modifies the object in place and returns it.

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> parser.from_disk("/path/to/parser")
> ```

| Name           | Description                                                                                     |
| -------------- | ----------------------------------------------------------------------------------------------- |
| `path`         | A path to a directory. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| _keyword-only_ |                                                                                                 |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~     |
| **RETURNS**    | The modified `DependencyParser` object. ~~DependencyParser~~                                    |

## DependencyParser.to_bytes {#to_bytes tag="method"}

> #### Example
>
> ```python
> parser = nlp.add_pipe("parser")
> parser_bytes = parser.to_bytes()
> ```

Serialize the pipe to a bytestring.

| Name           | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| _keyword-only_ |                                                                                             |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~ |
| **RETURNS**    | The serialized form of the `DependencyParser` object. ~~bytes~~                             |

## DependencyParser.from_bytes {#from_bytes tag="method"}

Load the pipe from a bytestring. Modifies the object in place and returns it.

> #### Example
>
> ```python
> parser_bytes = parser.to_bytes()
> parser = nlp.add_pipe("parser")
> parser.from_bytes(parser_bytes)
> ```

| Name           | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| `bytes_data`   | The data to load from. ~~bytes~~                                                            |
| _keyword-only_ |                                                                                             |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~ |
| **RETURNS**    | The `DependencyParser` object. ~~DependencyParser~~                                         |

## DependencyParser.labels {#labels tag="property"}

The labels currently added to the component.

> #### Example
>
> ```python
> parser.add_label("MY_LABEL")
> assert "MY_LABEL" in parser.labels
> ```

| Name        | Description                                            |
| ----------- | ------------------------------------------------------ |
| **RETURNS** | The labels added to the component. ~~Tuple[str, ...]~~ |

## DependencyParser.label_data {#label_data tag="property" new="3"}

The labels currently added to the component and their internal meta information.
This is the data generated by [`init labels`](/api/cli#init-labels) and used by
[`DependencyParser.initialize`](/api/dependencyparser#initialize) to initialize
the model with a pre-defined label set.

> #### Example
>
> ```python
> labels = parser.label_data
> parser.initialize(lambda: [], nlp=nlp, labels=labels)
> ```

| Name        | Description                                                                     |
| ----------- | ------------------------------------------------------------------------------- |
| **RETURNS** | The label data added to the component. ~~Dict[str, Dict[str, Dict[str, int]]]~~ |

## Serialization fields {#serialization-fields}

During serialization, spaCy will export several data fields used to restore
different aspects of the object. If needed, you can exclude them from
serialization by passing in the string names via the `exclude` argument.

> #### Example
>
> ```python
> data = parser.to_disk("/path", exclude=["vocab"])
> ```

| Name    | Description                                                    |
| ------- | -------------------------------------------------------------- |
| `vocab` | The shared [`Vocab`](/api/vocab).                              |
| `cfg`   | The config file. You usually don't want to exclude this.       |
| `model` | The binary model data. You usually don't want to exclude this. |
