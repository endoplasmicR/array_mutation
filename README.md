# array_mutation


## Thoughts:
it is not too clear what is needed for multithreading, if needed at the python interpreter level, we need
to add locks for each post and use a thread safe dictionary implementations

the index lookup operation is not very efficient since it is a linear scan
