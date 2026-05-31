# WHAT IS RUN-TIME TYPE INFORMATION?

It is a mechanism that allows us to determine the type of an object at run-time. We can use dynamic_cast (converts base class pointer to derived class pointer-downcasting) operator, type_info (holds the type of an object) class and typeid (returns the type of an object) operator in this concept.

The base class must have one virtual function (run-time polymorphism) for the dynamic_cast conversion to be successful.
