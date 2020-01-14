# Data Science Testing

> Eric J. Ma
>
> Novartis Institutes for Biomedical Research
>
> Ann Arbor, 2020-01-15

---

## About Me

- Scientific Data Analysis, NIBR IT
- https://ericmjl.github.io/

---

## How many of you are data scientists?

Keep your hands up...

----

## How many of you write code in a programming language >50% of your time?

Keep your hands up...

----

## How many of you write tests for your code?

----

## For those who don't write tests, what are your reasons?

----

## For those who do write tests, what are your reasons?

---

## Today's Main Point

> As a data scientist,
> investing time in writing tests today
> will save you time later on,
> thus increasing your productivity.

The same holds for basic software engineering skills in general:
documentation, testing, refactoring.
<!-- .element: class="fragment" -->

---

## Two Stories

1. How automated testing revealed weakspots in an app
<!-- .element: class="fragment" -->
2. How testing accelerated our data analysis workflow
<!-- .element: class="fragment" -->


---

## How automated testing revealed weakspots in an app

----

### HIV Drug Resistance Prediction

- Input: an amino acid sequence (string)
<!-- .element: class="fragment" -->
- Output: drug resistance (a number)
<!-- .element: class="fragment" -->

----

### Model Training Functions

```python
# utils.py

def read_protein(filename):
	sequence = ... # stuff happens
    return sequence

# Returns array 5 times the length of sequence.
def featurize(sequence):
    features = ... # stuff happens
    return features

# Return model predictions.
def predict(features):
    model = ... # load model
    return model.predict(features)
```

What have we assumed here
that probably ought to be tested?
<!-- .element: class="fragment" -->

----

### Let's write a test for featurize!

```python
from utils import featurize

def test_featurize():
    sequence = "MKALVIELQDPG..."  # something 99 amino acids long.
    feats = featurize(sequence)

    assert feats.shape[0] == 1
    assert feats.shape[1] == len(sequence) * 5
```

----

### Let's write a test for predict!

<!-- Q&A

Q: How did you decide on just doing an integration test for predict,
but not for featurize?

A: Most of the function calls inside `predict` was well-tested already,
so we don't need to duplicate those test.
Rather, we only wanted to make sure that they worked together,
given certain inputs.
 -->

```python
from utils import predict

# An integration test for the predict function.
def test_predict():
    sequence = "MKALVIELQDPG..."  # something 99 amino acids long.
    feats = featurize(sequence)
    preds = predict(feats)
```

Cool! Are we done?

----

### Clearly not!

One **huge** assumption we made here was the length of the input string.

Let's revisit that test.

----

- Featurize intended before predict.
- Prediction function uses a model that has been trained only on 495-long vectors per sample.
<!-- .element: class="fragment" -->
- Vectors came from 99-letter strings.
<!-- .element: class="fragment" -->
- Those 99-letter strings need to be drawn from a valid alphabet; not all 26 letters are valid.
<!-- .element: class="fragment" -->

----

_If a user inputs a string that is not 99 letters long, the program should crash._


----

_If a user inputs a string with invalid characters, the program should crash._

----

Let's write make the code more robust.

```python
acceptable_letters = set('ACDEFGHIJKLMNPQRSTVWXY')
def featurize(sequence):
    if not len(sequence) == 99:
        raise ValueError("put informative error here.")
    if not set(sequence).issubset(acceptable_letters):
        raise Exception("put informative error here.")
    features = ... # stuff happens
    return features

```
<!-- .element: class="fragment" -->

----

We can now make the test also more robust, using `Hypothesis`.

```python
from hypothesis import strategies as st, given
# other imports here...

@given(
    sequence=st.text(alphabet=..., min_size=0, max_size=200))
)
def test_featurize(sequence):
    if len(sequence) != 99:
        with pytest.raises(ValueError):
            feats = featurize(sequence)
    else:
        feats = featurize(sequence)
        assert feats.shape[0] == 1
        assert feats.shape[1] == len(sequence) * 5
```
<!-- .element: class="fragment" -->

We can do the same for invalid characters.
<!-- .element: class="fragment" -->

----

### We are in a much better position now

- Function is defensively robust against unexpected inputs.
<!-- .element: class="fragment" -->
- Tests help us catch breaking changes.
<!-- .element: class="fragment" -->

----

### In practice...

We caught this issue by using `Hypothesis`, and worked backwards.
<!-- .element: class="fragment" -->

Writing tests helps you catch bugs.
---

## How testing accelerated our data analysis workflow

----

### Protein engineering platform

- Large but simple codebase.
<!-- .element: class="fragment" -->
- Many independent utilities with some function sharing.
<!-- .element: class="fragment" -->

**Focus on data access.**
<!-- .element: class="fragment" -->

----

### Our data: a complex, evolving schema

- As platform gets built out, data requirements change.
<!-- .element: class="fragment" -->
- New enzyme = _similar_ schema.
<!-- .element: class="fragment" -->
- The database is as far normalized as possible
<!-- .element: class="fragment" -->

Lots of `join`s needed to make get data in human-readable form.
<!-- .element: class="fragment" -->

----

### Things to safeguard against:

- Evolving schema.
- Unexpected schema changes.

> For tabular data, the schema is the data's API!
<!-- .element: class="fragment" -->

----

### Tests for Data

- Column names
<!-- .element: class="fragment" -->
- Column data types
<!-- .element: class="fragment" -->
- Nullity
<!-- .element: class="fragment" -->
- Bounds
<!-- .element: class="fragment" -->

Are there anything else you can think of?
<!-- .element: class="fragment" -->

----

### Testing Data

```python
def test_query_function():
    data = query_function()

    # Column tests:
    expected_columns = [...]
    assert set(expected_columns) == set(data.columns)

    # Null checks: this column __must__ be fully populated
    assert pd.isnull(df[column_name]).sum() == 0
```

----

### I Expect Great Things of You

[Great Expectations](https://greatexpectations.io/) (GE) is right on our list of next things to try.
<!-- .element: class="fragment" -->

Only dealing with urgent matters have blocked us from using GE.
<!-- .element: class="fragment" -->

----

### Tests have accelerated our workflow

Because of data caching and data testing...
<!-- .element: class="fragment" -->

...a non-routine data query that may have taken half a day to get right...
<!-- .element: class="fragment" -->

...instead took 10 minutes to finish and be confident in.
<!-- .element: class="fragment" -->

---

## Tips for Building a Culture of Testing

----

### Bake it into your infrastructure

Set up a CI system that mandates checks on code.
<!-- .element: class="fragment" -->

Don't allow code to be committed without review and passing tests.
<!-- .element: class="fragment" -->

----

### Find the wins that build credibility

Provide testimonials to your DevOps team (where applicable).
<!-- .element: class="fragment" -->

Deliver talks about testing.
<!-- .element: class="fragment" -->

---

## Conclusions

- If you write code that you need to depend on, write tests for your code.
<!-- .element: class="fragment" -->
- If you use data, declare expectations of your data.
<!-- .element: class="fragment" -->
- "Escape auto-manual testing with Hypothesis."
<!-- .element: class="fragment" -->

---

## Resources

- [Essays on Data Science](https://ericmjl.github.io/essays-on-data-science/)
- [Personal Blog](https://ericmjl.github.io/blog/)
-
