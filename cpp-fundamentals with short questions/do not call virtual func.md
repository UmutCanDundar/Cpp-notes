# WHEN SHOULD NOT WE CALL VIRTUAL FUNCTION?

Virtual functions should not be called in a constructor or a destructor because they are called in a certain order because of the nature of the inheritance, which means that virtual functions are not virtual during the construction and destruction processes.
