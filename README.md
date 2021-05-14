# Session 2 assignment of EPAi3.0

## Object Mutability and Interning

The objective is to understand the functions defined in session2.py and how they are tested in test_session2.py

### Circular references

When we create an object, Python Memory Manager keeps track of all the references to it. For example, `felix=Felicis()`, this means that there is one reference to the object at its memory address. But once we delete the variable *felix*, the reference count goes back to 0. So, the Python Memory Manager removes the object and frees up that space in the memory. 

But there are certain instances called Circular references where this does not happen

In the code block below, in **critical_function** function, we have a loop for a certain range in which we keep calling the **add_something** function to which we send 2 arguments that is - a variable collection which is a list and the loop counter

	def critical_function():
		collection = list()
		for i in range(1, 1024 * 128):
			add_something(collection, i)
		clear_memory(collection)

In the code block below, in **add_something** function,  we have the variable collection, a list, to which we append an instance of the *class Something* after making changes to its attribute

	def add_something(collection: List[Something], i: int):
		something = Something()
		something.something_new = SomethingNew(i, something)
		collection.append(something)

When we create an instance the *class Something* from `something = Something()`, we have its attribute `something.something_new = None`.This happens in `__init__` which is a method called when an object is created from a class and allows it to initialize the attributes of a class.  And in the second line, `something.something_new = SomethingNew(i, something)`, we reference it to some other object.

	class Something(object):

		def __init__(self):
			super().__init__()
			self.something_new = None

		def __repr__(self):
			return 'something_new is pointing to {0}'.format(self.something_new)

This object is an instance of the *class SomethingNew* which takes in 2 arguments - loop counter and something, which was instantiated in the previous line. It has two attributes of its own, `SomethingNew.i=i` and 'SomethingNew.something=something'. The first attribute is a reference to the loop counter and the second attribute is a reference to the object `something` itself. And this is how a circular reference occurs.

	class SomethingNew(object):

		def __init__(self, i: int = 0, something: Something = None):
			super().__init__()
			self.i = i
			self.something = something

		def __repr__(self):
			return 'i is pointing to {0}'.format(self.i)

###  Garbage collection

Now if we go back to the **critical function**, the last line of code calls **clear_memory** function.  This function basically clears the variable `collection`. So therefore its references to any objects will vanish, and ideally, all the memory occupied by these objects should be freed up.  But because there is a circular reference, the reference counter is still not yet zero. So, this is where we use the garbage collection to find such referencess and remove them.

### Python optimizations - String interning
Interning is basically where the memory addresses of identifiers are always  remembered. Some strings are automatically interned, and some are not. But we can force them to be interened. But why would we want to do so?

When we compare two strings for equality, a lot of lookups are required, as we have to compare each character. But if we interened the strings, we compare the references of these i.e they should reference to the same memory address.

We have two functions **compare_strings_old(n)** and **compare_strings_new(n)**. In the first function, we have two long strings, and we are performing two operations in a loop - checking their equality and checking if a certain character is present in one of the strings. In the second function, we have just used python time sleep fuction which basically halts the execution of the program for given time mentioned. We need to replace this with an improved version of the first function code.

1.  2 same loops are running for the two separate operations. Ths clubbed into a single loop
2.  In `char_list = list(a)`, we are breaking the string into a list and then checking for a certain character in it. But this need not be done, we can use set(a) instead as set membership test is faster that list membership test
3. We forcefully intern the strings, and compare using `a is b` as this compares the memory addresses