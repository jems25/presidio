# Recognizers development - best practices

Recognizers are the main building blocks in Presidio. Each recognizer is in charge of detecting one or more entities in one or more languages. 
Recognizers define the logic for detection, as well as the confidence a prediction receives and a list of words to be used when context is leveraged.


* [Implementation considerations](#implementation-considerations)
  + [Accuracy](#accuracy)
  + [Performance](#performance)
  + [Environment](#environment)
* [Types of recognizers](#types-of-recognizers)
  + [Black lists](#a-black-lists)
  + [Pattern based](#b-pattern-based)
  + [Machine Learning (ML) based or rule-based](#c-machine-learning--ml--based-or-rule-based)
    - [Utilize spaCy](#utilize-spacy)
    - [Utilize Scikit-learn or similar](#utilize-scikit-learn-or-similar)
    - [Apply custom logic](#apply-custom-logic)
    - [Deep learning based methods](#deep-learning-based-methods)


## Implementation considerations

### Accuracy

Each recognizer, regardless of its complexity, could have false positives and false negatives. When adding new recognizers, we try to balance the effect of each recognizer on the entire system. A recognizer with many false positives would affect the system's usability, while a recognizer with many false negatives might require more work before it can be integrated. We are working on a tool to automatically test new recognizers. In the mean time, it would be best if you clarified how you tested the recognizer's accuracy, and what datasets you've used.

### Performance

Make sure your recognizer doesn't take too long to process text. Anything above 100ms per request with 100 tokens is probably not good enough.

### Environment

When adding new recognizers that have 3rd party dependencies, make sure that the new dependencies don't interfere with Presidio's dependencies. In the case of a conflict, one can create an isolated model environment (in a sidecar container or external endpoint) and implement a RemoteRecognizer on presidio's side to interact with the model's endpoint. In addition, make sure the license on the 3rd party dependency allows you to use it for any purpose.
Types of recognizers

## Types of recognizers

Generally speaking, there are three types of recognizers:

### a. Black lists

A black list is a list of words that should be removed during text analysis. For example, a list of titles (`["Mr.", "Mrs.", "Ms."]` etc.)
This type of recognizer could be added via API or code. In case of contribution, a code-based recognizer is a better option as it gets added to the list of predefined recognizers already implemented in Presidio.
See [this documentation](docs/custom_fields.md#via-code) on adding a new recognizer via code. The [PatternRecognizer](presidio-analyzer/presidio_analyzer/pattern_recognizer.py) class already has support for a black-list input.

### b. Pattern based

Pattern based recognizers use regular expressions to identify entities in text. 
This type of recognizer could be added by API or code. In case of contribution, 
a code-based recognizer is a better option as it gets added to the list of predefined recognizers already implemented in Presidio.
See [this documentation](docs/custom_fields.md#via-code) on adding a new recognizer via code. 
The [PatternRecognizer](presidio-analyzer/presidio_analyzer/pattern_recognizer.py) class should be extended.
See some examples here:
  - [Credit card recognizer](presidio-analyzer/presidio_analyzer/predefined_recognizers/credit_card_recognizer.py)
  - [Email recognizer](presidio-analyzer/presidio_analyzer/predefined_recognizers/email_recognizer.py)

### c. Machine Learning (ML) based or rule-based

Many PII entites are undetectable using naive approaches like black-lists or regular expressions. 
In these cases, we would wish to utilize a Machine Learning model capabable of identifying entities in text, 
or a rule-based recognizer. There are four options for adding ML and rule based recognizers:

#### Utilize spaCy

Presidio currently uses [spaCy](https://spacy.io/) as a framework for text analysis and Named Entity Recognition (NER). 
As to not introduce new tools for no reason, it is recommended to first try to use spaCy over other tools. 
spaCy provides descent results compared to state-of-the-art NER models, but with much better computational performance. 
spaCy could be trained from scratch, used in combination with pre-trained embeddings, or retrained to detect new entities. 
When building such model, you would need to extend the [EntityRecognizer](presidio-analyzer/presidio_analyzer/entity_recognizer.py) class.

#### Utilize Scikit-learn or similar

Scikit-learn models tend to be fast, but usually have lower accuracy than deep learning methods. However, for well defined problems with well defined features, they can provide very good results. 
Since deep learning models tend to be complex and slow, we encourage you to first test a traditional approach (like Conditional Random Fields) before going directly into state-of-the-art *Sesame-Street* character based models... 
When building such model, you would need to extend the [EntityRecognizer](presidio-analyzer/presidio_analyzer/entity_recognizer.py) class.
   
#### Apply custom logic

In some cases, rule-based logic provides the best way of detecting entities. 
The Presidio EntityRecognizer API allows you to use spaCy extracted features like lemmas, part of speech, dependencies and more to create your logic. When building such model, you would need to extend the [EntityRecognizer](presidio-analyzer/presidio_analyzer/entity_recognizer.py) class.

#### Deep learning based methods

Deep learning methods offer excellent detection rates for NER. 
They are however more complex to train, deploy and tend to be slower than traditional approaches. 
When contributing a DL based method, the best option would be to create a sidecar container which is isolated from the presidio-analyzer container. On the `presidio-analyzer` side, one would extend the [RemoteRecognizer](presidio-analyzer/presidio_analyzer/remote_recognizer.py) class and implement the network interface between `presidio-analyzer` and the endpoint of the model's container.

### General note

New predefined recognizers should also be listed in the field types list Presidio supports here: https://github.com/microsoft/presidio/blob/master/docs/field_types.md and added to the proto files here
