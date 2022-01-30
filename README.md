# array_mutation

To run, use
```bash
python3 src/array_mutation/array_mutation.py
```

the expected output is:
```bash
{"$update": {"posts.0.value": ["too"]}}
{"$update": {"posts.1.mentions.0.text": ["pear"]}}
{"$add": {"posts": [{"value": "four"}]}}
{"$add": {"posts.1.mentions": [{"text": "banana"}]}}
{"$remove": {"posts.0": true}}
{"$remove": {"posts.1.mentions.1": true}}
{"$update": {"posts.0.value": ["too"]}, "$add": {"posts": [{"value": "four"}]}, "$remove": {"posts.2": true}}
```

## Thoughts:
it is not too clear what is needed for multithreading, if needed at the python interpreter level, we need
to add locks for each post and use a thread safe dictionary implementations

the index lookup operation is not very efficient since it is a linear scan
